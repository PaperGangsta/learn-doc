# IndexedDB

### IndexDB简介

这里直接引用一下MDN中对`IndexedDB`使用上的注意事项

> IndexedDB API是强大的，但对于简单的情况可能看起来太复杂。如果你更喜欢一个简单的API，请尝试类库，如[localForage](https://localforage.github.io/localForage/), [dexie.js](http://www.dexie.org/), [PouchDB](https://pouchdb.com/), [idb](https://www.npmjs.com/package/idb), [idb-keyval](https://www.npmjs.com/package/idb-keyval), [JsStore](https://jsstore.net/) and [lovefield](https://github.com/google/lovefield) ，这些使IndexedDB更具工程友好性。

*如果像MDN中所表达的朋友可以看完基本介绍然后就略过本文剩余部分的内容。*



**IndexedDB**:

* 是一个事务型数据库；

* 是一个基于JavaScript的面向对象数据库;

  ( IndexedDB 里面没有表和记录的概念，它的数据的最小单位是 JavaScript 对象（object）)

* 是一个浏览器环境提供的本地数据库;
*  是一个非关系型的数据库。

它能够存储和检索用`键`索引的对象。

但是IndexedDB存储的对象(Object)是结构化的数据而且支持对象的嵌套。(和js完美结合)

也就是说它不能够存储像function那样的非结构化的数据。

相比于大家经常使用的`localStorage`它却需要把数据格式化为string才能存储。

>**注意:** IndexedDB遵循同源策略。所以当你在某个域名下操作储存数据的时候，你不能操作其他域名下的数据。

另外需要注意的一点就是,IndexedDB执行的操作是**异步**的，其目的是为了防止阻塞应用程序。

最初的IndexedDB包含了同步和异步的API，但目前同步的API除了在Web Workers中使用以外，已经被移除了。

MDN中给出的移除的解释:

> 因为尚不清晰是否需要。但如果Web开发人员有足够的需求，则可以重新引入同步API。