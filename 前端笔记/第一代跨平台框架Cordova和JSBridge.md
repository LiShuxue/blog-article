## 跨平台开发框架

**Cordova**：Cordova 是混合开发的先驱之一，最早是开源的 PhoneGap，由 Adobe 收购，后来捐赠给 Apache 软件基金会。它允许开发人员使用 Web 技术（HTML、CSS 和 JavaScript）构建移动应用程序，并通过将 Web 应用程序封装在本地容器中，使其能够访问设备原生功能。Cordova 支持多个平台，包括 iOS、Android 和 Windows Phone。

**React Native**：React Native 是由 Facebook 开发的一款基于 React.js 的跨平台开发框架。它允许开发人员使用 JavaScript 和 React 的语法来构建原生应用程序。React Native 使用原生组件和 API，以提供更好的性能和用户体验。它支持 iOS 和 Android 平台，并且在开发过程中可以实时预览应用程序的变化。

**Flutter**：Flutter 是由 Google 开发的一款跨平台移动应用程序开发框架。它使用 Dart 编程语言，并提供了一套丰富的 UI 组件和工具，用于构建高性能、美观的应用程序。Flutter 使用自己的渲染引擎，可以直接绘制应用程序的用户界面，从而实现快速的渲染和流畅的动画效果。Flutter 支持 iOS、Android 和 Web 平台。

**Uni-app**：Uni-app 是由 DCloud 开发的一款基于 Vue.js 的跨平台开发框架。它允许开发人员使用 Vue.js 编写一次代码，然后将其编译为多个平台的应用程序，包括 iOS、Android、Web 和各个平台的小程序。Uni-app 提供了一套统一的 API，使开发人员可以轻松地访问设备功能和平台特定的功能。

**Taro**：Taro 是一款基于 React.js 的跨平台开发框架，可以将一套代码编译成多个平台的应用程序，包括 iOS、Android、Web 和小程序。Taro 提供了一套统一的 API 和组件库，使开发人员可以使用 React 的语法来开发小程序和 App。

## 混合开发

混合开发是一种应用程序开发方法，它结合了原生应用程序开发和 Web 技术开发的优势。在混合开发中，开发人员使用 Web 技术（如 HTML、CSS 和 JavaScript）来构建应用程序的用户界面，并使用原生开发技术（如 Java、Objective-C 或 Swift）来访问设备功能和执行高性能操作。

混合开发的一个主要优势是编写界面比较快，且可以跨多个平台，例如 iOS 和 Android。然而，混合应用程序可能不如原生应用程序那样快速和高效，此外，某些设备功能可能无法通过混合开发框架直接访问，需要使用原生开发技术进行扩展。

## Cordova

Cordova 是混合开发的先驱之一，最早是开源的 PhoneGap，由 Adobe 收购，后来捐赠给 Apache 软件基金会。它允许开发人员使用 Web 技术（HTML、CSS 和 JavaScript）构建移动应用程序，并通过将 Web 应用程序封装在本地容器中，使其能够访问设备原生功能。Cordova 支持多个平台，包括 iOS、Android 和 Windows Phone。

Cordova 在原生应用中嵌入了一个 WebView，所以可以加载和呈现 Web 内容。WebView 允许将 HTML、CSS 和 JavaScript 渲染出来，同时用户也可以交互。WebView 是原生应用和 Web 内容之间的桥梁。其实相当于一个浏览器加载网页一样。不过加载的内容，可以是本地文件或者一个 URL。这种开发模式开发的 App 既有原生应用代码又有 Web 应用代码，因此又被称为 Hybrid App（混合应用程序）。

在 WebView 中，对于原生的一些系统服务，通过 JS Bridge 来调用，由于这些调用不是很频繁，JS Bridge 并不会成为性能瓶颈。然而，一个完整 HTML5 页面的展示要经历浏览器控件的加载、解析和渲染三大过程，性能消耗要比原生开发增加 N 个数量级，所以这种方案的瓶颈在于 WebView 对 H5 页面的渲染。

## Cordova 程序目录结构

