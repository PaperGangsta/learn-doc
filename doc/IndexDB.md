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



### **IndexedDB使用**

#### 创建数据库容器

如果要操作数据那么必须要有个数据库的容器。

数据库的所有的设计包括内容都要建立在容器上

IndexedDB 数据库打开或者创建成功后通过 onsuccess 事件回调获取到了数据库容器

```javascript
let open = window.indexedDB.open('mydb',1);
open.onsuccess = (e)=>{
   /*IndexedDB 数据库的事件回调中都会在事件对象中带有数据库容器对象，
    可以通过 event.target.result 获取.
    */
    let db = e.target.result
}
```

#### 数据库版本

有些同学可能已经发现`window.indexedDB.open`包含2个参数，除了第一个参数为name以外，第二个参数就是其对应的版本号。

在你需要创建、添加、修改、删除一个或一些`objectStore`（下文中有讲解）的时候，就需要对应的更新版本。

假设此时有版本**1**和版本**2**，那么在你通过`open`打开数据库的时候2者是不同的。

所以你需要时时刻刻确保，自己的版本要使用正确才行。

而从实际的开发角度来讲，正是因为这个设计，所以版本的变更通常不会允许其发生在程序运行状态下，也就是说我们通常不会通过程序去更改其版本。

每次升级应该发生在代码的更新迭代中，手动的为了修改数据库的结构从而去修改版本。



#### **对象仓库**

`objectStore`是IndexedDB中的核心概念。它是数据的存储仓库,在关系型数据库中我们通常把它称之为`表`。这个所谓的“表”内存放着具有相关性的数据，相关具体是指，所有存放在objectStore这个“表”内的数据都需要具备一个属性名可以理解为是**主键**。这个在IndexedDB中被称为`keyPath`。

> 注:关系型数据库表中不一定要有主键，但objectStore中一定需要keyPath

如果不存在这个属性并且也没有设置为自增(autoIncrement)就会报错(设置为自增时如果没有会自动创建)

数据表表要在数据库的基础上进行，同样的objectStore的操作也要在IndexedDB的容器上进行，具体是指在IndexedDB的`onupgradeneeded `事件中进行操作。

还要说明的是,objectStore不允许同名。如果同名了会报错。

为了防止程序报错而导致的bug,我们需要在创建时对objectStore的名字进行重名检查。

```javascript
let open = window.indexedDB.open('mydb',2);
open.onupgradeneeded = (e)=>{
    let db = e.target.result;
    let objectStore;
    if(!db.objectStoreNames.contains('myObjectStore')){
        objectStore = db.createObjectStore('myObjectStore',{keyPath:'id'})
    }
    else{
        objectStore = e.target.transaction.objectStore('myObjectStore');
    }
}
```

> 一旦objectStore被创建了,那么它的`name`和`keyPath`就无法被修改



#### 索引

IndexedDB中索引和关系型数据库中的索引不同，其的目的在于,为`objectStore`提供除了`keyPath`以外的其他的检索方式。

通常在IndexedDB中获取方式会使用`objectStore.get()`的方式获取数据，其默认值为`keyPath`所对的值。

所以IndexedDB才需要方法来建立新的索引检索方式

下面我们创建一条名为id的索引，其索引的属性为id，并把它的值设为唯一的。

```javascript
let open = window.indexedDB.open('mydb',3);
open.onupgradeneeded = (e)=>{
    let db = e.target.result;
    let objectStore;
    if(!db.objectStoreNames.contains('myObjectStore')){
        objectStore = db.createObjectStore('myObjectStore',{keyPath:'id'})
    }
    else{
        objectStore = e.target.transaction.objectStore('myObjectStore');
    }
     objectStore.createIndex('id','id',{unique:true});
}
```

`objectStore.createIndex（indexName,keyPath,options）`

- indexName：索引名字自己定义。
- keyPath：keyPath对应的是 object 的属性名， 必须和 object 的属性对应。
- options: unique||multiEntry||locale。

#### **事务**

数据库事务是一个很常见的概念。

还是那个最为经典的例子:

