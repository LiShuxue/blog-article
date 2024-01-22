## Gradle

Gradle 是一种以 Groovy 语言为基础的自动化构建工具。Gradle 中有两个重要的概念，分别是 Project 和 Task，Project 表示一个正在构建的项目，而 Task 表示某一个具体的任务。

### Project

每个 Project 都对应一个我们在 setting.gradle 中配置的 Project。一个 Project 可以创建新的 Task，添加依赖关系和配置，并应用插件和其他的构建脚本。Project 是通过我们常见的 build.gradle 文件实例化出来的，我们在 build.gradle 文件中进行的各种配置最终都会应用到 Project 中去。

### Task

Task 定义了一个当前任务执行时的最小单元，一个 Project 中一般会存在很多个 Task，通过执行这些 Task 来完成整个项目的构建。也就是说，项目构建的实际工作是由一个个的 Task 来完成的。

### 生命周期

Gradle 构建具有三个不同的阶段。Gradle 按顺序运行这些阶段：初始化，配置，执行。

- 初始化：检测`settings.gradle`文件，并执行文件中的内容，确定哪个项目和哪个 moudle 需要被 include，然后创建一个 Project 实例。

- 配置阶段：解析所有的`build.gradle`文件，包括 include 的模块的`build.gradle`，解析所有 task 配置。

- 执行阶段：根据 task 的依赖关系，按顺序执行。

## 安装

