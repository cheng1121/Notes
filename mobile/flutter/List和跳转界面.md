# List
展示一个列表，点击列表条目展示为选中状态

##### 添加第三方工具
在pubspec.yaml文件中,下方代码位置添加english_words,3.1.0版本。
然后点击右上角的Packages get 把包加入到项目中

```
dependencies:
  flutter:
    sdk: flutter

  # The following adds the Cupertino Icons font to your application.
  # Use with the CupertinoIcons class for iOS style icons.
  cupertino_icons: ^0.1.2
  #添加english_words
  english_words: ^3.1.0
```
##### 一个列表的完整代码

```
import 'package:flutter/material.dart';
import 'package:english_words/english_words.dart';

void main() => runApp(MyApp());

class MyApp extends StatelessWidget {
  @override
  Widget build(BuildContext context) {
    return MaterialApp(
      title: "一个列表",
      //修改主题颜色为白色
      theme: ThemeData(
        primaryColor: Colors.white
      ),
      home: RandomWords(),
    );
  }
}

//定义一个随机单词控件，  StatefulWidget本身不可变，只是用来创建 State
class RandomWords extends StatefulWidget {
  @override
  State<StatefulWidget> createState() {
    return RandomWordsState();
  }
}

//state类是可变的，并且有自己的生命周期
class RandomWordsState extends State<RandomWords> {
  //创建一个保存单词对的list _开头的变量为私有的
  final _suggestions = <WordPair>[];

  //字体大小
  final _biggerFont = const TextStyle(fontSize: 18.0);

  //set集合，用户保存用户选中的条目
  final _save = Set<WordPair>();

  //构建显示单词的列表
  Widget _buildSuggestions() {
    return ListView.builder(
        padding: const EdgeInsets.all(16.0),
        // itemBuilder 在每次生成一个单词对时被回调
        // 每一行都是用 ListTile 代表
        // 对于偶数行，这个回调函数都添加一个 ListTile 组件来保存单词对
        // 对于奇数行，这个回调函数会添加一个 Divider 组件来在视觉上分割单词对。
        // 注意，在小设备上看到分割物可能会非常困难
        itemBuilder: (context, i) {
          //在 ListView 组件的每行之前，先添加一个像素高度的分割。
          if (i.isOdd) {
            return Divider();
          }
          // 这个"i ~/ 2"的表达式将i 除以 2，然后会返回一个整数结果。
          // 例： 1, 2, 3, 4, 5 会变成 0, 1, 1, 2, 2。
          // 这个表达式会计算 ListView 中单词对的真实数量
          final index = i ~/ 2;
          //如果到达了单词对列表的结尾处...
          if (index >= _suggestions.length) {
            //然后生成10个单词对到建议的名称列表中。
            _suggestions.addAll(generateWordPairs().take(10));
          }

          return _buildRow(_suggestions[index]);
        });
  }

// 在每次生成单词对后，_buildSuggestions 会调用 _buildRow。这个函数使用 ListTile 组件显示每一个新的单词对，这使得每一行更加好看
  Widget _buildRow(WordPair pair) {
    //判断是否已经保存了该条目
    final alreadySaved = _save.contains(pair);

    return ListTile(
      title: Text(
        pair.asPascalCase,
        style: _biggerFont,
      ),
      //添加条目右侧的心形图片
      trailing: Icon(
        //设置图片
        alreadySaved ? Icons.favorite : Icons.favorite_border,
        //设置图片颜色
        color: alreadySaved ? Colors.red : null,
      ),
      //条目点击事件
      onTap: () {
        //在Flutter的响应式框架中，调用setState() 会触发对 State 对象的 build() 方法的调用，从而导致UI的更新。
        setState(() {
          if (alreadySaved) {
            _save.remove(pair);
          } else {
            _save.add(pair);
          }
        });
      },
    );
  }

  //跳转  当用户点击APP顶栏上的列表图标时，创建一个 route ，并将它添加到导航栈中，这个动作会使APP显示一个新界面。
  void _pushSaved() {
    //添加一条新路径
    Navigator.of(context).push(
      new MaterialPageRoute(
        builder: (context) {
          final tiles = _save.map(
            (pair) {
              return ListTile(
                title: Text(
                  pair.asPascalCase,
                  style: _biggerFont,
                ),
              );
            },
          );
          //每行添加水平间距
          final divided = ListTile.divideTiles(
            tiles: tiles,
            context: context,
          ).toList();
          return Scaffold(
            appBar: AppBar(
              title: Text("第二个界面"),
            ),
            body: ListView(children: divided,),
          );
        },
      ),
    );
  }

  @override
  Widget build(BuildContext context) {
    //随机生成字符串
//    final randomWords = WordPair.random();
//
//    return Text(randomWords.asPascalCase);

    return Scaffold(
      appBar: AppBar(
        title: Text("这是一个标题"),
        //添加动作   图片按钮，跳转
        actions: <Widget>[
          IconButton(
            icon: Icon(Icons.list),
            onPressed: _pushSaved,
          ),
        ],
      ),
      body: _buildSuggestions(),
    );
  }
}

```

步骤参考：

[创建你的第一个Flutter APP](https://zhuanlan.zhihu.com/p/34617369)