```bash
project/
├── platforms/                # 各个平台的构建目录
│   ├── android/              # Android 平台的构建目录
│   │   ├── app/              # Android 应用程序的主要源代码和资源
│   │   │   ├── assets/       # Android 资源文件，如 HTML、CSS、JavaScript
│   │   │   ├── src/          # 自定义 Android 源代码目录
│   │   │   │   ├── com/
│   │   │   │   │   └── example/
│   │   │   │   │       └── demo/
│   │   │   │   │           └── MainActivity.java # 自定义的 MainActivity，一般继承了CordovaActivity
│   │   │   │   └── ...
│   │   │   └── ...
│   │   ├── cordovaLib/        # Cordova 库的源代码（作为 Android 库项目）
│   │   ├── cordova/           # Cordova 框架的核心代码和资源
│   │   ├── platform_www/      # 包含cordova.js文件，和plugins相关文件，相当于这个android平台特殊的文件，最终会与外层的www文件一起copy到assets下面。
│   │   └── ...
├── plugins/                  # Cordova 插件目录
├── www/                      # 项目的 Web 内容目录，Cordova 构建工具会将 platform_www 目录中的内容与这个 www 目录中的内容合并，并一起复制到 Android 项目的 assets 目录下。
│   ├── css/                  # 样式表文件
│   ├── img/                  # 图片文件
│   ├── js/                   # JavaScript 文件
│   ├── index.html            # 项目的入口 HTML 文件
│   └── ...
├── config.xml                # 项目配置文件，用于定义项目的名称、描述等信息，最终会被合并到特定平台的config.xml里。
└── ...
```

## Cordova 核心功能

### WebView 加载页面

```java
// 加载本地 HTML 文件
webView.loadUrl("file:///android_asset/www/index.html");
// 加载远程 URL
webView.loadUrl("https://www.example.com");
```

### 定制 WebView 行为

WebViewClient 是用于处理 WebView 中页面加载事件的类。比如开始加载，加载完成，请求了某个 URL 等。在 Cordova 中是 SystemWebViewClient。

WebChromeClient 是用于处理与 JavaScript 交互和弹窗相关事件的类。它允许你捕获 WebView 中的 JavaScript 弹窗、处理文件选择、获取页面标题等。在 Cordova 中是 SystemWebChromeClient。

如果需要定制手机返回键，是 webview 中的返回，而不是 app 返回，需要覆写 onKeyDown 方法。

```java
public class MainActivity extends CordovaActivity {
    private WebView webView;
    @Override
    public void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        loadUrl(launchUrl);

        // 设置 WebViewClient
        webView = (WebView) appView.getEngine().getView();
        webView.setWebViewClient(new MyWebViewClient((SystemWebViewEngine) appView.getEngine()));
        webView.setWebChromeClient(new MyWebChromeClient((SystemWebViewEngine) appView.getEngine()));
    }

    @Override
    public boolean onKeyDown(int keyCode, KeyEvent event) {
        // 检查是否按下返回按钮
        if ((keyCode == KeyEvent.KEYCODE_BACK) && webView.canGoBack()) {
            // 如果 WebView 可以后退，则执行后退操作
            webView.goBack();
            return true; // 返回 true 表示已处理返回事件
        }

        // 如果 WebView 不能后退或没有按下返回按钮，继续默认的处理
        return super.onKeyDown(keyCode, event);
    }

    private class MyWebViewClient extends SystemWebViewClient {
        public MyWebViewClient(SystemWebViewEngine parentEngine) {
            super(parentEngine);
        }

        // shouldOverrideUrlLoading

        @Override
        public void onPageFinished(WebView view, String url) {
            super.onPageFinished(view, url);
            // 页面加载完成后的处理逻辑
        }
    }

    private class MyWebChromeClient extends SystemWebChromeClient {
        public MyWebChromeClient(SystemWebViewEngine parentEngine) {
            super(parentEngine);
        }

        @Override
        public boolean onJsAlert(WebView view, String url, String message, JsResult result) {
            return false;
        }
    }
}
```

### 事件处理

Cordova 提供了许多事件来帮助开发者处理与移动设备和应用程序生命周期相关的事件。
比如：deviceready，pause，resume，backbutton，menubutton，online，offline 等。

如果需要定制手机返回键功能，也可以在 js 中监听 backbutton 事件来实现。

```js
document.addEventListener(
  'backbutton',
  () => {
    // 自定义逻辑
  },
  false
);
```

### 插件系统

```bash
# 添加插件
cordova plugin add cordova-plugin-device
```

1. 然后它会为我们的 project/plugins 文件夹下添加 cordova-plugin-device 插件（这个是整个这个插件模块，可用于安卓和 ios）

2. 同时也会在我们的 platforms/android/platform_www/plugins 下面添加对应的安卓的插件 cordova-plugin-device（只可用于本安卓平台）

