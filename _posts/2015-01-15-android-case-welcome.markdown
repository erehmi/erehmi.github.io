---
date: "2015-01-15 13:10:00"
description: ""
layout: post
permalink: android-case-welcome
categories:
  - blog
  - android
  - case
title: Android开发案例 - 欢迎界面
---

本文详细描述了如何实现如下图中的微信启动界面. 该类启动界面的特点是在整个Application的生命周期里, 它只会出现在第一次进入应用时, 即便按回退键到桌面之后. 使用该类启动界面的应用还有: QQ, QQ音乐, 网易云音乐和微博等等.

![](<http://erehmi.github.io/assets/image/wechat-welcome.png>)

知识要点:
---------
-   AndroidManifest.xml 中 activity 的 android:noHistory 属性, 即 Intent.FLAG\_ACTIVITY\_NO\_HISTORY
-   隐式Intent
-   回退栈(BackStack)

详细内容见[官方文档](<http://developer.android.com/develop/index.html>). 

实现代码:
---------
**\> 定义**
-   SplashActivity 为启动界面
-   MainActivity 为主界面 

**\> AndroidManifest.xml**
{% highlight xml %}
<!-- 该文件为AndroidManifest.xml, 以下代码为application下的activity声明 -->

<!-- 启动界面 -->
<activity android:name=".SplashActivity"
          android:label="@string/app_name"
          android:noHistory="true">
    <intent-filter>
        <action android:name="android.intent.action.MAIN"/>
        <category android:name="android.intent.category.LAUNCHER"/>
    </intent-filter>
</activity>

<!-- 主界面 -->
<activity android:name=".MainActivity" android:label="@string/app_name">
</activity>
{% endhighlight %} 

特别需要注意的是, 在上述Activity-XML定义中,
我们设置了SplashActivity为noHistory的属性为true,
该设置是告诉系统只要离开该activity, 则请把该activity从回退栈中清除. 另外,
直接在Intent中设置Intent.FLAG\_ACTIVITY\_NO\_HISTORY标识的效果同设置该属性为true的.

**\> SplashActivity.java**
{% highlight java %}
import ...

public abstract class SplashActivity extends Activity implements Runnable {
    final Handler mHandler = new Handler();

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_splash);

        mHandler.postDelayed(this, 2000);
    }

    @Override
    public void run() {
        Intent intent = new Intent(this, MainActivtiy.class);
        startActivity(intent);
        // 此处可以不需要调用finish()了, 因为已经设置了noHistory属性, 从而使得系统接管finish操作
    }
}
{% endhighlight %} 

**\> MainActivity.java**
{% highlight java %}
import ...

public abstract class MainActivity extends Activity {
    ...

    @Override
    public void onBackPressed() {
        // 方法 1: goto the default launcher. It's not recommended.
        // Intent i = new Intent(Intent.ACTION_MAIN);
        // i.setFlags(Intent.FLAG_ACTIVITY_NEW_TASK);
        // i.addCategory(Intent.CATEGORY_HOME);
        // startActivity(i);

        // 方法 2: goto the default launcher. It's recommended.
        moveTaskToBack(true);
    }
}
{% endhighlight %} 

上述代码中, 提供了两个方法,
第一个方法通过隐式Intent来切换到桌面应用(即Launcher),
第二个方法则是将当前activity所在的task切换到后台, 需要注意的是,
moveTaskToBack(boolean nonRoot) 的 nonRoot 参数, 如果nonRoot=false,
则要求当前activity为栈顶activity, 否则, 调用将不起任何效果, 如果nonRoot=true,
则忽略nonRoot=false时的条件, 因此, 我们在这里直接设置nonRoot=true 

到这里, 我们可以一直按回退键, 直到切换到桌面,
这时SplashActivity已经被系统清理了,
MainActivity连同它所在的Task已经切换到后台了. 下次我们再启动应用时,
只要MainActivity没有被系统回收,
那么我们再看到的MainActivity还是退回到桌面前的那个MainActivity.


**END. \>\> SEE MORE:**
[http://erehmi.github.io/](<**http://erehmi.github.io/**>)
