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



### Options.ts

options中主要定义了基础选项。

如：横纵轴方向初始化。横向滚动、纵向滚动、自由滚动锁轴之类的。以及对一些基本事件的选项。边缘回弹动画，回弹时间的开关等。

```typescript
constructor() {
    this.startX = 0
    this.startY = 0
    this.scrollX = false
    this.scrollY = true
    this.freeScroll = false
    this.directionLockThreshold = 5
    this.eventPassthrough = EventPassthrough.None
    this.click = false
    this.dblclick = false
    this.tap = ''

    this.bounce = {
      top: true,
      bottom: true,
      left: true,
      right: true
    }
    this.bounceTime = 800

    this.momentum = true
    this.momentumLimitTime = 300
    this.momentumLimitDistance = 15

    this.swipeTime = 2500
    this.swipeBounceTime = 500

    this.deceleration = 0.0015

    this.flickLimitTime = 200
    this.flickLimitDistance = 100

    this.resizePolling = 60
    this.probeType = Probe.Default

    this.stopPropagation = false
    this.preventDefault = true
    this.preventDefaultException = {
      tagName: /^(INPUT|TEXTAREA|BUTTON|SELECT|AUDIO)$/
    }
    this.tagException = {
      tagName: /^TEXTAREA$/
    }

    this.HWCompositing = true
    this.useTransition = true

    this.bindToWrapper = false
    this.disableMouse = hasTouch
    this.disableTouch = !hasTouch
    this.autoBlur = true
  }
  merge(options?: { [key: string]: any }) {
    if (!options) return this
    for (let key in options) {
      this[key] = options[key]
    }
    return this
  }
```



```typescript
process() {
    this.translateZ =
      this.HWCompositing && hasPerspective ? ' translateZ(0)' : ''

    this.useTransition = this.useTransition && hasTransition

    this.preventDefault = !this.eventPassthrough && this.preventDefault

    this.resolveBounce()

    //增加锁轴的选项
    this.scrollX =
      this.eventPassthrough === EventPassthrough.Horizontal
        ? false
        : this.scrollX
    this.scrollY =
      this.eventPassthrough === EventPassthrough.Vertical ? false : this.scrollY

    // 根据锁轴的条件判断过程
    this.freeScroll = this.freeScroll && !this.eventPassthrough

    // 当自由滚动选项开启会覆盖掉之前锁轴的选项或者说优先级最高
    this.scrollX = this.freeScroll ? true : this.scrollX
    this.scrollY = this.freeScroll ? true : this.scrollY

    this.directionLockThreshold = this.eventPassthrough
      ? 0
      : this.directionLockThreshold

    return this
  }
```

### animater

#### Base.ts

Base是定义了一种规范，不太了解ts中的概念，类似于Java中的抽象类。

抽象类中定义了一系列的规范，和相对应的规范性的方法(抽象方法)

同时抽象类无法被实例化，它只是单纯提供规范而已。

实现类需要继承抽象类来实现具体的抽象方法但抽象类中可以有自己具体的实例化方法。

```javascript
export default abstract class Base

export default class Animation extends Base

export default class Transition extends Base
```

Base中定义了2个抽象方法。分别对应了animation和transition中需要实现的操作。

```typescript
 abstract move(
    startPoint: TranslaterPoint,
    endPoint: TranslaterPoint,
    time: number,
    easing: string | EaseFn,
    isSilent?: boolean
  ): void
  abstract stop(): void
```

#### Animation.ts

js实现滚动的动画效果

具体解释的会写在代码里面