3. 还会为我们的 src 下对应的 java 文件，/src/main/java/org/apache/cordova/device/Device.java。
   res/xml/config.xml 中也增加了对应的内容。

我们在自己的 js 中可以使用插件的方法，如 `device.getInfo()` ，当然也可以直接使用 `device.uuid` 等类似的，因为插件声明时定义了 `<clobbers target="device" />` ，表示插件的功能可以通过 device 对象直接访问。

### 插件原理

1. 我们在业务代码调用插件 js 中的方法

2. 插件的本身的 device.js 中，声明了 getInfo 方法，调用了 exec 方法

   ```js
   // 插件的js
   var exec = require('cordova/exec');

   Device.prototype.getInfo = function (successCallback, errorCallback) {
     exec(successCallback, errorCallback, 'Device', 'getDeviceInfo', []);
   };
   ```

3. exec 是 cordova/exec 中引入的，这是 requirejs 的写法。所以需要找到 cordova/exec 的定义，它定义在 cordova.js 中，我只抽取了核心的一些代码。可以看到最终调用了浏览器的原生 prompt 方法。

   Cordova 正是通过调用 prompt 函数来与原生代码进行通信的。这是 Cordova 自己实现的一种机制，用于跨越 JavaScript 和原生之间的隔阂。

   ```js
   // cordova.js
   define('cordova/exec', function (require, exports, module) {
     var nativeApiProvider = require('cordova/android/nativeapiprovider');
     function androidExec(success, fail, service, action, args) {
       var msgs = nativeApiProvider.get().exec(bridgeSecret, service, action, callbackId, argsJson);
     }

     module.exports = androidExec;
   });

   define('cordova/android/nativeapiprovider', function (require, exports, module) {
     var currentApi = require('cordova/android/promptbasednativeapi');

     module.exports = {
       get: function () {
         return currentApi;
       },
     };
   });

   define('cordova/android/promptbasednativeapi', function (require, exports, module) {
     module.exports = {
       exec: function (bridgeSecret, service, action, callbackId, argsJson) {
         return prompt(
           argsJson,
           'gap:' + JSON.stringify([bridgeSecret, service, action, callbackId])
         );
       },
     };
   });
   ```

4. 我们可以在 android/CordovaLib/src/org/apache/cordova/engine/SystemWebChromeClient.java 中，找到 Cordova 是如何处理 js 的 prompt 方法的。

   ```java
   // android/CordovaLib/src/org/apache/cordova/engine/SystemWebChromeClient.java
   @Override
   public boolean onJsPrompt(WebView view, String origin, String message, String defaultValue, final JsPromptResult result) {
       String handledRet = parentEngine.bridge.promptOnJsPrompt(origin, message, defaultValue);
       result.confirm(handledRet);
   }

   // android/CordovaLib/src/org/apache/cordova/CordovaBridge.java
   public String promptOnJsPrompt(String origin, String message, String defaultValue) {
       if (defaultValue != null && defaultValue.startsWith("gap:")) {
           JSONArray array = new JSONArray(defaultValue.substring(4));
           int bridgeSecret = array.getInt(0);
           String service = array.getString(1);
           String action = array.getString(2);
           String callbackId = array.getString(3);
           String r = jsExec(bridgeSecret, service, action, callbackId, message);
           return r == null ? "" : r;
       }
   }
   public String jsExec(int bridgeSecret, String service, String action, String callbackId, String arguments) throws JSONException, IllegalAccessException {
       pluginManager.exec(service, action, callbackId, arguments);
   }

   // android/CordovaLib/src/org/apache/cordova/PluginManager.java
   public void exec(final String service, final String action, final String callbackId, final String rawArgs) {
       CordovaPlugin plugin = getPlugin(service);
       CallbackContext callbackContext = new CallbackContext(callbackId, app);
       boolean wasValidAction = plugin.execute(action, rawArgs, callbackContext);
       PluginResult cr = new PluginResult(PluginResult.Status.INVALID_ACTION);
       callbackContext.sendPluginResult(cr);
   }
   ```

## JSBridge

### 介绍

JavaScript 是运行在单独的 JS Context 中（Webview 容器、JSCore 等），与原生运行环境隔离，所以需要有一种机制实现 Native 端和 Web 端的双向通信，这就是 JSBridge。

JSBridge 简单来讲，主要是 给 JavaScript 提供调用 Native 功能的接口，让混合开发中的『前端部分』可以方便地使用地址位置、摄像头甚至支付等 Native 功能。它的核心是 构建 Native 和非 Native 间消息通信的通道，而且是双向通信的通道。

