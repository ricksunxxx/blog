## 1、背景

最近在用小程序 canvas 做礼花从天空飘洒落地的动画，动画效果大概是这样的：

<img src="https://user-images.githubusercontent.com/9975520/123826304-3bcfd300-d932-11eb-88e6-d5927f035275.png" width = "50%" height = "50%" align=center />

## 2、问题一：canvas 版本问题

> 小程序 canvas 是个原生组件，有新旧版本之分，基础库 2.9.0 起使用新版本 canvas 2D。

1.旧版本不支持同层渲染，z-index 不起作用，层级比非原生组件都要高。

2.新版本支持同层渲染，z-index 可以控制组件的层级，像其他正常的内置组件一样使用。

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
