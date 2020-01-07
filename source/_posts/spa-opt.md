title: 提升移动端SPA页面体验探索与实践
date: 2019-12-27
author: 启宇
subtitle: 目前使用的是基于Taro的多端开发方案，只书写一套代码，再通过Taro的编译工具，将源码分别编译出在小程序和H5上运行的代码。
tags: [Taro, 性能]

categories: 移动端
---

目前设计师App上使用SPA的场景越来越多、越来越复杂。目前使用的是基于Taro的多端开发方案，只书写一套代码，再通过Taro的编译工具，将源码分别编译出在小程序和H5上运行的代码。

随着Taro组件库的搭建完成，目前技术场景能够满足现有需求。接下来关注页面的性能及体验

随机找一个SPA页面进行检查：
![企业微信截图_d878c392-5cdd-427c-bf67-ff8181e912af.png](https://cdn.nlark.com/yuque/0/2019/png/697894/1577354821872-d65558cc-cb3b-461c-9dc5-b0c3079a039e.png#align=left&display=inline&height=621&name=%E4%BC%81%E4%B8%9A%E5%BE%AE%E4%BF%A1%E6%88%AA%E5%9B%BE_d878c392-5cdd-427c-bf67-ff8181e912af.png&originHeight=621&originWidth=347&size=56922&status=done&style=none&width=347)![企业微信截图_aeb4b481-8584-410b-8107-4821898e8a70.png](https://cdn.nlark.com/yuque/0/2019/png/697894/1577354846546-37d8df72-9d9f-40b1-88d7-18d06587634e.png#align=left&display=inline&height=530&name=%E4%BC%81%E4%B8%9A%E5%BE%AE%E4%BF%A1%E6%88%AA%E5%9B%BE_aeb4b481-8584-410b-8107-4821898e8a70.png&originHeight=530&originWidth=337&size=38826&status=done&style=none&width=337)![企业微信截图_1356f5aa-aa7e-41c4-81b8-b8f5c96a9987.png](https://cdn.nlark.com/yuque/0/2019/png/697894/1577354856524-8eb75272-bfa1-4af7-8efe-9735d0024874.png#align=left&display=inline&height=547&name=%E4%BC%81%E4%B8%9A%E5%BE%AE%E4%BF%A1%E6%88%AA%E5%9B%BE_1356f5aa-aa7e-41c4-81b8-b8f5c96a9987.png&originHeight=547&originWidth=345&size=64766&status=done&style=none&width=345)

通过检测工具可以看到出辅助功能外，性能（主要是速度）及支持pwa是有待提升的。随着项目中spa页面使用越来越频繁，优化移动web页面在手机上的用户体验正迫在眉睫。


基于目前react技术栈Rax是一个不错的方案。超轻量，高性能，易上手的前端解决方案。一次开发多端运行，解放重复工作，专注产品逻辑，提升开发效率

## Rax
Rax是一个基于 React 写法的跨容器的 js 框架，与目前的React技术栈吻合，具有跨容器、高性能、轻量等特点。整合如下优势：
![企业微信截图_c5007624-e8b2-4ee6-b5df-b308ea154a38.png](https://cdn.nlark.com/yuque/0/2019/png/697894/1577344706513-a8a41051-3b5d-4fc8-b6c9-581d80bf88fb.png#align=left&display=inline&height=187&name=%E4%BC%81%E4%B8%9A%E5%BE%AE%E4%BF%A1%E6%88%AA%E5%9B%BE_c5007624-e8b2-4ee6-b5df-b308ea154a38.png&originHeight=187&originWidth=739&size=28970&status=done&style=none&width=739)

### 符合的特征：
#### JSX
JSX 是一种 JavaScript 的语法扩展，在 Rax 中 我们使用 JSX 来描述页面结构.
#### 生命周期
Rax 的生命周期与 React 中的概念是相同的
1、渲染阶段: componentWillMount、render、componentDidMount
2、存在阶段: componentWillReceiveProps、shouldComponentUpdate、            componentWillUpdate、componentDidUpdate

#### Hooks
支持Hooks

### 使用
```
npm init rax <projectName>
```
初始化项目过程中， 您可以根据提示选择一个或多个需要投放的端:
![企业微信截图_47dcb5f4-3267-4e21-91ba-52e8d47c29a7.png](https://cdn.nlark.com/yuque/0/2019/png/697894/1577346263138-af279268-aeaa-4760-9fa0-5f812f50535e.png#align=left&display=inline&height=242&name=%E4%BC%81%E4%B8%9A%E5%BE%AE%E4%BF%A1%E6%88%AA%E5%9B%BE_47dcb5f4-3267-4e21-91ba-52e8d47c29a7.png&originHeight=242&originWidth=572&size=33455&status=done&style=none&width=572)

spa页面
![企业微信截图_d4b86d2c-f5e1-43ca-89a7-9a0f314ff20e.png](https://cdn.nlark.com/yuque/0/2019/png/697894/1577346312642-b0119179-bc04-4f59-8181-2e1477f7f5b4.png#align=left&display=inline&height=109&name=%E4%BC%81%E4%B8%9A%E5%BE%AE%E4%BF%A1%E6%88%AA%E5%9B%BE_d4b86d2c-f5e1-43ca-89a7-9a0f314ff20e.png&originHeight=109&originWidth=581&size=26847&status=done&style=none&width=581)

服务器端渲染
![企业微信截图_b65ff346-b19f-413b-bbf8-0bebbfc20b53.png](https://cdn.nlark.com/yuque/0/2019/png/697894/1577346398224-40e51922-5a5a-4d47-94f0-564a38ea710e.png#align=left&display=inline&height=170&name=%E4%BC%81%E4%B8%9A%E5%BE%AE%E4%BF%A1%E6%88%AA%E5%9B%BE_b65ff346-b19f-413b-bbf8-0bebbfc20b53.png&originHeight=170&originWidth=570&size=41601&status=done&style=none&width=570)

### 性能
#### Single Page Web Application
Rax内置统一的SPA方案，无需维护Router组件。

对于移动端 SPA 加载性能是我们必须要考虑的问题，一起打包所有页面的资源必然会产生很大的 JS Bundle，在网络环境较差的情况下，容易阻塞资源加载加长等待时间，影响页面的可用性。Rax打包时将页面代码分离到不同的 JS Bundle 中，可得到更小的 JS Bundle，极大影响加载时间，控制资源大小。

#### 消除白屏
App Shell 是指支持页面展示的最小 HTML、CSS 和 JS 的资源集合，内容可为页面结构，页面片段，骨骼图等。它往往是纯 HTML 片段，一般只包括内联 CSS 和 base64 图片，不强依赖于 JS 框架。可以在加载、解析、执行 JS 之前就渲染出来，几乎消除了白屏时间，大大提高用户体验，确保提供即时、可靠的加载交互体验。

Rax中的 App Shell 可使用 JSX 进行开发，它会被提前构建并将渲染结果插入 HTML 中，使其可快速加载到用户屏幕。在 JS 运行时，App Shell 会复用 HTML 中的节点绑定事件并激活组件。

App Shell 的内容可为页面最外层通用可交互结构。比如 TabBar，将其注入 HTML 中，可方便用户快速切换页面进行页面预览,还可以通过骨骼图在白屏位置占位。骨骼图需要在最短的时间内渲染给用户，期望浏览器在加载完 HTML 之后就能先显示骨架屏，所以在 App Shell 中还可以通过骨骼图提升用户体验

#### 快照
用户所处环境网速较差导致 JS 文件加载慢时，白屏时间增长会造成不好的体验。如果在内容变化不大的 Web 应用中可将页面内容填充为用户上一次访问时对应的内容，极速展示页面内容，提升首屏加载速度。

页面会记录当前页面内容称之为快照，下次访问相同页面时，快速将快照填充渲染至当前空白页面，待 JS 加载完成后再进行状态和事件绑定，完成页面渲染。Chrome 弱网调试环境

#### 缓存控制
在 Web 应用中我们可以通过 Service Worker （以下简称 SW） 控制页面缓存，它是 PWA 中最重要的概念之一。它独立于浏览器的主线程运行，不仅可以拦截用户的网络请求，还可以操作缓存，支持 Push 和后台同步等功能

#### 服务端渲染
SSR 是指将页面渲染逻辑前置到服务器端执行，由服务器端直接返回渲染好的页面，然后再由浏览器端进行状态和事件绑定，来达到页面的可交互状态。服务端控制数据请求可彻底摆脱弱网环境，将数据请求网络延迟减少到了最小程度。

#### 支持PWA
无需打开浏览器并输入 URL 地址访问 Web 应用，通过点击保存至桌面的PWA，用户可快速定位并启动应用。不仅支持全屏预览应用，还能定制 PWA 的启动画面的图标和颜色等，增强 Web 应用与操作系统的集成能力，提供更好的类 Native 体验


可以看到Rax对于移动Web页面的体验支持上还是很丰富的，涵盖了所有可以优化的点，并且在很多项目中实践。是一个不错的方案。参考Rax实现方案及日常项目中的总结，对目前SPA页面针对性能上总结目前项目中实践的优化。


### 优化实践
#### 资源优化
减少HTTP请求，因为手机浏览器同时响应请求为4个请求（Android支持4个，iOS 5后可支持6个），所以要尽量减少页面的请求数，首次加载同事请求数不能超过4个

所有Taro发布的页面资源，都要经过uglify压缩。减少资源大小可以加快网页显示速度，所以要对HTML、CSS、JavaScript等进行代码压缩。写在HTML头部的JavaScript（无异步），和写在HTML标签中的Style会阻塞页面的渲染，因此CSS放在页面头部并使用Link方式引入，避免HTML标签中写Style,JavaScript放在页面尾部或使用异步方式加载

#### 首次加载优化
按需加载

1、将不影响加载的资源和当前屏幕资源不用的放到用户需要的时候才加载，可以大大提升重要资源的显示速度和降低总体流量。
2、使用Taro组件提供的LazyLoad和滚屏加载，实现图片等资源的懒加载
3、在自定义的Taro的组件库中，实现骨架屏（占位）组件。防止页面白屏

压缩图片，图片是目前项目中占流量最大的资源，多个页面会因此尽量使用最合适的格式和大小。

1、选择合适的图片(1. webP优于JPG 2. PNG8优于GIF)
2、图片压缩及使用其它方式代替图片(1. 使用CSS3 2. 使用SVG 3. 使用IconFont)
3、选择合适的大小（1. 首次加载不大于1014KB 2. 不宽于640（基于手机屏幕一般宽度）

#### 渲染优化
HTML使用Viewport,可以加速页面的渲染，使用
```
<meta name=”viewport” content=”width=device-width, initial-scale=1″>
```
减少Dom节点，Dom节点太多影响页面的渲染，尽量减少节点

#### 数据预加载
数据预加载是将页面中需要请求的数据提前，项目中可能存在多个数据请求，可以让后端合并多个数据请求，答复缩短用户看到页面的有效数据的等待时间，提高用户体验。

综合以上优化点，项目中新的SPA页面加载有了很大的提升，旧页面将持续改造。后续将持续关注业界优秀的跨平台方案，继续探索与实践任何可以优化移动Web页面体验的方案，提升用户体验

