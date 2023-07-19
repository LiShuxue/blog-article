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

自动生成的 gradle 文件夹是 Gradle 包装器，其中包含一个 jar 文件和一个 配置文件，使用这个包装器可以让 Gradle 运行在一个特定的版本上，目的是创造一个独立于系统、系统配置和 Gradle 版本的可靠和可重复构建。

gradle.properties：用于指定 Gradle 构建工具包本身的设置，比如是否使用 AndroidX，最大堆大小等等。

```gradle
// setting.gradle
pluginManagement {
    repositories {
        google()
        mavenCentral()
        gradlePluginPortal()
    }
}
dependencyResolutionManagement {
    repositoriesMode.set(RepositoriesMode.FAIL_ON_PROJECT_REPOS)
    repositories {
        google()
        mavenCentral()
    }
}
rootProject.name = "Weather"
include ':app'

```

pluginManagement 指插件管理，指定插件下载的仓库。dependencyResolutionManagement 指依赖管理，指定依赖库下载的仓库。这些仓库是有顺序的，顺序决定先从哪个仓库下载。rootProject.name 指项目名称，include 用于指定参与构建的模块，一个根工程可以有很多子工程，也就是很多的 module。一个子工程只有在 setting 里设置了，gradle 才会去识别，才会在构建的时候被包含进去。

```gradle
// Project 下的 build.gradle
plugins {
    id 'com.android.application' version '7.3.1' apply false
    id 'com.android.library' version '7.3.1' apply false
    id 'org.jetbrains.kotlin.android' version '1.7.20' apply false
}
```

这里只是 plugin 的引用，apply false 的意思是不将该 plugin 应用于当前项目。

```gradle
// Module 下的 build.gradle
plugins {
    id 'com.android.application'
    id 'org.jetbrains.kotlin.android'
}

android {
    namespace 'com.example.weather'
    compileSdk 33

    defaultConfig {
        applicationId "com.example.weather"
        minSdk 24
        targetSdk 33
        versionCode 1
        versionName "1.0"
    }
    buildFeatures {
        viewBinding true
    }
    buildTypes {
        release {
            minifyEnabled false
            proguardFiles getDefaultProguardFile('proguard-android-optimize.txt'), 'proguard-rules.pro'
        }
    }
    compileOptions {
        sourceCompatibility JavaVersion.VERSION_1_8
        targetCompatibility JavaVersion.VERSION_1_8
    }
    kotlinOptions {
        jvmTarget = '1.8'
    }
}

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

先来看下 android { ... }，它是 Android 工程配置的入口。

namespace：应用程序的命名空间，主要用于访问应用程序资源。

compileSdk：编译所依赖的 Android SDK 的版本。

applicationId：APP 的唯一标识，也是包名。

minSdk：支持的最低 API Level，如指定 23，代表低于 Android 6 的机型不能使用这个 APP。

targetSdk：指该 APP 是基于哪个 Android 版本开发的，如指定 32，代表适配到 Android 12。

versionCode 和 versionName：指版本号和版本名称。

buildTypes：构建类型，这个主要用于打包。

compileOptions：指定 Java 环境版本。

kotlinOptions：指定 JVM 环境。

dependencies：添加依赖。
