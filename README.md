# Notification使用总结
##主要内容

 - <a href="#demo">Demo效果图</a>
 - <a href="#demoCode">示例代码</a>
 - <a href="#principle">实现原理</a>
 - <a href="#SourceCode">1. 源码解析</a>
    - <a href="#showNotificationAnalysis">1.1 显示通知</a>
        - <a href="#Builder">1.1.1 Builder</a>
        - <a href="#Parcelable">1.1.2 Parcelable</a>
        - <a href="#Parcel">1.1.3 Parcel</a>
        - <a href="#NotificationManager">1.1.4 NotificationManager(发送给Notification进程)</a>
    - <a href="#clickNotificationAnalysis">1.2 点击通知栏</a>
        - <a href="#PendingIntent">1.2.1 PendingIntent</a>
        - <a href="#IIntentSender">1.2.2 IIntentSender</a>
    - <a href="#myNotificationAnalysis">1.3 自定义通知栏</a>
        - <a href="#RemoteViews">1.3.1 RemoteViews</a>
        - <a href="#ApplicationInfo">1.3.2 ApplicationInfo</a>
    - <a href="#summary">2. 总结 </a>
    - <a href="#hrefs">3. 相关链接 </a>


### <div id="demo">Demo效果图</div>

<img src="https://github.com/Allyns/NotificationAnalysis/blob/master/Untitled.gif"/>

## <div id="demoCode">示例代码</div>

```java

    /**
     * 自定义通知栏
     */
        NotificationCompat.Builder mBuilder = new Builder(this);
        RemoteViews mRemoteViews = new RemoteViews(getPackageName(), R.layout.view_custom_button);
        mRemoteViews.setImageViewResource(R.id.custom_song_icon, R.drawable.sing_icon);
        //API3.0 以上的时候显示按钮，否则消失
        mRemoteViews.setTextViewText(R.id.tv_custom_song_singer, "周杰伦");
        mRemoteViews.setTextViewText(R.id.tv_custom_song_name, "七里香");
        //如果版本号低于（3。0），那么不显示按钮
        if(BaseTools.getSystemVersion() <= 9){
            mRemoteViews.setViewVisibility(R.id.ll_custom_button, View.GONE);
        }else{
            mRemoteViews.setViewVisibility(R.id.ll_custom_button, View.VISIBLE);
        }
        //
        if(isPlay){
            mRemoteViews.setImageViewResource(R.id.btn_custom_play, R.drawable.btn_pause);
        }else{
            mRemoteViews.setImageViewResource(R.id.btn_custom_play, R.drawable.btn_play);
        }
        //点击的事件处理
        Intent buttonIntent = new Intent(ACTION_BUTTON);
        /* 上一首按钮 */
        buttonIntent.putExtra(INTENT_BUTTONID_TAG, BUTTON_PREV_ID);
        //这里加了广播，所及INTENT的必须用getBroadcast方法
        PendingIntent intent_prev = PendingIntent.getBroadcast(this, 1, buttonIntent, PendingIntent.FLAG_UPDATE_CURRENT);
        mRemoteViews.setOnClickPendingIntent(R.id.btn_custom_prev, intent_prev);
        /* 播放/暂停  按钮 */
        buttonIntent.putExtra(INTENT_BUTTONID_TAG, BUTTON_PALY_ID);
        PendingIntent intent_paly = PendingIntent.getBroadcast(this, 2, buttonIntent, PendingIntent.FLAG_UPDATE_CURRENT);
        mRemoteViews.setOnClickPendingIntent(R.id.btn_custom_play, intent_paly);
        /* 下一首 按钮  */
        buttonIntent.putExtra(INTENT_BUTTONID_TAG, BUTTON_NEXT_ID);
        PendingIntent intent_next = PendingIntent.getBroadcast(this, 3, buttonIntent, PendingIntent.FLAG_UPDATE_CURRENT);
        mRemoteViews.setOnClickPendingIntent(R.id.btn_custom_next, intent_next);

        mBuilder.setContent(mRemoteViews)
                .setContentIntent(getDefalutIntent(Notification.FLAG_ONGOING_EVENT))
                .setWhen(System.currentTimeMillis())// 通知产生的时间，会在通知信息里显示
                .setTicker("正在播放")
                .setPriority(Notification.PRIORITY_DEFAULT)// 设置该通知优先级
                .setOngoing(true)
                .setSmallIcon(R.drawable.sing_icon);
        Notification notify = mBuilder.build();
        notify.flags = Notification.FLAG_ONGOING_EVENT;
        mNotificationManager.notify(notifyId, notify);

```
## <div id="principle">原理分析</div>
   Notification并不是表面的在通知栏显示通知那么简单,显示通知栏和点击通知栏的交互都需要使用进程通信，即使当前的App被完全关闭了通知栏也不会被关闭，
   因为Notification和所属的App并不会是在同一个进程，每一条通知栏显示后它都分配了单独的进程的并且优先级属于可见进程仅次于前台进程，就像搜狗输入法一样，
   平时不使用的时候看不到软键盘界面，等你点击输入框之后他就显示出来了，区别在于Notification是属于系统级别的！下面的源码解析将证明我的结论！

   gif图证明我的结论：

   <img src="https://github.com/Allyns/NotificationAnalysis/blob/master/notificationCourse.gif"/>


