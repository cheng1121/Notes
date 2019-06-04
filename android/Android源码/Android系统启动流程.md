# Android系统启动流程
系统启动即使init进行启动的过程，由多个源文件共同组成，这些文件位于源码目录ststem/core/init中

## 启动电源以及系统启动
当电源按下时移动芯片代码从预定义的地方(固化在ROM)开始执行。加载引导程序BootLoader到RAM中，然后执行

## 引导程序BootLoader
是在Android操作系统开始运行前的一个小程序，它的主要作用是把系统OS拉起来并运行

## Linux内核启动
当内核启动时，设置缓存、被保护存储器、计划列表、加载驱动。在内核完成系统设置后，它首先在系统文件中寻找init.rc文件，并启动init进程

## init进程启动
init进程做的工作比较多，主要用来初始化和启动属性服务，也用来启动Zygote进程。
- 创建和挂载文件系统
- 初始化和启动属性服务
- 解析init.rc文件，并启动Zygote进程

## zygote进程启动
主要工作是：
- 创建AndroidRuntime并调用start函数启动Zygote进程
- 创建Java虚拟机并为Java虚拟机注册JNI函数
- 通过JNI调用ZygoteInit的main方法，进行Java框架层
- 创建服务器Socket和等待AMS请求
- 启动SystemServer进程

## SystemServer进程启动
主要工作：
- 创建Binder线程池
- 创建SystemServiceManager，对服务进行创建、启动和生命周期管理
- 启动系统服务

## Launcher启动过程
被SystemServer进程启动的AMS会启动Launcher，Launcher启动后会将已安装应用的快捷图标显示到界面上


