# 第一个Flutter App
```
void main(){
  runApp(MyApp());
}


/**
 * 应用顶层的Widget
 *
 * 这个widget是无状态的
 * 与之对应的 有状态的widget 继承 StatefulWidget
 *
 */

class MyApp extends StatelessWidget{


  @override
  Widget build(BuildContext context) {
    // 创建内容  使用Material风格的应用
    return MaterialApp(
      //应用标题
      title: "我的FlutterApp",

      //应用主页
      home: Scaffold(
        //应用的标题栏
        appBar: AppBar(
          title: Text("FlutterDemo"),
        ),
        //应用标题栏下方的内容
        body: Center(
          //点击按钮
           child: RaisedButton(
             onPressed: _onPressed,
             //点击按钮上的文字
             child: Text("roll"),

           ),
        ),
      ),
    );
  }
}

//点击事件
void _onPressed(){
 debugPrint("点击了按钮");
}
```
下面分析下各个方法的作用

### main()
main方法是程序的入口，在其内部执行 runApp方法来得到应用顶层的视图

### MyApp类
在Flutter中所有的东西都是widget。继承StatelessWidget表示无状态的widget，也就是其只可以用于显示信息，不能有其他动作,比如弹出dialog
如果需要弹出dialog 则需要使用StatefulWidget

### MaterialApp 
Material风格的 app 
title设置app标题的名字，body设置要展示的内容

其他的内容见上方代码中的注释

热加载快捷键为command + \ 

保存也会自动进行热加载

debugPrint()在控制台输出log

**遇到bug时，最好先结束app，然后再点击绿色三角运行，不要使用热加载**


### 完整代码
展示了一个弹出dialog 并展示两个随机数的demo

```
import 'dart:math';

import 'package:flutter/material.dart';

void main() {
  runApp(MyApp());
}

/**
 * 应用顶层的Widget
 *
 * 这个widget是无状态的
 * 与之对应的 有状态的widget 继承 StatefulWidget
 *
 */

class MyApp extends StatelessWidget {
  @override
  Widget build(BuildContext context) {
    // 创建内容  使用Material风格的应用
    return MaterialApp(
      //应用标题
      title: "我的FlutterApp",

      //应用主页
      home: Scaffold(
        //应用的标题栏
        appBar: AppBar(
          title: Text("FlutterDemo"),
        ),
        //应用标题栏下方的内容
        body: Center(
          //点击按钮
          child: RollingButton(),
        ),
      ),
    );
  }
}

//定义一个按钮
class RollingButton extends StatefulWidget {
  //返回一个状态 state
  @override
  State<StatefulWidget> createState() {
    return _RollingState();
  }
}

//定义一个按钮状态
class _RollingState extends State<RollingButton> {
  @override
  Widget build(BuildContext context) {
    return RaisedButton(
      child: Text("Roll"),
      onPressed: _onPressed,
    );
  }

  //随机数
  final _random = Random();
  //随机数集合
  //这里故意写错，这时返回的两个数字应该是相同的
  /*List<int> _roll(){
     final roll1 = _random.nextInt(6)+1;
     final roll2 = _random.nextInt(6)+1;
     return [roll1,roll1];
  }*/
  
  List<int> _roll(){
    final roll1 = _random.nextInt(6)+1;
    final roll2 = _random.nextInt(6)+1;
    return [roll1,roll2];
  }


  //点击事件
  void _onPressed() {
    debugPrint("点击了按钮");
    final _rollResult = _roll();
    //显示dialog
    showDialog(
        context: context,
        builder: (_) {
          return AlertDialog(
            //因为调用了两次_roll方法所以结果和预期相同，但是实际执行过程和预期不同
           // content: Text("roll result :(${_roll()[0]},${_roll()[1]})"),
            content: Text("roll result :(${_rollResult[0]},${_rollResult[1]})"),
          );
        });
  }
}

```