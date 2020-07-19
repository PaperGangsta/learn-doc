# Kotlin

## 基础

### 函数

```kotlin
fun main(a:Int,b:Int):Int{
  //返回Int  
}

fun sum(a:Int,b:Int){
    //返回类型自动腿短
}

fun printSum(a:Int,b:Int):Unit{
    //返回无意义的值
    println("sum of $a and $b is ${a+b}")
}

//Unit可以被省略
fun printSum(a:Int,b:Int){
    //返回无意义的值
    println("sum of $a and $b is ${a+b}")
}
```



### 变量

```kotlin
//val为只读局部变量--只能赋值一次
val a:Int = 1
val v = 2 //自动推断Int类型
val c: Int //如果无初始值,初始类型不能被省略(变量声明)
c = 1 //变量赋值

//可重新赋值的变量 var
var x = 1
x += 1
```



### 字符串模板

```kotlin
var a = 1

var s1  = "a is $a"
var s2 = "${s1.replace("is","was"))},but now is $a"
```



### 空值与null检测

某个变量的值可以为 *null* 的时候，必须在声明处的类型后添加 `?` 来标识该引用可为空。

如果 `str` 的内容不是数字返回 *null*：

```kotlin
fun parseInt(str:String):Int?{

}
```



### 类型自动转换

某个变量的值可以为 *null* 的时候，必须在声明处的类型后添加 `?` 来标识该引用可为空。

如果 `str` 的内容不是数字返回 *null*：

```kotlin
fun getStringLength(obj: Any): Int? {
    if (obj is String) {
        // `obj` 在该条件分支内自动转换成 `String`
        return obj.length
    }

    // 在离开类型检测分支后，`obj` 仍然是 `Any` 类型
    return null
}
```

**Kotlin中判断字符串或对象是否相等可以直接使用 `==`**



### 循环

#### 区间

```kotlin
fun main(){
    for(i in 0..10){
        print(i)
    }
}

//0..10 双端闭区间
//0 until 10 左闭右开 [0,10)

//step跳过某元素
fun main(){
    for(i in 0 until 10 step 2){
        printLn(i)
    }
}

//.. 和 until 都要求左边<右边
// downTo可以构建一个降序区间 如 10 downTo 1 => [10,1]

```



## 面向对象

Kotlin中默认权限是`public`

`internal`同一模块中的类可见

Kotlin中无`default`



 Kotlin默认所有非抽象类都不可以被继承

`open`关键字--使类可以被继承

`：`用于继承

`（）`表示继承类构造函数的传参,即使是无参构造函数也不能不写括号

```kotlin
class Student:Person(){
	
}
```

### 主构造函数和次构造函数

#### 主构造函数

沒有函数体,直接定义在类名后面即可。

如果想在主构造函数中写入一些逻辑,需要在结构体`init`中写,所有主构造函数中的逻辑都可以写在里面

```kotlin
class student(val sno: String, val grade: Int) : Person(){
    init(){
        println(sno)
        println(grade)
    }
}
```

继承父类的构造函数通过`()`中的参数指定。

#### 次构造函数

当一个类既有主构造函数也有次构造函数的时候，所有的次构造函数都必须调用主构造函数。

```kotlin
class Student(val sno: String , val grade :Int,name:String ,age:Int):person(name,age){
    constructor(name: String,age:Int):this("",0,name,age){
        
    }
}
```



没有主构造函数,那么继承类的时候就不用再加上括号了



### 接口

和Java中一样不多写了



### 数据类和单例类

#### 数据类

```kotlin
data class CellPhone(val brand: String, val price :Double)
```

Kotlin会根据主构造函数中的参数,自动生成`equals()`、`hashCode()`、`toString()`等方法。



#### 单例类

Kotlin会自动帮我们创建一个Singleton类的实例,并保证全局只存在一个实例。

```kotlin
object Singleton{}
```



### Lambda

#### 集合

`listof()`创建一个不可变集合

`mutableListOf()`创建一个可变集合



#### 集合的函数式api

通常不建议在Lambda中写过长的代码

**Lambda**语法结构：

```kotlin
{参数名1:参数类型, 参数名2:参数类型 -> 函数体}
```

`->`符号表示参数列表的结束和函数体的开始

函数体中最后一行代码会成为函数体的返回值



```kotlin
val list = listOf("Apple","Banana","Orange")
val lambda = {fruit:String -> fruit.length}
var maxLengthFruit = list.maxBy(lambda)
```



如果Lambda的参数列表中只有一个参数的时候可以将其简化为`it`

```kotlin
var maxLengthFruit = list.maxBy{it.length}
```