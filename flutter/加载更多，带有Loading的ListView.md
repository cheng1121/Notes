# 带有Loading的ListView

```
import 'package:flutter/material.dart';

void main() => runApp(MyApp());

class MyApp extends StatelessWidget {
  @override
  Widget build(BuildContext context) {
    return MaterialApp(
      title: 'ListView Demo',
      theme: ThemeData(
        primarySwatch: Colors.blue,
      ),
      home: MyHomePage(title: 'Home page'),
    );
  }
}

class MyHomePage extends StatefulWidget {
  MyHomePage({Key key, this.title}) : super(key: key);
  final String title;

  @override
  _MyHomePageState createState() => _MyHomePageState();
}

class _MyHomePageState extends State<MyHomePage> {
  List<int> items = List.generate(20, (i) => i); //创建数据 每次创建10条
  ScrollController _scrollController = ScrollController(); //滑动控制器
  bool isPerformingRequest = false; //是否有请求正在进行

  //State的生命周期 初始化   State创建时调用一次
  @override
  void initState() {
    // TODO: implement initState
    super.initState();
    _scrollController.addListener(() {
      if (_scrollController.position.pixels ==
          _scrollController.position.maxScrollExtent) {
        _getMoreData();
      }
    });
  }

  //当widget配置改变的时候，调用此方法
  @override
  void didUpdateWidget(MyHomePage oldWidget) {
    // TODO: implement didUpdateWidget
    super.didUpdateWidget(oldWidget);
  }

  //当widget的state状态改变时 调用此方法
  @override
  void didChangeDependencies() {
    // TODO: implement didChangeDependencies
    super.didChangeDependencies();
  }

  //当从view tree中删掉该widget时调用该方法(此时widget可能还会再次添加回tree中)
  @override
  void deactivate() {
    // TODO: implement deactivate
    super.deactivate();
  }

  //永久删除该widget时，调用此方法
  @override
  void dispose() {
    // TODO: implement dispose
    super.dispose();
    _scrollController.dispose();
  }

  //模拟加载数据
  _getMoreData() async {
    if (!isPerformingRequest) {
      //判断是否有请求正在执行
      setState(() => isPerformingRequest = true);
      //获取10条数据
      List<int> newEntries = await fakeRequest(items.length, items.length + 20);
      //设置当前widget的状态
      setState(() {
        items.addAll(newEntries);
        isPerformingRequest = false; //请求执行完毕，可以进行下一次请求了
      });
    }
  }

  Widget _buildProgressIndicator(){
    return  Padding(
        padding: const EdgeInsets.all(8.0),
      child:  Center(
        //Opacity 控制widget的透明度 来控制widget是显示还是隐藏
        child: Opacity(
            opacity: isPerformingRequest ? 1.0: 0.0,
           //圆形进度条
          child: CircularProgressIndicator(),
        ),
      ),
    );
  }

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(
        title: Text(widget.title),
      ),
      body: ListView.builder(
          itemCount: items.length +1,
          itemBuilder: (context, index) {
            if(index == items.length){ //是否为最后一个条目,是则显示圆形进度条
               return _buildProgressIndicator();
            }else {
              return ListTile(title: Text('number $index'),);
            }
          },
        controller: _scrollController,

          ),
    );
  }
}

//模拟网络请求 通过Future获取返回结果
Future<List<int>> fakeRequest(int from, int to) async {
  //延时2秒 执行
  return Future.delayed(Duration(seconds: 2), () {
    //返回to - from 条数据
    return List.generate(to - from, (i) => i + from);
  });
}

```