title: React-Native 升级到 0.57.0 遇到的问题
date: 2018-10-29
author: 海豹
subtitle: 最近把 RN 从 0.51.0 升级到 0.57.0 后，发现项目跑不起来了。摸索了半天终于解决了问题，这里做个记录。
tags: [React-Native, 坑]

categories: React-Native
---
移动端在酷家乐是一个重要入口，为了更快的进行 iOS 和 Android 端的迭代，我们采用了 React-Native 的解决方案。踩了很多坑，最近把 RN 从 0.51.0 升级到 0.57.0 后，发现项目跑不起来了。摸索了半天终于解决了问题，这里做个记录。

# 升级

- 升级 react native 到 0.57.0 ，升级 react 到 16.5

- 修改 babel-preset 依赖从 "babel-preset-react-native": "^5" 改为 "metro-react-native-babel-preset": "^0.45.0"，并更新.babelrc文件

``` json
{
"presets": ["module:metro-react-native-babel-preset"]
}
```

- 增加 @babel/plugin-external-helpers

- 增加 schedule@0.4.0

升级完上述依赖之后可能还会碰到 babel runtime 报错 此时需要增加"@babel/runtime": "^7.1.2"。

Android 还需要升级 native 的一些工具，按文档上的升级就行了，没什么大坑，这里不详细讲，参考[官方文档](https://github.com/react-native-community/react-native-releases/blob/master/CHANGELOG.md#057)。

## iOS 升级额外的一些坑


由于我们使用Cocoapod作为第三方库管理器，所以RN也是采用pod的方式集成在项目中，升级中发现一些依赖的问题。

- 在 podfile 中使用 CxxBridge 替换 BatchedBridge(0.45以后的 bridge 开始使用了 RCTCxxBridge, 之前都是RCTBatchedBridge , 而在0.54版本 RCTBatchedBridge 已经被废弃)

- 如果使用的是CxxBridge，则加入下面三个第三方编译依赖

``` JavaScript
pod 'DoubleConversion', :podspec => '/path/to/ReactNative/node_modules/react-native/third-party-podspecs/DoubleConversion.podspec'

pod 'glog', :podspec => '/path/to/ReactNative/node_modules/react-native/third-party-podspecs/glog.podspec'

pod 'Folly', :podspec => '/path/to/ReactNative/node_modules/react-native/third-party-podspecs/Folly.podspec'
```

- 需要将项目支持的最低版本提升至 iOS 9.0 以上

# 填坑

### 全局修改组件的 defaultProps

之前iOS通过设置
```JavaScript
Text.defaultProps.allowFontScaling = false;
```

从而使Text组件的字体大小不受系统设置的影响，但是在升级之后这么写会报错，正确的写法如下
```JavaScript
Text.allowFontScaling = false;
```

### react-native-scrollable-tab-view

升级完之后运行项目发现还是报错，原因是项目中用到了 react-native-scrollable-tab-view 这个三方库，代码中多了个逗号，去掉就行了。
> const {shouldUpdated, ...props, } = Props;

但是多人合作的项目中这么做不合适。目前这个库还没有发布新版本修复。一种做法是 fork 这个项目自己修改后再依赖修改后的库。查看这个库的 github，果然发现有人提 issue。实际上这个 bug 已经在 master 分支上已经修复，我们可以切换到 master 分支上

``` json
"react-native-scrollable-tab-view": "git+https://github.com/happypancake/react-native-scrollable-tab-view.git"
```

这里还有吐槽一下这个库，提供的 ScrollableTabBar 还有好几个 bug 没修，不知道下个 release 会不会一起修掉。

### cookie 失效

修完上面的 bug 之后项目终于可以正常运行，但是这时候出现了更大的坑，cookie 失效了。回滚新版本增加的 feature 代码，问题任然存在， 回退 rn 版本到 0.51.0 问题消失，升级到 0.57.0 问题出现，所以可以断定是高版本 rn 的问题。继续查阅 github 上的 issue 发现从 0.56.0 版本开始就有这个问题。解决方案是 在 fetch 的时候加上 credentials: 'include'

``` JavaScript
fetch(route, {method: 'GET', headers, credentials: 'include'})
```

加上之后解决 cookie 失效问题，踩坑之旅暂时告一段落。

# 总结

RN 开发相对于原生开发方便了很多，但是目前还是有很多坑，不过好在 RN 社区还算活跃，大部分问题都能解决。比如这次升级，碰到了很多意想不到的坑，好在 google 了一圈也都能够解决。

# 参考文献

- [How To Upgrade Project to React-Native 0.57 (RN 0.57)](https://medium.com/@oleg2014/how-to-upgrade-project-to-react-native-0-57-rn-0-57-1a7e9fd8098)

- [React Native changelog](https://github.com/react-native-community/react-native-releases/blob/master/CHANGELOG.md#057)

- [Cookie-Based Authentication Is Not Possible](https://github.com/facebook/react-native/issues/19958)

- https://github.com/happypancake/react-native-scrollable-tab-view/issues/937