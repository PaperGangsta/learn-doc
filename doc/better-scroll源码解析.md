# better-scroll源码阅读

主要阅读源码中的core核心模块和infinity无限下拉的部分

## Core

### 目录结构

![image-20200714172207687](C:\Users\yqm\AppData\Roaming\Typora\typora-user-images\image-20200714172207687.png)

### utils

utils内主要是`bubbling.ts`

`bubbling`中主要做的是对于事件的处理。

包括了事件传递，从源事件到目标事件一些系列的过程。

![image-20200714173818818](C:\Users\yqm\AppData\Roaming\Typora\typora-user-images\image-20200714173818818.png)

### translater

translater下只有一份`index.ts`的文件，它的目的也很明确

就是翻译或者或转译。

计算位置将源数据位置转译为css的translate偏移量。

通过偏移改变节点的位置。

![image-20200714174213611](C:\Users\yqm\AppData\Roaming\Typora\typora-user-images\image-20200714174213611.png)

![image-20200714174223674](C:\Users\yqm\AppData\Roaming\Typora\typora-user-images\image-20200714174223674.png)



### base

base中主要是对一些基本的事件做的处理。

preventDefault:包括阻止默认的事件发生(阻止浏览器默认行为)。

preventDefaultException:阻止原生的滚动，这样也同时阻止了一些原生组件的默认行为.

stopPropagation:阻止事件冒泡

![image-20200714175201051](C:\Users\yqm\AppData\Roaming\Typora\typora-user-images\image-20200714175201051.png)

**更具体些**：

在base中对原有的事件做了封装，并控制一些原生的事件，比如touch和对应的鼠标事件、click这种。

根据浏览器的环境控制了应该注册的事件类型：

是应当注册触摸事件还是注册鼠标事件。

```typescript
private handleDOMEvents() {
    const { bindToWrapper, disableMouse, disableTouch, click } = this.options
    const wrapper = this.wrapper
    const target = bindToWrapper ? wrapper : window
    const wrapperEvents = []
    const targetEvents = []
    const shouldRegisterTouch = hasTouch && !disableTouch
    const shouldRegisterMouse = !disableMouse

    if (click) {
      wrapperEvents.push({
        name: 'click',
        handler: this.click.bind(this),
        capture: true
      })
    }

    if (shouldRegisterTouch) {
      wrapperEvents.push({
        name: 'touchstart',
        handler: this.start.bind(this)
      })

      targetEvents.push(
        {
          name: 'touchmove',
          handler: this.move.bind(this)
        },
        {
          name: 'touchend',
          handler: this.end.bind(this)
        },
        {
          name: 'touchcancel',
          handler: this.end.bind(this)
        }
      )
    }

    if (shouldRegisterMouse) {
      wrapperEvents.push({
        name: 'mousedown',
        handler: this.start.bind(this)
      })

      targetEvents.push(
        {
          name: 'mousemove',
          handler: this.move.bind(this)
        },
        {
          name: 'mouseup',
          handler: this.end.bind(this)
        }
      )
    }
    this.wrapperEventRegister = new EventRegister(wrapper, wrapperEvents)
    this.targetEventRegister = new EventRegister(target, targetEvents)
  }
```

此外，对start、move、end三种状态做了处理。

down-move-up

在**start**中主要进行事件的初始化，包括了对起始点坐标的初始化，

对事件类型包括后续监听的事件类型的初始化。(是手还是鼠标)

![image-20200714175831802](C:\Users\yqm\AppData\Roaming\Typora\typora-user-images\image-20200714175831802.png)

**move**

主要计算滑动距离，以及根据移动距离触发回弹

```javascript
private move(e: TouchEvent) {
    if (eventTypeMap[e.type] !== this.initiated) {
      return
    }
    this.beforeHandler(e, 'move')
    let point = (e.touches ? e.touches[0] : e) as Touch
    let deltaX = point.pageX - this.pointX
    let deltaY = point.pageY - this.pointY
    this.pointX = point.pageX
    this.pointY = point.pageY

    if (
      this.hooks.trigger(this.hooks.eventTypes.move, {
        deltaX,
        deltaY,
        e
      })
    ) {
      return
    }
    // auto end when out of wrapper
    let scrollLeft =
      document.documentElement.scrollLeft ||
      window.pageXOffset ||
      document.body.scrollLeft
    let scrollTop =
      document.documentElement.scrollTop ||
      window.pageYOffset ||
      document.body.scrollTop

    let pX = this.pointX - scrollLeft
    let pY = this.pointY - scrollTop

    if (
      pX >
        document.documentElement.clientWidth -
          this.options.momentumLimitDistance ||
      pX < this.options.momentumLimitDistance ||
      pY < this.options.momentumLimitDistance ||
      pY >
        document.documentElement.clientHeight -
          this.options.momentumLimitDistance
    ) {
      this.end(e)
    }
  }
```

**end**

结束这一次的滑动操作

![image-20200714192737313](C:\Users\yqm\AppData\Roaming\Typora\typora-user-images\image-20200714192737313.png)

