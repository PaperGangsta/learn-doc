# Cache Api

`Cache`接口为缓存的Request/Response对象对提供存储机制。

例如：作为ServiceWorker的生命周期中的一部分。

通过查阅[MDN](https://developer.mozilla.org/zh-CN/docs/Web/API/Cache)发现:

> 尽管Cache是定义在了ServiceWorker标准中，但它不必一定来配合ServiceWorker来使用

Cache API 提供了一系列方法实现了请求响应对象的缓存管理，它可以作为资源请求响应的缓存仓库，为 Service Worker 实现离线缓存提供基础支持。

```JavaScript
//判断浏览器是否支持Cache Api
if('caches' in self){
    console.log('环境支持 Cache Api')
}
```

**开启Cache仓库（异步)**

返回的 Promise 对象在 resolve 时会返回成功打开的 Cache 对象。

```javascript
caches.open(cacheName).then(cache=>{});// cacheName(String)表示要打开的 Cache 对象的名称。
```

<img src="C:\Users\yqm\AppData\Roaming\Typora\typora-user-images\image-20200710101654436.png" alt="image-20200710101654436" style="zoom:80%;" />

Chrome打开即可发现,我这里定义了一个名为“myCache”的仓库



然后，我们来看看Cache的都有什么方法供我们使用:

| [`Cache.match(request, options)`](https://developer.mozilla.org/zh-CN/docs/Web/API/Cache/match) | 返回一个 [`Promise`](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/Promise)对象，resolve的结果是跟 [`Cache`](https://developer.mozilla.org/zh-CN/docs/Web/API/Cache) 对象匹配的第一个已经缓存的请求。 |
| :----------------------------------------------------------- | :----------------------------------------------------------- |
| [`Cache.matchAll(request, options)`](https://developer.mozilla.org/zh-CN/docs/Web/API/Cache/matchAll) | 返回一个[`Promise`](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/Promise) 对象，resolve的结果是跟[`Cache`](https://developer.mozilla.org/zh-CN/docs/Web/API/Cache)对象匹配的所有请求组成的数组。 |
| [`Cache.add(request)`](https://developer.mozilla.org/zh-CN/docs/Web/API/Cache/add) | 抓取这个URL, 检索并把返回的response对象添加到给定的Cache对象.这在功能上等同于调用 fetch(), 然后使用 Cache.put() 将response添加到cache中. |
| [`Cache.addAll(requests)`](https://developer.mozilla.org/zh-CN/docs/Web/API/Cache/addAll) | 抓取一个URL数组，检索并把返回的response对象添加到给定的Cache对象 |
| [`Cache.put(request, response)`](https://developer.mozilla.org/zh-CN/docs/Web/API/Cache/put) | 同时抓取一个请求及其响应，并将其添加到给定的cache            |
| [`Cache.delete(request, options)`](https://developer.mozilla.org/zh-CN/docs/Web/API/Cache/delete) | 搜索key值为request的[`Cache`](https://developer.mozilla.org/zh-CN/docs/Web/API/Cache) 条目。如果找到，则删除该[`Cache`](https://developer.mozilla.org/zh-CN/docs/Web/API/Cache) 条目，并且返回一个resolve为true的[`Promise`](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/Promise)对象；如果未找到，则返回一个resolve为false的[`Promise`](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/Promise)对象。 |
| [`Cache.keys(request, options)`](https://developer.mozilla.org/zh-CN/docs/Web/API/Cache/keys) | 返回一个[`Promise`](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/Promise)对象，resolve的结果是[`Cache`](https://developer.mozilla.org/zh-CN/docs/Web/API/Cache)对象key值组成的数组。 |

**在仓库内添加缓存**

```javascript
cache.put(
    new Request('../img/cityCard_03.jpg'),
    new Response(JSON.stringify({code:'success'}))
);
```



<img src="C:\Users\yqm\AppData\Roaming\Typora\typora-user-images\image-20200710103811597.png" alt="image-20200710103811597" style="zoom: 80%;" />

**结合fetch**

```javascript
fetch('../img/cityCard_03.jpg').then(response=>{
    if(response.ok){
        cache.put(new Request('../img/cityCard_03.jpg'),response);
    }
})
```



<img src="C:\Users\yqm\AppData\Roaming\Typora\typora-user-images\image-20200710104214897.png" alt="image-20200710104214897" style="zoom:50%;" />



**Cache.add(request) 和 Cache.addAll(requests)**

`add()` 只能请求和缓存一个资源，而 `addAll()` 能够抓取并缓存多个资源。

`add()` 和 `addAll()` 方法会缓存 Response.ok 为 true 的响应。

请求跨域资源返回的不透明的 Response 对象，也会缓存下来。

```javascript
cache.add('../img/cityCard_03.jpg').then(()=>{})
cache.addAll([
    '../img/cityCard_03.jpg',
    '../img/cityCard_04.jpg'
]).then(()=>{})
```



**cache.match() 和 cache.matchAll()**

上面有了增，现在来介绍一下查

根据方法名,顾名思义:

`match()` 会返回第一个匹配条件的缓存结果，而 `matchAll()` 则会返回所有满足匹配条件的缓存结果。

```javascript
cache.match('../img/cityCard_03.jpg').then(response => {
    if (response == null) {
        //未匹配到资源
    } else {
        console.log(response);
    }
})
cache.matchAll('../img/cityCard_03.jpg').then(responses => {
    if (!responses.length) {
        //未匹配到资源
    } else {
        console.log(responses);
    }
})
```

![image-20200710110041150](C:\Users\yqm\AppData\Roaming\Typora\typora-user-images\image-20200710110041150.png)

第二个参数`options`

我们可以通过传入第二个参数来控制match中的匹配过程,如下:

```javascript
cache.match('../img/cityCard_03.jpg?id=10', {ignoreSearch: true}).then(response => {
 if (response == null) {
         //未匹配到资源
     } else {
         console.log(response);
     }
})
```

![image-20200710110451140](C:\Users\yqm\AppData\Roaming\Typora\typora-user-images\image-20200710110451140.png)

**match.keys()**

`match()`、`matchAll()`是用来获取相应的，同样的cache为我们提供了获取匹配的请求的方法

```javascript
//不传入任何参数,默认返回当下Cache中所有的缓存的请求 
cache.keys().then(requests=>{
     if(!requests.length){
         //未匹配到
     }
     else{
         console.log(requests)
     }
 })
cache.keys('../img/cityCard_03.jpg', {ignoreSearch: true}).then(requests => {
  /* requests 可能包含
      ../img/cityCard_03.jpg
      ../img/cityCard_03.jpg?id=1
      ../img/cityCard_03.jpg?id=2 
      等等请求对象
      如果匹配不到任何请求，则返回空数组
	*/
})

```

![image-20200710110832346](C:\Users\yqm\AppData\Roaming\Typora\typora-user-images\image-20200710110832346.png)

**删除缓存**

`cache.delete()` 方法可以实现对缓存的清理

整个过程和上面的类似。

如果删除成功返回true，否则返回false

```javascript
cache.delete('../img/cityCard_03.jpg').then(success => {
    console.log(success)
})
cache.delete('/nonexistent.txt').then(success => {
    console.log(success)//false(删除失败)
})
cache.delete('../img/cityCard_03.jpg?id=10', {ignoreSearch: true}).then(success => {
    console.log(success)//true cityCard_03.jpg?id=10被删除
})
```

