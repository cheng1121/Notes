# flutter插件相关

## 一、创建插件

使用如下命令创建插件

```shell
flutter create -t plugin <pluginName>
```

- 创建完成后是没有android和ios目录的，需要手动添加支持的平台;
- 默认的平台语言是android: kotlin   ios: swift



## 二、添加android文件夹

进入插件目录下，执行下方命令：

```shell
flutter create -org <包名> -t plugin android .
```

执行完成后进入进入`pubspec.yaml`文件中:

1. 修改`some_platform`为：android
2. `android:`下方添加package:包名和pluginClass:android/src.main文件下的文件名（注意缩进：要有一个空格，表示是`android`的下一级）

## 三、添加ios文件夹

和android文件夹的添加方式一样执行命令：

```shell
flutter create -org <包名> -t plugin ios .
```



与`android`同级添加ios平台：需要添加pluginClass,为ios/Classes中的xxxPlugin，注意不要使用swift文件的名字



## 四、最终pubspec.yaml文件中的样式

```yaml
flutter:
  plugin:
    platforms:
      android:
        package: com.cheng.flutter_jim
        pluginClass: FlutterJimPlugin
      ios:
        pluginClass: FlutterJimPlugin

```





## 五、ios导入库

在ios目录下的`xxx.podspec`文件中添加

```podspec
s.dependency 'JMessage'
```

如果要导入的是静态库，则需要添加

```podspec
 s.static_framework = true
```

