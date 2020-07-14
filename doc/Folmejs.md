# Folmejs

### 简介

Folme 是一款高性能的 JavaScript 动画引擎

**特点** 

可随时打断

跨平台

**使用建议**

Folme依赖于`requestAnimationFrame`，所以如果当浏览器环境下主线程被阻塞，`requestAnimationFrame`也会被阻塞调用，会导致动画卡顿。

当页面中有大量阻塞主线程的逻辑时, 应该优先选择 CSS 动画

**CSS transition**

如果目标元素的某 CSS 属性设置了 transition-duration, 那么引擎将无法正确修改改属性. 所以应避免使用 `transition: all`, 以免出现无法预料的问题.

### 按需加载

@folmejs/core 没有默认导出，应该自己选择需要导入什么变量

如:

```javascript
import { to, set, easings } from "@folmejs/core";
```



### 快速开始

实现一个跟随鼠标平滑移动的小球

![folmeball](C:\Users\yqm\Desktop\gif\folmeball.gif)

### 目标元素与属性

在 Folme 中, 可以创建动画效果的只有 `to` 函数.

```javascript
function to(target: Target, properties: Properties, options?: Options): void;
```

`target` 为目标元素, `properties` 为目标属性, `options` 则一些额外的配置项.

#### 目标元素

Folme 可以作用于普通 HTML 元素, SVG 元素, 甚至 Window, 目标元素不同, 有效的目标属性也是不同的.

```typescript
type Target = string | Element | Element[] | NodeListOf<Element> | Window;
```

传递目标元素

```javascript
// Selector string
to(".target");

// Element
to(document.querySelector("#target"));

// Element array
to([document.querySelector("#target-1"), document.querySelector("#target-2")]);

// NodeList
to(document.querySelectorAll(".target"));

// Window
to(window);
```

```javascript
// Selector string
to(".target");

// Element
to(document.querySelector("#target"));

// Element array
to([document.querySelector("#target-1"), document.querySelector("#target-2")]);

// NodeList
to(document.querySelectorAll(".target"));

// Window
to(window);
```

```javascript
// Selector string
to(".target");

// Element
to(document.querySelector("#target"));

// Element array
to([document.querySelector("#target-1"), document.querySelector("#target-2")]);

// NodeList
to(document.querySelectorAll(".target"));

// Window
to(window);
```

**普通 HTML 元素目标属性**

```javascript
to(".target", {
  width: 100,
  backgroundColor: "#fff",
  color: "rgba(255,255,255, 0.3)",
});
```

#### 属性的初始值及转化

首次使用 `to` 设置目标属性时, Folme 首先会读取目标属性的值作为当前值, 然后再开始计算。

> **即便首次使用 to, width 也不会从 0 开始运动到目标值, 而是读取当前值作为动画的初始值.**

![folmeball-change](C:\Users\yqm\Desktop\gif\folmeball-change.gif)



### **停止动画**

```javascript
function cancel(target: Target, properties?: string | string[]): void;

cancel(".target"); // 取消目标元素所有动画
cancel(".target", "width"); // 取消 width 属性的动画
cancel(".target", ["width", "height"]); // 取消 width 和 height 属性的动画
```



![folmeball-cancel](C:\Users\yqm\Desktop\gif\folmeball-cancel.gif)

**何时使用**

1. 当你需要停止某些属性的动画时;
2. 大量元素或属性使用动画, 动画停止后你可以使用cancel释放内存.



### 自定义动画

![image-20200714141402305](C:\Users\yqm\AppData\Roaming\Typora\typora-user-images\image-20200714141402305.png)



**预设的缓动函数**

```javascript
const easings = {
    // 弹簧
    spring(damping = 0.85, response = 0.3) // damping 阻尼 response 响应时间(s)
    // 传统缓动模型
    linear(duration = 0.3) // duration 持续时间(s)
    quadIn(duration = 0.3)
    quadOut(duration = 0.3)
    quadInOut(duration = 0.3)
    cubicIn(duration = 0.3)
    cubicOut(duration = 0.3)
    cubicInOut(duration = 0.3)
    quartIn(duration = 0.3)
    quartOut(duration = 0.3)
    quartInOut(duration = 0.3)
    quintIn(duration = 0.3)
    quintOut(duration = 0.3)
    quintInOut(duration = 0.3)
    sinIn(duration = 0.3)
    sinOut(duration = 0.3)
    sinInOut(duration = 0.3)
    expoIn(duration = 0.3)
    expoOut(duration = 0.3)
    expoInOut(duration = 0.3)
}
```

```javascript
import { to, easings } from "@folmejs/core";

to(".target", {
  x: {
    value: 100,
    easing: easings.linear(1),
  },
});
```

### 多元素编排

start (起始值), step(步长), reverse(是否逆向),

```javascript
function defineStagger(start: number | string, step: number, reverse?: boolean);
```

使用方式

```javascript
to(".target", {
  x: defineStagger(0, 100),
});

to(".target", {
  x: {
    value: defineStagger(0, 100),
    delay: defineStagger(0, 0.1),
  },
});
```

**自定义stagger**

Stagger 是一个纯函数, 可以自己提供一个符合要求的函数, 根据接收到的参数计算出返回值.

```typescript
type Stagger = (
  index: number, // 元素的下标
  propName: string, // 属性名
  element: Element | Window, // 元素
) => number | string;
```

`defineStagger` 也接收一个回调函数

```javascript
to(".target", {
  x: index => index * 100,
});
to(".target", {
  x: defineStagger(index => index * 100);
});
```

### 回调函数优化

 folme在全局维护一个回调队列, 如果调用 `to` 时提供了回调函数, 就将这些回调函数加入回调队列.

引擎每计算一次就会遍历一次回调队列, 寻找匹配的回调函数调用, 并在动画停止后删除.



**回调爆炸**

如果连续使用`to`为同一个元素的相同属性创建动画效果, 并且每一个`to`都提供回调函数. 那么在动画完全停止之前, 有关回调函数都无法被释放, 会被一次又一次重复调用, 回调队列会越来越大, 导致页面卡顿甚至崩溃.

> **非常不推荐连续调用 `to` 时添加回调函数.**

如果必须要添加, 你可以在配置项中设置参数`key`, 可以为 `string` 或 `Symbol`.

```javascript
const uniqueKey = Symbol();
to(
  ".target",
  {
    x: Math.random() * 100,
  },
  {
    key: uniqueKey,
    onUpdate(state) {...},
    onComplate(state) {...},
  },
);

```

如果提供了 `key`, Folme 在向回调队列中添加回调函数时将优先替换 `key` 值相同的项, 避免出现回调爆炸.