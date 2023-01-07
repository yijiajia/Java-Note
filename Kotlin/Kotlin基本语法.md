## 基本语法
### 1. 变量定义

* val : 不可变
* var ：可变
```
var a = 10  // 自动类型推导为Int类型
var a: Int = 10  // 显示指定类型
```

### 2. 函数定义
```
// max() 方法是Kotlin内部包，需要在类中import导入

// 定义函数名为methodName，参数为两个整数，返回值为Int
// 返回为空时可省略括号外的Int
fun methodName(num1 : Int , num2 : Int) : Int {
    return max(num1,num2)
}

// 可省略函数体，直接用等号，因为有类型推导，所以函数的返回类型Int也可以省略
fun largeNumber(num1: Int, num2: Int) = max(num1,num2)

```

### 3. 逻辑控制

* if

if  与Java使用方式类似，区别在于Kotlin的 If可以返回值
```
//使用方式 ①
fun largeNumber(num1: Int,num2: Int): Int {
    var value = 0
    if(num1 > num2) {
        value = num1
    }else {
        value = num2
    }
    return value
}
//使用方式 ②
fun largeNumber(num1: Int,num2: Int):Int {
   var value = if (num1 > num2) {
       num1
   }else {
       num2
   }
   return value
}


//也可以更精简下，使用方式 ③
fun largeNumber(num1: Int,num2: Int) = if (num1 > num2) {
       num1
   }else {
       num2
   }

//甚至
fun largeNumber(num1: Int,num2: Int) = if(num1>num2) num1 else num2

```

* when
> when 相当于Java的switch，用于多支匹配。格式为`匹配值 -> {执行逻辑}`。使用when 不需要使用break这样的语句来避免往下执行。
```
fun getScore(name: String) = when(name) {
    "Tom" -> 86
    "Jim" -> 77
    "Jack" -> 95
    else -> 0
}

// 也可以类型匹配
fun checkNum(num: Number) {
    when (num) {
        is Int -> println("number is Int")
        is Double -> println("number is Double")
        else -> println("not match")
    }
}

// when括号内也可以不带参数，自定义匹配值
fun getScore(name: String) = when {
    name.startsWith("Tom") -> 86
    name == "Jim" -> 92
    else -> 0
}

```


### 4. 循环语句

* while 循环
> 与Java使用方式一致

* for-in循环
> Kotlin使用.. 来表示区间范围，例如val range = 0..10，即表示range是0到10的区间

```
// 依次打印0到10的数字
fun main() {
    for (i in 0..10) {
        println(i)
    }
}
```

> 使用until来创建左闭右开的区间，例如val range = 0 until 10，即表示0到9的区间
```
// 依次打印0到9的数字
fun main() {
    for (i in 0 until 10) {
        println(i)
    }
}

// 打印0到10之间的偶数，step 2 表示每次累加 2
fun main() {
    for (i in 0 until 10 step 2) {
        println(i)
    }
}

// 使用downTo降序打印10到1
fun main() {
    for (i in 10 downTo 1) {
         println(i)
    }
}
```

## 面向对象

### 1. 类与对象

与Java基本类似
```
class Person {
    var name = ""
    var age = 0
    
    fun eat() {
        println(name + " is eating .He is " + age + "years old")
    }
}

fun  main() {
    var p = Person() // 实例化对象
    p.name = "Jack"
    p.age = 14
    p.eat() 
}
```

### 2. 继承与实现
> Kotlin使用冒号: 来表示继承类或实现接口

Kotlin除开抽象类，默认类都是不可继承的。如果可被继承需要加上open关键字才可以。



```
// 这样的类才是可继承的
open class Person {
    
}
// 继承Person类
class Student : Person() {

}
```
#### 2.1 主构造函数和次构造函数
Kotlin 的类包含**主构造函数以及次构造函数**。
每个类默认都是带有一个不带参数的主构造函数。

