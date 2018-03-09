---
title: React-Native多个项目完成后的总结
abbrlink: CQvxFvnTvi1f5d62wARM4w
date: 2017-07-07 15:01:29
categories:
- 手机APP开发
- React Native
tags:
- React
- React Native
---
> 时隔一年，已经有2个RN的项目上线发布，从对手机APP开发的一无所知，到现在可以熟练开发发布应用，期间遇到了很多让人绝望的问题，好在最后都解决了。本文就把那些在开发过程中所遇到的问题及解决方案记录下来。

1. FlatList 初始化时不显示列表，需要拖拽一下才显示
   当 React Navigation 与 FlatList 配合使用时，会出现初始化时不显示列表的bug，必须要拖拽一下，才会正常显示。
   <!--more-->
   **解决方案：** 此时需要给FlatList设置`removeClippedSubviews={false}` 属性，即可解决问题(目前最新版0.46貌似已经解决了此bug)，官方文档解释如下：
> 注意：removeClippedSubviews属性目前是不必要的，而且可能会引起问题。如果你在某些场景碰到内容不渲染的情况（比如使用LayoutAnimation时），尝试设置removeClippedSubviews={false}。我们可能会在将来的版本中修改此属性的默认值。

2. 安卓端React Navigation的TabNavigator选项卡与react-native-scrollable-tab-view、FlatList一起使用，只显示第一页的内容。
   **解决方案：** 给TabNavigator增加`lazy: true` 属性，官方解释，*whether to lazily render tabs as needed as opposed to rendering them upfront*

3. Windows平台下安卓端USB真机调试错误

   使用数据线连接电脑，保证手机和电脑在同一局域网下，如果`adb devices`找不到设备，需要确保手机驱动已安装成功（可以使用各类手机助手或`echo 0x2a45 > .android/adb_usb.ini` 尝试安装驱动），building到97%后会报错：
   <!-- more -->
```sh
BUILD FAILED

FAILURE: Build failed with an exception.

* What went wrong:
Execution failed for task ':app:installDebug'.
> com.android.builder.testing.api.DeviceException: com.android.ddmlib.InstallException: Unable to upload some APKs

* Try:
Run with --stacktrace option to get the stack trace. Run with --info or --debug option to get more log output.
Total time: 1 mins 25.602 secs
Could not install the app on the device, read the error above for details.
Make sure you have an Android emulator running or a device connected and have
set up your Android development environment:
https://facebook.github.io/react-native/docs/android-setup.html
[vscode-react-native] Finished executing: react-native.cmd run-android
[Error] "Could not debug. Unknown error"
```

​	**错误原因：**国产安卓定制系统（例如：魅族Flyme）的错误。

​	**解决方案：**
	在项目中`android/build.gradle` 如下第8行，版本号改为 1.2.3后重新编译即可解决问题。issue：https://github.com/facebook/react-native/issues/8868
```Java
buildscript {
    repositories {
        jcenter()
    }
    dependencies {
        classpath 'com.android.tools.build:gradle:1.2.3'  //这里的版本号改为1.2.3
    }
}
```

4. 没有百分比宽高的解决办法

   flex:1 可以相当于 height:100% ；
   alignSelf:'stretch' 可以相当于 width:100% ；

   issue： https://github.com/facebook/css-layout/issues/57#ref-issue-86696744

5. TextInput的样式兼容问题
   android有默认的padding，需要给`padding: 0`，重置一下，才能和ios表现一致。

6. 如何修改Android初始化项目包名？

   * 修改`android/app/build.gradle`里的`applicationId`，为新包名。
   * 修改`android/app/src/main/AndroidManifest.xml`里的`package`，为新包名。
   * 在`android/app/src/main/java/com`下根据新包名中多出的层级，新建目录。
   * 打开`android/app/src/main/java/com/xxx/yyy/myProject/MainActivity.java`和`MainApplication.java`修改第一行为：`package com.xxx.yyy.myproject;`