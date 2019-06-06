---
date: "2016-03-22 12:00:00"
description: ""
layout: post
permalink: android-case-abslistview-count-down
categories:
  - blog
  - android
  - case
title: Android开发案例 – 在AbsListView中使用倒计时
---


在App中, 有多种多样的倒计时需求, 比如:

1.  在单View上, 使用倒计时, 如(如图-1)

2.  在ListView(或者GridView)的ItemView上, 使用倒计时(如图-2)

![](<http://mo0n1andin.github.io/assets/image/count-down.png>)

相比需求-1, 需求-2的难度更大, 性能要求更高:
因为AbsListView会涉及到ItemView重用的问题会使得管理定时器很麻烦,
另外如果定时地通过Base\#notifyDataChanged()去刷新数据, 性能又相对较低,
也会引起滚动卡顿的问题. 因此,
此文主要解决的问题是如何合理地在AbsListView中使用倒计时.

知识要点
--------

1.  AbsListView ItemView重用机制

2.  android.os.CountDownTimer

基本思路
--------

1.  使用CountDownTimer来完成基本倒计时功能

2.  按倒计时的时间间隔来分组管理CountDownTimer,
    即相同时间间隔的Item使用同一个CountDownTimer

3.  每组CountDownTimer倒计时的时间取组内的最大值, 一旦Item到达自身的倒计时时间,
    就会从该组倒计时中被移除

4.  定义一个倒计时任务, 用来管理上述分组

5.  每个业务可以根据需要创建并启动多个倒计时任务,
    且可以在适当的页面生命周期函数中停止该任务

实现代码
--------

-   github
    – [mo0n1andin](<https://github.com/mo0n1andin>)**/**[CountDownTask](<https://github.com/mo0n1andin/CountDownTask>) 

具体用法
--------

1.  普通页面: [演示代码](<https://github.com/mo0n1andin/CountDownTask/blob/master/samples/src/main/java/io/github/mo0n1andin/samples/SimpleActivity.java>)

2.  列表页面: [演示代码](<https://github.com/mo0n1andin/CountDownTask/blob/master/samples/src/main/java/io/github/mo0n1andin/samples/ListActivity.java>)


**END. \>\> SEE MORE:**
[http://mo0n1andin.github.io/](<**http://mo0n1andin.github.io/**>)
