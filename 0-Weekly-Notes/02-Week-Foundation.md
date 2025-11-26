# 第 2 周：Kotlin + UI 基础

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

Kotlin 函数调用时的命名参数
* 平常使用的是**位置参数**参数按位置传递，不需要参数名
* 命名参数，明确指定每个参数的名称
```kotlin
Text(
    text = msg.author,
    color = MaterialTheme.colorScheme.secondary, 
    style = MaterialTheme.typography.titleSmall
)
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

### 1.5.类与实例
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

创建实例时，构造函数分为有主构造函数和没有主构造函数，并且**init**块会在对象创建时，主构造函数执行后，自动执行，一个类可以有多个init块，它们会按照在类中出现的顺序依次执行。
1. 子类有主构造函数的情况，基类必须在主构造函数中立即初始化
2. 子类没有主构造函数的情况，必须在每一个二级构造函数中用`super`关键字初始化基类，Kotlin中构造函数的关键字为`constructor`。
```kotlin
constructor(标识符: 类型, ...) : super(值, ...)
```

#### 1.5.1.data关键字
用于创建DTO/POJO/POCO（Data Transfer Object）
data关键字用于修饰一个类，告诉Kotlin编译器，这个类是**专门用来保存数据的**，为我自动生成一些标准的功能。
#### 1.5.2.sealed关键字
Kotlin中用于定义**密封类**的关键字。
密封类：
1. 子类必须在同一文件中
2. 不能直接实例化密封类本身
3. 所有子类在编译时已知

实际上是语法糖🍬：允许在类体内定义子类，即使父类"尚未完成定义"
```kotlin
sealed class UiState {
    object Loading : UiState()
    data class Success(val newsList: List<NewsItem>) : UiState()
    data class Error(val message: String) : UiState()
}
```

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

### 1.12.委托关键字 by
`by`是Kotlin属性委托的语法，含义是将这个属性的setter和getter委托为xxx来处理
```kotlin
private val viewModel: NewsViewModel by viewModels()
```
### 1.13.try-catch异常处理
try尝试执行一段可能会抛出异常的代码，如果确实发生了异常，则用catch捕获，并进行处理，避免程序崩溃，最后还可以使用finally(可选)块执行无论如何都要执行的清理代码。

**注意**：捕获异常的顺序应该是从，具体 -> 一般
```kotlin
fun writeToFile(filename: String, text: String) {
    val writer = FileWriter(filename)
    try {
        writer.write(text) // 尝试写入，可能会失败
        println("写入成功")
    } catch (e: IOException) {
        println("写入失败: ${e.message}")
    } finally {
        // 无论写入成功还是失败，都必须关闭文件流，释放资源
        writer.close()
        println("文件流已关闭")
    }
}
```


## 2.Activity & Fragment 生命周期
### 2.1.Activity
类似于Qt的QMainWindow，主窗口类，应用的主要窗口/界面

| Activity 生命周期 | 与 Qt QMainWindow 类比 | 功能描述                            |
| :---------------- | :--------------------- | :------------------------------- |
| onCreate          | 构造函数                | 构造函数+初始化UI，仅调用一次        |
| onStart           | show()                 | 显示窗口                          |
| onResume          | 获得焦点                | 获得焦点，窗口激活，可交互           |
| onPause           | 失去焦点                | 失去焦点                          |
| onStop            | hide()                 | 隐藏窗口                          |
| onRestart         |                        | 使处于已停止状态的Activity恢复      |
| onDestroy         | 析构函数                | 析构函数，清理资源                  |

### 2.2.Fragment
类似于Qt中的控件如QWidget，QDialog
| 序号 | Fragment 生命周期      | 与 Qt 控件最贴切的类比                          | 功能描述（2025 年推荐做法）                                                                 |
| :--: | :--------------------- | :--------------------------------------------- | :------------------------------------------------------------------------------------------------- |
|  1   | **onAttach()**         | QObject 被添加到 parent（setParent）           | Fragment 与 Activity 建立关联，可安全获取 Activity（常用于接口强转或 activityViewModels）          |
|  2   | **onCreate()**         | 构造函数（QWidget 子类构造函数）                | Fragment 实例化完成，初始化 ViewModel、非 UI 数据、读取 savedInstanceState（不准碰 UI）           |
|  3   | **onCreateView()**     | QWidget::create() 或 inflate QML（布局加载）    | 创建并返回 Fragment 的视图层次（ViewBinding.inflate / LayoutInflater），只负责返回 root            |
|  4   | **onViewCreated()**    | QWidget::showEvent() 第一阶段（布局已完成）     | ★ 最重要！视图已创建完成，安全找控件、设置点击事件、observe LiveData/Flow、启动动画                |
|  5   | **onStart()**          | QWidget::showEvent()（真正显示到屏幕）         | Fragment 变得对用户可见（但可能被半透明 Activity 盖住），适合注册广播、传感器、前台服务            |
|  6   | **onResume()**         | QWidget::focusInEvent() + 窗口激活              | Fragment 完全可见且可交互（位于栈顶），开启摄像头、动画、定位、刷新 UI（用户真正看到并能操作）     |
|  7   | **onPause()**          | QWidget::focusOutEvent() 或窗口失去焦点         | 即将失去交互能力（另一个 Activity 弹起、按 Home），立即暂停动画、摄像头、定位等                     |
|  8   | **onStop()**           | QWidget::hideEvent()（完全不可见）              | Fragment 完全不可见，停止耗时任务、注销广播、前台服务等                                           |
|  9   | **onDestroyView()**    | QWidget 布局被清除（removeWidget 但对象未 delete）| ★ 必须清空 Binding！视图被销毁（切 Fragment、返回栈弹出），清理所有 View 相关资源，防止内存泄漏   |
| 10   | **onDestroy()**        | QObject::~QObject() 即将析构                   | Fragment 实例即将被销毁，极少写代码，清理全局资源（如线程池、单例引用）                           |
| 11   | **onDetach()**         | QObject::parent() 被设为 nullptr                | Fragment 与 Activity 彻底脱离，之后不能再访问 Activity，基本空着                                 |

## 3.常见布局（LinearLayout、ConstraintLayout）
### 3.1.LinearLayout(线性布局)
这种布局方式会将子视图按照水平/垂直方向依次排列。
- horizontal：水平排列（默认）
- vertical：垂直排列
结构简单，易于理解和使用，适合简单的线性排列场景`嵌套层级过深时会影响性能`，复杂布局难以实现。

### 3.2.ConstraintLayout(约束布局)
通过约束关系定位子视图，允许在平面上灵活摆放控件，有效减少布局嵌套，支持比例约束，角度约束等高级功能。

通过约束布局，定义控件之间的相对位置关系，减少层级嵌套。从而缓解嵌套布局导致的层级深，计算量大，耗时长，产生的渲染变慢，滑动卡顿的问题。
扁平化不是要完全消除嵌套，而是消除那些不必要的、只起包装作用的中间视图容器。

## 4.RecyclerView
### 4.1.RecyclerView概念
为一个功能强大且灵活的视图容器，可以用其高效的显示大量数据，列表项滚动出屏幕时，RecyclerView不会销毁它，而是放入回收池中等待复用，他的设计基于几个关键的概念。
| 名字                 | 功能                                                                                                                              |
| :------------------- | :-------------------------------------------------------------------------------------------------------------------------------- |
| RecyclerView         | 本身是一个ViewGroup，它本身就是视图，在屏幕上占据一块位置，负责每个列表项的摆放，与何时回收不可见项。                                  |
| Adapter              | Adapter是RecyclerView与数据源之间的桥梁，负责创建视图、将数据填充到已存在的ViewHolder中                                               |
| ViewHolder           | 是一个静态内部类，负责缓存视图的查找结果，每个列表项对应一个ViewHolder实例。它持有对列表项布局内各个子视图的引用。                         |
| LayoutManager        | 负责决定列表项如何排列在屏幕上，Android提供了几种默认实现，LinearLayoutManager(线性布局)，GridLayoutManager(网格布局)，StaggeredGridLayoutManager(交错的网格布局)。也可以自定义LayoutManager来实现特殊的布局，但比较复杂。 |
| ItemDecoration       | 负责绘制列表项之间的分割线、高亮、偏移。                                                                                            |
| ItemAnimator         | 负责处理列表项的动画效果，如当数据被添加、删除、移动时，项的出现/消失动画，默认提供简单的动画效果。                                       |

在使用Recycler之前先熟悉一下它的各组件之间关系。
`RecyclerView，Dataset，Adapter，ViewHolder，Item`
**一个ViewHolder对应一个Item，一个Adapter负责管理所有View Holder并bind Dataset中的一条数据到一个View Holder中。**
手机屏幕一次只能显示有限数量的项目，例如一次只能显示五条，RecyclerView只会创建略多于这个数量的ViewHolder，例如6,7个。之后循环使用。

### 4.2.使用RecyclerView步骤
实现RecyclerView的基本步骤
1. 确定列表/网格的外观，一般来说，可以使用RecyclerView库的某个标准布局管理器。
2. 设计列表中每个元素的外观和列表。根据设计扩展ViewHolder类。
3. 定义用于将您的数据与 ViewHolder 视图相关联的 Adapter。
此外，您还可以使用高级自定义选项根据自己的具体需求定制 RecyclerView。如设计状态改变动画，布局管理等。
#### 4.2.1.规划布局
RecyclerView 中的列表项由 LayoutManager 类负责排列。RecyclerView 库提供了三种布局管理器，用于处理最常见的布局情况：
* `LinearLayoutManager` 将各个项排列在一维列表中。
* `GridLayoutManager` 将各个项排列在二维网格中
  * 如果网格垂直排列，`GridLayoutManager` 会尽量使每行中所有元素的宽度和高度相同，但不同的行可以有不同的高度。
  * 如果网格水平排列，`GridLayoutManager` 会尽量使每列中所有元素的宽度和高度相同，但不同的列可以有不同的宽度。
* StaggeredGridLayoutManager 与 GridLayoutManager 类似，但不要求同一行中的列表项具有相同的高度（垂直网格有此要求）或同一列中的列表项具有相同的宽度（水平网格有此要求）。其结果是，同一行或同一列中的列表项可能会错落不齐。

#### 4.2.2.实现适配器和View Holder
确定布局后，您需要实现 **Adapter** 和 **ViewHolder**。这两个类配合使用，共同定义数据的显示方式。ViewHolder 是包含列表中各列表项的布局的 View 的封装容器。Adapter 会根据需要创建 ViewHolder 对象，还会为这些视图设置数据。将视图与其数据相关联的过程称为“绑定”。

定义适配器时，您需要替换三个关键方法：
* `onCreateViewHolder()`：每当 RecyclerView 需要创建新的 ViewHolder 时，它都会调用此方法。此方法会创建并初始化 ViewHolder 及其关联的 View，但不会填充视图的内容，因为 ViewHolder 此时尚未绑定到具体数据。
* `onBindViewHolder()`：RecyclerView 调用此方法将 ViewHolder 与数据相关联。此方法会提取适当的数据，并使用该数据填充 ViewHolder 的布局。例如，如果 RecyclerView 显示的是一个名称列表，该方法可能会在列表中查找适当的名称，并填充 ViewHolder 的 TextView widget。
* `getItemCount()`：RecyclerView 调用此方法来获取数据集的大小。例如，在通讯簿应用中，这可能是地址总数。 RecyclerView 使用此方法来确定什么时候没有更多的列表项可以显示。

### 4.3.RecyclerView具体实现
1.确定列表/网格布局
```kotlin
val recycler: RecyclerView = findViewById(R.id.recyclerView)
recycler.layoutManager = LinearLayoutManager(this)
```
2.实现Adapter类和ViewHolder类
ViewHolder类，继承自RecyclerView.ViewHolder，需要View进行初始化。在ViewHolder类具体处理根视图下的详细子试图
```kotlin
class MyViewHolder(view: View): RecyclerView.ViewHolder(view){
    val hintName: TextView = itemView.findViewById(R.id.hintName)
    val hintPhone: TextView = itemView.findViewById(R.id.hintPhone)
    val changeButton: Button = itemView.findViewById(R.id.changeButton)
}
```
Adapter类，继承自RecyclerView.Adapter<T>，需要数据集进行初始化，在类内需要重写`onCreateViewHolder`,`onBindViewHolder`,`getItemCount`,这三个函数(父类纯虚)。

```kotlin
class MyAdapter(private val dataList: List<Person>): RecyclerView.Adapter<MyViewHolder>(){
    override fun onCreateViewHolder(parent: ViewGroup, viewType: Int): MyViewHolder {
        // 创建View Holder，及关联View
        val itemView = LayoutInflater.from(parent.context).inflate(R.layout.item, parent, false)
        return MyViewHolder(itemView)
    }
    override fun onBindViewHolder(holder: MyViewHolder, position: Int) {
        val currentItem = dataList[position]
        holder.hintName.text= currentItem.name
        holder.hintPhone.text = currentItem.phone
    }