* 主构造函数
```
// 不带参数的主构造函数
open class Person {
    ...
}

// 带参数的主构造函数，使用init结构体可在实例化时执行里面的逻辑
class Student(var sno: String ,var grade: Int) : Person() {
    init {
        println("sno is " + sno)
        println("grade is " + grade)
    }
}
// 实例化Student时会输出学号以及成绩
val student = Student("a123",5)

// 若父类的主构造函数存在参数，则子类继承也需要写上父类构造函数的参数
open class Person(val name:String, val age:Int) {
    
}

// 子类继承时构造函数中的参数name和age不用指定变量类型，因为会自动跟父类主构造函数的参数类型绑定
class Student(val sno: String, val grade: Int, name :Sting,age: Int) : Person(name,age) {
    
}
```

* 次构造函数

Kotlin规定，当一个类既有主构造函数又有次构造函数时，所有的次构造函数都必须调用主构造 函数（包括间接调用）

```
class Student(val sno:String, val grade:Int, name:String, age: Int) : Person(name,age) {
    // 调用Student类的主构造函数
    constructor(name: String, age: Int) : this("", 0, name, age) {
        
    }
    constructor() : this("",0)
    
}

// 实例化
val stu1 = Student()
val stu2 = Student("Jack",18)
val stu3 = Student("a123", 5, "Jack", 18)
```

#### 2.2 接口
> 与Java类型，使用interface关键字
```
interface Study {
    fun read()
    fun write()
    // 定义函数体，默认实现
    fun talk() {
        println("talk talk me")
    }
}

// 实现接口
class Student(name: String, age: Int) : Person(name, age), Study {
    override fun read() {
        println("read book book")
    }
    override fun write() {
        println("write home work")
    }
}
```

