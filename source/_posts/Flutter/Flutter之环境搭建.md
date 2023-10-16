---
title: Flutter之环境搭建
date: 2023-10-10 15:18:42
tags: Flutter
---

参考文档

[flutter 环境搭建](https://blog.csdn.net/woyebuzhidao321/article/details/128413281)（推荐）

[fultter安装配置](https://blog.csdn.net/Milan__Kundera/article/details/125780788)

[flutter doctor --android-licenses 【报错】Exception in thread “main“ Android sdkmanager tool was foun](https://blog.csdn.net/weixin_45862329/article/details/129861888)

[Flutter(二)之环境搭建](https://juejin.cn/post/6844903935132581902)

[Android项目仓库配置国内加速阿里云镜像](https://blog.csdn.net/aa390481978/article/details/123823571)

<!-- more -->

## 环境搭建整体流程

1. 选择操作系统（macOS）
2. 安装 [Flutter SDK](https://flutter.dev/docs/development/tools/sdk/releases)
3. 配置环境变量
4. 安装模拟器（iOS和Android）
5. 安装开发工具（Android Studio）
6. 创建 Flutter 项目

## 选择操作系统

* 操作系统 macOS
* 处理器 Intel

## 安装 Flutter SDK

[下载 Flutter SDK](https://docs.flutter.dev/release/archive?tab=macos#macos)

![01](Flutter之环境搭建/01.png)

1. 选择自己的操作系统和最新稳定的版本（Stable版本）
2. mac 安装包有 arm64 和 x64，处理器是 Intel 选 x64 下载，如果是 Apple M1 就选 arm64 下载。

此次下载的是第一个 3.13.6（x64）。

安装：

1. 解压下载好的 SDK
2. 将 flutter 文件拖到应用程序中（/Applications/flutter）

## 配置环境变量

因为后面需要使用命令行执行 Flutter 命令，所以需要配置对应的环境变量。

一、前往文件：`~/.bath_profile`
二、配置环境变量

```js
# 确定 flutter 文件路径
export FLUTTER_HOME=/Applications/flutter
# 配置 Flutter 环境变量
export PATH=$PATH:$FLUTTER_HOME/bin
# 配置 Dart 环境变量
export PATH=$PATH:$FLUTTER_HOME/bin/cache/dart-sdk/bin
```

执行 `source ~/.bash_profile` 命令，使其生效。

后续打开终端第一次运行 flutter 命令前，需要先执行以下 `source ~/.bash_profile` 命令：

```js
% source ~/.bash_profile
% flutter doctor 
```

三、检查是否安装成功

使用 `flutter --version` 命令查看 flutter 版本，检查是否安装成功。

```js
 % flutter --version
Flutter 3.13.6 • channel stable • https://github.com/flutter/flutter.git
Framework • revision ead455963c (13 days ago) • 2023-09-26 18:28:17 -0700
Engine • revision a794cf2681
Tools • Dart 3.1.3 • DevTools 2.25.0

  ╔════════════════════════════════════════════════════════════════════════════╗
  ║                 Welcome to Flutter! - https://flutter.dev                  ║
  ║                                                                            ║
  ║ The Flutter tool uses Google Analytics to anonymously report feature usage ║
  ║ statistics and basic crash reports. This data is used to help improve      ║
  ║ Flutter tools over time.                                                   ║
  ║                                                                            ║
  ║ Flutter tool analytics are not sent on the very first run. To disable      ║
  ║ reporting, type 'flutter config --no-analytics'. To display the current    ║
  ║ setting, type 'flutter config'. If you opt out of analytics, an opt-out    ║
  ║ event will be sent, and then no further information will be sent by the    ║
  ║ Flutter tool.                                                              ║
  ║                                                                            ║
  ║ By downloading the Flutter SDK, you agree to the Google Terms of Service.  ║
  ║ Note: The Google Privacy Policy describes how data is handled in this      ║
  ║ service.                                                                   ║
  ║                                                                            ║
  ║ Moreover, Flutter includes the Dart SDK, which may send usage metrics and  ║
  ║ crash reports to Google.                                                   ║
  ║                                                                            ║
  ║ Read about data we send with crash reports:                                ║
  ║ https://flutter.dev/docs/reference/crash-reporting                         ║
  ║                                                                            ║
  ║ See Google's privacy policy:                                               ║
  ║ https://policies.google.com/privacy                                        ║
  ╚════════════════════════════════════════════════════════════════════════════╝


The Flutter CLI developer tool uses Google Analytics to report usage and
diagnostic data
along with package dependencies, and crash reporting to send basic crash
reports.
This data is used to help improve the Dart platform, Flutter framework, and
related tools.

Telemetry is not sent on the very first run.
To disable reporting of telemetry, run this terminal command:

flutter --disable-telemetry.
If you opt out of telemetry, an opt-out event will be sent,
and then no further information will be sent.
This data is collected in accordance with the
Google Privacy Policy (https://policies.google.com/privacy).

You have received two consent messages because the flutter tool is migrating to
a new analytics system. Disabling analytics collection will disable both the
legacy and new analytics collection systems. You can disable analytics reporting
by running `flutter --disable-telemetry`
```

四、配置镜像

flutter 项目有很多依赖，在国内下载这些依赖比较慢，可以将他们的安装源换成国内的。

编辑 `~/.bash_profile` 文件：

```js
export PUB_HOSTED_URL=https://pub.flutter-io.cn
export FLUTTER_STORAGE_BASE_URL= https://storage.flutter-io.cn
```

执行 `source ~/.bash_profile` 命令，使其生效。

## 配置 iOS 环境

1. 前往 Appstore 下载 Xcode。

2. 打开 Xcode，点击左上角 Xcode - Open Developer Tool - Simulator 打开模拟器

3. 模拟器被打开后，点击 Hardware - Device 选择需要的模拟器

![02](Flutter之环境搭建/02.png)

![03](Flutter之环境搭建/03.png)

## 配置 Android 环境

一、下载安装 Android Studio

1. 下载 [Android Studio](https://developer.android.com/studio?utm_source=android-studio&hl=zh-cn)
2. 下载完成后，双击安装包开始安装。
3. 「下一步」。
4. 安装完成后，在终端输入 `java -version` 验证是否安装成功。

```js
% java -version
java version "14.0.2" 2020-07-14
Java(TM) SE Runtime Environment (build 14.0.2+12-46)
Java HotSpot(TM) 64-Bit Server VM (build 14.0.2+12-46, mixed mode, sharing)
```

二、设置显示选项

![04](Flutter之环境搭建/04.png)

三、打开模拟器

![05](Flutter之环境搭建/05.png)

添加设备：【Create device】-【Phone】-【Pixel 3】-【Next】

![06](Flutter之环境搭建/06.png)

运行模拟器：

![07](Flutter之环境搭建/07.png)

![08](Flutter之环境搭建/08.png)

四、配置flutter环境

1、创建一个新项目，点击【New Project】

![09](Flutter之环境搭建/09.png)

2、设置项目信息，点击 【finish】

![10](Flutter之环境搭建/10.png)

3、创建完成

![11](Flutter之环境搭建/11.png)

4、打开设置页面，点击【Android Studio】-【Settings】

![12](Flutter之环境搭建/12.png)

5、下载安装 flutter，点击【Pluglns】- 搜索“flutter” - 点击【Install】开始下载，下载完成后点击【Restart IDE】重启 Android Studio。

![13](Flutter之环境搭建/13.png)

重启后打开【Pluglns】可以看到 Flutter、Dart 都已经下载安装好了。

![14](Flutter之环境搭建/14.png)

五、安装 Android SDK

Android SDK 是针对安卓开发的套件，如果最新的 Android SDK 存在兼容性问题，可以单独安装指定版本的 Android SDK。如添加 Android SDK Platform 29：

1、打开【Tools】-【SDK Manager】（或点击【Android Studio】-【Settings】）

![15](Flutter之环境搭建/15.png)

2、点击【Appearance & Behavior】-【System Settings】-【Android SDK】，添加 Android SDK Platform 29

【SDK Platforms】-【SDK Tools】-【Show Package Details】-【29.0.2】

![17](Flutter之环境搭建/17.png)

【SDK Platforms】-【Show Package Details】-【Android SDK Platform 29】，最后点击【OK】

![16](Flutter之环境搭建/16.png)

![18](Flutter之环境搭建/18.png)

下载完成点击【Finish】

![19](Flutter之环境搭建/19.png)

六、管理设备

1、点击【Tools】-【Device Manager】

![20](Flutter之环境搭建/20.png)

2、点击【Create Device】

![21](Flutter之环境搭建/21.png)

3、创建新的设备

![22](Flutter之环境搭建/22.png)

![23](Flutter之环境搭建/23.png)

![24](Flutter之环境搭建/24.png)

![25](Flutter之环境搭建/25.png)

4、创建成功

![26](Flutter之环境搭建/26.png)

七、执行 `flutter doctor` 命令检查当前 Flutter 环境

如果是第一次运行 flutter，需要先执行 `source ~/.bash_profile`，在执行 `flutter doctor`。

![27](Flutter之环境搭建/27.png)

按照提示，执行 `flutter doctor --android-licenses`

![28](Flutter之环境搭建/28.png)

然后，一路输入 `y` + 回车

![29](Flutter之环境搭建/29.png)

![30](Flutter之环境搭建/30.png)

![31](Flutter之环境搭建/31.png)

![32](Flutter之环境搭建/32.png)

![33](Flutter之环境搭建/33.png)

最后再次执行 `flutter doctor`，成功 ✌🏻

![34](Flutter之环境搭建/34.png)

## 遇到的问题

第一次下载 Android SDK 时选择的是最新版本

![35](Flutter之环境搭建/35.png)

执行 `flutter doctor` 报错

![36](Flutter之环境搭建/36.png)

这个时候，执行 `flutter doctor --android-licenses` 就会报错：

![37](Flutter之环境搭建/37.png)

参考文档：[记坑：flutter doctor --android-licenses 【报错】Exception in thread “main“ Android sdkmanager tool was foun](https://blog.csdn.net/weixin_45862329/article/details/129861888)

解决方法：将Android Studio ->Android SDK ->SDK Tools处，勾选版本为8.0 的Android SDK Command-line Tools，并取消勾选版本为9.0 的Android SDK Command-line Tools.

![38](Flutter之环境搭建/38.png)

安装成功后，在执行 `flutter doctor --android-licenses` 就不会报错了。

## 创建 Flutter 项目

### 使用 Android Studio 创建

点击【File】-【New】-【New Flutter Project】

![39](Flutter之环境搭建/39.png)

![40](Flutter之环境搭建/40.png)

![41](Flutter之环境搭建/41.png)

打开一个新的窗口【New Window】

![42](Flutter之环境搭建/42.png)

运行项目

1. 选择设备
2. 运行项目
3. 停止运行

![43](Flutter之环境搭建/43.png)

### 使用终端创建

1. 进入到需要创建项目的目录
2. 使用 `flutter create helloflutter` 命令创建（注意：后面的名称不能由特殊符号，也不能由大写）

```js
 % cd /Users/xx/Desktop/Flutter/HelloFlutter 
 % flutter create helloflutter
Signing iOS app for device deployment using developer identity: "iPhone
Developer: Xiaoshuo Li (SH8N382R65)"
Creating project helloflutter...
Resolving dependencies in helloflutter... 
Got dependencies in helloflutter.
Wrote 129 files.

All done!
You can find general documentation for Flutter at: https://docs.flutter.dev/
Detailed API documentation is available at: https://api.flutter.dev/
If you prefer video documentation, consider:
https://www.youtube.com/c/flutterdev

In order to run your application, type:

  $ cd helloflutter
  $ flutter run

Your application code is in helloflutter/lib/main.dart.

```
