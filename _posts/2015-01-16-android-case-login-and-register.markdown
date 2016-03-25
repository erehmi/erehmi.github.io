---
date: "2015-01-16 08:30:00"
description: ""
layout: post
permalink: android-case-login-and-register
categories:
  - blog
  - android
  - case
title: Android开发案例 - 注册登录
---

本文只涉及UI方面的内容, 如果您是希望了解非UI方面的访客, 请跳过此文.

在微博, 微信等App的注册登录过程中有这样的交互场景(如下图):

1.  打开登录界面

2.  在登录界面中, 点击注册, 跳转到注册界面

3.  如果取消注册, 则回退到登录界面, 如果完成注册,跳转到主页面

**在整个交互场景里有个问题, 就是如何在完成注册之后关闭之前已经打开过的界面.
那如果层级更深的话, 应该如何处理?**

![](<http://erehmi.github.io/assets/image/weibo-login-and-register.png>)

~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
注: 按此文分割线以下的方式处理, 存在一些问题, 如下:
> 假设注册登录之前的界面是欢迎界面, 并注册登录成功
> 这时, 以Intent.FLAG_ACTIVITY_CLEAR_TASK & Intent.FLAG_ACTIVITY_NEW_TASK方式启动主界面
> 进入应用管理强制停止应用, 再从最近使用过应用的列表进入应用
> 这时, 问题就来了, 应用没有进入欢迎界面, 而是直接进入了主界面

建议使用Intent.FLAG_ACTIVITY_CLEAR_TOP来实现注册登录的界面跳转, 大体思路, 如下:
> 设置欢迎界面的activity.launchMode属性为singleTop, 并接管onNewIntent(Intent), 若满足跳转主界面的条件, 则直接进入主界面
> 注册登录成功后, 使用Intent.FLAG_ACTIVITY_CLEAR_TOP方式跳转到欢迎界面
> 这时, 程序会运行到欢迎界面的onNewIntent(Intent), 这是因为欢迎界面现在是BackStack的栈顶activity了. 最后将直接进入主界面
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~


 **知识要点**
-   Intent.FLAG\_ACTIVITY\_CLEAR\_TASK
-   Intent.FLAG\_ACTIVITY\_NEW\_TASK

**P.S.** Intent.FLAG\_ACTIVITY\_CLEAR\_TASK与
Intent.FLAG\_ACTIVITY\_CLEAR\_TOP不同, 它是在API-Level-11时引入的,
使用它会清空当前task, 并让新activity成为一个空task的root activity,
另外它要求与Intent.FLAG\_ACTIVITY\_NEW\_TASK一起使用.
而Intent.FLAG\_ACTIVITY\_CLEAR\_TOP的调用效果同activity的singTask启动模式.

　　官方文档如下:

>   public static final int **FLAG\_ACTIVITY\_CLEAR\_TASK**
>   Added in API level 11
>   If set in an Intent passed to Context.startActivity(), this flag will cause
>   any existing task that would be associated with the activity to be cleared
>   before the activity is started. That is, the activity becomes the new root
>   of an otherwise empty task, and any old activities are finished. This can
>   only be used in conjunction with FLAG\_ACTIVITY\_NEW\_TASK.
>    public static final int **FLAG\_ACTIVITY\_NEW\_TASK**
>   Added in API level 1
>   If set, this activity will become the start of a new task on this history
>   stack. A task (from the activity that started it to the next task activity)
>   defines an atomic group of activities that the user can move to. Tasks can
>   be moved to the foreground and background; all of the activities inside of a
>   particular task always remain in the same order. See Tasks and Back Stack
>   for more information about tasks.
>   This flag is generally used by activities that want to present a "launcher"
>   style behavior: they give the user a list of separate things that can be
>   done, which otherwise run completely independently of the activity launching
>   them.
>   When using this flag, if a task is already running for the activity you are
>   now starting, then a new activity will not be started; instead, the current
>   task will simply be brought to the front of the screen with the state it was
>   last in. See FLAG\_ACTIVITY\_MULTIPLE\_TASK for a flag to disable this
>   behavior.
>   This flag can not be used when the caller is requesting a result from the
>   activity being launched.

**实现代码**
**\> 定义**
-   LoginActivity - 登录界面
-   RegisterActivity - 注册界面
-   MainActivity - 主界面

**\> LoginActivity.java**
略. 常规代码实现, 即直接调用startActivity来启动注册界面, 因此,
在注册界面按回退键后, 还是会回到该界面.

**\> RegisterActivity.java**
{% highlight java %}
import ...

public class RegisterActivity extends Activity {
    ...

    /**
     * 注册成功, 跳转到主界面
     */
    private void finishWhenRegisterSucceeded() {
        ComponentName component = new ComponentName(this, MainActivity.class);
        Intent intent = IntentCompat.makeRestartActivityTask(component);
        startActivity(intent);
    }
}
{% endhighlight %} 

在上述代码中, 使用了android-support-v4中的IntentCompat类,
该类封装了上述两个Intent-Flag.

到这里, 注册成功后, 进入了MainActivity后, MainActivity就是一个新task的root
activity了, 而其他activity则都被清理了.
需要说明的是, finishWhenRegisterSucceeded() **没有**对启动RegisterActivity的LoginActivity做任何处理或者Intent回传.


**END. \>\> SEE MORE:**
[http://erehmi.github.io/](<**http://erehmi.github.io/**>)
