title: 修复 Xcode 升级到 10.x 后 App 随机崩溃问题
date: 2018-11-12
author: 武松
subtitle: 最近我们新版 App 上线后，突然出现大量崩溃记录，在调试修复时，发现遇到了苹果最坑的 Xcode 更新，有两个会导致崩溃的 Bug。
tags: [iOS, 坑]

categories: iOS
---
最近我们新版App上线后，突然出现大量崩溃记录，在调试修复时，发现遇到了苹果最坑的Xcode更新，有两个会导致崩溃的Bug。


# 一、performSelector 返回值出错

performSelector 是 Objective-C 的常用黑科技，在RN调用原生功能时起很大作用。但是在 Xcode 10 和 iOS 12.0 中，如果一个函数返回类型是 void，那么 performSelector 的返回值不是 nil，而是传入的参数；而在部分旧版本系统中，比如 9.0，返回值不可观测，如果尝试将返回值赋值给某个对象，则会崩溃。
真是个莫名其妙的 Bug，解决方法也很简单，先判断返回值类型，决定是否要获取返回值。代码如下：
```objectivec
NSString *selectorStr = @“FuncName”;
if ([self respondsToSelector:NSSelectorFromString(selectorStr)]) {
    Method m = class_getInstanceMethod([self class], NSSelectorFromString(selectorStr));
    char ret[256];
    method_getReturnType(m, ret, 256);
    if (*ret != 'v') {
        return [self performSelector:NSSelectorFromString(selectorStr) withObject:parameter];
    } else {
        [self performSelector:NSSelectorFromString(selectorStr) withObject:parameter];
    return nil;
}
```

XCode 10.1 和 iOS 12.1 中已经正确返回 nil了。


# 二、系统版本为 9.x 的设备，运行 AppStore 上下载的 App 时随机崩溃

这个是这次最坑的 Bug，线下测试、beta 测试都是正常的，只有苹果处理后，在TestFlight 或 AppStore 下载使用才会崩溃，而且必须是 9.x 的系统。
在修复这个Bug中，首先我们搜到了一篇 [《关于iOS 9.2.1 从App Store下载出现不规则崩溃的问题》](https://www.jianshu.com/p/4bc0c5b3b597)，由于表现一致，一度以为就是这个原因，也参考着写了一个脚本获取所有图片描述，因为我们有多个模块，所以需要遍历所有Asset.car：
```bash
#!/bin/bash
DIRECTORY=$1
find "$DIRECTORY" -name 'Assets.car' -print0 | while read -d $'\0' file; 
do 
echo "---------$file"
xcrun --sdk iphoneos assetutil --info "$file" >> ./Assets.json
done
```
结果并没有找到 P3 的图片。尝试将所有 PNG 图片全部转一遍后，发现在 iOS9.1 上还是会随机崩溃，并不是图片的问题。

<br>
继续进行搜索，发现了有人在 StackOverflow 上提出了同样的问题 [《our app crashed in iOS 9 which upload by Xcode 10》](https://stackoverflow.com/questions/52364231/our-app-crashed-in-ios-9-which-upload-by-xcode-10)，然后翻了下 Xcode 10.1 的 Release Note。
![Xcode Release Note](https://qhyxpicoss.kujiale.com/2018/11/12/LPUOLAIKAQBZODD3AAAAAAA8_1022x298.png?x-oss-process=image/resize,m_lfit,w_640)

之前为了使用 React-Native 的新特性，我们将 React-Native 升级了新版本，详见[《React-Native 升级到 0.57.0 遇到的问题》](https://kujiale-mobile.github.io/2018/10/29/rn-upgrade/)，Deployment Target 也跟着 RN 升级到了 9.0。Xcode 也因为 iOS12 的发布，升级到了最新，刚好赶上了这个 Bug，导致崩溃率陡升。
最终由于无法调试，我们只能使用 Xcode 9.4.1 重新打包上传，暂时规避这个 Bug。iOS 的封闭导致对于这种系统问题，能做的事不多，只能希望苹果能尽快修复这个 Bug。


<br>
<br>
PS: 1 分差评能帮我们删了吗...
![1 Star](https://qhyxpicoss.kujiale.com/2018/11/12/LPUN5UIKAQBZMUQCAAAAACY8_679x222.png?x-oss-process=image/resize,m_lfit,w_320)