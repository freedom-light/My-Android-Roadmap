# 第四周内容
## 1.MVVM 架构模式
MVVM架构，Model - View - ViewModel，是一种架构模式，核心思想是`数据驱动``关注点分离`，它与具体的编程语言无关。
1. Model：数据层只负责数据的获取与存储，对外提供数据接口。
2. ViewModel：处理业务逻辑并提供 View 所需的数据，暴露可观察数据给View
3. View：仅根据数据进行展示，并响应用户UI交互

## 2.Kotlin 协程（Coroutines）和 Flow
### 2.1.协程(Coroutines)
协程可以被认为是轻量级的线程，协程不绑定到特殊线程，可以在一个线程中暂停协程，然后在另一个线程中恢复。
`launch`是一个协程构建器，它会启动一个协程与其余代码独立运行。可以将launch中的代码块提取出来，将获得一个带有`suspend`修饰符的新函数
```kotlin
fun main() = runBlocking { // this: CoroutineScope
    launch { doWorld() }
    println("Hello")
}

// this is your first suspending function
suspend fun doWorld() {
    delay(1000L)
    println("World!")
}
```
`delay`是一个特殊的暂停函数，它会将协程暂停一段时间，暂停协程不会阻塞底层线程，而是允许运行其他协程，并在线程中执行对应代码。
```kotlin
delay(1000L) // 非阻塞延迟1秒 (default time unit is ms)
```
`runBlocking`是一个协程构建器，顾名思义，运行它的线程在调用期间会被阻塞直到内部所有协程执行完毕，但阻塞线程效率低下，通常不使用。`runBlocking`会自动建立相应的协程作用域。
* 一般用于教学示例中，可以让初学者专注于协程的学习

`coroutineScope`也是一个协程构建器，可以用于声明作用域，它会创建一个协程作用域，直到其中所有子协程完成后才会完成。

协程遵循结构化并发原则，相比于C++中实现多线程异步执行，Android里的协程使用起来有以下四点优势
1. 父子层级关系

2. 自动传播取消

3. 自动等待完成

4. 自动传播异常


### 2.2.Flow

## 3.依赖注入(Hilt)

## Navigation 组件（多页面跳转）


