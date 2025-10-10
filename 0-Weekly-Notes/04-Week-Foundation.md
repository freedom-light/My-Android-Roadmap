# 第 4 周：架构与异步处理
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
#### 2.1.1.协程概念及基本使用
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

#### 2.1.2.协程使用注意事项
协程继承父协程的上下文，对于某些耗时操作，例如数据库读取、网络请求等，需要明确协程被创建在哪个线程，防止持有耗时操作等协程被创建在线程，应该UI界面刷新。
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

#### 3.2.2.使用Hilt
1. 所有使用Hilt的应用都必须包含`@HiltAndroidApp`注解的 `Application` 类，不使用Hilt的话可以不主动创建Application，使用系统默认的即可。
```kotlin
@HiltAndroidApp
class MyApplication : Application(){
    override fun onCreate() {
        ...
    }
}
```
2. 将依赖项注入Android类，Hilt可以为带有`@AndroidEntryPoint`注解的其他Android类提供依赖项。Hilt目前支持以下Android类
    * Application（通过使用 @HiltAndroidApp）
    * ViewModel（通过使用 @HiltViewModel）
    * Activity
    * Fragment
    * View
    * Service
    * BroadcastReceiver
3. 执行完第二步操作后，每个Android类会生成一个单独的Hilt组件，这些组件可以从它们各自的父类接收依赖项。如需从组件获取依赖项，使用`@Inject`注解执行字段注入。
4. 定义Hilt绑定，“绑定”包含将某个类型的实例作为依赖项提供所需的信息。
    * 构造函数注入，在某个类的构造函数中使用 @Inject 注解，以告知 Hilt 如何提供该类的实例`class AnalyticsAdapter @Inject constructor() { ... }`
    * 对于不能通过构造函数注入的，可以使用`Hilt模块`即一个带有`@Module`注解的类它会告知 Hilt 如何提供某些类型的实例。还需要使用 `@InstallIn` 为 Hilt 模块添加注解。指定这个模块安装在哪个Hilt组件中。
```
@InstallIn(SingletonComponent::class)  // 告诉 Hilt：这个模块在整个应用生命周期内有效
```
5. 使用@Binds注入接口实例，对于接口来说，你无法通过构造函数注入它，而应向Hilt提供绑定信息，通过带有`@Binds`注解的抽象函数，该函数会向Hilt提供以下信息：
    * 函数返回类型会告知Hilt，该函数提供哪个接口的实例
    * 函数参数会告知Hilt要提供哪种实现
6. 无法通过构造函数注入的另外情况，如类来自于外部库(Retrofit,OkHttpClient,Room数据库)，或者必须使用**构建器模式**创建实例。可以在**Hilt模块内部**创建一个函数并为函数添加`@Provides`注解，告知如何提供此类型的实例。带有注解的函数会向Hilt提供如下信息：
    * 函数返回类型会告知 Hilt 函数提供哪个类型的实例
    * 函数参数会告知 Hilt 相应类型的依赖项
    * 函数主体会告知 Hilt 如何提供相应类型的实例。每当需要提供该类型的实例时，Hilt 都会执行函数主体。
7. Hilt中的预定义限定符，Hilt提供了`@ApplicationContext` 和`@ActivityContext`限定符，在需要来自应用或activity的`context`类时使用。

在依赖注入中，应该优先使用 @Inject 构造函数，只有在无法控制类构造时才使用 @Provides。这样代码更简洁，也符合依赖注入的原则。
@Singleton // 用于指定依赖项的生命周期为单例

