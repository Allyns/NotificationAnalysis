# Notification源代码解析


解析文档编写中...


##简介
本次Notification(通知栏)解析主要基于android 5.0(API 21)版本;


##主要内容

 - <a href="#demo">Demo效果图</a>
 - <a href="#principle">实现原理</a>
 - <a href="#demoCode">示例代码</a>
 - <a href="#SourceCode">1. 源码解析</a>
    - <a href="#showNotificationAnalysis">1.1 显示通知</a>
        - <a href="#Notification">1.1.1 Notification</a>
        - <a href="#NotificationManager">1.1.2 NotificationManager</a>
        - <a href="#Builder">1.1.3 Builder</a>
        - <a href="#getSystemService">1.1.3 getSystemService</a>
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


### <div id="SourceCode">源码解析</div>

 - [MainActivity](https://github.com/Allyns/NotificationAnalysis/blob/master/app/src/main/java/com/example/apple/notificationanalysis/MainActivity.java)


### <div id="hrefs">相关链接</div>

 - [Buileder设计模式](http://www.cnblogs.com/bastard/archive/2011/11/21/2257625.html)

 - [Parcelable](http://www.cnblogs.com/renqingping/archive/2012/10/25/Parcelable.html)

 - [Parce](http://blog.csdn.net/mznewfacer/article/details/7847379)

 - [Android进程管理详解](http://blog.csdn.net/hyggt/article/details/7255043)

 - [Android 进程间通信实现原理分析](http://www.jb51.net/article/37797.htm)

 - [Android进程间通信](http://www.cnblogs.com/imlucky/p/3246013.html)