PhoneGap（Codova 的前身）作为 Hybrid 鼻祖框架，应该是最先被开发者广泛认知的 JSBridge 的应用场景；而对于 JSBridge 的应用在国内真正兴盛起来，则是因为杀手级应用微信的出现，主要用途是在网页中通过 JSBridge 设置分享内容。移动端混合开发中的 JSBridge，主要被应用在两种形式的技术方案上：

- 基于 Web 的 Hybrid 解决方案：例如微信浏览器、各公司的 Hybrid 方案
- 非基于 Web UI 但业务逻辑基于 JavaScript 的解决方案：例如 React-Native

### 原理

Web 端和 Native 可以类比于 Client/Server 模式，Web 端调用原生接口时就如同 Client 向 Server 端发送一个请求类似，JSB 在此充当类似于 HTTP 协议的角色，实现 JSBridge 主要是两点：

- JS 向 Native 发送消息 : 调用相关功能、通知 Native 当前 JS 的相关状态等。
- Native 向 JS 发送消息 : 回溯调用结果、消息推送、通知 JS 当前 Native 的状态等。

#### JS -> Native

**1. 拦截 Webview 请求的 URL Schema**

WebView 加载 web 页面之后，Web 发送的所有请求都会经过 WebView 组件，所以 Native 可以重写 WebView 里的方法，从来拦截 Web 发起的请求，然后对请求的格式进行判断，如果符合我们自定义的 URL Schema，对 URL 进行解析，进而调用原生 Native 的方法。

web 发请求有很多种，比如 a 标签，location.href 等，这些方法，a 标签需要用户操作，location.href 可能会引起页面的跳转丢失调用，所以一般使用没有啥副作用的 iframe.src。

```js
function callNativeFunction(arg) {
  // 构造Bridge请求URL，包括方法名和参数
  var url = 'jsbridge://nativeFunction/' + arg;

  // 发起Bridge请求
  iframe.src = url;
}

// 处理Native的回调
function callback(argument, result) {
  // 处理来自Native的结果
  console.log('JavaScript收到Native的回调：参数：' + argument + '，结果：' + result);
}
```

Native 拦截的时候，安卓提供了 shouldOverrideUrlLoading 方法拦截，UIWebView 使用 shouldStartLoadWithRequest，WKWebView 则使用 decidePolicyForNavigationAction。

```java
// Android中的WebView拦截URL
webView.setWebViewClient(new WebViewClient() {
    @Override
    public boolean shouldOverrideUrlLoading(WebView view, String url) {
        // 判断URL是否是Bridge请求
        if (url.startsWith("jsbridge://")) {
            // 解析URL参数
            String[] urlParts = url.split("://");
            String[] params = urlParts[1].split("/");
            String methodName = params[0];
            String argument = params[1];

            // 调用Native方法，执行操作
            if (methodName.equals("nativeFunction")) {
                // 执行相应的Native操作，例如调用相机等
                // ...

                // 返回结果给JavaScript
                String result = "Native操作成功";
                view.loadUrl("javascript:callback('" + argument + "', '" + result + "')");
                return true;
            }
        }
        return super.shouldOverrideUrlLoading(view, url);
    }
});

```

**2. 向 Webview 中注入原生对象**

通过 WebView 提供的接口，向 JavaScript 的 Context（window）中注入对象或者方法，让 JavaScript 调用时，直接执行相应的 Native 代码逻辑，达到 JavaScript 调用 Native 的目的。

Android 4.2 以下使用 WebViewClient 的 onJsPrompt 方式。

```java
// 创建一个本地Java对象，该对象将被注入到WebView中
class MyJavaScriptInterface {
    // @JavascriptInterface注释标记了一个可以供JavaScript调用的方法showToast。
    @JavascriptInterface
    public void showToast(String message) {
        // 在本地执行操作，例如显示一个Toast消息
        Toast.makeText(context, message, Toast.LENGTH_SHORT).show();
    }
}

// 在Activity或Fragment中使用WebView，并将本地对象注入到WebView中
WebView webView = findViewById(R.id.webView);
webView.getSettings().setJavaScriptEnabled(true);

// 创建一个实例并注入到WebView中，"jsInterface" 将在JavaScript中作为对象使用
MyJavaScriptInterface jsInterface = new MyJavaScriptInterface();
webView.addJavascriptInterface(jsInterface, "jsInterface");
```

