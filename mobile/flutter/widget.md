# widget 
flutter中UI控件就是widget，主要分为两种:
1. 无状态的widget,叫StatelessWidget，只能又来展示信息，不能有交互动作
2. 有状态的widget,叫StatefulWidget，通过改变状态使UI发生变化，可以包含用户交互


## StatelessWidget的使用
使用方式很简单，直接继承StatelessWidget，然后实现build方法就可以
```dart
class MyWidget extends StatelessWidget{
    @override
    Widget build(BuildContext context){
        return 
    }
}
```

## StatefulWidget的使用
1. 继承StatefulWidget，并实现createState方法
2. 继承State<Widget>，并实现build方法
3. createState方法中调用state类
```dart
//继承StatefulWidget
class MyWidget extends StatefulWidget{
  @override
  State<StatefulWidget> createState() {
    //创建_MyState类的实例
    return _MyState();
  }
}
//继承State类 并把MyWidget作为泛型传入
class _MyState extends State<MyWidget>{
  @override
  Widget build(BuildContext context) {
    // TODO: implement build
    return null;
  }
}
```

## StatelessWidget和StatefulWidget的区别
1. StatelessWidget用于展示内容等不需要交互的控件,build方法只调用一次
2. StatefulWidget需要和用户交互，所以每次State改变时都会执行build方法，用于重建UI


## Text控件
文本控件，用于展示文字,通过textStyle修改字体大小，颜色等
```
class _MyState extends State<MyWidget>{
  @override
  Widget build(BuildContext context) {
    // TODO: implement build
    return Center(
      child: Text(
        "文本控件",
        style: TextStyle(
          fontFamily: 'Roboto',
          fontSize: 18,
          fontWeight: FontWeight.bold,
          color: Colors.amber

        ),
      ),
    );
  }
}
```

## 图片Image
```
//展示资源图片
Image.asset(name);
//本地图片
Image.file(file);
//内存图片
Image.memory(bytes);
```


获取网络图片
```
class TestWidget extends StatelessWidget{
  @override
  Widget build(BuildContext context) {
    // TODO: implement build
    return Center(
      //展示一张网络图片
      child:Image.network('http://img17.3lian.com/d/file/201702/20/9f6f4686d0ccf754600b83377359234e.jpg'),
    );
  }
}
```

## 按钮
flutter提供了两种按钮
1. FlatButton,只显示文字区域，背景颜色为透明
2. RaisedButton，显示一个范围内的区域，带有背景颜色

```
 var flatButton = FlatButton(
 onPressed: () => print('flat按钮'), child: Text('FlatButton'),
    );

var raisedButton = RaisedButton(
onPressed: () => print('raisedButton'),
child: Text('Raisedbutton'),);
```

## 文本输入框

通过TextEditingController()控制器来获取输入的内容

```
class TextFieldWidget extends StatefulWidget{
  @override
  State<StatefulWidget> createState() {
    return _TextFieldState();
  }
}

class _TextFieldState extends State<TextFieldWidget>{
  //编辑框控制器
  var textController = TextEditingController();
  @override
  Widget build(BuildContext context) {
    // Row、Expand 都是用于布局的控件，这里可以先忽略它们
    return Row(
      children: <Widget>[
        Expanded(
          child: TextField(
            controller: textController,
          ),
        ),
        RaisedButton(
          child: Text('click'),
          onPressed: () => print('编辑框输入的内容是${textController.text}'),
        )

      ],
    );
  }

  @override
  void dispose() {
    super.dispose();
    //释放资源
    textController.dispose();
  }
}
```

## 显示弹窗

```
class DialogWidget extends StatefulWidget {
  @override
  State<StatefulWidget> createState() {
    // TODO: implement createState
    return _DialogState();
  }
}

class _DialogState extends State<DialogWidget> {
  @override
  Widget build(BuildContext context) {
    // TODO: implement build
    return RaisedButton(
      child: Text('显示Dialog'),
      onPressed: () {
        showDialog(
            context: context,
            builder: (_) {
              AlertDialog(
                content: Text('一个Dialog'),
                actions: <Widget>[
                  FlatButton(
                    child: Text('点击'),
                    onPressed: () => Navigator.pop(context),
                  )
                ],
              );
            });
      },
    );
  }
}
```

## Container、Padding和Center

### Container
Container控件可以设置一个控件的尺寸、背景、margin等
```
class _TestWidget extends StatelessWidget{
  @override
  Widget build(BuildContext context) {
    // TODO: implement build
    return Container(
      
      child: Text('Container'),
      //内边距
      padding: EdgeInsets.all(8.0),
      //外边距
      margin: EdgeInsets.all(8.0),
      //宽度
      width: 80.0,
      decoration: BoxDecoration(
        //背景色
        color: Colors.grey,
        //圆角
        borderRadius: BorderRadius.circular(5.0),
      ),
    );
  }
}
```

### Padding
只提供Padding属性
```

class _TestPaddingWidget extends StatelessWidget{
  @override
  Widget build(BuildContext context) {
    // TODO: implement build
    return Padding(
      padding: EdgeInsets.all(8.0),
      child: Text('Padding'),
    );
  }
}
```

### Center
内部控件处于中间，使用方式见上方Text控件

## 水平、竖直布局和Expand

### 水平控件 Row

```
class TestRowWidget extends StatelessWidget{
  @override
  Widget build(BuildContext context) {
    // 只有一个子元素的 widget，一般使用 child 参数来设置；Row 可以包含多个子控件，
    // 对应的则是 children。
    return Row(
      children: <Widget>[
        Text('row1'),
        Text('row2'),
        Text('row3'),
        Text('row4'),

      ],
    );
  }
}

```

### 竖直控件 Column
```
class TestColumnWidget extends StatelessWidget{
  @override
  Widget build(BuildContext context) {
    // TODO: implement build
    return Column(
      children: <Widget>[
        Text('Column1'),
        Text('Column2'),
        Text('Column3'),
        Text('Column4'),
        Text('Column5'),
        
      ],
    );
  }
}
```

### Expand控件
占满一行里剩余的所有空间，还可以通过flex设置所占比例

## Stack控件
使一个控件叠在另一个控件的上方。

默认情况下，子控件都按Stack的左上角对齐，可以通过设置alignment参数来改变这个对齐的位置，alignment的取值范围为-1,1



## 例子展示本地图片
1. 在项目根目录创建images文件夹，并把图片添加进去
2. 修改pubspec.yaml文件，把图片添加进去，点击右上角的packages get按钮
```
//添加以下代码
  assets:
    - images/anim_img.jpg
```
3. 展示图片
```
void main() => runApp(MyApp());

class MyApp extends StatelessWidget {
  @override
  Widget build(BuildContext context) {
    return MaterialApp(
      title: '标题',
      home: Scaffold(
        appBar: AppBar(
          title: Text("一个标题"),
        ),
        body: Image.asset('images/anim_img.jpg',
        width: 600.0,
          height: 240.0,
          // cover 类似于 Android 开发中的 centerCrop，其他一些类型，读者可以查看
          // https://docs.flutter.io/flutter/painting/BoxFit-class.html
          fit: BoxFit.cover,
        ),
      ),
    );
  }
}

```