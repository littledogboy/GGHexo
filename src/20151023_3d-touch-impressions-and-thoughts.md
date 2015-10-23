title: "3D Touch之我见"
date: 2015-10-23 09:00:00
tags: [Tomasz Szulc]
categories: [3D Touch, Swift 入门]
permalink: 3d-touch-impressions-and-thoughts

---
原文链接=http://szulctomasz.com/3d-touch-impressions-and-thoughts
作者=Tomasz Szulc
原文日期=2015-09-26
译者=saitjr
校对=pmst
定稿=千叶知风

Hey，现在有了支持3D Touch的iPhone和iOS9，这是一项伟大的科技杰作。让我们看看这为开发者和用户都带来了什么呢。本文主要是我对3D Touch的理解，然后列举了一些需要注意的点。

<!--more-->

## 出师不利

本来我想写一篇入门级关于3D Touch的导读文章，但是遇到一个问题——Xcode7的模拟器并不能模拟3D Touch。

苹果官方的说法是：

> Xcode7.0中，你必须使用支持3D Touch的真机进行开发。因为Xcode7.0的模拟器不支持3D Touch。

这是个悲伤的故事，但是又有什么办法呢。我想，苹果应该会想办法模拟出来。可能在下一个Xcode发布版本中就能看到。

另一个问题则是新的iPhone还没有在波兰发售，可能还要等2周（译者注：作者写文在波兰iPhone发售以前）。

还有一个办法，我10月要在San Francisco和Palo Alto呆上整整一月，所以我可能会买一部iPhone，但不确定是6s还是6s plus。

如果买了设备，那我肯定会发布一大波3D Touch的文章。

## Peek and pop操作

这是苹果介绍3D Touch的第一个特性——或者说是两个特性。

-   轻压(peek,轻度按压)操作能让用户在不离开当前界面的情况下预览内容。如果轻按某个选项能够弹出一个小的矩形视图，则表明它支持轻压操作。
-   peek手势弹出的视图应该足够大，这样内容就不会被手指挡住，从而用户可以选择是否重压，即pop操作（译者注：peek操作即轻按，用于预览，pop操作则是重压，用于进一步确认。pop能满足在预览之后，进入特定页面的需求）。
-   pop则是当用户在peek弹出的视图上加重按压力度，显示更详细的内容。
-   即使peek能给用户足够多的信息，你也应该让用户将该操作转化为pop。pop所显示的界面应该和用户点击进入界面相同。
-   请勿在peek预览的界面中放入按钮，因为用户手指一离开预览界面，预览界面就会消失，所以根本点不到按钮。
-   当用户在预览界面中向上滑动时，peek可以提供一些快捷操作(quick action)。你可以在预览界面中添加一些快捷操作，这样用户就可以上滑，然后选择一个你所提供的操作了。
-   如果你为某个选项提供了长按手势事件(touch-and-hold，或者叫long press)，那么你可以用peek来代替长按手势，这是一个很好的尝试。
-   如果你想使用快捷操作、peek和pop，那么记住在使用前先判断3D Touch是否可用。
-   不是每个设备都支持peek和pop操作的，并且3D Touch也可能处于禁用状态。所以不要让某些事件只能由peek来触发。最好有一个备选方案，即视图也能通过长按手势来展示。

## 快捷操作

接下来，介绍一些用户在重压应用图标时的一些快捷操作。

-   弹出框包含了主标题、副标题与图标。
-   该操作可以在应用程序更新时，显示更新信息。
-   你的应用在Home界面中，应该至少提供一个快捷选项。这样用户就可以使用手势操作你的应用，方便快捷。
-   医用最多可以提供四个快捷操作。
-   不要使用快捷操作来提醒用户更新、变更之类的事。如有需要，通知(*Notifications*)更能胜任这些任务。
-   快捷操作的命名应该简洁，如有需要，还应该有副标题和图标。尽量让用户明确该操作的作用。如果提供了副标题，那么标题栏将会更长，如果大小不能适应，系统会自动截取。如果没有副标题，那么主标题过长会自动换行。

## 总结

3D Touch是一个非常爽的特性，它提供了全新的交互方式。设备中包含了一个线性振动器(Taptic Engine)，所以按压屏幕时，设备能有一定响应——太好了，迫不及待想要试试。

非常遗憾，苹果没有在最新的Xcode 7 beta 2中解决无法模拟的问题，还是希望他们尽快搞定吧——或许他们不修复是为了增加销量，因为用户已经迫不及待的想要试试这个新特性了，而最简单粗暴的方式，就是买个新设备 :D。

我已经迫不及待的想要尝试基于3D Touch的应用了，如游戏和画图，还有一些我想象不到的应用。或许还可以看到计量重量的应用，比如称一个水果的重量，或是一个放置在屏幕上的物体的重量等等 :>。