```typescript
private animate(
    startPoint: TranslaterPoint,
    endPoint: TranslaterPoint,
    duration: number,
    easingFn: EaseFn
  ) {
    //记录了开始的时间
    let startTime = getNow()
    //定义目标时间
    let destTime = startTime + duration

    const step = () => {
      let now = getNow()
		//如果当前时间已经>=了目标时间
      //说明动画过程已经完成，可以移至终点
      if (now >= destTime) {
        this.translate(endPoint)

        this.hooks.trigger(this.hooks.eventTypes.move, endPoint)
        this.hooks.trigger(this.hooks.eventTypes.end, endPoint)
        return
      }

      now = (now - startTime) / duration
        //动画函数
      let easing = easingFn(now)
      //根据时间计算下一帧的新的目标点
      const newPoint = {} as { [key: string]: any }
      Object.keys(endPoint).forEach(key => {
        const startValue = startPoint[key]
        const endValue = endPoint[key]
        newPoint[key] = (endValue - startValue) * easing + startValue
      })
      this.translate(<TranslaterPoint>newPoint)

      if (this.pending) {
          //pending表示了当前的动画状态。
          //如果pending为false则动画结束
          //否则回调动画函数渲染下一帧
        this.timer = requestAnimationFrame(step)
      }

      if (this.options.probeType === Probe.Realtime) {
        this.hooks.trigger(this.hooks.eventTypes.move, newPoint)
      }
    }

    
    this.setPending(true)
    //取消动画的回调。完成动画
    cancelAnimationFrame(this.timer)
    step()
  }
```

在move()和stop()函数中定义了

```typescript
move(
    startPoint: TranslaterPoint,
    endPoint: TranslaterPoint,
    time: number,
    easingFn: EaseFn | string,
    isSlient?: boolean
  ) {
    //初始时time为0
    if (!time) {
      this.translate(endPoint)
      // 若在短时间内进行转换比如translateY 0->30->0
      // 则transitionend不会被触发
      // 所以我们需要强制的重拍来触发它
      this._reflow = this.content.offsetHeight
      // 如果静止则无需执行动作并直接结束
      if (isSlient) return

      this.hooks.trigger(this.hooks.eventTypes.move, endPoint)
      this.hooks.trigger(this.hooks.eventTypes.end, endPoint)
      return
    }
    this.animate(startPoint, endPoint, time, easingFn as EaseFn)
  }
  
stop() {
    //当开始执行stop函数时目前的状态当前还在动画函数回调的过程中(这很好理解因为还没有执行停止)
    if (this.pending) {
      this.setPending(false)
        //这里解绑事件或者说停止了动画效果完成了停止。
      cancelAnimationFrame(this.timer)
      const pos = this.translater.getComputedPosition()
      this.setForceStopped(true)

      if (this.hooks.trigger(this.hooks.eventTypes.beforeForceStop, pos)) {
        return
      }

      this.hooks.trigger(this.hooks.eventTypes.forceStop, pos)
    }
  }
```



### scroller

#### Scroller.ts

这里有点组件生命周期的味道。

所有的better-scroll中基本的功能的挂载也放在了这个中进行，

比如基本的动画功能，基本动作的派发，还有上面提到的转换功能

还有一个核心的计算函数`momentum`

同时这里定义了很多的钩子可以供开发者根据不同阶段的需求来使用。可以说大大的增强了组件在不同阶段的可用性。



```typescript
constructor(wrapper: HTMLElement, options: BScrollOptions) {
    this.hooks = new EventEmitter([
      'beforeStart',
      'beforeMove',
      'beforeScrollStart',
      'scrollStart',
      'scroll',
      'beforeEnd',
      'scrollEnd',
      'refresh',
      'touchEnd',
      'end',
      'flick',
      'scrollCancel',
      'momentum',
      'scrollTo',
      'ignoreDisMoveForSamePos',
      'scrollToElement',
      'resize'
    ])
    this.wrapper = wrapper
    this.content = wrapper.children[0] as HTMLElement
    this.options = options

    const { left = true, right = true, top = true, bottom = true } = this
      .options.bounce as BounceConfig
    // direction X
    this.scrollBehaviorX = new Behavior(
      wrapper,
      createBehaviorOptions(options, 'scrollX', [left, right], {
        size: 'width',
        position: 'left'
      })
    )
    // direction Y
    this.scrollBehaviorY = new Behavior(
      wrapper,
      createBehaviorOptions(options, 'scrollY', [top, bottom], {
        size: 'height',
        position: 'top'
      })
    )

    this.translater = new Translater(this.content)

    this.animater = createAnimater(this.content, this.translater, this.options)

    this.actionsHandler = new ActionsHandler(
      wrapper,
      createActionsHandlerOptions(this.options)
    )

    this.actions = new ScrollerActions(
      this.scrollBehaviorX,
      this.scrollBehaviorY,
      this.actionsHandler,
      this.animater,
      this.options
    )

    const resizeHandler = this.resize.bind(this)
    this.resizeRegister = new EventRegister(window, [
      {
        name: 'orientationchange',
        handler: resizeHandler
      },
      {
        name: 'resize',
        handler: resizeHandler
      }
    ])

    this.transitionEndRegister = new EventRegister(this.content, [
      {
        name: style.transitionEnd,
        handler: this.transitionEnd.bind(this)
      }
    ])

    this.init()
  }
```

