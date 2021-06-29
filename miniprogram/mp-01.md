## 1、背景

最近在用小程序 canvas 做礼花从天空飘洒落地的动画，动画效果大概是这样的

![image](https://user-images.githubusercontent.com/9975520/123826304-3bcfd300-d932-11eb-88e6-d5927f035275.png)

## 2、问题一：canvas 版本问题

小程序基础库 2.9.0 起支持一套新 Canvas 2D 接口（需指定 type 属性），同时支持同层渲染，原有接口不再维护
wx.createCanvasContext

```
<canvas type="2d" id="myCanvas"></canvas>
<canvas type="2d" id="myCanvas"></canvas>
<canvas type="2d" id="myCanvas"></canvas>
```
