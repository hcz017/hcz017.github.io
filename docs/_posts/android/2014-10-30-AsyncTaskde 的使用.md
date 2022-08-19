---
date: 2014-10-30 18:00
status: public
title: 'AsyncTaskde 的使用'
tags: AsyncTask
---

AsyncTask的定义：

    public abstract class AsyncTask<Params, Progress, Result> {  

AsyncTask是抽象类.AsyncTask定义了三种泛型类型 Params，Progress和Result。
* Params 启动任务执行的输入参数，比如HTTP请求的URL。
* Progress 后台任务执行的百分比。
* Result 后台执行任务最终返回的结果，比如String,Integer等。
在特定场合下，并不是所有类型都被使用，如果没有被使用，可以用java.lang.Void类型代替。

一个异步任务的执行一般包括以下几个步骤：
1. **execute(Params... params)**，执行一个异步任务，需要我们在代码中调用此方法，触发异步任务的执行。
2. **onPreExecute()**，在execute(Params... params)被调用后立即执行，一般用来在执行后台任务前对UI做一些标记。如在界面上显示一个进度条，或者一些控件的实例化，这个方法可以不用实现。
3. **doInBackground(Params... params)**，在onPreExecute()完成后立即执行，用于执行较为费时的操作，此方法将接收输入参数和返回计算结果。在执行过程中可以调用publishProgress(Progress... values)来更新进度信息。也可以另外写一个方法执行操作，然后在此调用。（如果你需要进行网络操作，那么相应的代码应当在此被执行。）
4. **onProgressUpdate(Progress... values)**，在调用publishProgress(Progress... values)时，此方法被执行，直接将进度信息更新到UI组件上。
5. **onPostExecute(Result result)**，当后台操作结束时，此方法将会被调用，result为doInBackGround的返回值。计算结果将做为参数传递到此方法中，直接将结果显示到UI组件上。

在使用的时候，有几点需要格外注意：
1. 异步任务（AsyncTask）的实例必须在UI线程中创建。
2. execute(Params... params)方法必须在UI线程中调用。
3. 不要手动调用onPreExecute()，doInBackground(Params... params)，onProgressUpdate(Progress... values)，onPostExecute(Result result)这几个方法。
4. 不能在doInBackground(Params... params)中更改UI组件的信息。
5. 一个任务实例只能执行一次，如果执行第二次将会抛出异常。
下面是一个例子：

        public class asyncTaskTest extends Activity {
            private TextView textView;
            private static final int DIALOG_KEY = 0;
            public String httpadd="http://codesimple.farbox.com/category";
            @Override
            protected void onCreate(Bundle savedInstanceState) {
                super.onCreate(savedInstanceState);
                setContentView(R.layout.activity_async_task_test);
                textView=(TextView)findViewById(R.id.textView);
                /**
                 *    1、异步任务在UI线程中被创建并执行,参数可以全为空
                 */
                new MyAsyncTask().execute(null, null, null);
            }
        
            private class MyAsyncTask extends AsyncTask<URL, Integer, String> {
        
                /**
                 *  2、execute被调用后立刻执行此方法，执行异步任务前对UI做些提示。
                 */
                protected void onPreExecute() {
                    showDialog(DIALOG_KEY);
                }
        
                /**
                 * 3、在onPreExecute()完成后立即执行。
                 * 此方法将接收输入参数和返回计算结果。
                 * 该方法并不运行在UI线程当中，主要用于异步操作，
                 * 这里的参数对应AsyncTask中的第一个参数，即输入参数。
                 * @return 对应AsyncTask的第三个参数，这个参数会传到onPostExecute（）里。
                 */
                protected String doInBackground(URL... urls) {
                    StringBuffer buffer=new StringBuffer();
                    //下面的代码需要jsoup的包，请自行搜索下载。
                    Document doc;
                    try {
                        doc = Jsoup.connect(httpadd).get();
                        Elements ListDiv = doc.getElementsByAttributeValue("class","title");
                        for (Element element :ListDiv) {
                            Elements links = element.getElementsByTag("a");
                            for (Element link : links) {
                                String linkHref = link.attr("href");
                                String linkText = link.text();
                                buffer.append("link=http://codesimple.farbox.com"+linkHref+"\r\n");
                                buffer.append("title="+linkText+"\r\n");
                            }
                        }
                    } catch (IOException e) {
                        // TODO Auto-generated catch block
                        e.printStackTrace();
                    }
                    return buffer.toString();
                }
        
                /**
                 * 4、onProgressUpdate在UI线程中执行，可以对UI空间进行操作。
                 * @param progress 对应AsyncTask中的第二个参数，代表后台执行进度。
                 */
                protected void onProgressUpdate(Integer... progress) {
        //            setProgressPercent(progress[0]);
                }
        
                /**
                 * 5、此方法在doInBackground方法执行结束之后被调用
                 * doInBackground的返回结果将做为参数传到此方法中。
                 * 此方法被UI线程调用，计算结果将传递到UI线程，并且在界面上展示给用户。
                 * @param result 对应AsyncTask中的第三个参数（也就是接收doInBackground的返回值）
                 */
                protected void onPostExecute(String result) {
                    textView.setText(result);
                    removeDialog(DIALOG_KEY);
                }
            }
            protected Dialog onCreateDialog(int id) {
                switch (id) {
                    case DIALOG_KEY: {
                        ProgressDialog dialog = new ProgressDialog(this);
                        dialog.setMessage("获取数据中  请稍候...");
                        dialog.setIndeterminate(true);
                        dialog.setCancelable(true);
                        return dialog;
                    }
                }
                return null;
            }
        }
        
布局文件xml代码：

    <LinearLayout xmlns:android="http://schemas.android.com/apk/res/android"
        xmlns:tools="http://schemas.android.com/tools"
        android:layout_width="match_parent"
        android:layout_height="match_parent"
        android:orientation="vertical" >
        <ScrollView
            android:layout_width="match_parent"
            android:layout_height="match_parent">
    
            <TextView
                android:layout_width="wrap_content"
                android:layout_height="wrap_content"
                android:textAppearance="?android:attr/textAppearanceMedium"
                android:text="Medium Text"
                android:id="@+id/textView" />
        </ScrollView>
    
    </LinearLayout>
        
            
更详细的说明请移步[详解Android中AsyncTask的使用](http://blog.csdn.net/liuhe688/article/details/6532519)、[Android中AsyncTask的简单用法](http://blog.csdn.net/cjjky/article/details/6684959)、[android AsyncTask介绍](http://www.cnblogs.com/devinzhang/archive/2012/02/13/2350070.html)。
  
**参考：**
1. [详解Android中AsyncTask的使用](http://blog.csdn.net/liuhe688/article/details/6532519)
2. [Android中AsyncTask的简单用法](http://blog.csdn.net/cjjky/article/details/6684959)
3. [android AsyncTask介绍](http://www.cnblogs.com/devinzhang/archive/2012/02/13/2350070.html)

感谢原作者。