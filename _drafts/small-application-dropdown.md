---
title: 小程序下拉刷新之定制版
date: 2018-05-08 14:59:23
tags:
- 小程序
- 组件
- 下拉刷新
---

最近在捣鼓小程序

准备把twitter H5所能看到的交互体验尽可能的在小程序里面还原一遍，来一个微信小程序版的twitter，当然twitter中可圈可点的用户体验细节吾等小辈无力洞悉，里面各种性能优化与数学原理我未能涉足，这里只模仿一些表面上的交互，学习学习小程序，玩玩而已。
<!-- more -->
在去体验twitter的下拉刷新时，只能用一个词来形容：完美！

![完美](http://asset.dawiwt.com/images/20180508161112.gif)

如果要多用两个词来形容：自然、流畅！

它是这个样子的：

![下拉刷新](http://asset.dawiwt.com/images/twitter-pull-down-refresh.gif)

好，感受一番之后，我们来分析具体需求：

1. 滚动区域滚动条距顶部为0时，继续下拉，触发下拉刷新功能
2. 滚动区内容跟随手指滑动
3. 下拉距离60像素以内，显示向下箭头，此时若离开屏幕，滚动区恢复原状
4. 下拉距离大于60像素，显示向上箭头，此时若离开屏幕，滚动区回弹至60像素，显示加载中动画，请求接口
5. 接口返回数据，执行渲染函数，滚动区恢复原状
6. 在一次下拉刷新为加载中时，反复下拉，不会重复请求接口

觉得文字多不好理解，不如来看流程图：

```flow
st=>start: Start|past:>http://www.google.com[blank]
e=>end: End:>http://www.google.com
op1=>operation: My Operation|past
op2=>operation: Stuff|current
sub1=>subroutine: My Subroutine|invalid
cond=>condition: Yes
or No?|approved:>http://www.google.com
c2=>condition: Good idea|rejected
io=>inputoutput: catch something...|request

st->op1(right)->cond
cond(yes, right)->c2
cond(no)->sub1(left)->op1
c2(yes)->io->e
c2(no)->op2->e
```

分析完需求后，我们来一一实现它。

记得微信小程序官方有下拉刷新的功能，我赶紧去学习了一下，但是学习过后，跟我的需求一比，有所失望，这完全不匹配啊；不过想想，这也不能怪小程序官方，毕竟这是通用功能，满足各种交互需求会给团队增加不少的工作量和不可控因素，就像浏览器的下拉框一样，只提供基本功能，想要美化它，必须通过一些特殊的手段来处理，而对于小程序的下拉刷新来说，也是这个道理吧。

那么我们来看一下官方下拉刷新是怎么用的：

1. 在app.json中开启小程序下拉刷新功能
```javascript app.json https://developers.weixin.qq.com/miniprogram/dev/framework/config.html#window docs
{
    window: {
        // 配置主题色，默认dark
        // light: 浅色
        // dark: 深色
        backgroundTextStyle: 'light',
        enablePullDownRefresh: true
    }
}
```
2. 在页面相关的js文件中，绑定下拉刷新的处理函数
```javascript index.js https://developers.weixin.qq.com/miniprogram/dev/framework/app-service/page.html docs
Page({
    onPullDownRefresh: function(){
        // Do something when pull down.
        // 数据处理完成，停止下拉刷新
        wx.stopPullDownRefresh()
    }
})
```

至此，官方给出的下拉刷新功能就配置完成了，效果如下：

[官方下拉刷新效果图](http://www.baidu.com)

哇，简直不能再简单了！但效果跟我们要求的相差太远，既然如此，作为一个爱折腾的程序员，不如，自己动手写一个。

开始吧！

对于这个需求，我将它的交互行为划分为三个主要部分：何时开始、持续计算、如何结束；
<!--stackedit_data:
eyJoaXN0b3J5IjpbNzExMDY0MjUzLC0xMjIyOTkzMTczXX0=
-->