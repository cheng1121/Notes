
# Flutter开发bug汇总

### 去掉下巴
```
SystemChrome.setSystemUIOverlayStyle(SystemUiOverlayStyle.dark.copyWith(
  systemNavigationBarColor: Colors.transparent,
));
```

### ios真机调试报错
*2020年11月26日00:27:34*
真机调试报错: Could not install build/ios/iphoneos/Runner.app on xxxdevice
```
1. Could not install build/ios/iphoneos/Runner.app on xxxxxx.
2. Try launching Xcode and selecting "Product > Run" to fix the problem:
3.  open ios/Runner.xcworkspace
4. Error launching application on iPhonesk8p.

```
原因: 真机上没有信任运行的app!
解决: 手机设置->通用->设备管理->找到运行app设置信任


### Flutter 更新SDK后,运行项目会出现 'flutter 无法打开“iproxy”，因为无法验证开发者。'的弹窗问题
解决办法：
进入到项目根根目录: 
```
  1.$ rm -rf build
  2.$ flutter clean
  3.$ flutter build ios --debug

```

### demo项目ios编译报错
*2020年08月14日14:30:09*
```
targeted OS version does not support use of thread local variables in _term_exit for architecture x86_64
```
解决办法：xcode打开项目，点击左侧Runner -> 右侧General -> 点击下方的TARGETS中的Runner -> 找到Deployment info 中的Target 点击后再下拉菜单中选择 ios 9.3(flutter_ffmpeg支持的最低版本是9.3)

### gradle下载报SSL peer shut down incorrectly
*2020年12月11日21:54:55*
android目录中的gradle.properties中添加如下代理:
```
systemProp.http.proxyHost=fodev.org
systemProp.http.proxyPort=8118
systemProp.http.nonProxyHosts=*.jitpack.io, *.maven.org
systemProp.https.proxyHost=fodev.org
systemProp.https.proxyPort=8118
systemProp.https.nonProxyHosts=*.jitpack.io, *.maven.org

```

### release打包后报couldn't find "libflutter.so"
*2020年12月12日16:57:00*
[问题分析见](https://blog.csdn.net/z2008q/article/details/108621188)
解决办法：
- 在build.gradle中添加如下代码指定cup架构
```
///必须在apply flutter.gradle之前添加
project.setProperty('target-platform', 'android-arm')

```

- 打包时添加命令
```
 --target-platform=android-arm 
 ///or
 --target-platform=android-arm64

```

### 打包apk，使用zipalign命令报错
*2020年12月12日20:40:29*
原因：google使用新库zipflinger构建的apk，会自动对齐apk。所以尝试再次对齐时报错。
使用zipalign -c -v 4 test.apk验证对齐会返回成功


### ImagePicker在ios模拟器中崩溃
*2021年01月16日15:59:12*  
报NSInternalInconsistencyException错误  
此错误只在模拟器中发生，真机正常  
[详情记录](https://github.com/flutter/flutter/issues/68283)
