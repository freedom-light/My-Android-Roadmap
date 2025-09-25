# 第四周内容
## 1.MVVM 架构模式
MVVM架构，Model - View - ViewModel，是一种架构模式，核心思想是`数据驱动`，`关注点分离`，它与具体的编程语言无关。
1. Model：数据层只负责数据的获取与存储，协调多个数据源，对外提供数据接口。
2. ViewModel：处理业务逻辑并提供 View 所需的数据，暴露可观察数据给View
3. View：仅根据数据进行展示，并响应用户UI交互
### 1.1.Model 数据层
#### 1.1.1.Repository
Repository(数据仓库)核心职责是对外提供统一的数据访问接口。
### 1.2.ViewModel 中间层
### 1.3.View UI层
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
```kotlin
// 父协程
val parentJob = CoroutineScope(Dispatchers.Main).launch {
    // 子协程1
    launch { 
        delay(1000)
        println("子任务1完成")
    }
    
    // 子协程2  
    launch {
        delay(2000)
        println("子任务2完成")
    }
}

// 结构：
// parentJob
//   ├── 子协程1
//   └── 子协程2
```
2. 自动传播取消
```kotlin
val parentJob = CoroutineScope(Dispatchers.Main).launch {
    val child1 = launch { 
        delay(5000) // 需要5秒
        println("这行不会执行")
    }
    
    val child2 = launch {
        delay(1000)
        println("子任务2完成")
    }
}

// 2秒后取消父协程
delay(2000)
parentJob.cancel() // 自动取消所有子协程！

// 输出：只有"子任务2完成"，child1被自动取消了
```
3. 自动等待完成
```kotlin
val parentJob = CoroutineScope(Dispatchers.Main).launch {
    val child1 = launch { 
        delay(1000)
        println("子任务1完成")
    }
    
    val child2 = launch {
        delay(2000) 
        println("子任务2完成")
    }
    
    // 父协程会自动等待所有子协程完成
    println("所有子任务完成后再执行这里")
}

// 输出：
// 子任务1完成
// 子任务2完成  
// 所有子任务完成后再执行这里
```
4. 自动传播异常
```kotlin
val parentJob = CoroutineScope(Dispatchers.Main).launch {
    launch {
        delay(1000)
        throw RuntimeException("子协程出错了！")
    }
    
    launch {
        delay(2000)
        println("这行不会执行")
    }
}

// 子协程的异常会自动传播给父协程
// 父协程会取消所有其他子协程
```
解决了传统方式启动许多线程导致的
1. 无法统一取消
2. 可能内存泄漏
3. 异常处理困难

### 2.2.Flow
常见的Flow有`StateFlow`和`SharedFlow`，它们是热数据流，只要该数据流被收集，或对它的任何其他引用在垃圾回收根中存在，该数据流就会一直存于内存中。

在使用的时候可以通过`asStateFlow()`将`MutableStateFlow`暴露为不可变`StateFlow`，MutableStateFlow由ViewModel层持有，暴露不可变的数据流给View层。还有一种方式是用`stateIn()`将Flow(Flow是父类包含许多种Flow)转变为StateFlow。stateIn()需要指定的三个参数
* scope（协程作用域）
* started（启动策略）
* initialValue（初始值）
```kotlin
stateIn(
        scope = viewModelScope,
        started = kotlinx.coroutines.flow.SharingStarted.WhileSubscribed(5000),
        initialValue = emptyList()
    )
```
`WhileSubscribed()`是启动策略的一种可以：`避免内存泄露`和`优化资源使用`，在有收集者的时候，流会保持活跃状态，正常发射数据，没有收集者时会等待5000毫秒后停止活跃，期间重新出新收集者会立即活跃。
`combine()`可以将多个流组合到一起，简化操作，combine 操作符的返回值是一个新的普通 Flow。

