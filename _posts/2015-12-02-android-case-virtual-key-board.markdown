---
date: "2015-12-02 17:52:00"
description: ""
layout: post
permalink: android-case-virtual-keyboard
categories:
  - blog
  - android
  - case
title: Android开发案例 - 自定义虚拟键盘
---

所有包含IM功能的App(如微信, 微博, QQ, 支付宝等)都提供了Emoji表情之类的虚拟键盘,
 如下图:

![](<http://erehmi.github.io/assets/image/wechat_chat.png>)

本文只着重介绍如何实现输入法键盘和自定义虚拟键盘的流畅切换,
而不介绍如何实现虚拟键盘, 因为后者实现相对容易, 而前者若实现不好,
则会出现体验的问题, 比如输入区域的视图在切换时会跳动等问题.

### **知识要点:**

-   AndroidManifest.xml: activity属性 *android:windowSoftInputMode *

-   InputMethodManager

-   Window 管理机制

-   View 管理机制

### 基本思路:

-   假设最外层视图为LinearLayout

-   虚拟键盘以layout形式直接添加到页面layout-xml中, 进入上述图中页面时,
    默认是隐藏该虚拟键盘布局的, 输入法键盘也是默认收起的

-   虚拟键盘高度应该为输入法键盘的高度, 因此, 输入法弹出时需要保存其高度值,
    另外, 如果弹出虚拟键盘前未弹出过输入法键盘, 那么这时是不知道其高度值的,
    因此需要预设一个高度值

-   在切换键盘时, 动态调整内容区域(如聊天内容列表)的高度, 当虚拟键盘弹出时,
    设置内容区域的高度为固定高度值, 而当虚拟键盘隐藏时, 还原内容区域的高度,
    这样就可以实现键盘间的流畅切换了

### 实现代码:
-   [github -
    VirtualKeyboardController](<https://github.com/erehmi/VirtualKeyboardController>) 

![](<http://images2015.cnblogs.com/blog/608747/201512/608747-20151204123852893-457357290.gif>)

### 具体用法:

[github - VirtualKeyboardController \#
Sample ](<https://github.com/erehmi/VirtualKeyboardController/tree/master/sample>)

### 扩展用法:
如果页面布局的最外层视图不是LinearLayout,
扩展 *VirtualKeyboardController.LayoutManager* 接口即可,
可参考 *LinearLayoutManager *实现代码.


### 相关:
-   <http://www.dss886.com/android/2015/10/18/10-41/>


**END. \>\> SEE MORE:**
[http://erehmi.github.io/](<**http://erehmi.github.io/**>)
