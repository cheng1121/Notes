# IPC的方式
## Bundle
 Activity、Service、Receiver都支持Intent传递Bundle数据，并且Bundle实现了Parcelable了接口，所以我们可以通过Bundle实现进程间通信，但是只能传递Bundle支持的数据类型

## 文件共享
两个进程通过读写同一个文件来进行数据交换，虽然可能会出现问题，比如并发读写问题。
Sp是个例外，系统对sp有内存缓存侧露，所以多个进程同时读写sp时，可能会丢失数据或者不准。所以，不建议在多进程通信中使用sp

## Messenger
通过它可以在不同进程间传递message对象。Messenger是一种轻量级的IPC方案，它的底层实现是AIDL
messenger的使用
1. 服务端代码
```
/**
 * Messenger
 * 服务端
 * 
 */

public class MessengerService extends Service {


    private static final String TAG = "MessengerService";


    private static class  MessengerHandler extends Handler{
        @Override
        public void handleMessage(Message msg) {
            switch (msg.what){
                case 1:
                    Log.i(TAG, "receive msg from client : "+ msg.getData().getString("msg"));
                    //服务端给客户端回复消息
                    Messenger client = msg.replyTo;
                    Message message = Message.obtain(null,2);
                    Bundle bundle = new Bundle();
                    bundle.putString("reply","已经收到了");
                    message.setData(bundle);
                    try {
                        client.send(message);
                    } catch (RemoteException e) {
                        e.printStackTrace();
                    }

                    break;
            }
        }
    }

    private final Messenger mMessenger = new Messenger(new MessengerHandler());

    @Nullable
    @Override
    public IBinder onBind(Intent intent) {
        return mMessenger.getBinder();
    }
}

```

2. 客户端代码
```
/**
 * Messenger客户端
 * 为了收到服务端的回复
 * 客户端也需要Handler和Messenger
 */

public class MessengerActivity extends AppCompatActivity {

    private static final String TAG = "MessengerActivity";
    private Messenger mService;
    private Messenger mReplyMessenger = new Messenger(new ClientHandler());


    private static class ClientHandler extends Handler {
        @Override
        public void handleMessage(Message msg) {
            switch (msg.what) {
                case 2:
                    Log.i(TAG, "handleMessage: "+ msg.getData().getString("reply"));
                    break;
                default:
                    super.handleMessage(msg);
            }

        }
    }

    //连接服务端 并发送消息
    private ServiceConnection serviceConnection = new ServiceConnection() {
        @Override
        public void onServiceConnected(ComponentName name, IBinder service) {
            mService = new Messenger(service);
            Message message = Message.obtain(null, 1);
            Bundle bundle = new Bundle();
            bundle.putString("msg", "hello, this is client");
            message.setData(bundle);
            //要接收服务端的回复需要把 mReplyMessenger 传递给服务端
            message.replyTo = mReplyMessenger;

            try {
                mService.send(message);
            } catch (RemoteException e) {
                e.printStackTrace();
            }
        }

        @Override
        public void onServiceDisconnected(ComponentName name) {

        }
    };

    @Override
    protected void onCreate(@Nullable Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_main);
        Intent intent = new Intent(this, MessengerService.class);

        bindService(intent, serviceConnection, BIND_AUTO_CREATE);
    }

    @Override
    protected void onDestroy() {
        unbindService(serviceConnection);
        super.onDestroy();

    }
}

```

此时运行后 服务端就会接收到客户端的消息

Messenger是以串行的方式处理客户端发来的消息，如果大量的消息同时发送到服务端，服务端仍然只能一个一个处理，这时使用Messenger就不太合适了

## AIDL
AIDL的使用见：文档AIDL的使用
大致流程为：
1. 首先创建一个Service和一个AIDL接口
2. 创建一个类继承自AIDL接口中的Stub类并实现Stub中的抽象方法，在Service的onBind方法中返回这个类的对象
3. 客户端绑定服务端Service，建立连接后就可以访问远程服务端的方法了
### Binder连接池
1. 每个业务模块创建自己的AIDL接口并实现此接口，这个时候不同业务模块之间是不能有耦合的，所有实现细节我们要单独开来，然后向服务端提供自己的唯一标识和其对应的Binder对象
2. 服务端只需要一个Service，并且提供一个queryBinder接口，这个接口能够根据业务模块的特征来返回相应的Binder对象

## ContentProvider
是android中提供的专门用于不同应用间进行数据共享的方式，底层实现是Binder。
1. query、update、insert、delete运行在Binder线程中，onCreate运行在主线程中，所以onCreate中不能执行耗时操作
2. query、update、insert、delete是存在多线程并发访问的，因此方法内部要做好线程同步

## Socket
Socket也称套接字，是网络通信中的概念，它分为流式套接字和用户数据报套接字两种，分别对应于网络的传输控制层中的TCP和UDP协议

1. TCP协议是面向连接的协议，提供稳定的双向通信功能，TCP连接的建立需要经过“三次握手”才能完成，为了提供稳定的数据传输功能，其本身提供了超时重传机制，因此具有很高的稳定性；
2. UDP是无连接的，提供不稳定的单向通信功能，当然UDP也可以实现双向通信功能，在性能上，UDP具有更好的效率，其缺点是数据不保证一定能正确传输，尤其是在网络拥塞的情况下



几种IPC方式的优缺点和适用场景

|  名 称 | 优点  | 缺点 | 使用场景 |
|:-------|:------|:-----|:---------|
|Bundle | 简单易用 | 只能传输Bundle支持的数据类型 | 四大组件间的进程间通信 |
|文件共享|简单易用|不适合高并发场景，并且无法做到进程间的即时通信|无并发访问情形，交换简单的数据实时性不高的场景|
|AIDL|功能强大，支持一对多并发通信，支持实时通信|使用稍复杂，需要处理好线程同步|一对多通信且有RPC需求|
|Messenger|功能一般，支持一对多串行通信，支持实时通信|不能很好处理高并发情形，不支持RPC，数据通过Message进行传输，因此只能传输Bundle支持的数据类型|低并发的一对多即时通信，无RPC需求，或者无须要返回结果的RPC需求|
|ContentProvider|在数据源访问方面功能强大，支持一对多并发数据共享，可通过Call方法扩展其他操作|可以理解为受约束的AIDL，主要提供数据源的CRUD操作|一对多的进程间的数据共享|
|Socket|功能强大，可以通过网络传输字节流，支持一对多并发实时通信|实现细节稍微有点繁琐，不支持直接的RPC|网络数据交换|










