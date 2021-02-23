
# Navigator2.0的使用

## 1.0版本路由

### 匿名路由
在`Navigator.of(context).push()`时push方法中的参数为`MaterialPageRoute`,代码如下:
```dart
Navigator.of(context).push(
    MaterialPageRoute<void>(
        builder: (BuildContext context) {
            return BookDetail(
                book: book,
            );
        },
    ),
);

```
### 命名路由：路由列表

在`MaterialApp/CupertinoApp` 有个`Map<String, WidgetBuilder>`类型的参数`routes` ，当定义好与其相同类型的Map并传递给routes后即可通过`pushName()`进行路由跳转，Map集合的key即：路由页面的名称；value即：要跳转的页面. 完整路由代码如下：
```dart
void main() {
  runApp(App());
}

///匿名路由和命名路由
class App extends StatefulWidget {
  @override
  _AppState createState() => _AppState();
}

class _AppState extends State<App> {
  ///路由列表
  final Map<String, WidgetBuilder> _routes = <String, WidgetBuilder>{
    'list': (BuildContext context) => Route1MyHomePage(
          title: '列表',
          list: <Book>[
            Book('book1', 'author1'),
            Book('book2', 'author2'),
            Book('book3', 'author3'),
          ],
        ),
    'detail': (BuildContext context) => Route1BookDetail(),
  };

  @override
  Widget build(BuildContext context) {
    return MaterialApp(
      routes: _routes,
      initialRoute: 'list',
    );
  }
}

class Route1MyHomePage extends StatefulWidget {
  const Route1MyHomePage({
    Key? key,
    required this.title,
    required this.list,
  }) : super(key: key);
  final String title;
  final List<Book> list;

  @override
  _Route1MyHomePageState createState() => _Route1MyHomePageState();
}

class _Route1MyHomePageState extends State<Route1MyHomePage> {
  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(
        title: Text(widget.title),
      ),
      body: ListView.builder(
        itemBuilder: (BuildContext context, int index) {
          final Book book = widget.list[index];
          return ListTile(
            ///传递参数
            onTap: () =>
                Navigator.of(context).pushNamed('detail', arguments: book),
            title: Text(book.title),
            trailing: Text(book.author),
          );
        },
        itemCount: widget.list.length,
      ),
    );
  }
}

class Route1BookDetail extends StatelessWidget {
  @override
  Widget build(BuildContext context) {
    final Object? argument = ModalRoute.of(context)?.settings.arguments;
    Book? book;
    if (argument is Book) {
      book = argument;
    } else {
      book = null;
    }
    return Scaffold(
      appBar: AppBar(),
      body: Center(
        child: Column(
          mainAxisSize: MainAxisSize.min,
          children: [
            Text('this is ${book?.title} detail; author is ${book?.author}'),

            ///匿名路由
            TextButton(
                onPressed: () {
                  if (book == null) {
                    return;
                  }
                  Navigator.of(context).push(
                    MaterialPageRoute<void>(
                      builder: (BuildContext context) {
                        return BookDetail(
                          book: book!,
                        );
                      },
                    ),
                  );
                },
                child: const Text('跳转下一页')),
          ],
        ),
      ),
    );
  }
}

class Book {
  Book(this.title, this.author);

  final String title;
  final String author;
}


```

### 命名路由：`onGenerateRoute`的用法

**当同时使用`routes`和`onGenerateRoute`时，路由遍历顺序如下:**
- 先遍历`routes`列表,如果查找到则返回，不再去`onGenerateRoute`中查找
- 如果`routes`中未找到则去`onGenerateRoute`查找，找到则返回,不再执行`onUnknownRoute`,
- 如果`routes`和`onGenerateRoute`都未查找到则返回`onUnknownRoute`中定义的路由页面