在 JavaScript 中可以直接调用 Java 对象中的方法

```js
jsInterface.showToast('这是来自JavaScript的消息');
```

**3. 重写 prompt 等原生 JS 方法**
这也是 Cordova 的实现原理。

WebView 有一个方法，叫 setWebChromeClient，可以设置 WebChromeClient 对象，而这个对象中有三个方法，分别是 onJsAlert,onJsConfirm,onJsPrompt，当 js 调用 window 对象的对应的方法，即 window.alert，window.confirm，window.prompt，WebChromeClient 对象中的三个方法对应的就会被触发，由于拦截上述方法会对性能造成一定影响，因此需要选择使用频率较低的方法，而在 Android 中，相比其它几个方法，几乎不会使用到 prompt 方法，因此占用 prompt 是最佳方案。

```java
webView.setWebChromeClient(new WebChromeClient() {
    @Override
    public boolean onJsPrompt(WebView view, String url, String message, String defaultValue, JsPromptResult result) {
        if (message.startsWith("jsbridge:")) {
            // 解析jsbridge请求
            String[] parts = message.split(":");
            String action = parts[1];
            String data = parts[2];

            // 调用原生代码处理jsbridge请求
            String nativeResponse = handleJsBridgeRequest(action, data);

            // 将原生响应传递回JavaScript
            result.confirm(nativeResponse);
            return true;
        }
        return super.onJsPrompt(view, url, message, defaultValue, result);
    }
});


private String handleJsBridgeRequest(String action, String data) {
    String response = null;

    // 根据传入的action执行相应的操作
    if ("getUserInfo".equals(action)) {
        // 执行获取用户信息的操作，data可能包含参数
        response = fetchUserInfo(data);
    } else if ("doSomethingElse".equals(action)) {
        // 执行其他操作，根据具体的action来处理
        response = performSomeAction(data);
    }

    // 返回原生响应给JavaScript
    return response;
}
```

JS 中调用时：

```js
// 发送一个简单的JSBridge请求
function sendJsBridgeRequest(action, data, callback) {
  var message = 'jsbridge:' + action + ':' + data;
  var response = prompt(message);

  // 处理原生返回的响应
  if (typeof callback === 'function') {
    callback(response);
  }
}
```

#### Native -> JS

相比于 JavaScript 调用 Native， Native 调用 JavaScript 较为简单，毕竟不管是 iOS 的 UIWebView 还是 WKWebView，还是 Android 的 WebView 组件，都以子组件的形式存在于 View/Activity 中，直接调用相应的 API 即可。

Native 调用 JavaScript，其实就是执行 JavaScript 字符串，让 WebView 调用 JavaScript 中的方法，因此 JavaScript 的方法必须在全局的 window 上。

Android 4.4 之前。

```java
String jsCode = String.format("window.showWebDialog('%s')", text);
webView.loadUrl("javascript: " + jsCode);
```

Android 4.4 之后。

```java
String jsCode = String.format("window.showWebDialog('%s')", text);
webView.evaluateJavascript(jsCode, new ValueCallback<String>() {
  @Override
  public void onReceiveValue(String value) {

  }
});
```

iOS 的 UIWebView

```
NSString *jsStr = @"执行的JS代码";
result = [webView stringByEvaluatingJavaScriptFromString:jsStr];
```

iOS 的 WKWebView

```
[webView evaluateJavaScript:@"执行的JS代码" completionHandler:^(id _Nullable response, NSError * _Nullable error) {

}];
```

#### 带回调的调用

上面已经说到了 Native、Web 间双向通信的两种方法，但站在一端而言还是一个单向通信的过程 ，比如站在 Web 的角度：Web 调用 Native 的方法，Native 直接相关操作但无法将结果返回给 Web，但实际使用中会经常需要将操作的结果返回，也就是 JS 回调。

基于之前的单向通信就可以实现，我们在一端调用的时候在参数中加一个 callbackId 标记对应的回调，另一端接收到调用请求后，进行实际操作，如果带有 callbackId，对发送端再进行一次调用，将结果、callbackId 回传回去，发送端根据 callbackId 匹配相应的回调，将结果传入执行就可以了。

### 开源的 JSBridge

主要通过注入 API 的形式，[DSBridge for Android](https://github.com/wendux/DSBridge-Android)、[DSBridge for IOS](https://github.com/wendux/DSBridge-IOS)

主要通过拦截 URL Schema，[JsBridge](https://github.com/lzyzsd/JsBridge)
