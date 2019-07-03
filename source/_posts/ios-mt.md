title: iPadOS Multitasking：理论
date: 2019-06-30
author: 肥皂
subtitle: iPadOS 绝对是 WWDC19 中最让人雀跃的消息之一。iPad 将在全新的操作系统中释放出更加强大的生产力。`多任务（Multitasking）`便是 iPad 一直以来独有的特性，在这次 iPadOS 的升级中也被赋予更强大的能力。
tags: [iOS，多任务]

categories: iOS
---
# 简介

iPadOS 绝对是 WWDC19 中最让人雀跃的消息之一。iPad 将在全新的操作系统中释放出更加强大的生产力。`多任务（Multitasking）`便是 iPad 一直以来独有的特性，在这次 iPadOS 的升级中也被赋予更强大的能力。


![](https://qhyxpicoss.kujiale.com/2019/07/03/LUOE7JYKAQBZM2NSAAAAACQ8_2032x1169.png)
![](https://qhyxpicoss.kujiale.com/2019/07/03/LUOFGNYKAQBZOISIAAAAABQ8_2032x1169.png)


作为 iPad 端的移动设计工具，我们开始着手适配多任务。

多任务是 iOS 9 中引入的特性。多任务在 iOS 上有三种表现形式，分别是临时出现和交互的`滑动覆盖（Slide Over）`，真正的分屏同时操作两个 app 的`分割视图（Split View）`，以及在其他 app 里依然可以进行视频播放的`画中画（Picture in Picture）`模式。

而在启用分屏的同时，我们也不得不让 app 支持设备的所有朝向，这同样也会引起布局上的一些变化。

![](https://qhyxpicoss.kujiale.com/2019/07/03/LUOE7JYKAQBZM2NSAAAAACY8_1600x384.jpg)


针对设备的多个朝向以及多任务所带来的眼花缭乱的尺寸问题，Apple 从 iOS 8 开始引入 `Size Class` 体系。Apple 将自家的移动设备按照尺寸区别，将水平和竖直两个方向设计了 `Regular` 和 `Compact` 的组合。比如目前 iPhone 在竖屏时宽度是 Compact，高度是 Regular；横屏时 iPhone XS Max、iPhone XR 和 Plus 机型宽度是 Regular，高度是 Compact，而其他 iPhone 在横屏时高度和宽度都是 Compact；iPad 不论型号和方向，宽度和高度都是 Regular。

同样，面对多任务带来的各种尺寸组合情况，iOS 同样通过 Size Class 来处理。可以在这里看到目前所有的 Size Class。在我们开发的过程中，我们也不再关心具体设备的型号、尺寸、屏幕旋转和分屏等情景，而是根据指定的 Size Class 特性来呈现内容。


![Xcode 中的 Size Class 选项，可以看到 wC，hR](https://qhyxpicoss.kujiale.com/2019/07/03/LUOFGNYKAQBZOISIAAAAABY8_536x151.png)


# 为当前页面找些参照
通过参考一些系统 app 和比较知名的第三方 app 的适配手段，为我们的 app 提供一些思路。这里不会展示适配页面的具体视觉。

基于我们移动设计工具的特性，所有页面可以简单分为 1) 工具外，2) 工具内。

## 工具外
工具外的相当一部分页面是大同小异的列表页，例如方案列表，样板间列表，渲染图，商品清单，户型搜索等。这类页面需要制定每行个数随屏宽变化的阈值，间距的规则。

顶部栏（navigation bar），底部栏（tab bar）系统会有默认的处理方式，如果是 RN 或 Web 页，参考原生的行为即可。

各类筛选器需要支持左右滑动，以避免显示不完全的问题。

![](https://qhyxpicoss.kujiale.com/2019/07/03/LUOE7JYKAQBZM2NSAAAAADA8_800x498.png)

同时，值得注意的还有：

1. 所有的 popover 在 compact 上应被替换为 action sheet。如果选项非常多，它甚至可以是一个模态页面（e.g. 方案左上角的 account setting 入口）；
2. 所有的 form sheet 在 compact 上应被替换为模态页面（e.g. 新建户型、CAD 导入）；
3. 我们的弹窗基本没有采用系统原生的 alert。弹窗的尺寸较大且内容多种多样（重命名，删除，分享），这些弹窗需要一份适配的规则。


![](https://qhyxpicoss.kujiale.com/2019/07/03/LUOFGNYKAQBZOISIAAAAACA8_2388x1668.png) ![](https://qhyxpicoss.kujiale.com/2019/07/03/LUOE7JYKAQBZM2NSAAAAADI8_2388x1668.png)

>GoodNotes 中编辑单个 Note 的 popover，在 compact 下变更为 action sheet

![](https://qhyxpicoss.kujiale.com/2019/07/03/LUOFGNYKAQBZOISIAAAAACI8_2388x1668.png) ![](https://qhyxpicoss.kujiale.com/2019/07/03/LUOE7JYKAQBZM2NSAAAAADQ8_2388x1668.png)

>Keynote 中 add 的 popover，在 compact 下变更为全屏模态页面

## 工具内
各个设计类 app 对于工具页面本身采取的做法各不相同。这里列举一部分供参考。

1. 比较粗暴的照搬型：`AutoCAD`，`Concepts`；
2. 优化功能入口 / 精简部分功能：`Notes`，`Procreate`；
3. 禁止绝大部分功能: `SketchBook`。

![](https://qhyxpicoss.kujiale.com/2019/07/03/LUOFGNYKAQBZOISIAAAAACQ8_2388x1668.png) ![](https://qhyxpicoss.kujiale.com/2019/07/03/LUOE7JYKAQBZM2NSAAAAADY8_2388x1668.png)

>强大到不需要适配的 AutoCAD，右侧的 Blocks 栏和我们的模型库非常相似，在 regular 和 compact 上毫无差别。其他工具栏似乎也没有什么变化

![](https://qhyxpicoss.kujiale.com/2019/07/03/LUOFGNYKAQBZOISIAAAAACY8_2388x1668.png)

>Concepts，也没有做特殊处理

![](https://qhyxpicoss.kujiale.com/2019/07/03/LUOFGNYKAQBZOISIAAAAADA8_2388x1668.png) ![](https://qhyxpicoss.kujiale.com/2019/07/03/LUOE7JYKAQBZM2NSAAAAAEA8_2388x1668.png)

>Procreate，部分功能归类为一个整合入口

![](https://qhyxpicoss.kujiale.com/2019/07/03/LUOE7JYKAQBZM2NSAAAAAEI8_2388x1668.png)

>SketchBook，剔除了所有操作，只能预览

当处理我们的工具页面时，或许需要综合各种策略。同时也可以从其它非工具类 app 的适配方式中汲取灵感。在必要的情况下，考虑禁用部分功能。

![](https://qhyxpicoss.kujiale.com/2019/07/03/LUOFGNYKAQBZOISIAAAAADI8_2388x1668.png) ![](https://qhyxpicoss.kujiale.com/2019/07/03/LUOFGNYKAQBZOISIAAAAADQ8_2388x1668.png)

>Maps，原先位于左侧的信息搜索内容，在 compact 下放置于底部，类似于控制中心一样可以被拽起。模型库的适配可以参考

# 前进！
目前，移动设计工具已经就部分页面完成了适配视觉与界面交互逻辑。我们也在如火如荼地开发当中。眼花缭乱的列表页如何迎接多种尺寸？不同 Size Class 的页面的控件如何丝滑切换？显示不下的内容如何让它“服服帖帖”？分屏的适配还会遇到什么陷阱？

敬请期待——`《iPadOS Multitasking：实战》`。

# 引用与参考
[Apple Human Interface Guidelines: Multitasking](https://developer.apple.com/design/human-interface-guidelines/ios/system-capabilities/multitasking/)

[Apple Human Interface Guidelines: Adaptivity and Layout](https://developer.apple.com/design/human-interface-guidelines/ios/visual-design/adaptivity-and-layout/)

[Apple Support Issue](https://support.apple.com/en-us/HT207582)

[Apple WWDC15 Session 205](https://developer.apple.com/videos/play/wwdc2015/205/)

[Apple WWDC15 Session 211](https://developer.apple.com/videos/play/wwdc2015/211/)

[Apple WWDC15 Session 212](https://developer.apple.com/videos/play/wwdc2015/212/)