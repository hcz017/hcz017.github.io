---
date: 2014-09-27 11:14
status: public
title: android.os.NetworkOnMainThreadException
url: NetworkOnMainThreadException
tags: debug
---

**2014-10-29 更新内容**

代码中有网络请求，程序执行的时候LogCat报错如下：

android.os.NetworkOnMainThreadException
at android.os.StrictMode$AndroidBlockGuardPolicy.onNetwork

有趣的是我偶然发现当我按照LogCat给出的android.os.NetworkOnMainThreadException去搜索的时候，大多数结果都是使用Thread线程操作。
而我按照第二条at android.os.StrictMode$AndroidBlockGuardPolicy.onNetwork搜索的时候，得到了不少用AsyncTask异步解决的结果。

**原因：**
Android在4.0之前的版本支持在主线程中访问网络，但是在4.0以后对这部分程序进行了优化，也就是说访问网络的代码不能写在主线程中了（也有网页上说是3.0）。

**解决方法一：**使用Thread
将请求网络资源的代码使用Thread去操作。在Runnable中做HTTP请求，不用阻塞UI线程。


    public void onCreate(Bundle savedInstanceState) {
    super.onCreate(savedInstanceState);
    this.setContentView(R.layout.main_view);
     // 开启一个子线程，进行网络操作，等待有返回结果，使用handler通知UI  
    new Thread(runnable).start();
    }

    Handler handler = new Handler(){
        @Override
        public void handleMessage(Message msg) {
            super.handleMessage(msg);
            Bundle data = msg.getData();
            String val = data.getString("value");
            Log.i(TAG,"请求结果:" + val);
            // TODO  
            // UI界面的更新等相关操作  
        }
    }
    
    Runnable runnable = new Runnable(){
        @Override
        public void run() {
            // TODO: http request.
            // 在这里进行 http request.网络请求相关操作  
            Message msg = new Message();
            Bundle data = new Bundle();
            data.putString("value","请求结果");
            msg.setData(data);
            handler.sendMessage(msg);
        }
    }
**解决方法二：**使用AsyncTask

下面是google给的代码示例，出自[AsyncTask](http://developer.android.com/reference/android/os/AsyncTask.html)。
我们把网络请求写在doInBackground()方法里就好了。

>AsyncTask must be subclassed to be used. The subclass will override at least one method (doInBackground(Params...)), and most often will override a second one (onPostExecute(Result).)

AsyncTask的必须被继承使用。子类至少覆盖一个方法（doInBackground（参数...）），最经常将覆盖中的第二个方法是（onPostExecute（结果）。）

>Here is an example of subclassing:

下面是一个子类的示例：

     private class DownloadFilesTask extends AsyncTask<URL, Integer, Long> {
         protected Long doInBackground(URL... urls) {
             int count = urls.length;
             long totalSize = 0;
             for (int i = 0; i < count; i++) {
                 totalSize += Downloader.downloadFile(urls[i]);
                 publishProgress((int) ((i / (float) count) * 100));
                 // Escape early if cancel() is called
                 if (isCancelled()) break;
             }
             return totalSize;
         }

         protected void onProgressUpdate(Integer... progress) {
             setProgressPercent(progress[0]);
         }

         protected void onPostExecute(Long result) {
             showDialog("Downloaded " + result + " bytes");
         }
     }
 
>Once created, a task is executed very simply:

子类创建后用下面的代码执行：

 	new DownloadFilesTask().execute(url1, url2, url3);  
>Not all types are always used by an asynchronous task. To mark a type as unused, simply use the type Void:

并不是所有的参数总是被异步任务所使用，如果一个参数未使用，只需用Void代替：


	 private class MyTask extends AsyncTask<Void, Void, Void> { ... }

更多AsyncTask可以参看本博：[AsyncTaskde 的使用](http://codesimple.bitcron.com/post/android/asynctaskde-de-shi-yong "AsyncTaskde 的使用")
##AsyncTask和Handler对比##

至于哪个适合你的程序，看看下面的优缺点吧。

1 ） AsyncTask实现的原理,和适用的优缺点
AsyncTask,是android提供的轻量级的异步类,可以直接继承AsyncTask,在类中实现异步操作,并提供接口反馈当前异步执行的程度(可以通过接口实现UI进度更新),最后反馈执行的结果给UI主线程.
使用的优点:
-   简单,快捷
-  过程可控   
  
使用的缺点:
-  在使用多个异步操作和并需要进行Ui变更时,就变得复杂起来.

2 ）Handler异步实现的原理和适用的优缺点
在Handler 异步实现时,涉及到 Handler, Looper, Message,Thread四个对象，实现异步的流程是主线程启动Thread（子线程）àthread(子线程)运行并生成Message-àLooper获取Message并传递给HandleràHandler逐个获取Looper中的Message，并进行UI变更。
使用的优点：
- 结构清晰，功能定义明确
- 对于多个后台任务时，简单，清晰

使用的缺点：
- 在单个后台异步处理时，显得代码过多，结构过于复杂（相对性）

 
参考：
1. [[Android开发那点破事]解决android.os.NetworkOnMainThreadException](http://www.2cto.com/kf/201402/281526.html)
2. [Android之NetworkOnMainThreadException异常](http://blog.csdn.net/mad1989/article/details/25964495)
3. [android.os.NetworkOnMainThreadException的解决方案](http://www.cnblogs.com/kissazi2/p/3153307.html)
4. [AsyncTask](http://developer.android.com/reference/android/os/AsyncTask.html)

感谢原作者！