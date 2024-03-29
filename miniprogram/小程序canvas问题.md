# 小程序canvas问题

> canvas 是 HTML5 新增的元素，可用于通过使用 JavaScript 中的脚本来绘制图形。例如，它可以用于绘制图形、制作照片、创建动画，甚至可以进行实时视频处理或渲染。

微信小程序一开始就支持 canvas，但早期的 canvas 存在许多不足，canvas 层级过高覆盖其他组件的问题一直令人诟病。2.9.0起，小程序发布了一套新的 Canvas 2D 接口，可以支持同层渲染，解决了这个“心头大患”。

## 背景

最近在用小程序 canvas 做礼花从天空飘洒落地的动画，动画效果大概是这样的：

<img src="https://user-images.githubusercontent.com/9975520/123932101-b0068700-d9c3-11eb-9934-b1f7c719ad40.png" width = "30%" height = "30%" align=center />

## 问题一：canvas 版本问题

> 小程序 canvas 是个原生组件，有新旧版本之分，基础库 2.9.0 起使用新版本 canvas 2D。

1. 旧版本不支持同层渲染，z-index 不起作用，层级比非原生组件都要高。

2. 新版本支持同层渲染，z-index 可以控制组件的层级，像其他正常的内置组件一样使用。

对于新旧版 canvas，在使用时需要特别注意以下问题：

```js

// 使用旧版接口 wx.createContext 获取绘图上下文 context

<!-- canvas.wxml -->
<canvas canvas-id="myCanvas"></canvas>

// 在Page内
var context = wx.createCanvasContext('myCanvas')

// 在Component内
var context = wx.createCanvasContext('myCanvas',this) // this表示在这个自定义组件下查找，不可缺少

```

如果使用新版接口，这时候，代码又不一样了：

```js


// 使用新版接口获取绘图上下文 context

<!-- canvas.wxml -->
<canvas type="2d" id="myCanvas"></canvas> // 多了type和id,少了canvas-id

// 在Page内
const query = wx.createSelectorQuery()
query.select('#myCanvas')
    .fields({ node: true, size: true })
    .exec((res) => {
    const canvas = res[0].node
    const context = canvas.getContext('2d')
    })

// 在Component内
const query = this.createSelectorQuery() // 这里要使用this，不能使用wx
query.select('#myCanvas')
    .fields({ node: true, size: true })
    .exec((res) => {
    const canvas = res[0].node
    const context = canvas.getContext('2d')
    })

```

小程序新版本 canvas 向[标准的 HTML Canvas](https://html.spec.whatwg.org/multipage/canvas.html#the-canvas-element)对齐。

当前官方 canvas API 文档没有区分新旧，很容易让开发者搞混，因此开发时可以参考 HTML canvas API：

MDN：[Canvas API](https://developer.mozilla.org/en-US/docs/Web/API/Canvas_API)

中文：[Canvas API](https://www.canvasapi.cn/CanvasRenderingContext2D/)

## 问题二：canvas 新旧接口不一致问题

1. draw 方法

> CanvasContext.draw()方法：将之前在绘图上下文中的描述（路径、变形、样式）画到 canvas 中。

旧版本：

```js
const ctx = wx.createCanvasContext('myCanvas')

ctx.save()
ctx.setFillStyle('red')
ctx.fillRect(10, 10, 150, 100)
ctx.draw() // 如果没有draw，画布上是不会有东西的
```

新版本：**新版 canvas 没有 draw 方法**，只有 drawImage 方法，切记。

2. setFillStyle 方法与 fillStyle 属性

> CanvasContext.setFillStyle()方法:填充的颜色，默认颜色为 black。

旧版本：是个**方法**

```js
ctx.setFillStyle('red')
```

新版本：是个**属性**

```js
ctx.fillStyle = 'red'
```

## 问题三：canvas 新旧版本事件触发不一致

旧版本：canvas 设置 z-index 小于黑色遮罩，此时 canvas 还是覆盖在黑色遮罩**上面**的，因为旧版 canvas 并没有做同层渲染，这是正常的展示。但是神奇的是：点击黑色遮罩上按钮，居然可以触发按钮上的事件，理论上应该是不可以的，因为 canvas 挡住了按钮而且不存在事件冒泡的情况。

新版本：canvas 设置 z-index 小于黑色遮罩，因为同层渲染原因，此时 canvas 在黑色遮罩**下面**了，不符合需求，我们把 z-index 设置高一点，此时 canvas 在黑色遮罩**上面**了，但是此时按钮是触发不了的，符合逻辑。

解决方法：

旧版 canvas：不建议，不能保证所有平台和手机都一致，而且官方已经放弃维护。

新版 canvas：建议，可以对 z-index 这么设置：按钮 z-index > canvas z-index > 蒙层 z-index

## 问题四：canvas 锯齿问题

<img src="https://user-images.githubusercontent.com/9975520/123932805-5fdbf480-d9c4-11eb-8f0f-e5e36f3bb897.png" width = "30%" height = "30%" align=center />

问题有两个:

1、方块变大了，而且有明显的锯齿

2、下落速度比原来快了非常多

问题原因：

Retina 屏下 1px 使用了 n 个屏幕像素来绘制，其中 n 是像素大小的比率，一般叫 dpr，即设备的物理像素分辨率与 CSS 像素分辨率之比。

假如是 iPhone XS 手机，此时 n = 3 ，1px 被拉伸展示到 3 个设备像素来展示，这时绘制的颜色就有深有浅了，如下图：

<img src="https://user-images.githubusercontent.com/9975520/123936982-49379c80-d9c8-11eb-863d-e4a2f81b2487.png" width = "30%" height = "30%" align=center />

解决方法：

把画布放大，保持跟设备物理像素分辨率一致就可以了，代码如下：

```js
const dpr = wx.getSystemInfoSync().pixelRatio // 浏览器环境则使用 dpr = window.devicePixelRatio
const query = this.createSelectorQuery()
query
  .select('#myCanvas')
  .fields({
    node: true,
    size: true,
  })
  .exec((res) => {
    const canvas = res[0].node
    ctx = canvas.getContext('2d')

    // 以下处理是画布放大dpr倍（iPhone XS 的dpr = 3）
    canvas.width = res[0].width * dpr
    canvas.height = res[0].height * dpr
    ctx.scale(dpr, dpr)
  })
```

## 总结

1. canvas 新旧版本问题
2. canvas 接口不一致问题
3. canvas 事件与表现不一致问题
4. canvas 锯齿问题

小程序新版本 canvas 默认开启 GPU 加速，性能比旧版提升 50%，而且做了同层渲染，事件和样式都符合逻辑，官方推荐使用新版。
