---
title: Notification 简单使用
date: 2014-12-08 19:33
status: draft
---

Android Notification 简单使用，java代码如下：

    public class MainActivity extends Activity implements View.OnClickListener {
        NotificationManager notificationManager;//通知控制类
        int Notification_ID = 0;
        @Override
        protected void onCreate(Bundle savedInstanceState) {
            super.onCreate(savedInstanceState);
            setContentView(R.layout.activity_main);
            notificationManager= (NotificationManager) getSystemService(Context.NOTIFICATION_SERVICE);
            findViewById(R.id.btn_send).setOnClickListener(this);
            findViewById(R.id.btn_cancle).setOnClickListener(this);
        }
    
        @Override
        public void onClick(View v) {
            switch (v.getId()){
                case R.id.btn_send:
                    sendNotification();
                    break;
                case R.id.btn_cancle:
                    notificationManager.cancel(Notification_ID);
                    break;
            }
        }
        /**
         * 构造notification并发送到通知栏
         */
        private void sendNotification(){
    
            Intent intent=new Intent(this,MainActivity.class);
            PendingIntent pendingIntent=PendingIntent.getActivity(this, 0, intent, 0);
            Notification.Builder builder =new Notification.Builder(this);
            builder.setSmallIcon(R.drawable.ic_launcher);//图标
            builder.setTicker("hello");//手机状态栏的提示
            builder.setWhen(System.currentTimeMillis());//时间
            builder.setContentTitle("标题");
            builder.setContentText("内容");
            builder.setContentIntent(pendingIntent);//点击后的意图
    //        builder.setDefaults(Notification.DEFAULT_SOUND);//提示音
    //        builder.setDefaults(Notification.DEFAULT_VIBRATE);//指示灯
    //        builder.setDefaults(Notification.DEFAULT_LIGHTS);//震动
            builder.setDefaults(Notification.DEFAULT_ALL);//包括以上三种
            Notification notification=builder.build();//4.1以上的
            notificationManager.notify(Notification_ID,notification);
        }
    }

layout内容 为两个按钮