## 4.Navigation 组件（多页面跳转）
用于简化 Android 应用中多页面之间的导航和跳转。它提供了一种标准化、声明式的方式来处理应用内导航。
```kotlin
dependencies {
    def nav_version = "2.5.3"
    implementation "androidx.navigation:navigation-fragment-ktx:$nav_version"
    implementation "androidx.navigation:navigation-ui-ktx:$nav_version"
}
```
同样是为了减少样板代码而生的技术，Navigation 组件通过标准化的方式简化了 Android 应用中的导航实现，提高了代码的可维护性和开发效率。
## 5.Demo
### 将新闻APP重构为 ViewPager2 + Fragment + TabLayout 并 将数据库改为`监听`模式
#### ViewPager2 + Fragment + TabLayout使用方式
1. 第一步：创建 Fragment
使用这个技术首先将需要的页面创建出来，以新闻APP为例，一个全部新闻页、一个收藏新闻页。
```kotlin
// 示例：AllNewsFragment
class AllNewsFragment : Fragment() {
    override fun onCreateView(inflater: LayoutInflater, container: ViewGroup?, savedInstanceState: Bundle?): View? {
        // 在这里初始化你的 Fragment 布局和数据
        return inflater.inflate(R.layout.fragment_all_news, container, false)
    }
    // ... 其他逻辑
}
```
2. 第二步：创建适配器 (Adapter)
创建一个继承自 FragmentStateAdapter 的类。
* 构造函数：可以接收 FragmentActivity（用于 Activity 中）或 Fragment（用于嵌套在另一个 Fragment 中）。
* getItemCount()：返回 ViewPager 中页面的总数
* createFragment(position: Int)：根据位置创建并返回对应的 Fragment 实例。
```kotlin
class NewsPagerAdapter(fragmentActivity: FragmentActivity) : FragmentStateAdapter(fragmentActivity) {

    // 确定页面的数量
    override fun getItemCount(): Int = 2

    // 根据位置返回对应的 Fragment
    override fun createFragment(position: Int): Fragment {
        return when (position) {
            0 -> AllNewsFragment() // 第一页
            1 -> FavoritesFragment() // 第二页
            else -> throw IllegalArgumentException("Invalid position: $position")
        }
    }
}
```
3. 第三步：在 Activity 或 Fragment 中设置 ViewPager2
在你的布局 XML 中放置一个 ViewPager2 组件，然后在代码中获取它并设置适配器。
```kotlin
<?xml version="1.0" encoding="utf-8"?>
<LinearLayout xmlns:android="http://schemas.android.com/apk/res/android"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    android:orientation="vertical">

    <!-- 可选：与 ViewPager2 联动的 TabLayout -->
    <com.google.android.material.tabs.TabLayout
        android:id="@+id/tab_layout"
        android:layout_width="match_parent"
        android:layout_height="wrap_content" />

    <androidx.viewpager2.widget.ViewPager2
        android:id="@+id/view_pager"
        android:layout_width="match_parent"
        android:layout_height="0dp"
        android:layout_weight="1" />

</LinearLayout>
```
4. 第四步：使用TabLayoutMediator关联TabLayout和ViewPager2。关联TabLayout和ViewPager：使用TabLayoutMediator将标签布局与翻页器关联起来，实现：
* 点击标签切换对应页面
* 滑动页面时自动更新选中的标签
```kotlin
private fun setupViewPagerWithTabs() {
    val adapter = NewsPagerAdapter(this)
    binding.viewPager.adapter = adapter
    // 使用TabLayoutMediator关联TabLayout和ViewPager2
    tabLayoutMediator = TabLayoutMediator(binding.tabLayout, binding.viewPager) { tab, position ->
        tab.text = when (position) {
            0 -> "全部新闻"
            1 -> "我的收藏"
            else -> ""
        }
    }
    tabLayoutMediator.attach()
}
```
#### 数据库改为`监听`模式
主要是一点，数据流Flow是可以被collect的，冷数据流适用于一次性的操作，因为冷数据流每次都是独立的数据流，有收集者时才生产。
```kotlin
private fun observeFavorites(){
    viewModelScope.launch {
        repository.observeFavorites().collect { favorites ->
            _state.update { it.copy(myFavorites = favorites) }
        }
    }
}
```
```kotlin
override suspend fun observeFavorites(): Flow<List<NewsItem>> {
    return newsDao.observeFavorites()
}
```
```kotlin
// 查询所有收藏的新闻 - 新增Flow监听版本
@Query("SELECT * FROM news_items WHERE isFavorite = 1 ORDER BY favoriteTime DESC")
fun observeFavorites(): Flow<List<NewsItem>>
```
## 6.遇到的问题
启动程序就闪退，也没有日志输出，错误原因用Application()创建了一个新的Application实例，但Room数据库初始化时需要长期存在的、有效的上下文，应使用系统维护的application，该变量是系统在应用启动时自动创建的全局Application实例，具备完整的上下文功能。

数据库操作在主线程执行，可能会长时间阻塞UI导致问题，这里要注意，不是说开启一个新的协程就可以了，这个协程如果是在主线程里面的话，那还是会导致阻塞UI的问题，所以需要放到一个新的线程里面，IO线程(withContext(Dispatchers.IO))，suspend不自动切换线程，只是标记这个函数可以挂起。所以需要withContext(Dispatchers.IO) 来保证线程安全！

不调用数据库的函数，在工具里面看不到数据库的信息。

.java成功，.javaClass失败。

两个流操作困难，可以组合成一个数据流操作。
