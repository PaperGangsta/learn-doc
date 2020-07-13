# ServiceWorker

### ServiceWorker简介

Service Worker是一种调度机制。是PWA的核心技术之一。

**为什么需要ServiceWorker?**

Web的一个困境就是--无论多么好的网页或者说Web APP ，如果在你没有网络或者说网络不太健康的情况下，用户的体验是极其的差的。而Service Worker的出现就是为了实现可以离线访问、稳定访问以及对应的静态资源缓存处理等。

**Service Worker是独立于浏览器主线程的线程**

我们通常说的Service Worker就是指Service Worker工作线程。它与浏览器的主线程是隔离的，并且它拥有自己独立的上下文。

**Service Worker特点**

* Service Worker的安全要求: **必须运行在HTTPS协议之下。**本地的localhost和127.0.0.1的非HTTPS也被认为安全
* 完全是异步实现的。内部接口通过Promise 实现，并且在Service Worker中不能直接操作DOM
* 可以离线工作在后台，可以实现消息推送，资源同步等功能
* 可以拦截并代理请求，并相应的处理的返回内容。



### Service Worker的基本使用

#### Service Worker的注册

Service Worker在注册时会有自己的`作用域`,如果使用不当，就会发生难以预测的作用域污染问题。

那么，什么是Service Worker的作用域呢？

##### 作用域

Service Worker的作用域其实就是一个 `path` ，它所指的就是Service Worker能够控制的页面的范畴。

比如我们现在随便指定一个作用域:`https://xiaoming/quick/start/`

那么Service Worker就可以控制

* `https://xiaoming/quick/start/first.html`
* `https://xiaoming/quick/start/second.html`
* `https://xiaoming/quick/start/third.html`
* ...

控制页面就是指Service Workerl可以处理页面的资源、请求，然后可以 实现离线缓存等功能，如果页面不再Service Worker 的作用域内，那么就无法被其控制。

```javascript

```

