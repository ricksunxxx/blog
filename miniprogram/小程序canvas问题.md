## 1、背景

最近在用小程序 canvas 做礼花从天空飘洒落地的动画，动画效果大概是这样的

<img src="https://user-images.githubusercontent.com/9975520/123826304-3bcfd300-d932-11eb-88e6-d5927f035275.png" width = "50%" height = "50%" align=center />

## 2、问题一：canvas 版本问题

如果使用旧版接口，获取 canvas 绘图上下文时需要特别注意：

```

<!-- canvas.wxml -->
<canvas canvas-id="myCanvas"></canvas>

//在Page内
var context = wx.createCanvasContext('myCanvas') //使用 wx.createContext 获取绘图上下文 context

//在Component内
var context = wx.createCanvasContext('myCanvas',this) //表示在这个自定义组件下查找

```

如果使用新版接口，基础库在 2.9.0 起支持一套新 Canvas 2D 接口（需指定 type 属性），同时支持同层渲染，原有接口不再维护。

新版 canvas API 对齐 HTML canvas API，并且开启了 GPU 加速，性能比旧版提升了 50%，官方建议使用新版接口。

这时候，代码又不一样了：

```

<!-- canvas.wxml -->
<canvas type="2d" id="myCanvas"></canvas> //多了type和id,少了canvas-id


//在Page内
const query = wx.createSelectorQuery()
query.select('#myCanvas')
    .fields({ node: true, size: true })
    .exec((res) => {
    const canvas = res[0].node
    const ctx = canvas.getContext('2d')
    })


//在Component内
const query = this.createSelectorQuery() //这里要使用this，不能使用wx
query.select('#myCanvas')
    .fields({ node: true, size: true })
    .exec((res) => {
    const canvas = res[0].node
    const ctx = canvas.getContext('2d')
    })

```