转账是生活中常见的操作,比如从A账户转账100元到B账号。站在用户角度而言,这是一个逻辑上的单一操作,然而在数据库系统中,至少会分成两个步骤来完成:

- 1.将A账户的金额减少100元
- 2.将B账户的金额增加100元。

![img](https://img2018.cnblogs.com/blog/1422237/201811/1422237-20181122103041637-782687204.png)

在这个过程中可能会出现以下问题:

- 1.转账操作的第一步执行成功,A账户上的钱减少了100元,但是第二步执行失败或者未执行便发生系统崩溃,导致B账户并没有相应增加100元。
- 2.转账操作刚完成就发生系统崩溃,系统重启恢复时丢失了崩溃前的转账记录。
- 3.同时又另一个用户转账给B账户,由于同时对B账户进行操作,导致B账户金额出现异常。

为了便于解决这些问题,需要引入数据库事务的概念。



**数据库事务：是构成单一逻辑工作单元的操作集合**

其目的是为了使系统能够更方便的进行故障恢复以及并发控制,从而保证数据库状态的一致性。

**事务的ACID特性**:

**原子性(Atomicity)**:事务中的所有操作作为一个整体像原子一样不可分割，要么全部成功,要么全部失败。

**一致性(Consistency):**事务的执行结果必须使数据库从一个一致性状态到另一个一致性状态。一致性状态是指:1.系统的状态满足数据的完整性约束(主码,参照完整性,check约束等) 2.系统的状态反应数据库本应描述的现实世界的真实状态,比如转账前后两个账户的金额总和应该保持不变。

**隔离性(Isolation)**:并发执行的事务不会相互影响,其对数据库的影响和它们串行执行时一样。比如多个用户同时往一个账户转账,最后账户的结果应该和他们按先后次序转账的结果一样。

**持久性(Durability)**:事务一旦提交,其对数据库的更新就是持久的。任何事务或系统故障都不会导致数据丢失。

在事务的ACID特性中,C即一致性是事务的根本追求,而对数据一致性的破坏主要来自两个方面

- 1.事务的并发执行
- 2.事务故障或系统故障

![img](https://img2018.cnblogs.com/blog/1422237/201811/1422237-20181122103102223-1059881337.png)

下面就来说一下IndexedDB中的事务。其目的与设计理念是相同的就不再累述。

**为了规避异常,IndexedDB保证了操作的按照一定顺序执行:**

而且强制规定了任何 object 读写的操作都必须在一个事务中进行

并且在代码层面必须通过指定的`db.transcaction()`方法向容器提出具体的事务需求，才能对`objectStore`进行操作

`transcaction(storeNames,mode)`

* storeNames:一个数组指定了需要操作的objectStore的名字(空数组会抛异常) *(必选)*

* mode:定义读写权限。默认权限为只读 `'readonly'`||`'readwrite'` *(可选)*

  > mode参数优化策略:除非确实要写入数据库,否则不要打开读写事务，会使速度变慢。

```javascript
let open = window.indexedDB.open('mydb', 4)
open.onsuccess = e => {
  let db = e.target.result
  let transaction = db.transaction(
    ['myObjectStore'],
    'readonly'
  )
  let objectStore = transaction.objectStore('myObjectStore')
  let objectRequest = objectStore.get('xiaomi_ng')
  objectRequest.onsuccess = e => {
    let object = e.target.result
  }
}
```

在这里阐述一下一个比较容易混淆的地方:

 db ，只能得到 `objectStore `的名字列表

`transaction.objectStore()` 方法可以获取想要操作的 `objectStore`的实例 



#### 游标

**什么是游标?**

简单来说: 

游标是一种临时的数据库对象，即可以用来存放在数据库表中的数据行副本，也可以指向存储在数据库中的数据行的指针。

游标提供了在逐行的基础上操作表中数据的方法。

或者说是一个用来记录数组正在被操作的某个下标位置的变量。

这里专门说游标的原因是因为，IndexedDB**没有**直接提供类似的可以获取objectStore中全部的object的方法。

所以我们需要通过游标来解决这个问题:

```javascript
let open = window.indexedDB.open('mydb',5);
open.onsuccess = (e)=>{
    let db = e.target.result
    let transaction = db.transaction(
      ['myObjectStore'],
      'readonly'
    );
    let objectStore = transaction.objectStore('myObjectStore');
    //打开一个游标
    //objectStore.openCursor() 方法打开游标机制，该方法返回一个 Request 对象，
    //在这个 Request 对象的 onsuccess 回调中，
    //如果 cursor 没有遍历完所有 object，
    //那么通过执行 cursor.continue() 来让游标滑动到下一个 object，
    //onsucess 回调会被再次触发。
    //而如果所有的 object 都遍历完了，cursor 变量会是 undefined。
    let cursorRequest = objectStore.openCursor();//cursor 光标、游标
    //results 变量，它的声明必须放在 onsuccess 回调函数的外部，因为该回调函数会在遍历过程中反复执行。
    let results = [];
    cursorRequest.onsuccess = (e)=>{
        let cursor = e.target.result;
        if(cursor){
            results.push(cursor.value)
            cursor.continue()
        }else{
            console.log(results);
        }
    }
}
```



#### IndexedDB 的增删改查

**增**

`objectStore.add(object)`

这个就是字面意思不再多解释这个参数了。但要注意的是object的"主键"`keyPath`需要唯一,在这个对象中具体的是指`id`这个属性(具体原因是因为上面我们设置过了,忘记了可以翻上去看一看)。

但是,**尽量不要使用这个方法**：

理由如下。如果上述说的唯一值重复了，那么程序就会抛异常报错，在生产环境中我们要尽量避免此类的事情发生，以免造成程序崩溃。

所以这里我们通常会用`objectStore.put()`来代替（具体见"**改**"，这个版块是"**增**"）

```javascript
let open = window.indexedDB.open('mydb', 6)
open.onupgradeneeded = e => {
  let db = e.target.result
  let transaction = db.transaction(
    ['myObjectStore'],
    'readwrite'
  )
  let objectStore = transaction.objectStore('myObjectStore')
  objectStore.add({
    id: 'Xiaomi_ng',
    name: 'Xiao Ming',
  })
}
```

**删**

```javascript
let open = window.indexedDB.open('mydb', 6)
open.onupgradeneeded = e => {
  let db = e.target.result
  let transaction = db.transaction(
    ['myObjectStore'],
    'readwrite'
  )
  let objectStore = transaction.objectStore('myObjectStore')
  objectStore.delete('Xiaomi_ng')
}
```

大家可以发现这个代码格式基本上是固定的。

这样写是不是很啰嗦呀？？？

那我们是不是就可以偷个懒封装一下，然后传参调方法就行了,嘻嘻嘻。



**改**

偷个懒，偷个懒，重复性的东西不再放了。

这里具体要介绍一下`objectStore.put(object)`和`objectStore.add(object)`方法的区别:

add的已经说过了(具体见"**增**"，这个版块是"**改**"),这里说一下put:

* 如果`objectStore`中存在该id,那么更新;若没有则添加；

* 当设置了`autoIncrement`的时候，put()方法需要传第二个参数--`keyPath`(主键)的值，从而确定要更新的究竟是哪一个`keyPath`所对应的对象。

  如果不传在主键自增的情况下,很有很能会在`objectStore`中添加进去一个新的对象。

  这就是问题所在，所以这里强烈的建议，传的时候保证object中存在`keyPath`的值

```javascript
objectStore.put({
    id: 'Xiaomi_ng',
    name: 'Da Ming',
})
```

**查**

查还是要简单的说一下的，开头的时候有提到整个过程都是异步操作的。

这里通过`objectStore.get()`得到的是一个`Request`对象

*如果不清楚`Request`是什么的同学，可以查阅一下[MDN](https://developer.mozilla.org/zh-CN/docs/Web/API/Request/Request)中的讲解,写的十分的清楚(中文)大概10min左右能看完内容不多。*

所以需要在其`onsuccess`事件中才能得到对应的结果

```javascript
let objectRequest = objectStore.get('Xiaomi_ng')
objectRequest.onsuccess = e => {
    // 获取到的数据
	let object = e.target.result
}
```

