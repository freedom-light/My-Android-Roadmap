# 二.第二周内容

Kotlin文件以 .kt 为后缀

首先创建一个.kt文件方便练手

<img width="182" height="233" alt="image" src="https://github.com/user-attachments/assets/42c13442-b8aa-48b9-9797-23c86c4ffbe7" /><img width="195" height="234" alt="image" src="https://github.com/user-attachments/assets/4bb1be17-4468-42cc-81ca-2134700a7e26" />
<img width="142" height="114" alt="image" src="https://github.com/user-attachments/assets/a948bdb8-8a22-47fe-9eca-f97bdfe88597" />

## 1.Kotlin 基础语法
### 1.1.包声明
package：几乎所有的package名字都对应文件系统上的文件夹路径，从根本上避免了类名冲突的问题，可以理解为命名空间。
import：使用其他地方的代码，于C++include功能类型，不同的是更高效，不需要“复制粘贴”，更类似于创建快捷方式，编译单元大小不变，类文件只需编译一次，其他模块直接引用即可，并且可以通过封装类的访问权限来控制使用范围
```kotlin
package com.example.myapplication
import android.os.Bundle
```
### 1.2.输入输出
可以通过print打印，这样是不会自动换行的
- println打印是会自动换行的
- readln可以读取用户输入

$：是字符串模板，用来在字符串中插入变量/表达式的值  
```kotlin
fun main() {
    val res = add(a = 3, b = 5)
    println("res is $res")
}
```

对于简单的标识符直接使用$接标识符就可以了，如果是表达式或属性访问需要加上{}，例如如果没有加上大括号，则会认为找father这个标识符，但是father是一个对象，要想访问对象的某个属性的话，需要加上大括号。
```kotlin
println("$father.area")
println("${father .area}")
```
```kotlin
Father@17550481.area
200
```

如果$加简单的标识符加了大括号，编译器会报警告⚠

<img width="173" height="26" alt="image" src="https://github.com/user-attachments/assets/df009fcd-4aa0-4d23-9ee4-9e18c35bc596" />（移除花括号）

<img width="282" height="22" alt="image" src="https://github.com/user-attachments/assets/2942040b-579d-41ff-affc-4d16ccb27d65" />（移除单表达式字符模板）

### 1.3.函数定义

函数定义
* 使用关键字fun，参数格式为 参数:类型
```kotlin
fun add(a: Int, b: Int): Int {
    return a + b
}
```
表达式作为函数体，返回类型自动推导，但public方法必须明确写出返回类型，第二种方式没有错误，返回类型自动推导
```kotlin
fun sub(a: Int, b: Int) = a - b

public fun mul(a: Int, b: Int) = a * b
```

无返回值的函数
* 返回类型部分写Unit（可省略（public fun也同理））
```kotlin
public fun printNo() {
    println("no return")
}
```

可变长参数函数
* 在变量前加vararg，变长参数的本质实际上是接收多个单独的参数，需要在调用时加 * 展开运算符。
```kotlin
fun vars(vararg nums: Int) {
    for (v in nums) {
        println(v)
    }
}

fun main() {
    val nums: IntArray = intArrayOf(1, 2, 3, 4, 5)
    vars( ... nums = * nums) // 该行注释和省略号可能来自图片的遮挡
}
```

**出现Save ‘’ to dictionary是IDE[集成开发环境]认为拼写不规范建议修改，但不是错误。**
<img width="181" height="71" alt="image" src="https://github.com/user-attachments/assets/6823cee6-1ecb-4d49-84e0-611d5dc86d90" />

Lambda表达式
* 等号右边的是真正的lambda表达式，隐式返回最后表达式，自动捕获外部变量，将匿名函数赋值给一个命名变量，方便重复使用。
```kotlin
val addLambda: (Int, Int) -> Int = { x, y -> x + y }
//  类型声明     参数列表    返回类型  lambda主体
println(addLambda(10, 20))
```

表达式函数体
* fun 函数名(参数...) = 表达式
* =符号表示函数体是一个表达式，编译器会自动推断返回类型
```kotlin
fun add(a: Int, b: Int) = {a+b}
```

### 1.4.定义常量与变量
* 变量：var 标识符: 类型 = 初值
* 常量：val 标识符: 类型 = 初值
编译器支持自动类型判断，并且变量和常量都可以没有初始化值，但没有**初始化必须提供标识符类型，方便分配内存空间**，但是在**引用前必须初始化。**
可以在顶层声明，函数外创建全局变量

#### 1.4.1.lateinit
延迟初始化

