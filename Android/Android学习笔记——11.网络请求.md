## WebView

WebView 控件可以用来显示网页，先在布局文件中使用 WebView 控件，然后使用代码加载 url。

setJavaScriptEnabled()方法，让 WebView 支持 JavaScript 脚本。WebViewClient 可以处理 onPageFinished，shouldOverrideUrlLoading 等，WebChromeClient 可以处理 Javascript 的 alert（）和其他功能等。可以同时使用。

需要声明权限：`<uses-permission android:name="android.permission.INTERNET" />`

```kotlin
webView.settings.javaScriptEnabled=true
webView.webViewClient = WebViewClient()
webView.loadUrl("https://www.baidu.com")
```

更多使用参考：<https://blog.csdn.net/devilnov/article/details/117323956>

## HttpURLConnection

```kotlin
val url = URL("https://www.baidu.com")
val connection = url.openConnection() as HttpURLConnection
connection.requestMethod = "GET"

// 如果需要给服务器发数据
connection.requestMethod = "POST"
val output = DataOutputStream(connection.outputStream)
output.writeBytes("username=admin&password=123456")

// 单位：毫秒
connection.connectTimeout = 8000
connection.readTimeout = 8000

// 获取服务器返回的输入流（服务器响应）
val input = connection.inputStream

// 读取响应数据
val response = StringBuilder()
val reader = BufferedReader(InputStreamReader(input))
reader.use {
    reader.forEachLine {
        response.append(it)
    }
}

connection.disconnect()
```

## OkHttp

```kotlin
// get请求
val client = OkHttpClient()
val request = Request.Builder()
    .url("https://www.baidu.com")
    .build()
val response = client.newCall(request).execute()
val responseData = response.body?.string()

// post请求
val requestBody = FormBody.Builder()
    .add("username", "admin")
    .add("password", "123456")
    .build()
val request = Request.Builder()
    .url("https://www.baidu.com")
    .post(requestBody)
    .build()
```

## 解析 JSON 格式数据

### 使用 JSONObject

```kotlin
private fun parseJSONWithJSONObject(jsonData: String) {
    try {
        val jsonArray = JSONArray(jsonData)
        for (i in 0 until jsonArray.length()) {
            val jsonObject = jsonArray.getJSONObject(i)
            val id = jsonObject.getString("id")
            val name = jsonObject.getString("name")
            val version = jsonObject.getString("version")
        }
    } catch (e: Exception) {
        e.printStackTrace()
    }
}
```

### GSON

```kotlin
// 先定义一个类Person，加入json字符串中的字段。
val gson = Gson()
val person = gson.fromJson(jsonData, Person::class.java)

// 如果解析的是json数组
val typeOf = object : TypeToken<List<Person>>() {}.type
val people = gson.fromJson<List<Person>>(jsonData, typeOf)
```

## 封装请求类

最终的回调接口都还是在子线程中运行的，因此我们不可以在这里执行任何的 UI 操作，除非借助 runOnUiThread()

```kotlin
interface HttpCallbackListener {
    fun onFinish(response: String)
    fun onError(e: Exception)
}
object HttpUtil {
    // 使用HttpURLConnection
    fun sendHttpRequest(address: String, listener: HttpCallbackListener): String {
        thread{
            var connection: HttpURLConnection? = null
            try {
                val response = StringBuilder()
                val url = URL(address)
                connection = url.openConnection() as HttpURLConnection
                connection.connectTimeout = 8000
                connection.readTimeout = 8000
                val input = connection.inputStream
                val reader = BufferedReader(InputStreamReader(input))
                reader.use {
                    reader.forEachLine {
                        response.append(it)
                    }
                }
                // 回调onFinish()方法
                listener.onFinish(response.toString())
            } catch (e: Exception) {
                e.printStackTrace()
                // 回调onError()方法
                listener.onError(e)
            } finally {
                connection?.disconnect()
            }
        }
    }

    // 使用OkHttp
    fun sendOkHttpRequest(address: String, callback: okhttp3.Callback) {
        val client = OkHttpClient()
        val request = Request.Builder()
            .url(address)
            .build()
        client.newCall(request).enqueue(callback)
    }
}

// 使用sendHttpRequest
HttpUtil.sendHttpRequest(address, object : HttpCallbackListener {
    override fun onFinish(response: String) {
        // 得到服务器返回的具体内容
    }
    override fun onError(e: Exception) {
        // 在这里对异常情况进行处理
    }
})

// 使用sendOkHttpRequest
HttpUtil.sendOkHttpRequest(address, object : Callback {
    override fun onResponse(call: Call, response: Response) {
        // 得到服务器返回的具体内容
        val responseData = response.body?.string()
    }
    override fun onFailure(call: Call, e: IOException) {
        // 在这里对异常情况进行处理
    }
})
```

