---
date: "2015-12-21 17:29:00"
description: ""
layout: post
permalink: android-case-taobao-goods-detail
categories:
  - blog
  - android
  - case
title: Android开发案例 - 淘宝商品详情
---


所有电商APP的商品详情页面几乎都是和淘宝的一模一样(见下图):
-   采用上下分页的模式
-   商品基本参数 & 选购参数在上页展示
-   商品图文详情等其他信息放在下页展示

![](<http://erehmi.github.io/assets/image/taobao-goods-detail.png>)

**知识要点**

1.  垂直方向的ViewPager,
    git: [castorflex/VerticalViewPager](<https://github.com/castorflex/VerticalViewPager>) 

2.  手势拦截 & 处理

**实现思路**

1.  上下分页的设计完全可以用垂直分页来实现, 见**知识要点[1]** 

2.  如果使用垂直分页来实现, 那么问题就来了:
    上下分页中的内容肯定是支持垂直滚动的, 如此就会和ViewPager的手势冲突, 因此,
    上下分页内容的最外层视图(暂且叫作ContentContainer)必须要处理手势,
    即在垂直滚动内容时必须告知ViewPager, 当前状态的ContentContainer是否可滚动.
    p.s.少数View是已经处理了上述手势问题的, 但是, 像ListView,
    ScrollView都是没有处理的.
    具体处理方式可以参考: **android.support.v4.widget.NestedScrollView**

3.  建议: 上页部分可以采用ListView来实现, 扩展性更好.
    不推荐使用ScrollView来实现.

4.  其他UI细节, 不在此赘述.


**END. \>\> SEE MORE:**
[http://erehmi.github.io/](<**http://erehmi.github.io/**>)