#### 1.4.2.属性重写
属性重写使用override关键字，被重写的用open关键字修饰，属性必须具有兼容类型，每一个声明的属性都可以通过初始化程序或者getter方法被重写。可以用var重写val，反过来不行。
1. getter（获取器）：读取属性值，val和var都有，控制对外显示什么
2. setter（设置器）：修改属性值，只有var有，控制值如何被修改

### 1.5.创建类与实例
在kotlin中所有类都继承于Any类，Any是所有类的超类（父类）

Any默认提供三个函数：
1. equals( )：用于比较两个对象是否相同（比较内存地址）
2. hashCode( )：返回对象的哈希码，如果两个对象equals相同，则哈希码一定相同，但是哈希码相同，equals不一定相同（哈希冲突）
3. toString( )：返回对象的字符串表示，默认格式：类名@哈希码十六进制

子类必须满足父类的构造要求（**提供父类的所有参数**），但子类可以选择不将其全部设为自身的属性，而是通过固定值/计算值的方式给父类。这样做的原因显而易见，不给父类全部的属性它没法初始化。
```kotlin
class Son(height: Int) : Father(height, width = 20)
```

使用open关键字修饰需要被继承的类
```kotlin
open class Father(val height: Int, val width: Int){
    val area = height * width
}
```

创建实例时，构造函数分为有主构造函数和没有主构造函数
1. 子类有主构造函数的情况，基类必须在主构造函数中立即初始化
2. 子类没有主构造函数的情况，必须在每一个二级构造函数中用`super`关键字初始化基类，Kotlin中构造函数的关键字为`constructor`。
```kotlin
constructor(标识符: 类型, ...) : super(值, ...)
```

#### 1.5.1.data关键字
用于创建DTO/POJO/POCO（Data Transfer Object）
data关键字用于修饰一个类，告诉Kotlin编译器，这个类是**专门用来保存数据的**，为我自动生成一些标准的功能。

### 1.6.重写
#### 1.6.1.函数重写
在父类声明函数时，默认是final，不能被子类重写，对函数添加open修饰则可以重写，子类重写时使用override关键字。

### 1.7.for循环
1. for(val in 容器){}
2. for(val: Int in 容器){}
3. for(i in 容器.indices){} 编译器会将其优化为标准两分三段式，不会额外创建对象
4. for( (index, value) in 容器.withIndex() ){}每个元素是IndexedValue(包含值和索引)，会有轻微的开销。

### 1.8.when表达式
与C++中switch类似，when会按顺序检查，遇到第一个满足的条件执行对应的代码，之后立即退出，不会进入多个分支。
```kotlin
when(tmp) {
    1 -> println(1)
    2,3,4 -> println(2)
    in 4 <= … <= 6 -> println(3)
    is String -> println(4)
    else -> {
        println("123")
        println("456")
    }
}
```

### 1.9.标签
kotlin中`任何表达式`都可以用标签(lable)来标记，标签的格式为：标识符@，例如abc@，fooBar@都是有效的标签。要为一个表达式加标签，只需要在前面加标签即可。
```kotlin
loop@ for (i in 1 .. 10) {
    println(i)
}
```
会直接退出外层循环
```kotlin
loop@ for (i in 0 .. 9) {
    for (j in 1 .. 10) {
        if (j == 7) break@loop
    }
}
```
#### 1.9.1.隐式标签
隐式标签接受与该lambda表达式同名，加标签可以限制return
```kotlin
fun foo() {
    val tmp: IntArray = intArrayOf(1, 2, 3)
    tmp.forEach {
        println(it)
        if (it == 2) return@forEach
        println(it)
    }
}
```

### 1.10.空安全
#### 1.10.1.NULL检查机制
kotlin明确将可空性作为其类型系统的一部分，这意味着你可以显示声明哪些变量和属性允许为null，从而防止空指针异常(NPE)NullPointerException

#### 1.10.2.Elvis操作符 ?: 
专门用来处理空检查并提供默认值，val length = person?.name?.length ?: -1
1. 如果person为空，整个表达式结果就为空
2. 如果person不为空，person.name为空，整个表达式结果就为空
3. length同理，值 ?: 默认值，值为null则表达式结果为默认值，否则是原值

### 1.11.Gradle 构建脚本关键字
| 条目 | 说明 |
|:---|:---|
| dependencies | 声明当前项目（或模块）所依赖的代码库 |
| plugins | 声明和应用构建插件，这些插件会为项目添加额外的功能、任务和约定 | 