## Retrofit

Retrofit 只需要在接口文件中声明一系列方法和返回值，然后通过注解的方式指定该方法对应哪个服务器接口，以及需要提供哪些参数。当我们在程序中调用该方法时，Retrofit 会自动向对应的服务器接口发起请求，并将响应的数据解析成返回值声明的类型。Retrofit 还会将服务器返回的 JSON 数据自动解析成对象。

接口文件建议以具体的功能种类名开头，并以 Service 结尾，这是一种比较好的命名习惯。

返回值必须声明成 Retrofit 中内置的 Call 类型，并通过泛型来指定服务器响应的数据应该转换成什么对象。

当发起请求的时候，Retrofit 会自动在内部开启子线程，当数据回调到 Callback 中之后，Retrofit 又会自动切换回主线程，整个操作过程中我们都不用考虑线程切换问题。

```kotlin
// 1. 定义接口返回的json数据需要转成的对象
class App(val id: String, val name: String, val version: String)

// 2. 定义接口文件，里面可以包含多个接口请求
interface AppService {
    @GET("xx_data.json")
    fun getAppData(): Call<List<App>>
}

// 3. 发送请求并处理响应
val retrofit = Retrofit.Builder()
    .baseUrl("http://10.0.2.2/")
    .addConverterFactory(GsonConverterFactory.create())
    .build()

val appService = retrofit.create(AppService::class.java)
appService.getAppData().enqueue(object : Callback<List<App>> {
    override fun onResponse(call: Call<List<App>>, response: Response<List<App>>) {
        val list = response.body()
        if (list != null) {
            for (app in list) {
                Log.d("MainActivity", "id is ${app.id}")
                Log.d("MainActivity", "name is ${app.name}")
                Log.d("MainActivity", "version is ${app.version}")
            }
        }
    }
    override fun onFailure(call: Call<List<App>>, t: Throwable) {
        t.printStackTrace()
    }
})
```

### @Path("xxx")，@Query("xxx")，@Body，@Headers，@Header

注解可以用在方法型参中，解析出数据，作为方法的参数传入。

接口地址当中可以使用{xxx}形式的占位符，使用@Path("xxx")注解来解析对应的数据

接口中可以使用`?u=xx&t=yy`形式的参数，@Query("xxx")注解来解析。

使用 POST 请求来提交数据，需要将数据放到 HTTP 请求的 body 部分，可以借助@Body 注解来完成。

如果需要请求的 header 中指定参数，使用@Headers 注解，这个不是用在方法的参数中，需要声明在方法上面。如果想要动态的，声明在方法形参那里，需要使用@Header。没有`s`。

### 全局封装

```kotlin
object ServiceCreator {
    private const val BASE_URL = "http://10.0.2.2/"
    private val retrofit = Retrofit.Builder()
        .baseUrl(BASE_URL)
        .addConverterFactory(GsonConverterFactory.create())
        .build()

    // 第一种
    fun <T> create(serviceClass: Class<T>): T = retrofit.create(serviceClass)

    // 第二种，将泛型实化
    inline fun <reified T> create(): T = create(T::class.java)
}

// 第一种使用
val appService = ServiceCreator.create(AppService::class.java)

// 第二种使用
val appService = ServiceCreator.create<AppService>()
```
