title: 小程序分享图片生成方案实现
date: 2018-07-07
author: 生忘
subtitle: 一套名为Painter的工具， 为开发者提供一种简单实用的“绘制”图片的解决思路，让开发者可以自由地生成自己想要的图片文件。
tags: [小程序, 组件]

categories: 小程序
---

![](https://user-gold-cdn.xitu.io/2018/7/9/1647d4f9b1068841?w=1000&h=613&f=jpeg&s=70128)

在小程序界里，生成图片分享到朋友圈这个功能，是如此得光芒耀眼，以至于各个小程序都趋之若鹜地前来跪倒在她的石榴裙下。不幸的是，微信爸爸并没有提供给我们很好很便捷的相关工具；恰恰相反，屏幕截屏的功能被残忍丢进历史的垃圾桶，只留下一个Canvas组件以及围绕在其周围的深渊巨坑们。

所以我们准备了一套名为Painter的工具， 为开发者提供一种简单实用的“绘制”图片的解决思路，让开发者可以自由地生成自己想要的图片文件。

github传送门：https://github.com/Kujiale-Mobile/Painter

如果直接使用canvas进行绘图，那绝对是很酸爽的一次体验，除了失控的代码，还有无数的天坑。先来列举一下canvas 中踩过的坑以及我们的解决（或绕过）的方法。

### canvas的坑

painter从实现上来讲，是用了小程序的canvas作为载体来实现以上功能的。而canvas有很多著名的坑。有的坑，我们小心翼翼地绕了过去；有的坑，我们还是痛快淋漓地一脚踩了下去……

1. 在微信版本6.6.6的某些ios机型上，canvas的clip()方法不能被restore。导致在这些机型上无法进行切圆角的操作。迫于无奈在开发中我们不得已抛弃了这些机型，用了一个if语句将这些机器的切圆角功能阉割了。。。
2. 小程序的canvas提供了measuretText()方法，暂时只支持测量文本宽度，无法知道文字的具体高度。因此一些元素对齐的需求无法做到很漂亮。
3. 在绘制图片的时候，有几率会发生很神奇的表现，即canvas绘图的时候位置出现整体偏差，造成最后生成的图片有残缺。这种情况大多数时候发生在onLoad中调用painter的情况下。我们处理的方法是对图片的宽和高比例进行检测，一旦出现异常，就重新绘制一遍。
4. canvas不能绘制网络图片。canvas.drawImage(url)方法，给url传入一个网络链接，在模拟器上表现完美，然而在真机上无法绘制。我们在Painter中引入了一套自己的网络图片下载后绘制的机制，并在其中加入了LRU存储管理机制。
5. canvas是原生组件，始终位于视图的最上层，z-index设置对其无效。这个就不多说了。。很多人应该都踩过。
6. canvas要进行绘制，则canvas组件必须真实地被写在页面上，而且其wx:if不能为false。不过，允许把canvas组件放置在屏幕之外，如设置position:fixed;left:750rpx;。这一方法是可以解决5，6两点问题的黑科技

### Painter的功能

![](http://sinacloud.net/music-store/markdownPic/painter1.gif?KID=sina,2o3w9tlWumQRMwg2TQqi&Expires=1579069535&ssig=QOPvSJJ3AH)

如图所示

通过右边的类似于css又有点像json但其实上它是个js的寥寥几行代码，我们绘制出了左边的这样的图形，包含了背景图片、文字、图片、二维码这四种常用的元素。

Painter阅读完代码，绘制成图片以后，会将图片的链接返回给我们。此时，我们可以将图片上传、保存到本地或者显示在屏幕上。

它可以很方便地定制所需要的图片，还可以自由动态地给图片更换风格。

此外，小程序canvas.drawImage()方法在真机上不能绘制网络图片。而Painter 可以解决这个问题，如果有绘制网络图片的需求也可以考虑使用Painter。

### Painter其它优势

1. painter可以下载网络图片到本地，并对下载到本地的网络内容进行LRU管理。目前小程序允许的最大本地储存为10m，我们默认painter可使用的本地存储为6m，超出时会对本地存储进行清理。如果需要自定义，可以在/painter/lib/downloader.js中修改MAX_SPACE_IN_B属性。

1. 目前子 view 的 css 属性支持 object 或 array。允许将几个view公用的css属性提取出来。
2. 由于palette 是以 js 承载的 json，所以你可以在每一个属性中很方便的加上自己的逻辑。也可以把某些属性单独提取出来，让多个 palette 共用，做到模块化。

### 使用

#### demo下载

demo项目使用submodule的方式进行管理，因此在clone时需要运行

```
git clone https://github.com/Kujiale-Mobile/Painter.git --recursive
```

clone完成后可以看到目录。其中，/pages/example中存放的是使用示例，/components/painter就是我们所引入的功能组件。此外还有一个palette目录，里面存放是我们所需要绘图代码。实际工作时，painter会调取card.js里的信息，在图片上绘制出相应的图形，就像一支画笔在调色板上调制蘸取了颜料，然后在画布上创作一样。

<img src="http://sinacloud.net/music-store/markdownPic/painter2.png?KID=sina,2o3w9tlWumQRMwg2TQqi&Expires=1579069535&ssig=eb8cOp%2FfLR" width=200>

#### 将Painter引入到自己的项目

你可以直接将demo里的painter复制粘贴到自己的项目下，当然也可以更为优雅地运行一下这个代码：

```powershell
git submodule add https://github.com/Kujiale-Mobile/PainterCore.git painter
```

它会将Painter工具放置在你当前的目录下。我们推荐的做法是把它放在你的components下。

#### 引入组件

像其它的组件一样，在需要引入Painter的页面.json文件中添加：

```json
"usingComponents":{
  "painter":"/components/painter/painter"
}
```

#### 组件调用

在页面的xml文件中调用painter组件，并传入pallete规则的数据，以及绘制结束以后的回调。

```xml
<painter palette="{{data}}" bind:imgOK="onImgOK" bind:imgErr="onImgErr"/>
```

palette即是我们的调色板数据，以json形式根据一定规范创建，详细信息请移步下文。

#### 绘制回调

```js
bind:imgOK="onImgOK"
bind:imgErr="onImgErr"
```

数据传入后，painter就会开始绘制，无论绘制成功或是失败，都能在相应的回调方法里获取相关的信息，如：

<img src="http://sinacloud.net/music-store/markdownPic/painter3.png?KID=sina,2o3w9tlWumQRMwg2TQqi&Expires=1579069535&ssig=pqFHDi2C07" width=300>              <img src="http://sinacloud.net/music-store/markdownPic/painter4.png?KID=sina,2o3w9tlWumQRMwg2TQqi&Expires=1579069535&ssig=vIjymNrLl0" width=300>





### Pallette

说到底，Painter是一支画笔工具，具体要让这支画笔画什么东西，还得由我们，天资聪颖的程序猿们，来告诉它。告诉它应该画什么，在哪里画，画的时候用什么姿势……等等。这需要用一些别的手段，因为科学的实验证明过，试图用普通话这门语言跟它进行沟通，是不会有任何效果的。

#### 调色板属性

每一块调色板都它自己的整体属性，它一般规定了整个绘图范围的大小、样式、背景等

它处于整个json文件的最外层，需要指定以下几个属性：

| 属性       | 解释                                                         |
| :--------- | ------------------------------------------------------------ |
| background | 背景，可以是颜色值，也可以是图片链接，支持本地图片链接和网络图片链接 |
|width|宽度|
|height|高度|
|borderRadius|圆角|
|views|需要画在图上的其它元素，允许为空，但不允许省略|

示例代码：

```
{
      background: 'https://qhyxpicoss.kujiale.com/2018/06/12/LMPUSDAKAEBKKOASAAAAAAY8_981x600.png',
      width: '654rpx',
      height: '400rpx',
      borderRadius: '20rpx',
      views: []
 }
```

#### view属性

画完了调色板的整体属性以后，就可以向views中增加一些元素了。元素支持四种类型，用type字段进行区分分类。不同种类的view又要求提供有不同的数据，如image元素需要提供它的url，text元素需要提供text文字内容：

| type  | content | description              | 私有css属性 |
| ----- | ------- | ------------------------ | ----------- |
| image | url     | 图片资源地址，本地或网络 |             |
|text|text|文本元素，书写文字|fontSize:字体大小，color:文字颜色|
|rect|无|矩形|color:填充颜色|
|qrcode|content|画二维码|background:背景颜色，默认为透明|

除了各view的私有属性之外，view还有一些公共属性可以设置：

| 属性   | 作用                      |
| ------ | ------------------------- |
|left, top, right, bottom|元素的位置|
| rotate | 旋转角度，单位为360度的度 |
|borderRadius|圆角，如果需要设置图片为原形，请设置该属性为宽或高的一半|
|align |元素在水平方向的对齐方式，与left配合使用，可设置为left, center, right，默认为left。|

##### rotate

控制元素的旋转，如下图，将一行文字顺时针旋转了6度。

```json
{
    type: 'text',
    text: '酷家乐 移动前端',
    css: {
     left: '20rpx',
     top: '50rpx',
     fontSize: '40rpx'
   },
},
```

效果：

<img src="http://sinacloud.net/music-store/markdownPic/painter5.png?KID=sina,2o3w9tlWumQRMwg2TQqi&Expires=1579069535&ssig=%2FIQFDrXwM8" width=300>

##### borderRadius

代码(圆形)：

```js
        {
          type: 'image',
          url: this.cardInfo.avatar,
          css: {
            top: '48rpx',
            left: '448rpx',
            width: '192rpx',
            height: '192rpx',
            borderRadius:'96rpx',
          },
        },
```

方角-->8rpx圆角-->圆形

<img src="http://sinacloud.net/music-store/markdownPic/painter6.png?KID=sina,2o3w9tlWumQRMwg2TQqi&Expires=1579069535&ssig=mRPxmGLq%2FM" width=300>

##### align

这个属性值比较有意思，它被用来设置元素在水平方向的、相对于位置设置的对齐方式。

什么意思呢？

比如说你设置了某元素的left为100rpx，并设置align属性为left，那么该元素的左端就与100rpx对齐；若设置align为center，则该元素的中轴线与100rpx对齐。

在下面的例子中，三行文字的left都是230rpx，align分别为left, center, right。红线是横坐标为230rpx的轴线。

即，当设置了align属性的时候，left值表达的是元素属性中align的位置。

<img src="http://sinacloud.net/music-store/markdownPic/painter7.png?KID=sina,2o3w9tlWumQRMwg2TQqi&Expires=1579069535&ssig=NKWF9HpTSU" width=300>



代码：

```js
		{
          type: 'text',
          text: '酷家乐 移动前端',
          css: {
            left: '330rpx',
            top: '100rpx',
            fontSize: '40rpx',
          },
        },
        {
          type: 'text',
          text: '酷家乐 移动前端',
          css: {
            left: '330rpx',
            top: '200rpx',
            fontSize: '40rpx',
            align: 'center'
          },
        },
        {
          type: 'text',
          text: '酷家乐 移动前端',
          css: {
            left: '330rpx',
            top: '300rpx',
            fontSize: '40rpx',
            align: 'right'
          },
        },
```

有了这个属性，就可以设置元素的对齐形式，完成下面的布局要求了：

<img src="http://sinacloud.net/music-store/markdownPic/painter8.png?KID=sina,2o3w9tlWumQRMwg2TQqi&Expires=1579069535&ssig=n9f0Zi2sd5" width=300>          <img src="http://sinacloud.net/music-store/markdownPic/painter9.png?KID=sina,2o3w9tlWumQRMwg2TQqi&Expires=1579069535&ssig=We9qnzenXa" width=300>



**注意：align属性请和left属性配合使用，设置right值将造成错误。**



##### align与rotate

当align属性与rotate属性同时存在时，元素的旋转表现是以元素的中心点为中心的。

<img src="http://sinacloud.net/music-store/markdownPic/painter10.png?KID=sina,2o3w9tlWumQRMwg2TQqi&Expires=1579069535&ssig=smbGVG%2F3u3" width=300>



#### 尺寸单位

目前 Painter 中支持两种尺寸单位，px 和 rpx，代表的意思和小程序中一致。目前还没有很好地支持百分比的使用。

#### 保存图片演示

获得图片的url后，可以设置一个点击按钮，点击保存到本地

```
  onImgOK(e) {
    this.imagePath = e.detail.path;
  },

  saveImage() {
    wx.saveImageToPhotosAlbum({
      filePath: this.imagePath,
    })
  },
```

按钮绑定saveImage方法，点击进行保存：

<img src="http://sinacloud.net/music-store/markdownPic/painter12.gif?KID=sina,2o3w9tlWumQRMwg2TQqi&Expires=1599707408&ssig=4CxJKcKxV8" width=500>



### 生成朋友圈分享图

最后，利用Painter工具可以生成不同样式的朋友圈分享图(下图为微信小程序 酷咖名片 线上版部分截图)

<img src="http://sinacloud.net/music-store/markdownPic/share1.jpeg?KID=sina,2o3w9tlWumQRMwg2TQqi&Expires=1599707408&ssig=HgqWb35Pbh" width=230>                                     <img src="http://sinacloud.net/music-store/markdownPic/share2.jpeg?KID=sina,2o3w9tlWumQRMwg2TQqi&Expires=1599707408&ssig=XwFCwR67iz" width=230>  

<img src="http://sinacloud.net/music-store/markdownPic/share3.jpeg?KID=sina,2o3w9tlWumQRMwg2TQqi&Expires=1599707408&ssig=eUX6XhYCBg" width=230>                     <img src="http://sinacloud.net/music-store/markdownPic/share4.jpeg?KID=sina,2o3w9tlWumQRMwg2TQqi&Expires=1599707408&ssig=ZQRv4QwSOU" width=230>



### 结语

painter是酷家乐前端在小程序的实际开发中自制的一套工具，目前在朋友圈分享、皮肤模板替换等方面都在使用，觉得用起来喜忧参半。开源出来大家分享。如果它能帮助到任何一个人，我们都非常开心；我们也非常欢迎并感谢提issure或pr，来告诉我们一些我们自己没有能想到的东西，或者帮助解决Painter 中的大大小小的坑：

再一次传送门地址：https://github.com/Kujiale-Mobile/Painter