```dart
///build中的写法如下：其余代码同上

  @override
  Widget build(BuildContext context) {
    return MaterialApp(
      // routes: _routes,
      initialRoute: 'list',
      onGenerateRoute: (RouteSettings settings) {
        if (settings.name == 'list') {
          ///没有返回值，泛型传void或者不传，不传则为dynamic类型
          return MaterialPageRoute<void>(

              ///需要RouteSettings，否则popUtil时将无法找到此路由
              /// final Object? argument = ModalRoute.of(context)?.settings.arguments;
              /// ModalRoute方法也无法获取参数
              settings: settings,
              builder: (BuildContext context) {
                return Route1MyHomePage(
                  title: '列表',
                  list: <Book>[
                    Book('book1', 'author1'),
                    Book('book2', 'author2'),
                    Book('book3', 'author3'),
                  ],
                );
              });
        } else if (settings.name == 'detail') {
          ///此时可以从RouteSettings中获取到push中传的参数
          /// final Object? arguments =   settings.arguments;

          return MaterialPageRoute<void>(
              settings: settings,
              builder: (BuildContext context) {
                return Route1BookDetail();
              });
        }
      },
    );
  }
```



## 2.0版本

1.0版本的路由虽然能满足基本的需求，但是在一些特殊场景下，无法满足我们的需求，主要是不能自由操作路由栈，对系统返回键的监听不友好等等问题。因此google开发出了navigator2.0版本，2.0版本和1.0版本兼容，使用哪个要看项目需求


### 预定义路由栈方式
1. 在`MaterialApp` 中的`home`参数中使用`Navigator`，并且给`onPopPage`和`pages`参数传参。代码如下：
```dart

class _MyAppState extends State<MyApp> {
  Book? _selectedBook;
  bool show404 = false;
  List<Book> books = <Book>[
    Book('book1', 'author1'),
    Book('book2', 'author2'),
    Book('book3', 'author3'),
  ];

  @override
  Widget build(BuildContext context) {
    return MaterialApp(
      title: 'Flutter Demo',
      theme: ThemeData(
        primarySwatch: Colors.blue,
      ),

      ///navigator 2.0
      home: Navigator(
        onPopPage: (Route<dynamic> route, dynamic result) {
          if (!route.didPop(result)) {
            return false;
          }

          ///详情页点击返回时，清空选中
          setState(() {
            _selectedBook = null;
          });

          return true;
        },

        ///路由页面 (使用预定义路由栈)
        pages: <Page>[
          MaterialPage<void>(
            key: const ValueKey('BookListPage'),
            child: MyHomePage(
              title: '标题',
              list: books,
              onSelected: (Book book) {
                setState(() {
                  _selectedBook = book;
                  print('选中的 book ====${book.title}');
                });
              },
            ),
          ),
          if (show404)
            MaterialPage<void>(child: UnKnowScreen())
          else if (_selectedBook != null)

            ///自定义路由
            BookDetailPage<void>(_selectedBook!),
          // MaterialPage<void>(
          //     key: ValueKey(_selectedBook),
          //     child: BookDetail(
          //       book: _selectedBook!,
          //     )),
        ],
      ),
    );
  }
}

```
2. 添加页面`MyHomePage`、`BookDetail`和`UnKnowScreen`
```dart

class MyHomePage extends StatefulWidget {
  const MyHomePage(
      {Key? key,
      required this.title,
      required this.list,
      required this.onSelected})
      : super(key: key);
  final String title;
  final List<Book> list;
  final ValueSetter<Book> onSelected;

  @override
  _MyHomePageState createState() => _MyHomePageState();
}

class _MyHomePageState extends State<MyHomePage> {
  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(
        title: Text(widget.title),
      ),
      body: ListView.builder(
        itemBuilder: (BuildContext context, int index) {
          final Book book = widget.list[index];
          return ListTile(
            onTap: () => widget.onSelected(book),
            title: Text(book.title),
            trailing: Text(book.author),
          );
        },
        itemCount: widget.list.length,
      ),
    );
  }
}


class BookDetail extends StatelessWidget {
  const BookDetail({Key? key, required this.book}) : super(key: key);
  final Book book;

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(),
      body: Center(
        child: Column(
          mainAxisSize: MainAxisSize.min,
          children: [
            Text('this is ${book.title} detail; author is ${book.author}'),
            TextButton(
                onPressed: () {
                  Navigator.of(context).push(
                    MaterialPageRoute<void>(
                      builder: (BuildContext context) {
                        return BookDetail(
                          book: book,
                        );
                      },
                    ),
                  );
                },
                child: const Text('匿名路由跳转下一页')),
          ],
        ),
      ),
    );
  }
}

class UnKnowScreen extends StatelessWidget {
  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(),
      body: const Center(
        child: Text('page is not found:status is 404'),
      ),
    );
  }
}

```
3. 自定义路由动画
```dart
///自定义路由，继承page
class BookDetailPage<T> extends Page<T> {
  const BookDetailPage(this.book) : super(key: const ValueKey('bookDetail2'));

  final Book book;

  @override
  Route<T> createRoute(BuildContext context) {
    return PageRouteBuilder<T>(
        settings: this,
        pageBuilder: (BuildContext context, Animation<double> animation,
            Animation<double> secondAnimation) {
          final Tween<Offset> tween =
              Tween<Offset>(begin: const Offset(0, 1.0), end: Offset.zero);
          final CurveTween curveTween = CurveTween(curve: Curves.easeInOut);
          return SlideTransition(
            position: animation.drive(curveTween).drive(tween),
            child: BookDetail(
              book: book,
            ),
          );
        });
  }
}

```


