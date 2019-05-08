## Android异步的实现方式
- Author：WeicongLee
- Data：2019.05.07

#### 用Java的方法来实现异步
- 继承Thread类，重写run方法
- 实现Runnable接口，用于创建Thread对象

#### 使用Android特有的方法实现
- AsyncTask
- Handler消息机制
- RxJava第三方观察者机制


## 用Java的方法来实现异步
#### 1）继承Thread类，重写run()方法：
```JAVA
public class MyThread extends Thread {
    private Context context;
    //构造方法，接收上下文
    public MyThread(Context context) {
        this.context = context;
    }
    //重写run()方法，在里面做耗时操作
    @Override
    public void run() {
        ((Activity)context).runOnUiThread(new Runnable() {
            @Override
            public void run() {
                //UI更新操作
            }
        });
    }
}
//传递当前的上下文
MyThread thread = new MyThread(xxxActiivity.this);
thread.start(); 
```

#### 2）实现Runnable接口：
- 新建类实现Runnable接口，重写run()方法
    ```JAVA
    public class MyRunnable implements Runnable {
        private Context context;
        //构造方法，接收上下文
        public MyRunnable(Context context) {
            this.context = context;
        }
        //重写run()方法，在里面做耗时操作
        @Override
        public void run() {
            ((Activity)context).runOnUiThread(new Runnable() {
                @Override
                public void run() {
                    //UI更新操作
                }
            });
        }
    }
    //新建Runnable实例
    MyRunnable runnable = new MyRunnable(xxxActiivity.this);
    //将Runnable实例传递给Thread
    Thread thread = new Thread(runnable);
    thread.start(); 
    ```
- 直接new一个Thread实现内部类
    ```JAVA
    new Thread(new Runnable() {
        @Override
        public void run() {
            runOnUiThread(new Runnable() {
                @Override
                public void run() {
                    //UI更新操作
                }
            });
        }
    }).start();
    ```

## 使用Android特有的方法来实现异步
#### 1）实现一个内部类继承AsyncTask，并重写方法
```JAVA
class MyAsyncTask extends AsyncTask<String, Integer, Bitmap>{
    @Override
    protected void onPreExecute() {
        //处于主线程，在线程开始之前执行的
        super.onPreExecute();
    }
    @Override
    protected Bitmap doInBackground(String... strings) {
        //在后台子线程中执行的操作
        return null;
    }
    @Override
    protected void onCancelled() {
        //当任务被取消时回调
        super.onCancelled();
    }
    @Override
    protected void onProgressUpdate(Integer... values) {
        super.onProgressUpdate(values);
        //更新进度
    }
    @Override
    protected void onPostExecute(Bitmap bitmap) {
        super.onPostExecute(bitmap);
        //当任务执行完成时调用
    }
}
MyAsyncTask asyncTask = new MyAsyncTask();
asyncTask.execute("xxx");
```
用AsyncTask实现异步简单便携，各个过程都有明确的回调，过程可控，但缺点在于若执行多个异步，就会变得复杂

#### 2）使用Handler
- Handler常用来解决线程间通信，可以在子线程中提醒UI线程更新组件
    ```JAVA
    //实例化一个Handler，并重写handleMessage()方法
    private Handler handler = new Handler(){
        @Override
        public void handleMessage(Message msg) {
            //可根据传过来的参数判别是哪个操作
            switch (msg.what){
                case 0:
                    break;
            }
        }
    };
    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        //子线程
        new Thread(new Runnable() {
            @Override
            public void run() {
                //(可选)新建Message，参数赋值
                Message message = new Message();
                message.what = 0;
                //向主线程handler发送消息
                handler.sendMessage(message);
            }
        }).start();
    }
    ```
- 主线程也可以使用handler往子线程发送消息
    ```JAVA
    //创建Handler实例，默认为初始化
    private Handler handler = null;
    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        //子线程
        new Thread(new Runnable() {
            @Override
            public void run() {
                //(可选)新建Message，参数赋值
                Message message = new Message();
                message.what = 0;
                //向子线程的handler发送消息
                handler.sendMessage(message);
            }
        }).start();
    }
    //实现Thread类
    class MyThread extends Thread {
        @Override
        public void run() {
            super.run();
            //建立消息循环，初始化Looper
            Looper.prepare();
            //初始化主线程的handler
            handler = new Handler() {
                @Override
                public void handleMessage(Message msg) {
                    super.handleMessage(msg);
                    //可根据传过来的参数判别是哪个操作
                    switch (msg.what) {
                        case 0:
                            break;
                    }
                }
            };
            //启动消息循环
            Looper.loop();
        }
    }
    ```

#### 3）使用RxJava2
详细使用可以查阅[掘金](https://juejin.im/post/5b17560e6fb9a01e2862246f#heading-284)
- 先在gradle中添加依赖
    ```Java
    implementation 'io.reactivex.rxjava2:rxjava:2.1.4'
    implementation 'io.reactivex.rxjava2:rxandroid:2.0.2'
    ```
- 创建被观察者
    ```JAVA
    Observable observer = Observable.create(new ObservableOnSubscribe() {
        @Override
        public void subscribe(ObservableEmitter emitter) throws Exception {
            emitter.onComplete();
        }
    })
    ```
- 创建观察者
    ```JAVA
    Observable<String> reader = new Observer<String>() {
        @Override
        public void onSubscribe(Disposable d) {
            Log.i(TAG, "onSubscribe: ");
        }
        @Override
        public void onNext(String s) {
            Log.i(TAG, "onNext: " + s);
        }
        @Override
        public void onError(Throwable e) {
            Log.i(TAG, "onError: " + e.getMessage());
        }
        @Override
        public void onComplete() {
            Log.i(TAG, "onComplete: ");
        }
    }
    ```
- 建立订阅关系
    ```JAVA
    observer.subscribe(reader);
    ```

- 也可直接使用链式编程一步到位
    ```JAVA
    Observable.create(new ObservableOnSubscribe() {
        @Override
        public void subscribe(ObservableEmitter emitter) throws Exception {
            emitter.onComplete();
        }
    })
        //回调线程
        .observeOn(AndroidSchedulers.mainThread())
        //执行线程
        .subscribeOn(Schedulers.io())
        //添加订阅者
        .subscribe(new Observer<String>() {
            @Override
            public void onSubscribe(Disposable d) {
                Log.i(TAG, "onSubscribe: ");
            }
            @Override
            public void onNext(String s) {
                Log.i(TAG, "onNext: " + s);
            }
            @Override
            public void onError(Throwable e) {
                Log.i(TAG, "onError: " + e.getMessage());
            }
            @Override
            public void onComplete() {
                Log.i(TAG, "onComplete: ");
            }
        });
    ```
