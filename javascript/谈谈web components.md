# 谈谈 Web components

开发过微信小程序的同学应该对下图自定义组件在开发者工具的展示比较熟悉，其中有节点展示了#shadow-root，这个东西是什么呢，保持疑问去Google找答案。

![image](https://user-images.githubusercontent.com/9975520/124881579-1877e880-e002-11eb-9d32-a13a0e61ff26.png)

## Web components 是什么？

从[官方网站](https://www.webcomponents.org/introduction)的介绍可以得知：

>Web components are a set of web platform APIs that allow you to create new custom, reusable, encapsulated HTML tags to use in web pages and web apps.

从定义可以提取出几个关键词：

**custom（自定义）**，**reusable（可重用）**，**encapsulated（封装）**

这不就是自定义组件吗？

所以，大概可以猜到 Web components 应该就是在浏览器层面提供实现自定义组件的能力。

## Web components 四个主要规范

1. Custom elements（自定义元素）
2. Shadow DOM （影子DOM）
3. HTML Template （HTML模板）
4. ES Modules （ES 模块）

## Custom elements

>The Custom Elements specification lays the foundation for designing and using new types of DOM elements.

自定义组件分为两种类型：

1. Autonomous custom elements （基于 HTMLElement 抽象类）

    这是全新的元素, 继承自 HTMLElement 抽象类。

2. Customized built-in elements （基于内置元素）

   继承自内置的 HTML 元素，比如自定义 HTMLButtonElement 等。

**基于 HTMLElement 抽象类的语法：**

```js
class MyElement extends HTMLElement {
  constructor() {
    super();
    // 元素在这里创建
  }

  connectedCallback() {
    // 在元素被添加到文档之后，浏览器会调用这个方法
    //（如果一个元素被反复添加到文档／移除文档，那么这个方法会被多次调用）
  }

  disconnectedCallback() {
    // 在元素从文档移除的时候，浏览器会调用这个方法
    // （如果一个元素被反复添加到文档／移除文档，那么这个方法会被多次调用）
  }

  static get observedAttributes() {
    return [/* 属性数组，这些属性的变化会被监视 */];
  }

  attributeChangedCallback(name, oldValue, newValue) {
    // 当上面数组中的属性发生变化的时候，这个方法会被调用
  }

  adoptedCallback() {
    // 在元素被移动到新的文档的时候，这个方法会被调用
    // （document.adoptNode 会用到, 非常少见）
  }

  // 还可以添加更多的元素方法和属性
}

customElements.define('my-element', MyElement);
```

例如我们写个hello world的例子：

```html
<!DOCTYPE html>
<html lang="en">
  <body>
    <hello-world></hello-world>
    <script>
      class MyElement extends HTMLElement {
        connectedCallback() {
          this.innerHTML = '<h1>hello world</h1>'
        }
      }

      // 让浏览器知道我们新定义了新的标签 <hello-world>
      customElements.define('hello-world', MyElement)
    </script>
  </body>
</html>
```

**基于内置元素的语法：** 例如在按钮的基础上自定义组件

```js
// 抽象类 HTMLElement 变成了具体的类 HTMLButtonElement
class HelloButton extends HTMLButtonElement {
    /* custom element 方法 */    
}

// 提供定义标签的第三个参数
customElements.define('hello-button', HelloButton, { extends:'button' })

// 在 HTML 页面这么使用
<button is="hello-button"></button>
```

例如：两个按钮，一个可点击，一个不可点击

```html
<!DOCTYPE html>
<html lang="en">
  <body>
    <button is="hello-button">Click me</button>
    <button is="hello-button" disabled>Disabled</button>
    <script>
      // 这个按钮在被点击的时候说 "hello"
      class HelloButton extends HTMLButtonElement {
        constructor() {
          super()
          this.addEventListener('click', () => alert('Hello!'))
        }
      }

      customElements.define('hello-button', HelloButton, { extends: 'button' })
    </script>
  </body>
</html>
```

Custom elements 在各浏览器中的兼容性已经非常好了。Edge 支持地相对较差，但是我们可以使用 [polyfill](https://github.com/webcomponents/polyfills/tree/master/packages/webcomponentsjs)

## Shadow DOM

>Shadow DOM defines how to use encapsulated style and markup

Shadow DOM 定义如何封装样式和标签，它可以让一个组件拥有自己的「影子」DOM 树，我们不能使用一般的 JavaScript 调用或者选择器来获取内建 shadow DOM 元素。它们不是常规的子元素，而是一个强大的封装手段。

实际上一些原生的 HTML 元素也使用了 Shadow DOM 。例如你再一个网页中有一个 \<video> 元素，它将会作为一个单独的标签展示，但它也将显示播放和暂停视频的控件，当你在浏览器开发工具中查看 video 标签，是看不到这些控件。这些控件实际上就是 video 元素的 Shadow DOM 的一部分，因此默认情况下是隐藏的

一个 DOM 元素有两类 DOM 子树：

1. Light tree（光明树）
2. Shadow tree（影子树）

我们平时看到的HTML5 里的各种标签都是 Light tree 。

文章顶部微信开发者工具里看到就是Shadow tree。

我们再看看 Google 首页：
![image](https://user-images.githubusercontent.com/9975520/124908765-41a57280-e01c-11eb-918e-d098fa2ff1c2.png)

如果你看不到shadow tree 打开下面这个开关就可以了：
![image](https://user-images.githubusercontent.com/9975520/124909294-dc05b600-e01c-11eb-8b63-8adf7d0fd954.png)

还是刚才的 hello world，稍微改一下就可看到了：

```html
<!DOCTYPE html>
<html lang="en">
  <body>
    <hello-world></hello-world>
    <script>
      class MyElement extends HTMLElement {
        connectedCallback() {
          // 调用 this.attachShadow({ mode: ... }) 创建 shadow tree
          const shadow = this.attachShadow({ mode: 'open' })
          shadow.innerHTML = '<h1>hello world</h1>'
        }
      }

      // 让浏览器知道我们新定义了新的标签 <hello-world>
      customElements.define('hello-world', MyElement)
    </script>
  </body>
</html>
```

带有 shadow tree 的 hello world 效果如下：
![image](https://user-images.githubusercontent.com/9975520/124910116-cd6bce80-e01d-11eb-87d1-adb28662b884.png)

Shadow dom 是创建组件级别 DOM 的一种方法，可以使用 innerHTML 或者其他 DOM 方法来扩展 shadowRoot，需要注意的是：

1. 文档里面的样式对 shadow tree 没有任何效果
2. shadow tree内部的样式才起作用
3. 对主文档的 JavaScript 选择器隐身，比如 querySelector

当然，Shadow dom 还有插槽 slot（具名插槽和默认插槽），还可以引入样式，如 \<style> 或 \<link rel="stylesheet"> 。

为了保持细节简单，浏览器会重新定位事件。当事件在组件外部捕获时，shadow DOM 中发生的事件将会以 host 元素作为目标。但是，如果事件发生在 slotted 元素上，实际存在于 light DOM 上，则不会发生重定向。  

## HTML Template

>HTML template defines how to declare fragments of markup that go unused at page load, but can be instantiated later on at runtime.

\<template\>  元素用来存储 HTML 模板。浏览器将忽略它的内容，仅检查语法的有效性，但是我们可以在 JavaScript 中访问和使用它来创建其他元素。

模板的 content 属性可看作Document Fragment —— 一种特殊的 DOM 节点。

Talk is cheap ，show me your codes ！

```html
<!DOCTYPE html>
<html lang="en">
  <body>
    <div id="elem">Click me</div>

    <!-- 模板，可以看出一段封装好的可复用的 HTML -->
    <template id="tmpl">
      <style>
        p {
          font-weight: bold;
        }
      </style>
      <p id="message"></p>
    </template>

    <script>
      elem.onclick = function () {
        elem.attachShadow({ mode: 'open' })
        elem.shadowRoot.append(tmpl.content.cloneNode(true)) 
        elem.shadowRoot.getElementById('message').innerHTML =
          'Hello from the shadows!'
      }
    </script>
  </body>
</html>
```

![image](https://user-images.githubusercontent.com/9975520/124913795-2178b200-e022-11eb-9b8a-ee0ec7f353d7.png)

\<template\>  标签非常独特:

1. 浏览器将检查模板里的HTML语法
2. 插入其文档后，模板里面的脚本会运行
3. 模板不具有任何迭代机制，数据绑定或变量替换的功能

## ES Modules

>The ES Modules specification defines the inclusion and reuse of JS documents in a standards based, modular, performant way.

ES Modules 其实就是 JavaScript 官方的规范的模块化系统，它并不是 Web components 的产物。相反，Web components 的是基于ES Modules 来实现模块化定义和使用的。

如果你有使用过 [polymer](https://polymer-library.polymer-project.org/3.0/docs/devguide/feature-overview)，你会发现Web components 的模块化非常熟悉：[paper-button 具体用法](https://www.webcomponents.org/element/@polymer/paper-button)

```js
<script type="module" src="node_modules/@polymer/paper-button/paper-button.js"></script>

<paper-button class="indigo">raised</paper-button>
```

如果你熟悉 [Vite](https://vitejs.dev/) ，你会特别理解以上模块的引入方式。

## 总结

Web components 的优势：

1. 原生不需要框架
2. 易于继承，不需要编译
3. 真正的局部CSS作用域
4. 标准，只有HTML，CSS，JavaScript

Web components 的不足：

1. Web components 依托在 html 模版语言上，缺少 js 的灵活性
2. 兼容问题，各家浏览器厂商跟进缓慢

其实，国内的网站其实很难看到 Web components 被大量应用的身影。 国外相对多一点，比如Google首页就应用了非常多Web components。

随着前端框架种类越来越多，组件无法跨框架复用，甚至只能固定在框架的某个版本使用，随着技术的迭代进步，相信 Web components 会作为未来的 Web 组件标准，真正打通 Web 生态，让组件超越框架而存在，未来我们再不用为Vue、React、Angular谁更好这种题目而烦恼了。

革命尚未成功，同学尚需努力学习三大框架 **:)**

## 参考

[www.webcomponents.org](https://www.webcomponents.org/introduction)

[web-components](https://javascript.info/web-components)

[custom-elements](https://html.spec.whatwg.org/#custom-elements)

[web-components-will-replace-your-frontend-framework](https://www.dannymoerkerke.com/blog/web-components-will-replace-your-frontend-framework)

[Micro-Frontend using Web Components](https://medium.com/swlh/micro-frontend-using-web-components-e9faacfc101b)

[polymer 3.0](https://polymer-library.polymer-project.org/3.0/docs/devguide/feature-overview)