## 路由路径解析、自定义路由代理，自定义路由过度
上方的预定义路由无法处理平台发送过来的路由路径，不能刷新web上显示的的URL的路径，因此需要使用`RouteInformationParser`和`RouterDelegate`来进行路由路径解析和刷新路由页面  

当自定义路由代理、和路径时，使用`MaterialApp`的`router`构造函数,传入自定义的`routeInformationParser`和`routerDelegate`参数。如果需要在路由过度期间进行特殊操作(如：关闭入场动画、给指定路由栈内的路由传参等)，则需要在Navigator的构造中传入`transitionDelegate`，通过继承`TransitionDelegate`抽象类来自定义路由过度代理

### `RouteInformationParser`

使用方式是，继承`RouteInformationParser<T>`抽象类,其中泛型`T`是我们定义的路由信息类;  
然后实现其中的两个函数`Future<T> parseRouteInformation(RouteInformation routeInformation);`和` RouteInformation? restoreRouteInformation(T configuration) => null;`

#### `parseRouteInformation`函数

 **以下内容为方法注释的翻译**

- 转化路由信息，并传递给`TransitionDelegate`
- 返回值为`Future`类型，就是接受异步计算获取路由（如：网络）
- 考虑使用`SynchronousFuture`返回结果，这样就不需要等待下一个`microtask`把数据传递到`RouterDelegate`

实现代码(和*Learning Flutter’s new navigation and routing system*中的相同)如下：

```dart

 @override
  Future<RoutePath> parseRouteInformation(
      RouteInformation routeInformation) async {
    final Uri uri = Uri.parse(routeInformation.location!);
    if (uri.pathSegments.isEmpty) {
      return RoutePath.home();
    }

    if (uri.pathSegments.length == 2) {
      if (uri.pathSegments[0] != 'book') {
        return RoutePath.unKnown();
      }
      final String remaining = uri.pathSegments[1];
      final int id = int.tryParse(remaining) ?? -1;
      if (id == -1) {
        return RoutePath.unKnown();
      }

      return RoutePath.detail(id);
    }

    return RoutePath.unKnown();
  }


```

#### `restoreRouteInformation`函数

从配置（即路由信息）中恢复路由

- 可以能返回`null`此时浏览器的历史记录将不会更新
- 此方法返回值不为`null`时，`parseRouteInformation`必须返回有相同的`route`

代码如下：

```dart
  @override
  RouteInformation? restoreRouteInformation(RoutePath configuration) {
    if (configuration.isUnKnown) {
      return const RouteInformation(location: '/404');
    }

    if (configuration.isHomePage) {
      return const RouteInformation(location: '/');
    }

    if (configuration.isDetailPage) {
      return RouteInformation(location: '/book/${configuration.id}');
    }

    return null;
  }
```



### `RouterDelegate<T>`路由代理





### TransitionDelegate路由过度时期的代理



参考文章1：
[Learning Flutter’s new navigation and routing system(需翻墙)](https://medium.com/flutter/learning-flutters-new-navigation-and-routing-system-7c9068155ade)
参考文章2：
[Flutter Navigator 2.0 指南与原理解析](https://flutter.cn/community/tutorials/understanding-navigator-v2#navigator-20)