#### **Behavior.ts**

根据定义将事件做一层包装。以及上文提到了当快速划动的时强制触发并通过momentum来计算形成动画。

以及对应的窗口变化的时候动态的对组件进行计算的具体的计算方法的实现。

```typescript
 constructor(public wrapper: HTMLElement, public options: Options) {
    this.hooks = new EventEmitter(['momentum', 'end'])
    this.content = this.wrapper.children[0] as HTMLElement
    this.currentPos = 0
    this.startPos = 0
  }
```

重新计算 BetterScroll，当 DOM 结构发生变化的时候务必要调用确保滚动的效果正常

```typescript
refresh() {
    const { size, position } = this.options.rect
    const isWrapperStatic =
      window.getComputedStyle(this.wrapper, null).position === 'static'
    const wrapperRect = getRect(this.wrapper)
    this.wrapperSize = wrapperRect[size]

    const contentRect = getRect(this.content)
    this.contentSize = contentRect[size]

    this.relativeOffset = contentRect[position]
    if (isWrapperStatic) {
      this.relativeOffset -= wrapperRect[position]
    }

    this.minScrollPos = 0
    this.maxScrollPos = this.wrapperSize - this.contentSize
    if (this.maxScrollPos < 0) {
      this.maxScrollPos -= this.relativeOffset
      this.minScrollPos = -this.relativeOffset
    }

    this.hasScroll =
      this.options.scrollable && this.maxScrollPos < this.minScrollPos
    if (!this.hasScroll) {
      this.maxScrollPos = this.minScrollPos
      this.contentSize = this.wrapperSize
    }

    this.direction = 0
  }
```



#### Actions.ts

action将钩子中的定义的方法按照ts定义的规范，自定义的封装为一个事件，从而派发出去。

绑定各个生命周期钩子的处理方法,并根据快速滚动的时间和滚动起始的时间戳判断是否达到了触发momentum函数。为后续的处理方式做出判断

```typescript
 constructor(
    scrollBehaviorX: Behavior,
    scrollBehaviorY: Behavior,
    actionsHandler: ActionsHandler,
    animater: Animater,
    options: BScrollOptions
  ) {
    this.hooks = new EventEmitter([
      'start',
      'beforeMove',
      'scrollStart',
      'scroll',
      'beforeEnd',
      'end',
      'scrollEnd'
    ])

    this.scrollBehaviorX = scrollBehaviorX
    this.scrollBehaviorY = scrollBehaviorY
    this.actionsHandler = actionsHandler
    this.animater = animater
    this.options = options

    this.directionLockAction = new DirectionLockAction(
      options.directionLockThreshold,
      options.freeScroll,
      options.eventPassthrough
    )

    this.enabled = true

    this.bindActionsHandler()
  }
```

#### DerectionLock.ts

Animation.ts中反复提到的锁轴，锁轴后续的完善性的操作。

很核心的一段代码计算判断方向的锁定。

确认当前的滚动的方向，并传递给后续的事件通过处理(即当前滚动方向沿着当前锁定的轴，状态正确)

```typescript
checkMovingDirection(absDistX: number, absDistY: number, e: TouchEvent) {
  this.computeDirectionLock(absDistX, absDistY)

  return this.handleEventPassthrough(e)
}
```

如果判断当前是在沿着某个方向(水平或垂直)在滚动，则将它锁定。

```typescript
private computeDirectionLock(absDistX: number, absDistY: number) {
    if (this.directionLocked === DirectionLock.Default && !this.freeScroll) {
      if (absDistX > absDistY + this.directionLockThreshold) {
        this.directionLocked = DirectionLock.Horizontal // lock horizontally
      } else if (absDistY >= absDistX + this.directionLockThreshold) {
        this.directionLocked = DirectionLock.Vertical // lock vertically
      } else {
        this.directionLocked = DirectionLock.None // no lock
      }
    }
  }
```

