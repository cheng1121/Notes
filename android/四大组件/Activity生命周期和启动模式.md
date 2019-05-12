# Activity的生命周期和启动模式
## activity的生命周期
    1. onCreate：表示Activity正在被创建，可以在此方法中进行初始化工作。如：调用setContentView加载布局，初始化activity所需数据等
    2. onStart: 表示Activity正在被启动，即将开始，这时Activity已经可见，但是还没有出现在前台，无法和用户交互。也就是Activity已经显示出来了，但用户还看不到
    3. onRestart：表示Activity正在重新启动，当Activity从不可见状态变为可见状态时会调用此方法，如用户按了HOME键后activity变为不可见，接着又回到这个activity时，就会执行该方法
    4. onResume: 表示activity已经可见了，并且出现在前台开始活动；和onStart方法不同的是，onStart方法已经可见但是还在后台，而onRsume方法是已经可见，而且显示在前台
    5. onPause: 表示activity正在停止，可以在此方法中进行一些不太耗时的操作，比如数据存储，动画停止等
    6. onStop: 表示activity即将停止，可以做一些稍微重量级的回收工作，也是不能太耗时
    7. onDestory: 表示activity即将销毁，可以做一些回收和资源释放等工作

activity生命周期图：（图片来源：[4.1.1 Activity初学乍练 | 菜鸟教程](http://www.runoob.com/w3cnote/android-tutorial-activity.html)）

![877390-20170304103100126-1196400911.jpg](https://github.com/Freedom12521/Android/blob/master/android/images/activity生命周期.jpg)

注意：onResume和onStart 、onPause和onStop这几个方法从不同的角度区分有不同的意义，但是在使用的时候，对于我们来说没有区别

## 特殊情况下Activity的生命周期

1. 资源相关的系统配置发生改变导致Activity重新创建
典型案例：横竖屏切换导致Activity重新创建。
原因：横竖屏切换会导致系统配置发生改变，而Activity也会先销毁再重新创建以适应新的系统配置。
生命周期：Activity  ->意外情况  -> onSaveInstanceState -> onDestory  -> onCreate  -> onRestoreInstanceState
onSaveInstanceState方法的调用时机是在onStop方法之前，但是和 onPause方法没有时序之间的管理，可能在onPause之前，也可能在onPause方法之后。
onSaveInstanceState只有在Activity因为意外情况被销毁时才会被调用（如系统内存不足时，后台Activity被回收；没做特殊处理的横竖屏切换等）。
当Activity被重新创建后会回调onRestoreInstanceState,并把在onSaveInstanceState中保存的Bundle对象作为参数传递给onRestoreInstanceState和onCreate。
onRestoreInstance的调用时机在onStart之后
View保存和恢复数据的流程：
首先Activity调用onSaveInstanceState去保存数据，然后Activity会委Window保存数据，Window会委托它上面的顶级容器ViewGroup去保存数据，一般来说这个顶层容器很可能是DecorView，然后顶层容器再通知它的子元素来保存数据。数据恢复的过程也类似
整个流程是一种典型的委托思想：上层委托下层，父容器委托子元素去处理一件事情

2. 内存不足导致低优先级的Activity被回收
这种情况下保存和恢复数据的流程和上一种是一样的。
 Activity的优先级可以分为三种：
(1). 前台Activity：正在和用户交互的Activity，优先级最高
(2). 可见但非前台Activity：如弹出一个对话框，导致Actiivty可见但是位于后台，无法和用户直接交互
(3). 后台Activity：已经被暂停的Activity，比如执行了onStop，优先级最低

3. 系统配置改变导致Activity重新创建的解决办法
在清单文件中给Activity节点添加 configChanges属性
 android:configChanges=“orientation”
添加该属性后Activity 就不会重新创建了。
此时，当系统配置改变以后会调用 onConfigurationChanged，可以在此方法内做一些特殊处理

## Activity的启动模式
1. standard： 标准模式，Activity默认为该启动模式，没次startActivity都会在任务栈中重新创建一个Activity实例。使用非Activity类型的Context（如：ApplicationContext）启动Activity时，由于没有任务栈所以需要指定一个标记位（FLAG_ACTIVITY_NEW_TASK）,这样在启动的时候会创建一个新的任务栈来存放启动的Activity实例，此时Activity的启动模式为singleTask模式
2. singleTop：栈顶复用模式，如果新Activity已经位于任务栈的栈顶，那么此Activity不会重新创建实例，同时会回调它的onNewIntent方法。
onCreate和onStart方法不会被回调，因为Activity没有改变。
如果Activity没有位于栈顶，则会重新创建该Activity实例。
3. singleTask: 栈内复用模式，当前任务栈只会存在一个当前Activity的实例，多次重复启动该Activity会回调它的onNewIntent方法
注意：以singleTask模式启动的Activity，如果任务栈内已经存在该Activity的实例，并且其上方还有其他Activity的实例，则会把上方的其他Activity弹出站内
4. singleInstance: 单实例模式，这是一种加强的singleTask模式，除了拥有singleTask的特性外，启动模式为singleInstance的Activity所在的任务栈只会有该Activity一个实例，不会存在其他Activity的实例

5. TaskAffinity:任务相关性，这个参数标识了一个Activity所需要的任务栈的名字，默认情况下，所有Activity所需的任务栈的命在为应用包名，可以为每个Acitivty指定不同的任务栈，只要不和包名相同就行。任务栈分为前台任务栈和后台任务栈，后台任务栈中的Activity处于暂停状态，可以通过切换将后台任务栈在次调到前台
TaskAffinity主要和singleTask启动模式或者allowTaskReparenting属性结合使用。
和singleTask启动模式配合使用时，待启动的Activity会运行在名字和TaskAffinity形同的任务栈中。
和allowTaskReparenting使用时，如果A应用启动了B应用的Activity C，这时由于B应用未启动，所以Activity C在A应用的任务栈中，这时启动B应用后，Activity C 会从A应用的任务栈中回到B应用的任务栈中，因为Activity C本来就属于B应用

举个例子来看一下启动模式：
有A、B、C、D 四个Activity，A的启动模式为standard,B的启动模式为singleTop，C的启动模式为singleTask,D的启动模式为singleInstance
在不设置TaskAffinity的情况下
问：按照以下顺序调用这四个Activity后 任务栈内的情况？
A -> B -> C -> D -> A -> B -> C -> D

A -> B的任务栈：
![a跳转到b.jpg](https://github.com/Freedom12521/Android/blob/master/android/images/一a-b.png)

B -> C的任务栈
![image](https://github.com/Freedom12521/Android/blob/master/android/images/一b-c.png)

C -> D的任务栈
![image](https://github.com/Freedom12521/Android/blob/master/android/images/一c-d.png)

D -> A 的任务栈
![image](https://github.com/Freedom12521/Android/blob/master/android/images/一d-a.png)


A -> B 的任务栈
![image](https://github.com/Freedom12521/Android/blob/master/android/images/二a-b.png)


B -> C的任务栈
![image](https://github.com/Freedom12521/Android/blob/master/android/images/二b-c.png)


C -> D的任务栈
![image](https://github.com/Freedom12521/Android/blob/master/android/images/二c-d.png)

从以上中可以看出 最后有两个任务栈，一个是启动模式为singleInstance的D Activity 所在的任务栈。另一个是A、B、C三个Activity所在的任务栈




### 各种情况下的生命周期
1. 启动Activity：onCreate -> onStart -> onResume
2. Activity被其他Activity覆盖，或者锁屏时：调用 onPause
3. Activity从被覆盖状态回到前台，或者解锁屏时：调用 onResume
4. 跳转新界面或者按Home键时：onPause -> onStop
5. 回退到Activity时: onRestart -> onStart -> onResume
6. 当Activity处于后台时，被系统回收，此时再次回退到该Activity：onCreate -> onStart -> onResume
7. 退出当前Activity时：onPause -> onStop -> onResume
8. A跳转到B时，两个Activity的声明周期:A -> onCreate -> onStart -> onResume -> 跳转到B -> onPause -> B 执行 onCreate -> onStart -> onResume -> A 执行 onStop
9. 从B回退到A：B -> onPause -> A 执行 onRestart -> onStart -> onResume -> B执行 onStop -> onDestroy
10. 与Fragment的生命周期,启动时：Fragment onAttach -> onCreate -> onCreateView -> onViewCreate -> Activity onCreate -> Fragment onActivityCreated -> onStart -> Activity onStart -> onResume -> Fragment onResume
11. 与Fragment的生命周期，销毁时:Fragment onPause -> Activity onPause -> Fragment onStop -> Activity onStop -> Fragment onDestroyView -> onDestroy -> onDetach -> Activity onDestroy
12. Activity与menu的创建顺序：Activity执行完onResume 后创建menu