    override fun getItemCount(): Int {
        return dataList.size
    }
}
```
`LayoutInflater`是一个类，用于将XML布局文件转换成对于的View对象

`inflate`函数的三个参数：
* R.layout.item要加载的XML布局资源
* 新视图将要被添加到的父视图
* 是否要立即添加到父视图

3.主页面调用
```kotlin
val dataList: List<Person> = listOf(
    Person("a", "123"),
    Person("b", "456"),
    Person("c", "789"),
)
val adapter = MyAdapter(dataList)
recycler.adapter = adapter
```

## 5.小demo
### 5.1.完成一个「名片展示 App」：包含顶部头像、姓名、简介
<img width="213" height="173" alt="image" src="https://github.com/user-attachments/assets/423094b1-79e2-4786-8cbd-bfdce8d0c48a" />

### 5.2.写一个「联系人列表 Demo」：用RecyclerView 展示一组名字，点击后弹 Toast 提示。

## 6.遇到的问题
### 6.1.重启Android Studio后，遇到build.gradle.kts报错
<img width="99" height="20" alt="image" src="https://github.com/user-attachments/assets/23a3916b-14b3-4269-8c4d-46fb7997c6de" />

可能是因为缓存临时异常，Gradle状态不同步。可以通过ViSync Now强制重新去读取所有 Gradle 相关的配置文件，重新建立依赖关系、检查语法等。

<img width="125" height="170" alt="image" src="https://github.com/user-attachments/assets/f190e882-7c03-4542-b5b5-f2dffae26dbd" />

### 6.2.对图片重命名后报错
<img width="287" height="71" alt="image" src="https://github.com/user-attachments/assets/3d62fba7-2926-48d9-b9ba-d547c5110757" />

解决方案：资源文件的名称必须以字母开头，1.jpg开头有问题

### 6.3.ImageView未显示
ImageView默认不会起到占位作用，tools:srcCompat 只是 Android Studio 预览工具使用的占位图，仅在设计视图中显示，不会在实际运行时生效。
所以如果没有设置图片，在运行时会像没有加ImageView一样。
app:srcCompat = "@drawable/ic_launcher_background"
可以先设置一个默认图片。







