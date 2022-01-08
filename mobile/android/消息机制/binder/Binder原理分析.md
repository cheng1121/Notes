# Binder原理分析

本文内容和图片摘自[Android Bander设计与实现 - 设计篇](https://blog.csdn.net/universus/article/details/6211589)

## Linux中的IPC方式
1. 管道
2. System V IPC，即消息队列/共享内存/信号量
3. socket

Android系统使用client-server方式进行进程间通信的原因：
1. client-server方式，使应用程序只需要和server建立连接就可以完成各种功能，花费的时间和精力较少
2. linux系统中使用这种方式进行通信的只有socket，虽然也可以以其他方式为基础自建立一套协议实现client-server，但这样增加了系统的复杂性，在资源稀缺的手机中，可靠性也难以保证
3. 传输性能
   1. socket传输效率低，开销大，主要用于跨网络的进程间通信和本机上进程间通信的低速通信
   2. 消息队列和管道采用存储-转发方式，即数据先从发送方缓存区拷贝到内核开辟的缓存区，然后再从内核缓存区拷贝到接收方的缓存区，至少有两次拷贝
   3. 共享内存无需拷贝，但是控制复杂，难以使用

各种IPC方式数据拷贝次数
IPC | 拷贝次数
---|---
共享内存 | 0
Binder | 1
Socket/管道/消息队列 | 2

4. 安全考虑，传统IPC没有安全措施，依靠上层协议确保安全，有以下几方面不足：
   1. 接收方无法鉴别发送发的身份，也就是无法拿到UID/PID
   2. 只能由用户在数据包内添加UID/PID，不可靠，容易被恶意程序利用
   3. 接入点是开放的，无法建立私有通道

基于以上原因，Android建立了一套新的IPC机制来满足系统对通信方式，传输性能，安全性的要求，这就是Binder。

Binder基于Client-Server通信模式，传输过程只需一次拷贝，为发送者添加UID/PID身份，既支持实名Binder也支持匿名Binder，安全性高。


## Binder通信模型
Binder框架定义了四个角色：Server、Client、ServiceManager和Binder驱动。Service、ClientServiceManager运行在用户空间，驱动运行在内核空间

### Binder驱动
和设备硬件没有关系，只是实现方式和设备驱动一样的，提供open()、mmap()、poll()、ioctl()等标准文件操作。作用：
1. 进程间Binder通信的建立
2. Binder进程间的传递
3. Binder引用计数管理
4. 数据包在进程间的传递和交互等底层支持

驱动和应用程序之间定义了一套接口协议，主要功能由ioctl()接口实现，不提供read()、write()，因为ioctl()灵活方便，且能够一次调用实现先写后读以满足同步交互，而不必分别调用read()、write()

### ServiceManager和实名Binder
ServiceManager的作用是将字符形式的Binder转化成Client中对该Binder的引用，使得Client能通过Binder名字获得对Server中Binder实体的引用。注册了名字的Binder叫实名Binder。

Server创建了Binder实体，为其取一个名字后，将这个Binder连同名字以数据包的形式通过Binder驱动发送给ServiceManager，通知ServiceManager注册一个有名字的Binder，它位于Server中。

驱动为这个穿过进程边界的Binder创建位于内核中的实体节点以及ServiceManager对实体的引用，将名字及新建的引用打包传递给ServiceManager，ServiceManager接收数据包后，从中取出名字和引用填入一张查找表中

ServiceManager是一个进程,Server也是一个进程，它俩之间的通信也是进程间通信，使用的也是Binder进行通信。ServiceManager是Server端有自己的Binder对象，其他进程都是Client，需要通过这个Binder的引用来实现Binder的注册，查询和读取。ServiceManager的Binder没有名字也不需要注册，当一个进程使用BINDER_SET_CONTEXT_MGR命令将自己注册成ServiceManager时，Binder驱动会自动为它创建一个Binder实体。其次这个Binder的引用在所有Client中都固定为0，而无需通过其他手段获得

### Client获得实名Binder的引用
Server向ServiceManager注册了Binder实体及其名字后，Client就可以通过名字获得该Binder的应用了。Client也利用保留的0号引用向ServiceManager请求访问某个Binder。

获得引用的过程：
1. client申请名字叫book的Binder引用
2. ServiceManager收到这个连接请求，从请求数据包里得到binder的名字
3. ServiceManager根据Binder名字从查找表中找到对应的条目
4. ServiceManager从条目中取出Binder的引用
5. 把引用作为回复发送给发起请求的client

从以上过程可以看出，Binder对象的引用有两个，一个在ServiceManager中，一个位于发起请求的client中。如果有更多的client请求该Binder，系统中就会有更多引用指向binder，这些引用是强引用，从而确保了只要有引用Binder实体就不会被释放掉

### 匿名Binder
不是所有的Binder都需要注册给ServiceManager，Server端可以通过已经建立的Binder连接将创建的Binder实体传给Client，这个已经建立的Binder必须是通过实名Binder实现，由于这个创建的Binder没有向ServiceManager注册，所以是匿名Binder。

Client收到这个匿名Binder的引用后，通过这个引用向位于Server中的实体发送请求。匿名
Binder为通信双方建立一条私密通道，只要Server没有把匿名binder发给别的进程，比的进程就无法通过穷举或猜测等任何方式获得该Binder的引用，向binder发送请求

Binder通信示例：
![参与Binder通信的所有角色](http://hi.csdn.net/attachment/201102/27/0_1298798577tfS4.gif)

## Binder的表述
Binder存在于系统以下几个部分：
- 应用程序进程：分别位于Server进程和Client进程中
- Binder驱动：分别管理为Server端的Binder实体和Client端引用
- 传输数据：由于Binder可以跨进程传递，需要在传输数据中予以表述
在系统不同部分，Binder实现的功能不同，表现形式也不一样

### Binder在应用程序中
Binder本质上只是一种底层通信方式，和具体服务没有关系。为了提供具体服务，Server必须提供一套接口函数以便Client通过远程访问使用各种服务。这时通常采用Proxy设计模式：将接口函数定义在一个抽象类中，Server和Client都会以该抽象类为基类实现所有接口函数，所不同的是Server端是真正的功能实现，而Client端是对这些函数远程调用请求的包装

### Binder在Server中
做为Proxy设计模式的基础，首先定义一个抽象接口类封装Server所有功能，其中包含一系列纯虚函数留待Server和Proxy各自实现。由于这些函数需要跨进程调用，须为其一一编号，从而Server可以根据收到的编号决定调用哪个函数。其次就要引入Binder了。Server端定义另一个Binder抽象类处理来自Client的Binder请求数据包，其中最重要的成员是虚函数onTransact()。该函数分析收到的数据包，调用相应的接口函数处理请求。

接下来采用继承方式以接口类和Binder抽象类为基类构建Binder在Server中的实体，实现基类里所有的虚函数，包括公共接口函数以及数据包处理函数：onTransact()。这个函数的输入是来自Client的binder_transaction_data结构的数据包。前面提到，该结构里有个成员code，包含这次请求的接口函数编号。onTransact()将case-by-case地解析code值，从数据包里取出函数参数，调用接口类中相应的，已经实现的公共接口函数。函数执行完毕，如果需要返回数据就再构建一个binder_transaction_data包将返回数据包填入其中。

那么各个Binder实体的onTransact()又是什么时候调用呢？这就需要驱动参与了。前面说过，Binder实体须要以Binde传输结构flat_binder_object形式发送给其它进程才能建立Binder通信，而Binder实体指针就存放在该结构的handle域中。驱动根据Binder位置数组从传输数据中获取该Binder的传输结构，为它创建位于内核中的Binder节点，将Binder实体指针记录在该节点中。如果接下来有其它进程向该Binder发送数据，驱动会根据节点中记录的信息将Binder实体指针填入binder_transaction_data的target.ptr中返回给接收线程。接收线程从数据包中取出该指针，reinterpret_cast成Binder抽象类并调用onTransact()函数。由于这是个虚函数，不同的Binder实体中有各自的实现，从而可以调用到不同Binder实体提供的onTransact()。


以上内容都是摘自大佬的原理分析文章（因为发现后边的很多内容都看不懂，所以这里只是把自己理解的部分摘出来），看到这里虽然对Binder的整体运行机制有了个大致的了解，但是还有很多内容看还不是很不了解，就比如Binder在各个层级之中的表述，感觉还是晦涩难懂。因此为了更好的理解Binder的运行机制，下面以一个实例来分析

```
package com.cheng.chapter_2;
//使用Binder传输的接口都需要继承IInterface接口才可以传输 这个类是系统自动为我们生成的
  public interface IBookManager extends android.os.IInterface {
        /**
         * Local-side IPC implementation stub class.
         */
        public static abstract class Stub extends android.os.Binder implements com.cheng.chapter_2.IBookManager {
            private static final java.lang.String DESCRIPTOR = "com.cheng.chapter_2.IBookManager";

            /**
             * Construct the stub at attach it to the interface.
             */
            public Stub() {
                this.attachInterface(this, DESCRIPTOR);
            }

            /**
             * Cast an IBinder object into an com.cheng.chapter_2.IBookManager interface,
             * generating a proxy if needed.
             * 把服务端的Binder对象转换为客户端需要的aidl接口对象
             */
            public static com.cheng.chapter_2.IBookManager asInterface(android.os.IBinder obj) {
                if ((obj == null)) {
                    return null;
                }
                //查询是否在本地，如果返回为null，则说明是进程间调用，需要通过代理
                android.os.IInterface iin = obj.queryLocalInterface(DESCRIPTOR);
                if (((iin != null) && (iin instanceof com.cheng.chapter_2.IBookManager))) {
                    return ((com.cheng.chapter_2.IBookManager) iin);
                }
                //通过代理返回对应的IBookManager
                return new com.cheng.chapter_2.IBookManager.Stub.Proxy(obj);
            }

            //返回Stub 也就是IBinder
            @Override
            public android.os.IBinder asBinder() {
                return this;
            }

            //此方法在进程间通信时会调用，同一进程时不会调用
            @Override
            public boolean onTransact(int code, android.os.Parcel data, android.os.Parcel reply, int flags) throws android.os.RemoteException {
                java.lang.String descriptor = DESCRIPTOR;
                //根据code调用相应的方法
                switch (code) {
                    case INTERFACE_TRANSACTION: {
                        reply.writeString(descriptor);
                        return true;
                    }
                    case TRANSACTION_getBookList: {
                        data.enforceInterface(descriptor);
                        java.util.List<com.cheng.chapter_2.Book> _result = this.getBookList();
                        reply.writeNoException();
                        reply.writeTypedList(_result);
                        return true;
                    }
                    case TRANSACTION_addBook: {
                        data.enforceInterface(descriptor);
                        com.cheng.chapter_2.Book _arg0;
                        if ((0 != data.readInt())) {
                            _arg0 = com.cheng.chapter_2.Book.CREATOR.createFromParcel(data);
                        } else {
                            _arg0 = null;
                        }
                        this.addBook(_arg0);
                        reply.writeNoException();
                        return true;
                    }
                    default: {
                        return super.onTransact(code, data, reply, flags);
                    }
                }
            }
            //进程间通信时asInterface方法中调用的代理类
            private static class Proxy implements com.cheng.chapter_2.IBookManager {
                
                private android.os.IBinder mRemote;

                Proxy(android.os.IBinder remote) {
                    mRemote = remote;
                }

                @Override
                public android.os.IBinder asBinder() {
                    return mRemote;
                }

                public java.lang.String getInterfaceDescriptor() {
                    return DESCRIPTOR;
                }
                //运行在客户端，客户端线程调用此方法后，会进入挂起状态，直到服务端返回数据
                @Override
                public java.util.List<com.cheng.chapter_2.Book> getBookList() throws android.os.RemoteException {
                    android.os.Parcel _data = android.os.Parcel.obtain(); //发送给服务端请求
                    android.os.Parcel _reply = android.os.Parcel.obtain(); //服务端的响应
                    java.util.List<com.cheng.chapter_2.Book> _result; //从响应中得到的返回结果
                    try {
                        _data.writeInterfaceToken(DESCRIPTOR);
                        //服务端根据code调用相应的方法，并把响应存入  _reply中
                        mRemote.transact(Stub.TRANSACTION_getBookList, _data, _reply, 0);
                        _reply.readException();
                        //读取响应的返回的结果
                        _result = _reply.createTypedArrayList(com.cheng.chapter_2.Book.CREATOR);
                    } finally {
                        _reply.recycle();
                        _data.recycle();
                    }
                    return _result;
                }
                //和getBookList方法一样
                @Override
                public void addBook(com.cheng.chapter_2.Book book) throws android.os.RemoteException {
                    android.os.Parcel _data = android.os.Parcel.obtain();
                    android.os.Parcel _reply = android.os.Parcel.obtain();
                    try {
                        _data.writeInterfaceToken(DESCRIPTOR);
                        if ((book != null)) {
                            _data.writeInt(1);
                            book.writeToParcel(_data, 0);
                        } else {
                            _data.writeInt(0);
                        }
                        mRemote.transact(Stub.TRANSACTION_addBook, _data, _reply, 0);
                        _reply.readException();
                    } finally {
                        _reply.recycle();
                        _data.recycle();
                    }
                }
            }
            //两个方法对应的 code
            static final int TRANSACTION_getBookList = (android.os.IBinder.FIRST_CALL_TRANSACTION + 0);
            static final int TRANSACTION_addBook = (android.os.IBinder.FIRST_CALL_TRANSACTION + 1);
        }
        //声明服务端的两个方法
        public java.util.List<com.cheng.chapter_2.Book> getBookList() throws android.os.RemoteException;

        public void addBook(com.cheng.chapter_2.Book book) throws android.os.RemoteException;
    }
```