下载 [Gradle](https://gradle.org/releases/) 并配置环境变量 `PATH=$PATH:/Users/lishuxue/Documents/software/gradle-8.2.1/bin`

使用`gradle -v`验证正确安装

## 使用

Gradle 的执行就是由各种任务组合执行，来对项目进行构建的。

doFirst 和 doLast 将一个动作添加到任务动作列表的开头或结尾。当任务执行时，动作列表中的动作会依次执行。可以添加多次。

defaultTasks 可以用来指定默认执行的任务。

dependsOn 可以声明依赖于其他任务。

```gradle
// build.gradle

tasks.register('hello') {
    doLast {
        println 'Hello world!'
    }
}
tasks.register('intro') {
    dependsOn tasks.hello
    doLast {
        println "I'm Gradle"
    }
}
```

通过 `gradle -q myTask` 或者 `./gradlew -q myTask` 执行这个任务。-q 会抑制 Gradle 的日志消息，因此只显示任务的输出。

使用 `gradle wrapper` 生成对应的 gradlew 脚本。gradlew 是 Gradle 的包装脚本，用于在项目中执行 Gradle 命令。

## Android 项目

当我们新建一个项目后，Gradle 默认会生成一些编译脚本文件，主要有：setting.gradle、build.gradle 以及子项目中的 build.gradle 等等，还会在当前目录下生成一个 gradle 文件夹，下面分别介绍一些这些文件的作用：

```
├── gradle
│   └── wrapper
│       ├── gradle-wrapper.jar
│       └── gradle-wrapper.properties
├── gradlew  // gradlew是Gradle的包装脚本
├── gradlew.bat
├── settings.gradle // 定义 build 和 子项目
├── build.gradle // 主项目 build 脚本
├── gradle.properties // 主项目 build 脚本
└── app
    └── build.gradle // 模块 build 脚本
```

setting.gradle 用来告诉 gradle 这个项目包含了那些子项目。

build.gradle 是默认的构建脚本，当我们在执行 gradle 命令时，会首先到当前目录下寻找该文件，然后通过该文件的配置实例化一个 Project 对象。

app 模块下的 build.gradle，是模块构建使用的脚本。

自动生成的 gradle 文件夹是 Gradle 包装器，其中包含一个 jar 文件和一个 配置文件，使用这个包装器可以让 Gradle 运行在一个特定的版本上，目的是创造一个独立于系统、系统配置和 Gradle 版本的可靠和可重复构建。

gradle.properties：用于指定 Gradle 构建工具包本身的设置，比如是否使用 AndroidX，最大堆大小等等。

## Android 项目脚本启动流程

Gradle 构建脚本的执行是由 Gradle 构建引擎负责的。构建引擎在执行脚本时按照一定的生命周期和阶段来进行。以下是构建脚本执行的基本流程：

初始化阶段（Initialization）：确定哪些项目参与构建。为对应的项目创建 Project 对象。

- 加载项目根目录下的 settings.gradle 文件，解析 settings.gradle 文件，配置根项目的属性，如插件管理、仓库配置等。确定哪些项目参与构建。

配置阶段（Configuration）：解析构建脚本中的配置块。生成要执行的 task。

- 加载项目根目录和子项目的 build.gradle 文件，解析 build.gradle 文件。

- 执行 buildscript 配置块，配置构建脚本自身所需的资源，如插件和仓库。

- 执行 plugins 配置块（如果有），用于声明和配置项目所需要的插件。

- 执行 repositories 配置块，配置项目的依赖项仓库。

- 执行 dependencies 配置块，定义项目的依赖项。

- 执行 allprojects 配置块，配置所有项目（包括主项目和子项目）的通用设置。

- 执行 subprojects 配置块，配置所有子项目的通用设置。

执行阶段（Execution）：Gradle 会根据用户的输入或者默认的设置，确定要执行的任务集合，并按照任务之间的依赖关系，逐个执行任务。

- 执行根项目的任务：首先执行根项目（主项目）中定义的任务。

- 执行子项目的任务：执行所有子项目中定义的任务。

- 执行任务的依赖关系：Gradle 检查并执行任务之间的依赖关系。如果某个任务依赖于其他任务，Gradle 会确保在执行该任务之前先执行其依赖任务。

- 执行插件提供的任务：插件可能会提供一些任务，例如与构建、打包、测试等相关的任务。在执行阶段，Gradle 会执行这些插件提供的任务。

- 执行用户自定义的任务：如果在构建脚本中定义了自定义任务，这些任务也会在执行阶段被执行。

### 构建任务

在配置文件中，plugins 块声明了两个插件，分别是 Android 插件和 Kotlin 插件。这些插件会引入一系列默认的任务，其中包括构建任务和其他与 Android 构建相关的任务。

通过运行 ./gradlew tasks 命令查看所有可用的任务及其描述。

- assemble：用于生成 APK 文件的任务，它依赖于不同的构建变体的 assemble 任务，例如 assembleDebug 和 assembleRelease。

- build：用于执行 assemble 和 check 任务的任务，它依赖于所有的构建变体的 assemble 和 check 任务。

- install：用于安装 APK 文件到设备或模拟器的任务，它依赖于不同的构建变体的 install 任务，例如 installDebug 和 installRelease。

- uninstallAll：用于卸载 APK 文件从设备或模拟器的任务，它依赖于不同的构建变体的 uninstall 任务，例如 uninstallDebug 和 uninstallRelease。

- check：用于运行所有检查任务，如代码静态分析、测试等。

- test：用于运行单元测试的任务，它依赖于不同的构建变体的 test 任务，例如 testDebugUnitTest 和 testReleaseUnitTest。

- lint：运行 Lint 静态代码分析。例如 lintDebug 和 lintRelease。

- clean：用于清除构建输出的任务，它删除了所有的构建变体的输出目录。

### 根级别的 setting.gradle

```gradle
// 项目初始化的时候执行 setting.gradle

// pluginManagement 配置和管理构建脚本自身需要使用的 Gradle 插件，以及下载插件的仓库地址。
pluginManagement {
    repositories {
        google()
        mavenCentral()
        gradlePluginPortal()
    }
}

// dependencyResolutionManagement 用于配置项目的依赖解析策略，比如指定依赖库下载的仓库。
dependencyResolutionManagement {
    repositoriesMode.set(RepositoriesMode.FAIL_ON_PROJECT_REPOS)
    repositories {
        google()
        mavenCentral()
    }
}

// 用于声明和配置项目中的使用的插件，apply false 的意思是不将该 plugin 应用于当前项目。
plugins {
    id "dev.flutter.flutter-plugin-loader" version "1.0.0"
    id "com.android.application" version "7.3.0" apply false
}

// 设置项目的根项目的名称。
rootProject.name = "Weather"

// 用于指定项目中包含的子项目（模块），一个根工程可以有很多子工程。子工程只有在 setting 里设置了，gradle 才会去识别，才会在构建的时候被包含进去。
include ':app'
```

### 根级别的 build.gradle

```gradle
// 项目级别的配置文件，配置一些通用的

// buildscript 指定构建脚本需要的插件和其他工具的版本，以及从哪些仓库获取这些依赖项。这些仓库只对构建脚本自身的依赖项生效，对于项目中的模块或子项目的依赖项并不生效。
buildscript {
    // ext 是 Gradle 中的一种扩展机制，允许你在 build.gradle 文件中定义额外的属性。通过ext定义的属性，可以在同一个build.gradle文件的任何位置使用。
    ext.kotlin_version = '1.7.10'
    repositories {
        google()
        mavenCentral()
    }

    dependencies {
        classpath "org.jetbrains.kotlin:kotlin-gradle-plugin:$kotlin_version"
    }
}

// 配置所有项目（包括主项目和子项目）的通用设置。包括仓库。
allprojects {
    repositories {
        google()
        mavenCentral()
    }
}

// 在 Gradle 构建脚本中，rootProject 是一个默认存在的变量，通过 rootProject 访问了根项目，并配置了根项目的构建目录。
rootProject.buildDir = '../build'

// subprojects 用于指定所有子项目的通用设置，它可以出现多次，每次都会应用到所有子项目中。
subprojects {
    project.buildDir = "${rootProject.buildDir}/${project.name}"
}
subprojects {
    // 指定所有子项目在进行评估（evaluation）时依赖于 ':app' 项目的评估。在 Gradle 中，"评估"（evaluation）是指 Gradle 构建脚本的解析和执行过程。
    project.evaluationDependsOn(':app')
}

// 注册了一个名为 "clean" 的自定义任务，这个任务是由 Delete 类提供的。可以通过 " ./gradlew clean "来执行该任务
tasks.register("clean", Delete) {
    delete rootProject.buildDir
}
```

### 模块（子项目）级别的 build.gradle

```gradle
// 模块（子项目）级别的 build.gradle

// 配置子项目中使用的插件
plugins {
    id 'com.android.application' // 用于配置 Android 应用程序的构建。
    id 'org.jetbrains.kotlin.android' // 支持 Kotlin 在 Android 项目中的使用。
}

// Android 工程配置的入口。
android {
    // 设置了 Android 命名空间，通常用于生成 R 类的包名。
    namespace 'com.example.weather'
    // 指定在构建（编译）你的应用程序时使用的 Android SDK 版本。也可以用 compileSdkVersion
    compileSdk 33
    // 使用的 NDK 版本的属性，NDK（Native Development Kit）是用于开发 Android 应用中的本地代码（C/C++）的工具包。
    ndkVersion flutter.ndkVersion

    // 提供应用程序的基本配置信息
    defaultConfig {
        applicationId "com.example.weather" // APP 的唯一标识，也是包名。
        minSdk 24 // 支持的最低 API Level，如指定 23，代表低于 Android 6 的机型不能使用这个 APP。
        targetSdk 33 // 指该 APP 在运行时所针对的 Android 平台的版本。具体来说，它告诉 Android 系统你的应用程序在这个版本被设计和测试的，也就是可以充分利用这个版本的特性。
        versionCode 1 // versionCode 和 versionName：指版本号和版本名称。
        versionName "1.0"
    }

    // 指定用于签署应用程序的密钥库文件、密钥别名、密码等信息。
    // 当你开发和测试应用程序时，可以自己创建一个 keystore 文件来签署应用程序。这个过程通常包括使用 Java 的 keytool 工具生成密钥对，并将它们存储在 keystore 文件中。
    // 在发布应用程序时，通常建议使用由认可的证书颁发机构签发的证书进行签名。
    signingConfigs {
        release {
            storeFile file('path/to/your/keystore.jks')
            storePassword 'your_keystore_password'
            keyAlias 'your_key_alias'
            keyPassword 'your_key_password'
        }
    }

    // 配置不同的构建类型，例如 release 和 debug
    buildTypes {
        release {
            minifyEnabled true // 是否启用代码混淆和压缩。
            proguardFiles getDefaultProguardFile('proguard-android-optimize.txt'), 'proguard-rules.pro' // 指定 ProGuard 配置文件的路径
            signingConfig signingConfigs.release
        }
        debug {
            // 为应用程序的 applicationId 添加一个后缀。在同一设备上同时安装 release 版本和 debug 版本的应用程序，使用不同的 applicationId 后缀，可以确保两个版本的应用程序互不冲突。
            applicationIdSuffix ".debug"
        }
    }

    // 设置 Java 编译选项，包括源代码和目标兼容性。
    compileOptions {
        sourceCompatibility JavaVersion.VERSION_1_8
        targetCompatibility JavaVersion.VERSION_1_8
    }
    // 设置 Kotlin 编译选项，包括 JVM 目标版本。
    kotlinOptions {
        jvmTarget = '1.8'
    }

    // 配置构建中的一些特定功能
    buildFeatures {
        viewBinding true // 启用了 View Binding 功能。
    }

    // sourceSets 可以用于指定不同类型的源代码，例如主要的应用程序代码、测试代码、Android 测试代码等。
    sourceSets {
        main {
            java.srcDirs = ['src/main/java']
            res.srcDirs = ['src/main/res']
        }

        // test 源代码集用于存放针对普通 Java 代码的单元测试（Unit Tests）。
        test {
            java.srcDirs = ['src/test/java']
        }

        // androidTest 源代码集用于与 Android 平台交互的集成测试。
        androidTest {
            java.srcDirs = ['src/androidTest/java']
        }
    }
}

// 列举应用程序的依赖项。使用了 AndroidX 库、Kotlin 标准库、Material Design 库、Retrofit HTTP 客户端库、Kotlin 协程库等。
dependencies {
    implementation 'androidx.core:core-ktx:1.8.0'
    implementation 'androidx.appcompat:appcompat:1.6.1'
    implementation 'com.google.android.material:material:1.5.0'
    implementation 'androidx.constraintlayout:constraintlayout:2.1.4'
    implementation 'androidx.recyclerview:recyclerview:1.3.0'
    implementation "androidx.lifecycle:lifecycle-extensions:2.2.0"
    implementation "androidx.lifecycle:lifecycle-livedata-ktx:2.6.1"
    implementation "androidx.swiperefreshlayout:swiperefreshlayout:1.1.0"
    implementation 'com.squareup.retrofit2:retrofit:2.6.1'
    implementation 'com.squareup.retrofit2:converter-gson:2.6.1'
    implementation "org.jetbrains.kotlinx:kotlinx-coroutines-core:1.6.4"
    implementation "org.jetbrains.kotlinx:kotlinx-coroutines-android:1.6.4"
}
```
