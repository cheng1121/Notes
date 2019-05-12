# AIDL的使用
## 创建aidl
1. 
![aidl创建.jpg](https://github.com/Freedom12521/Android/blob/master/android/images/创建aidl.png)
创建aidl文件后会自动在java同级目录下生成一个aidl文件夹，里边存放的就是aidl文件。
注意：直接创建Book.java的同名Book.aidl文件时会报错，需要先随便写一个xx.aidl文件然后再修改为Book.aidl文件

2. 一个实现Parcelable接口的 Book.java类

```
package com.cheng.chapter_2;

import android.os.Parcel;
import android.os.Parcelable;

public class Book implements Parcelable {


    public int bookId;
    public String name;

    public Book(int bookId, String name) {
        this.bookId = bookId;
        this.name = name;
    }


    @Override
    public int describeContents() {
        return 0;
    }

    @Override
    public void writeToParcel(Parcel dest, int flags) {
        dest.writeInt(this.bookId);
        dest.writeString(this.name);
    }

    protected Book(Parcel in) {
        this.bookId = in.readInt();
        this.name = in.readString();
    }

    public static final Creator<Book> CREATOR = new Creator<Book>() {
        @Override
        public BookcreateFromParcel(Parcelsource) {
            return new Book(source);
        }

        @Override
        public Book[] newArray(int size) {
            return new Book[size];
        }
    };
}


```

两个aidl文件
```
// Book.aidl
package com.cheng.chapter_2;


parcelable Book;


```

```
// IBookManager.aidl
package com.cheng.chapter_2;

import com.cheng.chapter_2.Book;

interface IBookManager{
     List<Book>  getBookList();
     
     //in表示输入型参数
     //out表示输出型参数
     //inout表示输入输出型参数
     void addBook(in Book book);
}

```

都创建完成后点击make project 后会自动在generatedJava文件夹下生成 IBookManager.java文件

3. IBookManager.java解析

DESCRIPTOR ：是Binder的唯一标识，一般用当前的类名+包名表示
如：
   [image:A6EF5E2C-5C1C-4712-933E-060A842EA3F6-26914-0001BA24719E3600/63DAF43E-D1AD-405D-9C66-BB666E7B0EAD.png]

asInterface(android.os.IBinder obj)： 用于将服务端的Binder对象转换成客户端所需的AIDL接口类型的对象，这个转换过程是区分进程的，如果客户端和服务端位于同一进程，那么此方法返回的就是服务端的Stub对象本身，否则的话返回的就是系统封装后的 Stub.proxy对象

asBinder：返回当前对象本身

onTransact：这个方法运行在服务端中Binder线程池中，当客户端发起跨进程请求时，远程请求会通过系统底层封装后交由此方法来处理

Proxy.getBookList:运行在客户端，当客户端远程调用此方法
Proxy.addBook：运行在客户端

图片来源 [Android Studio中如何创建AIDL - 技术丶从积累开始 - 博客园](https://www.cnblogs.com/rookiechen/p/5352053.html)
![binder工作机制.jpg](https://github.com/Freedom12521/Android/blob/master/android/images/binder工作机制.jpg)

4. 客户端和服务端进行通信
 
[image:4F1FD141-E2FF-4160-977D-5ECC24CBB8A2-26914-0001C506F0164871/2987EA4F-8C17-449E-B2E6-B63120A78645.png]
在创建一个module，包名和服务端一样，把服务端的aidl文件夹直接复制到客户端的main文件夹下，服务端的Book.java文件复制到客户端，然后点击make project,会在gen目录下生成IBookMainager.java文件

在MainActivity中添加以下代码
```
IBookManager mIBookManager;
TextViewmTv;

@Override
protected void onCreate(Bundle savedInstanceState) {
    super.onCreate(savedInstanceState);
    setContentView(R.layout.activity_main);
    mTv = findViewById(R.id.tv);

    Intent intent = new Intent();
    intent.setAction("com.cheng.chapter_2.BookService");
    intent.setPackage("com.cheng.chapter_2");

    bindService(intent, connection, BIND_AUTO_CREATE);
}

ServiceConnection connection = new ServiceConnection() {
    @Override
    public void onServiceConnected(ComponentName name, IBinder service) {
        //通过asInterface方法获得  IBookManager实例
        mIBookManager = IBookManager.Stub.asInterface(service);
        if(mIBookManager != null) {
            try{
                mIBookManager.addBook(new Book(1, "初识AIDL"));


                mTv.setText(mIBookManager.getBookList().toString());
            } catch(RemoteException e) {
                e.printStackTrace();
            }
        }
    }

    @Override
    public void onServiceDisconnected(ComponentName name) {

    }
};


@Override
protected void onDestroy() {
    super.onDestroy();
    unbindService(connection);
}

```

之后先启动服务端，然后再运行客户端 即可成功调用服务端方法
