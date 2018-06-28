title: 小程序开发十问
date: 2018-05-11
author: 冬青
subtitle: 小程序有着自己的开发模式，新手入门免不了需要各种自己摸索，本篇文章就带领大家入个门，看看小程序的一些基本开发形式。
tags: [小程序, 基础]

categories: 小程序
---

![](https://user-gold-cdn.xitu.io/2018/5/11/1634fc24425fe797?w=734&h=381&f=jpeg&s=58819)

### 第一问：小程序页面间如何传递数据？
A 跳转到 B 时，可以通过 url 中 query 传递数据。

B 页面 onLoad(options) 方法中的 options 会包含 query 中的 key-value 的内容。

如果需要传递如 json 或数组这样的结构化数据，我们可以先把结构化数据做 string 化和 encode 一下后，再通过该方式传递。
```
encodeURIComponent(JSON.stringify(xxx))
```

在 B 页面中，获得内容后，通过以下方法，解析出数据。
```
JSON.parse(decodeURIComponent(xxx))
```
### 第二问：页面间如何回传数据?
比如 A 打开了 B，B 中一些数据需要传送到 A。可以先获取前一个页面实例，然后直接调用前一个页面方法进行数据传输。
```
const pages = getCurrentPages();
const prevPage = pages[pages.length - 2];
prevPage.methodOfPrevPage(data);
```

### 第三问：小程序如何与服务端保持会话？
因小程序框架并无 Cookie 管理机制，并且小程序也未提供向 WebView 设置 Cookie 的方法。所以如果我们想继续使用 Session-Cookie 机制，则需要自己实现一套，我们可以简单的提取出 set-cookie 头中有效的 cookie 内容，然后存储在内存和本地中，在下一次请求的时候，把这些 cookie 组装起来使用。当涉及到 WebView 时，我们可以通过 query 的方法，把这些 Cookie 内容传给 Web 端，用来维持和服务端的有效会话。

当然你也可以采用 Token 机制，与服务端保持会话。

### 第四问：如何调用子组件中的方法？
我们可以在自定义组件中加上一个 id，然后在 js 代码中使用如下方法：
```
this.selectComponent('#id').methodOfComponent(data);
```

### 第五问：子组件中如何调用父组件的方法？
使用组件事件方法，详细文档：[《组件事件文档》](https://developers.weixin.qq.com/miniprogram/dev/framework/custom-component/events.html)

在父组件中使用子组件时，可以定义一个
```
bind:customMethod='parentMethod'
```

然后子组件中，可以使用以下代码调用父类的方法
```
this.triggerEvent('customMethod', data);
```


### 第六问：小程序如何进行数据分析？
小程序后台提供了数据分析能力，具体可见：[《小程序数据分析文档》](https://developers.weixin.qq.com/miniprogram/analysis/)

并且如果需要把数据接入到自己的服务中，也可以通过调用微信接口的方式拿到数据：[《小程序数据分析接口文档》](https://developers.weixin.qq.com/miniprogram/dev/api/analysis.html)

如果需要自定义数据，我们可以在小程序中调用方法：
```
wx.reportAnalytics(eventName, data)
```

不过在使用前，需要在小程序管理后台自定义分析中新建事件，配置好事件名与字段。另外自定义事件的数据无法通过接口获得，

所以如果你需要在自己的服务器上也分享自定义事件，那只能自己开发几个接口了。

### 第七问：微信小程序的二维码生成有次数限制吗？
微信提供了三种方式生成微信二维码，详情可查看：[《小程序二维码相关文档》](https://developers.weixin.qq.com/miniprogram/dev/api/qrcode.html)

此三种类型二维码都需要服务端端通过 access_token 调用微信接口生成。并且其中 接口B仅能生成已发布的小程序页面的二维码，所以你的小程序先得上线后才能测试该功能。有点坑。

其中接口A、和接口C有次数限制，接口A加上接口C，总共生成的码数量限制为100,000。

接口 B 次数无限制，但调用频率有限制，5000次／分钟。

接口 A 和接口 C 相对接口 B 可以传入一个最大长度不超过 128 字节的 path，你可以在 path 中通过 query 的形式传入参数。

接口 B 相对 A、C，把 PATH 拆成了，page 和 scene，其中 scene 最大为 32 个字符。可以在 page 的 onLoad 方法中通过 options.scene 方式获得这个 scene。

### 第八问：普通二维码可以打开小程序吗？
可以，需要在小程序管理后台添加，添加后，即可扫描以下内容的二维码就可跳转到小程序的指定页面了。

详情可查看：[《小程序普通二维码文档》](https://developers.weixin.qq.com/miniprogram/introduction/qrcode.html)


![](https://user-gold-cdn.xitu.io/2018/5/11/1634f9c00dbf361a?w=1137&h=250&f=png&s=33232)

### 第九问：小程序版本的兼容情况如何？
小程序运行在微信上，并且小程序的基础库随微信版本而发版。所以不同的微信版本会对小程序的表现有所影响。
有关各个版本的基础库的覆盖率可以查看以下链接。
[《小程序基础库版本介绍》](https://developers.weixin.qq.com/miniprogram/dev/framework/client-lib.html)

目前微信推荐的最低基础库版本，可以覆盖 80% 以上的微信用户。另外低版本的微信在使用使用高基础库版本的小程序会提示升级微信。

### 第十问：小程序代码可以运行在浏览器中吗？
小程序使用的是自己的一套框架，只是借用了目前主流的 html + js + css 的开发形式，所以小程序代码本身是无法直接运行在浏览器中的。

目前美团开源了一套自己的方案：[mpvue](https://github.com/Meituan-Dianping/mpvue) ，使用 vue 的形式来编写小程序。并且可以通过改变打包配置的方式，让同一套代码可以同时运行在小程序和浏览器中。

> 最近在公司做了几款小程序，对小程序开发有了一些经验，如果你对小程序开发有更多疑问，可以加入小程序开发者交流微信群，一起交流学习。
> ![](https://user-gold-cdn.xitu.io/2018/6/16/164063c67855ac65?w=1080&h=1578&f=jpeg&s=61627)