**核心特性是：当组合的任意一个源流发射新值时，它都会触发一次组合计算并发射新的结果。**
## 3.依赖注入(Hilt)
### 3.1.手动依赖注入
Android推荐的应用架构推荐将代码划分为多个类，以达到**分离关注点**。这就需要将更多更小的类连接到一起，以实现彼此之间的依赖关系。
<img width="960" height="720" alt="image" src="https://github.com/user-attachments/assets/e1569f42-6289-474c-86a8-31de26d40477" />

每个类都连接到其所依赖的类形成连接，依赖注入有助于建立这种连接，从而方便更换实现以进行测试。

将流程视为应用中与某项功能相对于的一组画面，例如登陆流程，LoginActivity是登陆流程的入口点，用户与activity进行交互，因此，LoginActivity 需要创建 LoginViewModel 及其所有依赖项。
<img width="1705" height="726" alt="image" src="https://github.com/user-attachments/assets/2e8b7be2-98ac-4492-b106-51f9387fea95" />

迭代
* 使用传统的方式
  * 存在大量的样板代码，需要在另一部分使用时，需要使用重复代码
  * 必须按顺序声明依赖项，底层的创建出来后，上层的才可以创建
  * 很难重复使用对象，如需重复使用则要使用单例模式，但单例使测试变得困难(所有测试共享相同的实例)
* 使用容器管理代码，将重复的部分写为工厂方法
  * 必须自行管理 AppContainer，手动为所有依赖项创建实例。
  * 仍然有大量样板代码。需要手动创建工厂或参数，具体取决于是否要重复使用某个对象。
* 管理应用流程中的依赖项，流程开始时创建，流程结束时销毁，如可以用activity管理容器的生命周期，onCreate中创建，onDestroy中销毁(使用生命周期观察)

### 3.2.使用Hilt作为依赖项注入工具，自动管理依赖项
手动依赖项注入要求您手动构造每个类及其依赖项，并借助容器来重复使用和管理依赖项。
#### 3.2.1.在项目中配置Hilt
project build gradle
```kotlin
alias(libs.plugins.kotlin.kapt) apply false
alias(libs.plugins.hilt.android) apply false
```
Module buile gradle
```kotlin
alias(libs.plugins.kotlin.kapt)
alias(libs.plugins.hilt.android)

implementation(libs.hilt.android)
kapt(libs.hilt.compiler)
implementation(libs.hilt.navigation.fragment)
```
libs.versions.toml
```kotlin
[versions]
hilt = "2.48.1"
hiltNavigation = "1.0.0"

[libraries]
hilt-android = { group = "com.google.dagger", name = "hilt-android", version.ref = "hilt" }
hilt-compiler = { group = "com.google.dagger", name = "hilt-compiler", version.ref = "hilt" }
hilt-navigation-fragment = { group = "androidx.hilt", name = "hilt-navigation-fragment", version.ref = "hiltNavigation" }

[plugins]
kotlin-kapt = { id = "org.jetbrains.kotlin.kapt", version.ref = "kotlin" }
hilt-android = { id = "com.google.dagger.hilt.android", version.ref = "hilt" }
```

#### 3.2.2.使用Hilt流程
1. @HiltAndroidApp：标志应用的入口点，让Hilt能够开始工作

## 4.Navigation 组件（多页面跳转）

## 5.Demo

## 6.遇到的问题
启动程序就闪退，也没有日志输出，错误原因用Application()创建了一个新的Application实例，但Room数据库初始化时需要长期存在的、有效的上下文，应使用系统维护的application，该变量是系统在应用启动时自动创建的全局Application实例，具备完整的上下文功能。
数据库操作在主线程执行，可能会长时间阻塞UI导致问题，这里要注意，不是说开启一个新的协程就可以了，这个协程如果是在主线程里面的话，那还是会导致阻塞UI的问题，所以需要放到一个新的线程里面，IO线程(withContext(Dispatchers.IO))，suspend不自动切换线程，只是标记这个函数可以挂起。所以需要withContext(Dispatchers.IO) 来保证线程安全！
不调用数据库的函数，在工具里面看不到数据库的信息。
.java成功，.javaClass失败。
两个流操作困难，可以组合成一个数据流操作。