### <div id="SourceCode">源码解析</div>

本次Notification(通知栏)解析主要基于android 5.0(API 21)版本;


### <div id="showNotificationAnalysis">显示通知</div>

### <div id="Builder">Builder</div>

 说到notification的显示通知，不得不提到一个辛勤的劳动者Buileder，他是一种设计模式，通知到的内容和布局并不是固定的，他是根据设定的值来变换布局的，
 比如说有两个通知一个是有进度条的一个是没有的.改变了显示内容并不需要我们去改变布局，Buileder已经帮我们做好了这些事情，它可以将一个复杂对象的构建与它的表示分离，
 同样的构建过程可以创建不同的表示。每一步的构造过程中可以引入参数，使得经过相同的步骤创建最后得到的对象的展示不一样。


   Builder部分源代码

```java

public static class Builder {

       ／***
       ....
     ＊/
       private CharSequence mContentTitle; 
       private CharSequence mContentText; 
       private CharSequence mContentInfo; 
       private CharSequence mSubText; 
       private PendingIntent mContentIntent; 
       private RemoteViews mContentView; 
       private CharSequence mTickerText; 
       private RemoteViews mTickerView; 

       ／***
       ....
     ＊/
            RemoteViews contentView = new BuilderRemoteViews(mContext.getApplicationInfo(), resId);

            resetStandardTemplate(contentView);

            boolean showLine3 = false;
            boolean showLine2 = false;
            boolean contentTextInLine2 = false;

            if (mLargeIcon != null) {
                contentView.setImageViewIcon(R.id.icon, mLargeIcon);
                processLargeLegacyIcon(mLargeIcon, contentView);
                contentView.setImageViewIcon(R.id.right_icon, mSmallIcon);
                contentView.setViewVisibility(R.id.right_icon, View.VISIBLE);
                processSmallRightIcon(mSmallIcon, contentView);
            } else { // small icon at left
                contentView.setImageViewIcon(R.id.icon, mSmallIcon);
                contentView.setViewVisibility(R.id.icon, View.VISIBLE);
                processSmallIconAsLarge(mSmallIcon, contentView);
            } else {
                contentView.setViewVisibility(R.id.text2, View.GONE);
                if (hasProgress && (mProgressMax != 0 || mProgressIndeterminate)) {
                    contentView.setViewVisibility(R.id.progress, View.VISIBLE);
                    contentView.setProgressBar(
                            R.id.progress, mProgressMax, mProgress, mProgressIndeterminate);
                    contentView.setProgressBackgroundTintList(
                            R.id.progress, ColorStateList.valueOf(mContext.getColor(
                                    R.color.notification_progress_background_color)));
                    if (mColor != COLOR_DEFAULT) {
                        ColorStateList colorStateList = ColorStateList.valueOf(mColor);
                        contentView.setProgressTintList(R.id.progress, colorStateList);
                        contentView.setProgressIndeterminateTintList(R.id.progress, colorStateList);
                    }
                    showLine2 = true;
                } else {
                    contentView.setViewVisibility(R.id.progress, View.GONE);
                }
            }

    /***
      .....
    */
        private int getBaseLayoutResource() {
            return R.layout.notification_template_material_base;
        }

}
```

 
### <div id="hrefs">相关链接</div>

 - [Buileder设计模式](http://www.cnblogs.com/bastard/archive/2011/11/21/2257625.html)

 - [Parcelable](http://www.cnblogs.com/renqingping/archive/2012/10/25/Parcelable.html)

 - [Parce](http://blog.csdn.net/mznewfacer/article/details/7847379)

 - [Android进程管理详解](http://blog.csdn.net/hyggt/article/details/7255043)

 - [Android 进程间通信实现原理分析](http://www.jb51.net/article/37797.htm)

 - [Android进程间通信](http://www.cnblogs.com/imlucky/p/3246013.html)
