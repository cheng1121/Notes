# 内容提供者(ContentProvider)
主要作用就是将程序内部的数据提供给外部进行共享，为数据提供外部访问接口，被访问的数据主要以数据库的形式存在，而且还可以选择共享哪一部分数据。是跨进程通信的方式之一

### 使用系统的ContentProvider
1. 获取ContentResolver实例
2. 确定Uri的内容，并解析为具体的Uri实例
3. 通过ContentResolver实例来调用相应的方法，传递相应的参数，但是第一个参数总是Uri，它制定了我们要操作的的数据的具体地址

### 自定义ContentProvider
1. 继承ContentProvider类，并在清单文件中添加contentProvider节点
2. 重写onCreate、query、getType、insert、delete、update方法

注意：重写的这几个方法执行在UI线程，其中onCreate方法是在Application.onCreate之前执行，也就是在进程启动时执行

Uri的形式一般有以下两种：

1. 以路径名为结尾，这种Uri请求的是整个表的数据，如: content://com.demo.androiddemo.provider/tabl1 标识我们要访问tabl1表中所有的数据
2. 以id列值结尾，这种Uri请求的是该表中和其提供的列值相等的单条数据。 content://com.demo.androiddemo.provider/tabl1/1 标识我们要访问tabl1表中_id列值为1的数据。

这里只简单介绍ContentProvier，以后会有从源码开始分析的ContentProver