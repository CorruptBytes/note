# 概述

- Kotlin类文件后缀为 .kt

# 函数

Kotlin弱化了Java中一切皆对象的强制约束，允许函数独立声明，而不是绑定在一个类中。

```
//无返回值函数
fun func_name() {

}

fun func_name(arg1: type1,arg2: type2): return_type {
  return return_value
}
//简单函数的简化写法
fun func_name(arg1: type1, arg2: type2) = expression 
```

```
fun main(args: Array<String>) {
  
}

// 也可以不声明Main函数接收参数

fun main() {

}
```

# 类与对象

```kotlin
//类定义
class Class_Name {
  //字段
  val field1: type1 = value //val声明只读属性，对象创建后属性无法重新赋值
  var field2: type2 = value //var声明可变属性
  //方法
  fun func_name() {}

}
//创建对象
val object = ClassName(value1,value2,...)
//调用方法
object.func_name()
```

### 构造器

Kotlin中类的构造器分为主构造器和次构造器。

主构造器在类签名处声明。

```
class ClassName constructor(val field1: type1, var field2: type2) {

}
//constructor关键字可省略
class ClassName(val field1: type1, var field2: type2) {
}
```

- 主构造器的参数有三种

|                  |                                       |
| ---------------- | ------------------------------------- |
| val name: String | 声明参数 + 只读属性 + getter          |
| var age: Int     | 声明参数 + 可变属性 + getter + setter |
| name: String     | 仅参数                                |

- 主构造器不能包含代码，自定义初始化逻辑需要放在init块中。init块中可以访问主构造器参数

```
class Person(val name: String, var age: Int) {

  init {
    // 主构造器执行时调用
    require(age >= 0) { "Age cannot be negative" }
    println("Person $name created")

  }
}
```

次构造器通过constructor关键字定义

 

### getter/setter

在 Kotlin 中，所有属性都具有getter/setter方法，属性的读写操作(`object.field`)会直接调用getter/setter。

```
class Person {

  var name: String = "Alice"

    get() {
      println("getter 被调用")

      return field

    }
    set(value) {

      println("setter 被调用")

      field = value
    }
}
```

- 如果不自定义getter/setter，编译器会自动生成
- `field` 表示属性的 backing field（幕后字段）。

### 数据类

类似于Java中的 record ，是一种为了减少数据类样板代码的语法糖，但比 record 限制更少，功能更强大。

```
data class Class_name(val/var field1:type1,...)
```

编译器会自动生成：

- `equals()`
- `hashCode()`
- `toString()`
- `componentN()`
- 属性访问器

### Lambda表达式

```
{ 参数列表 -> 函数体 }
```

- 默认情况下，Lambda表达式的最后一个表达式的值为返回值

### Lambda 类型声明

#### 普通 Lambda

```
var handler: (arg_type1, ...) -> return_type
```

#### 可空 Lambda

```
val handler: ((arg_type1) -> return_type)? = null
```

#### 无返回值 Lambda

```
val printer: (arg_type1) -> Unit
```

### 简化写法

Lambda 作为最后一个参数，可以移动到函数括号外：

```
list.filter() { x -> x > 0 }
```

进一步，如果只有一个参数，可以省略括号简写为：

```
list.filter { x -> x > 0 }
```

如果 Lambda 只有一个参数，可以省略参数声明，使用 `it`：

```
list.filter {
    it > 0
}
```

### 扩展方法

可以在不改变原有类代码的基础上，为类扩展新的方法

```
fun ClassName.methodName() {

}
```

# 变量

```
val name: String = "Alice"   
var age: Int = 25 
```

- val：不可变变量

- var：可变变量

Kotlin支持类型自动推断，通常可以省略类型声明，如 var age = 25 ,会被推导为 Int

# 数据类型

```
val str: String = "Hello"

val num: Int = 42

val pi: Double = 3.14

val flag: Boolean = true

val char: Char = 'A'
```

# 空安全

Kotlin中所有类型的变量都不能置空， 如果想要允许变量赋值为空，则可声明变量为type?表示可空类型，可空类型与普通类型的变量不属于同一类型，不能直接相互赋值

```
var nullableStr: String? = null

val length = nullableStr?.length ?: 0 
```

?. 表示安全调用，如果对象为空，则不调用方法，直接返回 null。对于可空类型的变量，必须使用安全调用才可以调用其方法。

## Elvis 操作符 

当左侧表达式为 null 时，返回右侧的默认值。常与安全调用配合，返回默认值

```
val result = nullableValue ?: defaultValue
```

# 流程控制语句

```
if (score >= 60) {
    println("Pass")
} else {
    println("Fail")
}

//如果分支中有多行语句，必须使用{}包裹，单行语句可以省略
when(day) {

    1 -> println("Monday")

    2 -> println("Tuesday")

    else -> println("Other")
}
//when可以作为表达式使用，当 when 作为表达式使用且分支具有多行语句时，该分支的值为最后一个语句的值

val result = when(num) {

    1 -> {
        val x = 10
        x
    }

    else -> 0
}

//Iterable可以是实现了Iterable接口的类或数组，Pair等

for (item in iterable) {

  println(item )

}

while (count < 10) {

  count++

}
```

# 集合

```
val list = listOf(1, 2, 3)   //不可变集合

val mutableList = mutableListOf(1, 2, 3)  //可变集合

val map = mapOf("key1" to "value1", "key2" to "value2")
```

# 区间

区间是Kotlin中的一种特殊内置类，它实现了 Iterable 接口，可用于 for 迭代。

## 语法糖

Kotlin 提供了 .. 语法糖用于快速创建区间对象，等同于 rangeTo() 方法

```
//创建一个 a 到 b 的闭区间
var range = a .. b
```

# 委托

将行为转交给另一个对象实现

## 类委托

把接口实现委托给另一个对象，避免手写样板代码。如需覆盖某个方法，可单独重写，未重写的方法继续走委托。

```
interface Printer {

    fun print()

}


class DefaultPrinter : Printer {

    override fun print() {
        println("默认打印")
    }

}
//// 所有 Printer 方法都自动转给 p

class MyPrinter(
    p: Printer
) : Printer by p
```

## 属性委托/变量委托

把属性/变量的 get / set 交给委托对象处理。

```
val name: String by lazy { "computed once" }
```

Kotlin提供了功能不同的标准委托：

| 委托                                                   | 作用                     |
| ------------------------------------------------------ | ------------------------ |
| lazy                                                   | 首次访问时初始化         |
| observable                                             | 值变化时回调             |
| vetoable                                               | 修改前校验               |
| Delegates.notNull()                                    | 类似 lateinit 的非空 var |
| operator fun getValue(...) /operator fun setValue(...) | 自定义属性委托           |

# 异常

## 异常捕获

```
try {

    val result = 10 / 0

} catch(e: ArithmeticException) {

    println("Division by zero")

} finally {

    println("Done")

}
```

# 其他

## 模板字符串

Kotlin支持通过 $var_name 在字符串中嵌入变量值。

```
print("Hello,$name")
```

