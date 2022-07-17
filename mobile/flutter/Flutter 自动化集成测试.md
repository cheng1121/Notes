# Flutter 自动化集成测试

目前Selenium不支持原生及Flutter App的自动化集成测试，不论是Selenium Webdriver层面还是SeleniumIDE(脚本录制、回放)。Flutter App要执行集成测试，可以借助Flutter Driver来实现（Flutter Driver等同于WebDriver在web测试中的作用）在真机和模拟器上的集成化测试。 

> 注：Selenium WebDriver使用了[W3C WebDriver规范](https://www.w3.org/TR/webdriver/),WebDriver为语言中立的相关协议，可以在外部程序中控制浏览器的行为。也可参考[WebDriver的Github站点](https://github.com/w3c/webdriver)

Flutter Driver提供了编写和运行Flutter集成测试的基础，但是存在一些限制，目前只支持使用Dart语言编写集成测试脚本，并且一次运行只能支持一台真机或者模拟器。优点是纯粹简洁，只需要Flutter SDK和flutter_driver及integration_test等两个第三方包即可实现自动化集成测试。

> 注：Flutter Driver对于多台设备和模拟器的支持，可以通过脚本运行多个命令进程来执行多个真机的同时测试。

Google针对Flutter App的集成化测试提供另外的方式，可以支持设备农场式式（device farm）的测试。但是要借助google firebase/firestore服务，不推荐使用。（firebase为google的Baas-Backend as a Service服务，  [Firebase](https://firebase.google.com/)  gives you access to backend services for mobile applications—including  authentication, storage, database, and hosting—without maintaining your  own servers，firestore为google提供的云端数据库）

另外，App的集成化测试还存在一些更高级的工具，比如[Appium](http://appium.io)，[Katalon](https://www.katalon.com),其中Katalon可能还支持App测试脚本的录制（非编程方式制作脚本）。但是可用性、易用性还需要进一步研究。



## Flutter App集成测试样例

要编写Flutter App的自动化集成测试用例，实际上[单独使用flutter_driver也可以完成](https://flutter.dev/docs/cookbook/testing/integration/introduction)，但是目前google建议[配合integration_test包进行集成测试](https://flutter.dev/docs/testing/integration-tests)，integration_test包对flutter driver进行了一些扩展，使得集成测试的编写更加简洁（类似UI窗体测试用例），并且扩展了集成测试的运行场景。不论哪种方式编写的flutter集成测试用例，最终都需要使用`flutter driver`命令运行。

### 项目基本设置、测试用例、集成测试执行命令样例解析

- 引入集成测试所需要的第三方包，在yaml文件的dev_dependencies下添加

  ```yaml
  dev_dependencies:
  
    #flutter 单元测试和Widgets测试
    flutter_test:
      sdk: flutter
    #flutter 集成测试
    integration_test: ^1.0.0
    flutter_driver:
      sdk: flutter
  ```

- 项目根目录下添加`integration_test`目录(要和lib目录同级)，并在目录中添加包含具体测试用例的`xx_test.dart`文件

  ```dart
  ///xx_test.dart文件内容
  import 'package:flutter_test/flutter_test.dart';
  import 'package:integration_test/integration_test.dart';
  
  void main(){
    ///集成测试环境的初始化
    IntegrationTestWidgetsFlutterBinding.ensureInitialized();
  
    testWidgets('test====', (WidgetTester tester) async{
      
      ///启动MyApp widget，pumpWidget后不会再次刷新widget，除非  
      await tester.pumpWidget(MyApp());
  
      
      ///find是用来查找目标Widget及其中相关内容的辅助对象
      ///expect执行验证widget行为是否符合预期
      ///`findsOneWidget`常量表示有符合预期的widget，`findsNothing`表示没有符合预期的widget
      expect(find.text('0'), findsOneWidget);
      expect(find.text('1'), findsNothing);
        
      ///模拟add icon的点击事件30次
      for (int i = 0; i < 30; i++) {
        await tester.tap(find.byIcon(Icons.add));
  
        ///重建Widget(即刷新页面)
        await tester.pump(const Duration(milliseconds: 500));
        expect(find.text('${i +1}'), findsOneWidget);
      }
        
      ///结束状态的验证
      expect(find.text('30'), findsOneWidget);
    });
  }
  ```

- 同时在项目根目录下添加`test_driver`文件夹，在此文件夹中添加`integration_test_driver.dart`文件

    ```dart
    ///`integration_test_driver.dart`文件内容
    import 'package:integration_test/integration_test_driver_extended.dart';
    
    ///flutter driver命令运行的main方法
    Future<void> main() {
      return integrationDriver();
    }
    ```
    
- 包含集成测试的项目目录结构

  ```
  
  lib/
    ...
  integration_test/
    demo_test.dart
    other_test.dart
    ...
  test/
    widget_test.dart
    ...
  test_driver/
    integration_test_driver.dart
  
  ```
  
  
  
- 使用`flutter_driver`命令运行程序

  

  - 在终端执行命令在真机/模拟器中执行集成测试
  
    ```shell
  flutter driver --driver=test_driver/integration_test_driver.dart --target=integration_test/demo_test.dart
    ```

  - 测试结果

    android真机测试过程录屏
  
    <video src=".\flutter_integration_test_video_capture.mp4"></video>
    
    测试通过的结果
    
    ```shell
    $>>> flutter driver --driver=test_driver/integration_test_driver.dart --target=integration_test/demo_test.dart
    Using device EVR AL00.
    Starting application: integration_test/demo_test.dart
    "build/app/outputs/flutter-apk/app.apk" does not exist.
    Running Gradle task 'assembleDebug'...                                  
    Running Gradle task 'assembleDebug'... Done                        33.3s
    ✓ Built build/app/outputs/flutter-apk/app-debug.apk.
    Installing build/app/outputs/flutter-apk/app.apk...                10.1s
    I/flutter (24738): Observatory listening on http://127.0.0.1:39839/dMaCVohstJ4=/
    VMServiceFlutterDriver: Connecting to Flutter application at http://127.0.0.1:60777/dMaCVohstJ4=/
    VMServiceFlutterDriver: Isolate found with number: 1021301541835079
    VMServiceFlutterDriver: Isolate is paused at start.
    VMServiceFlutterDriver: Attempting to resume isolate
    I/flutter (24738): 00:00 +0: test====
  VMServiceFlutterDriver: Connected to Flutter application.
    I/flutter (24738): 00:00 +1: (tearDownAll)
I/flutter (24738): 00:00 +2: All tests passed!
    All tests passed.
  Stopping application instance.
    ```
    
    测试失败项的结果：
    
    ```shell
    $>>> flutter driver --driver=test_driver/integration_test_driver.dart --target=integration_test/demo_test.dart
    Using device EVR AL00.
    Starting application: integration_test/demo_test.dart
    Installing build/app/outputs/flutter-apk/app.apk...                 6.0s
    Running Gradle task 'assembleDebug'...                                  
    Running Gradle task 'assembleDebug'... Done                        13.9s
    ✓ Built build/app/outputs/flutter-apk/app-debug.apk.
    I/flutter (26295): Observatory listening on http://127.0.0.1:39077/K34itROO7Lo=/
    I/flutter (26295): 00:00 +0: test====                                   
    I/flutter (26295): 00:00 +1: (tearDownAll)                              
    I/flutter (26295): 00:00 +2: All tests passed!                          
    Installing build/app/outputs/flutter-apk/app.apk...                10.9s
    I/flutter (26494): Observatory listening on http://127.0.0.1:33961/sVRk9n0QwpI=/
    VMServiceFlutterDriver: Connecting to Flutter application at http://127.0.0.1:61577/sVRk9n0QwpI=/
    VMServiceFlutterDriver: Isolate found with number: 2745649757703519
    VMServiceFlutterDriver: Isolate is paused at start.
    VMServiceFlutterDriver: Attempting to resume isolate
    I/flutter (26494): 00:00 +0: test====
    VMServiceFlutterDriver: Connected to Flutter application.
    I/flutter (26494): (The following exception is now available via WidgetTester.takeException:)
    I/flutter (26494): ══╡ EXCEPTION CAUGHT BY FLUTTER TEST FRAMEWORK ╞════════════════════════════════════════════════════
    I/flutter (26494): The following TestFailure object was thrown running a test:
    I/flutter (26494):   Expected: <5>
    I/flutter (26494):   Actual: <4>
    I/flutter (26494): 
    I/flutter (26494): When the exception was thrown, this was the stack:
    I/flutter (26494): #4      main.<anonymous closure>
    (file:///Users/cheng/flutter/projects/UnitTestDemo/integration_test/demo_test.dart:9:5)
    I/flutter (26494): #5      testWidgets.<anonymous closure>.<anonymous closure>
    (package:flutter_test/src/widget_tester.dart:146:29)
    I/flutter (26494): <asynchronous suspension>
    I/flutter (26494): #6      testWidgets.<anonymous closure>.<anonymous closure>
    (package:flutter_test/src/widget_tester.dart)
    I/flutter (26494): #7      TestWidgetsFlutterBinding._runTestBody (package:flutter_test/src/binding.dart:784:19)
    I/flutter (26494): <asynchronous suspension>
    I/flutter (26494): #10     TestWidgetsFlutterBinding._runTest (package:flutter_test/src/binding.dart:764:14)
    I/flutter (26494): #11     LiveTestWidgetsFlutterBinding.runTest (package:flutter_test/src/binding.dart:1592:12)
    I/flutter (26494): #12     IntegrationTestWidgetsFlutterBinding.runTest
    (package:integration_test/integration_test.dart:194:17)
    I/flutter (26494): #13     testWidgets.<anonymous closure> (package:flutter_test/src/widget_tester.dart:138:24)
    I/flutter (26494): #14     Declarer.test.<anonymous closure>.<anonymous closure>
    (package:test_api/src/backend/declarer.dart:175:19)
    I/flutter (26494): <asynchronous suspension>
    I/flutter (26494): #15     Declarer.test.<anonymous closure>.<anonymous closure>
    (package:test_api/src/backend/declarer.dart)
    I/flutter (26494): #20     Declarer.test.<anonymous closure> (package:test_api/src/backend/declarer.dart:173:13)
    I/flutter (26494): #21     Invoker.waitForOutstandingCallbacks.<anonymous closure>
    (package:test_api/src/backend/invoker.dart:231:15)
    I/flutter (26494): #26     Invoker.waitForOutstandingCallbacks (package:test_api/src/backend/invoker.dart:228:5)
    I/flutter (26494): #27     Invoker._onRun.<anonymous closure>.<anonymous closure>.<anonymous closure>
    (package:test_api/src/backend/invoker.dart:383:17)
    I/flutter (26494): <asynchronous suspension>
    I/flutter (26494): #28     Invoker._onRun.<anonymous closure>.<anonymous closure>.<anonymous closure>
    (package:test_api/src/backend/invoker.dart)
    I/flutter (26494): #33     Invoker._onRun.<anonymous closure>.<anonymous closure>
    (package:test_api/src/backend/invoker.dart:370:9)
    I/flutter (26494): #34     Invoker._guardIfGuarded (package:test_api/src/backend/invoker.dart:415:15)
    I/flutter (26494): #35     Invoker._onRun.<anonymous closure> (package:test_api/src/backend/invoker.dart:369:7)
    I/flutter (26494): #42     Invoker._onRun (package:test_api/src/backend/invoker.dart:368:11)
    I/flutter (26494): #43     LiveTestController.run (package:test_api/src/backend/live_test_controller.dart:153:11)
    I/flutter (26494): (elided 31 frames from dart:async and package:stack_trace)
    I/flutter (26494): 
    I/flutter (26494): This was caught by the test expectation on the following line:
    I/flutter (26494):   file:///Users/cheng/flutter/projects/UnitTestDemo/integration_test/demo_test.dart line 9
    I/flutter (26494): The test description was:
    I/flutter (26494):   test====
    I/flutter (26494): ════════════════════════════════════════════════════════════════════════════════════════════════════
    I/flutter (26494): (If WidgetTester.takeException is called, the above exception will be ignored. If it is not, then the
    above exception will be dumped when another exception is caught by the framework or when the test ends, whichever happens
    first, and then the test will fail due to having not caught or expected the exception.)
    I/flutter (26494): ══╡ EXCEPTION CAUGHT BY FLUTTER TEST FRAMEWORK ╞════════════════════════════════════════════════════
    I/flutter (26494): The following TestFailure object was thrown running a test:
    I/flutter (26494):   Expected: <5>
    I/flutter (26494):   Actual: <4>
    I/flutter (26494): 
    I/flutter (26494): When the exception was thrown, this was the stack:
    I/flutter (26494): #4      main.<anonymous closure>
    (file:///Users/cheng/flutter/projects/UnitTestDemo/integration_test/demo_test.dart:9:5)
    I/flutter (26494): #5      testWidgets.<anonymous closure>.<anonymous closure>
    (package:flutter_test/src/widget_tester.dart:146:29)
    I/flutter (26494): <asynchronous suspension>
    I/flutter (26494): #6      testWidgets.<anonymous closure>.<anonymous closure>
    (package:flutter_test/src/widget_tester.dart)
    I/flutter (26494): #7      TestWidgetsFlutterBinding._runTestBody (package:flutter_test/src/binding.dart:784:19)
    I/flutter (26494): <asynchronous suspension>
    I/flutter (26494): #10     TestWidgetsFlutterBinding._runTest (package:flutter_test/src/binding.dart:764:14)
    I/flutter (26494): #11     LiveTestWidgetsFlutterBinding.runTest (package:flutter_test/src/binding.dart:1592:12)
    I/flutter (26494): #12     IntegrationTestWidgetsFlutterBinding.runTest
    (package:integration_test/integration_test.dart:194:17)
    I/flutter (26494): #13     testWidgets.<anonymous closure> (package:flutter_test/src/widget_tester.dart:138:24)
    I/flutter (26494): #14     Declarer.test.<anonymous closure>.<anonymous closure>
    (package:test_api/src/backend/declarer.dart:175:19)
    I/flutter (26494): <asynchronous suspension>
    I/flutter (26494): #15     Declarer.test.<anonymous closure>.<anonymous closure>
    (package:test_api/src/backend/declarer.dart)
    I/flutter (26494): #20     Declarer.test.<anonymous closure> (package:test_api/src/backend/declarer.dart:173:13)
    I/flutter (26494): #21     Invoker.waitForOutstandingCallbacks.<anonymous closure>
    (package:test_api/src/backend/invoker.dart:231:15)
    I/flutter (26494): #26     Invoker.waitForOutstandingCallbacks (package:test_api/src/backend/invoker.dart:228:5)
    I/flutter (26494): #27     Invoker._onRun.<anonymous closure>.<anonymous closure>.<anonymous closure>
    (package:test_api/src/backend/invoker.dart:383:17)
    I/flutter (26494): <asynchronous suspension>
    I/flutter (26494): #28     Invoker._onRun.<anonymous closure>.<anonymous closure>.<anonymous closure>
    (package:test_api/src/backend/invoker.dart)
    I/flutter (26494): #33     Invoker._onRun.<anonymous closure>.<anonymous closure>
    (package:test_api/src/backend/invoker.dart:370:9)
    I/flutter (26494): #34     Invoker._guardIfGuarded (package:test_api/src/backend/invoker.dart:415:15)
    I/flutter (26494): #35     Invoker._onRun.<anonymous closure> (package:test_api/src/backend/invoker.dart:369:7)
    I/flutter (26494): #42     Invoker._onRun (package:test_api/src/backend/invoker.dart:368:11)
    I/flutter (26494): #43     LiveTestController.run (package:test_api/src/backend/live_test_controller.dart:153:11)
    I/flutter (26494): (elided 31 frames from dart:async and package:stack_trace)
    I/flutter (26494): 
    I/flutter (26494): This was caught by the test expectation on the following line:
    I/flutter (26494):   file:///Users/cheng/flutter/projects/UnitTestDemo/integration_test/demo_test.dart line 9
    I/flutter (26494): The test description was:
    I/flutter (26494):   test====
    I/flutter (26494): ════════════════════════════════════════════════════════════════════════════════════════════════════
    I/flutter (26494): 00:00 +0: test==== [E]
    I/flutter (26494):   Test failed. See exception logs above.
    I/flutter (26494):   The test description was: test====
    I/flutter (26494):   
    I/flutter (26494): 00:00 +0 -1: (tearDownAll)
    Failure Details:
    Failure in method: test====
    ══╡ EXCEPTION CAUGHT BY FLUTTER TEST FRAMEWORK ╞═════════════════
    The following TestFailure object was thrown running a test:
      Expected: <5>
      Actual: <4>
    
    When the exception was thrown, this was the stack:
    #4      main.<anonymous closure> (file:///Users/cheng/flutter/projects/UnitTestDemo/integration_test/demo_test.dart:9:5)
    #5      testWidgets.<anonymous closure>.<anonymous closure> (package:flutter_test/src/widget_tester.dart:146:29)
    <asynchronous suspension>
    #6      testWidgets.<anonymous closure>.<anonymous closure> (package:flutter_test/src/widget_tester.dart)
    #7      TestWidgetsFlutterBinding._runTestBody (package:flutter_test/src/binding.dart:784:19)
    <asynchronous suspension>
    #10     TestWidgetsFlutterBinding._runTest (package:flutter_test/src/binding.dart:764:14)
    #11     LiveTestWidgetsFlutterBinding.runTest (package:flutter_test/src/binding.dart:1592:12)
    #12     IntegrationTestWidgetsFlutterBinding.runTest (package:integration_test/integration_test.dart:194:17)
    #13     testWidgets.<anonymous closure> (package:flutter_test/src/widget_tester.dart:138:24)
    #14     Declarer.test.<anonymous closure>.<anonymous closure> (package:test_api/src/backend/declarer.dart:175:19)
    <asynchronous suspension>
    #15     Declarer.test.<anonymous closure>.<anonymous closure> (package:test_api/src/backend/declarer.dart)
    #20     Declarer.test.<anonymous closure> (package:test_api/src/backend/declarer.dart:173:13)
    #21     Invoker.waitForOutstandingCallbacks.<anonymous closure> (package:test_api/src/backend/invoker.dart:231:15)
    #26     Invoker.waitForOutstandingCallbacks (package:test_api/src/backend/invoker.dart:228:5)
    #27     Invoker._onRun.<anonymous closure>.<anonymous closure>.<anonymous closure> (package:test_api/src/backend/invoker.dart:383:17)
    <asynchronous suspension>
    #28     Invoker._onRun.<anonymous closure>.<anonymous closure>.<anonymous closure> (package:test_api/src/backend/invoker.dart)
    #33     Invoker._onRun.<anonymous closure>.<anonymous closure> (package:test_api/src/backend/invoker.dart:370:9)
    #34     Invoker._guardIfGuarded (package:test_api/src/backend/invoker.dart:415:15)
    #35     Invoker._onRun.<anonymous closure> (package:test_api/src/backend/invoker.dart:369:7)
    #42     Invoker._onRun (package:test_api/src/backend/invoker.dart:368:11)
    #43     LiveTestController.run (package:test_api/src/backend/live_test_controller.dart:153:11)
    (elided 31 frames from dart:async and package:stack_trace)
    
    This was caught by the test expectation on the following line:
      file:///Users/cheng/flutter/projects/UnitTestDemo/integration_test/demo_test.dart line 9
    The test description was:
      test====
    ═════════════════════════════════════════════════════════════════
    
    end of failure 1
    
    
  
    I/flutter (26494): 00:00 +1 -1: Some tests failed.
  Stopping application instance.
    Driver tests failed: 1
    
    ```
    
    


## Flutter 集成测试参考资料

### 官网资料

package地址:https://pub.dev/packages/integration_test

github地址：https://github.com/flutter/plugins/tree/master/packages/integration_test

flutter中文资源站：https://flutter.cn/docs/testing/integration-tests

### 其他资料

[Flutter集成测试的文章](https://www.cnblogs.com/fnng/p/13664254.html)，没有图形化的界面录制集成测试用例。可以模拟点击和触摸事件。

[Appium详细应用案例](https://blog.csdn.net/weixin_42717928/article/details/106753390)

[主流的自动化测试工具](https://cloud.tencent.com/developer/article/1458016)

[Katalon自动化测试工具软件](https://www.katalon.com)  

[Katalon移动端文档](https://www.katalon.com/mobile-testing/)

[Appium支持Flutter测试的开源项目，还不成熟](https://github.com/truongsinh/appium-flutter-driver)



### 扩展参考资料：

#### Selenium

##### 简介

一款开源的web自动化测试工具，支持分布式，支持java，js，python等语言，支持市面上几乎所有的浏览器，并且在所有支持这些浏览器的系统中都运行良好。

不支持非web项目

[官网地址](https://www.selenium.dev)

网址：https://www.selenium.dev

中文文档地址1：https://selenium-python-zh.readthedocs.io/en/latest/index.html

中文文档地址2：https://wizardforcel.gitbooks.io/selenium-doc/content/official-site/introduction.html

#### Appium

##### 简介

1. appium是开源的移动端自动化测试框架；

2. appium可以测试原生的、混合的、以及移动端的web项目；

3. appium可以测试ios，android应用（还有firefox os）

4. appium是跨平台的，可以用在osx，windows以及linux桌面系统上；

5. appium支持多种语言Ruby、Python、Java、JavaScript、Objective C、php、C#、RobotFramework

#### 官网和参考文档

官网：http://appium.io

官网文档：http://appium.io/docs/en/about-appium/intro/

官方中文文档：http://appium.io/docs/cn/about-appium/intro/

知乎上的简介：https://zhuanlan.zhihu.com/p/61971173