**Java 与 Kotlin 可修饰符的对比**
![z7Y2b8.png](https://s1.ax1x.com/2022/12/16/z7Y2b8.png)

#### 2.3 数据类与单例类

数据类用data关键字修饰，表示该类已自动生成有equals()、hashCode()、toSting()等固定的方法。

```
data class CellPhone(val brand: String, val price: Double)
```

单例类使用object关键字修饰
```
object Singleton {
    fun test() {
        println("test is run...")
    }
}

// 调用单例类的方法，类似调用Java的static静态方法
fun main() {
    Singleton.test()
}
```



### 空指针检查
>https://book.kotlincn.net/text/null-safety.html





### 标准函数

1. let 函数
2. with函数
3. run函数
4. apply函数



### 静态方法

#### 1. 模拟静态方法

​	Kotlin 中可以通过单例类以及companion object来模拟静态方法

```kotlin
/**
* 可以直接通过Util.doAction()方法调用
**/
object Util {
    fun doAction() {
        println("do action")
    }
}

/**
* doAction1 为实例方法，需要Util2的对象才能调用
* doAction2 可以直接通过Util2.doAction2()调用
**/
class Util2 {
    fun doAction1() {
        println("do action1")
    }
    companion object {
          fun doAction2() {
        	println("do action2")
    	 }
    }
   
}
```

​	Util2中的doAction2 方法并非真正的静态方法，而是companion object 在Util2中生成的伴生类中的方法。调用Util2.doAction2() 实际上是调用Util2 中伴生类对象的doAction2方法。

​	以上的方法虽然能模拟静态方法的使用，不过这样定义在Java中并不能调用。为满足在Java中调用Kotlin中的静态方法，可以通过**注解**或者**顶层文件**来解决。

#### 2. 静态方法

##### 2.1 使用注解

使用@JvmStatic 注解来表示该方法是个静态方法，该注解只能配合 companion object 关键字使用。

```kotlin
class Util2 {
    fun doAction1() {
        println("do action1")
    }
    @JvmStatic 
    companion object {
          fun doAction2() {
        	println("do action2")
    	 }
    }
   
}
```

##### 2.2 顶层文件

新建--> New Kotlin Class/File --> File，新建一个Helper.kt的文件。在该文件中没有类，可以定义方法。里面的方法都为静态方法。在Kotlin中可以直接调用，在Java中通过 Helper.xxx()直接调用。

* **Helper.kt**

```kotlin
package com.example.helloworld

fun doSomeThing() {
    println("Helper doSomeThing")
}
```

* 在Kotlin 中直接调用

```kotlin
class NormalActivity : AppCompatActivity() {


    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        setContentView(R.layout.activity_normal)
        doSomeThing() // 直接调用
    }
}
```

* 在Java中直接调用

```java
class Test {
	public void test() {
		Hepler.doSomeThing();
	}
}
```



### 几个关键字

* const

  定义静态常量，需要配合单例类、companion object使用

* lateinit

  标记变量，表示该变量延迟初始化，可不必指定值。如果该变量后面也没初始化，那么会抛出UninitializedPropertyAccessException异常。

  ::变量名.isInitialized 可用于判断该变量是否已初始化。

* sealed class

  标记为密封类。表示该类的实现对应的条件全部都会被处理。常用于分支判断时，例如成功失败

  *  未标记前，需要额外增加else分支来满足编译

  ```kotlin
  interface Result
  class Success(val msg:String) : Result()
  class Failure(val error : Expception) : Result()
  
  fun getMsgResult(result : Result) = when(result) {
  	is Success -> result.msg
  	is Failure -> "Error is ${result.error.message}"
  	else -> throw IllegalArgumentException()
  }
  ```

  * 标记后，省略了else分支。当新增Result子类时，when分支必须新增该子类的判断，否则会编译报错

  ```kotlin
  sealed class Result
  class Success(val msg:String) : Result
  class Failure(val error : Expception) : Result
  
  fun getMsgResult(result : Result) = when(result) {
  	is Success -> result.msg
  	is Failure -> "Error is ${result.error.message}"
  }
  ```




### 扩展函数

> 扩展函数表示即使在不修改某个类的源码的情况下，仍然可以打 开这个类，向该类添加新的函数

语法结构：

```kotlin
fun ClassName.methodName(param1: Int, param2: Int): Int{
	return 0
}
```

假设需要在String类中添加统计字符个数的方法，那么就可以使用扩展函数来解决这个问题。

1. 创建String.kt 的顶层文件（建议是在哪个类中添加扩展函数就定义一个同名的Kotlin文件）

2. 在String.kt中编写以下的代码：

   ```kotlin
   fun String.lettersCount() : Int {
   	var count = 0
   	for (char in this) {
   		if(char.isLetter()) {
   			count++
   		}
   	}
   	return count
   }
   ```

3. 这样String类就有了lettersCount 这个扩展函数。使用方式如下：

   ```kotlin
   val count = "ABC123@#abc".lettersCount()
   ```

   

### 运算符重载

> 在Kotlin中可以对内置的运算符进行重载，对其赋值不一样的功能。比如两个Money对象直接相加就可以返回一个新的Money对象。

以加号运算符为例，如果实现两个对象相加的功能，语法结构如下：

```kotlin
class Obj {
	operator fun plus(obj: Obj): Obj {
		// 处理相加的逻辑
	}
}
```

假设两个Money对象相加，那么即可重载加号的函数

```kotlin
class Money(val value: Int) {
	operator fun plus(money: Money): Money {
		val sum = money.value + value
		return Money(sum)
	}
    // 也可以用不一样的参数重载，这里可以直接加数值
    operator fun plus(newValue: Int): Money {
        val sum = value + newValue
        return Money(sum)
    }
}
```

那么，两个Money对象相加，使用方式如下：

```kotlin
val money1 = Money(5)
val money2 = Money(10)
val money3 = money1 + money2
val money4 = money3 + 5
```



除了加号外，Kotlin还提供了许多可以重载的运算符。

[![zxLU8H.png](https://s1.ax1x.com/2022/12/26/zxLU8H.png)](https://imgse.com/i/zxLU8H)









### 相关网站

Kotlin官方文档中文版：
https://book.kotlincn.net/text/basic-syntax.html



