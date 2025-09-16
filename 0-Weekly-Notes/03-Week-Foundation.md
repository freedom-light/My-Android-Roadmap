# 三.第三周内容

## 1.学会在本地存储数据（SharedPreferences、Room）
### 1.1.Room
#### 1.1.1.Room数据库
Room 持久性库在 SQLite 上提供了一个抽象层，以便在充分利用 SQLite 的强大功能的同时，能够流畅地访问数据库。具体来说，Room 具有以下优势：
| 特性 | 传统方式 | Room |
| :--- | :--- | :--- |
| **编译时验证** | SQL语句作为普通字符串，编译时无法发现错误，只能在运行时暴露。 | 在编译时分析SQL查询的语法和语义，提前发现表名、列名等错误。 |
| **开发效率与样板代码** | 需要编写大量重复、易出错的样板代码（如解析Cursor、构建ContentValues）。 | 通过注解（如`@Dao`, `@Query`, `@Insert`）自动生成实现代码，极大减少样板代码。 |
| **数据库迁移** | 需要手动处理`onUpgrade`逻辑，编写复杂的SQL并维护版本变化，容易出错。 | 提供清晰的`Migration`类，可安全地指定从某个版本到另一个版本的迁移路径，结构清晰且易于测试。 |

| 注解 | 作用 |
| :--- | :--- |
| `@Entity` | 标记一个类为数据库中的一张表。这是定义数据实体最重要的注解。 |
| `@PrimaryKey` | 定义表的主键。每个被 `@Entity` 注解的类至少必须有一个被此注解标记的字段。 |
| `@ColumnInfo` | 自定义表中列的属性，如列名（`name`）、是否索引（`index`）等。 |
| `@Ignore` | 忽略被标记的字段，不会在数据库表中为其创建对应的列。 |
| `@ForeignKey` | 定义外键约束，用于关联另一张表，保证引用的完整性。 |
| `@Index` | 为表的特定列创建索引，以提高查询速度，但可能会降低插入和更新数据的速度。 |

#### 1.1.2.在项目中使用Room
1. 在顶级build.gradle.kts中声明KSP插件
```kotlin
// Top-level build file where you can add configuration options common to all sub-projects/modules.
plugins {
    alias(libs.plugins.android.application) apply false
    alias(libs.plugins.kotlin.android) apply false
// apply false只声明插件，但不应用到当前模块，通常在顶级build...中使用
    alias(libs.plugins.ksp) apply false
}
```
2. 在模块级build.gradle.kts中启用KSP
```kotlin
plugins {
    alias(libs.plugins.android.application)
    alias(libs.plugins.kotlin.android)

    // 添加 KSP 插件
    alias(libs.plugins.ksp)
}
```
3. 在libs.versions.toml中代码规范化
```kotlin
[versions]中添加
ksp = "2.0.21-1.0.27"
[plugins]
ksp = { id = "com.google.devtools.ksp", version.ref = "ksp" }
```
4. Gradle 构建脚本中添加一个核心配置块（最简方式）
```kotlin
dependencies {
    val room_version = "2.7.2"

    // 1. Room 运行时库 - 必需
    implementation("androidx.room:room-runtime:$room_version")

    // 2. Room 编译器 - 必需（二选一）
    // 如果你用 Kotlin，用 KSP：
    ksp("androidx.room:room-compiler:$room_version")

    // 3. Kotlin 扩展和协程支持 - 强烈推荐（因为你是 Kotlin 项目）
    implementation("androidx.room:room-ktx:$room_version")
}
```

这样就可以正确使用Room了

### 1.2.SharedPreferences

## 2.掌握网络请求（Retrofit + OkHttp + Gson）
去文档中先大概了解Retrofit，将其配置到自己的项目中。
初次学习Retrofit 的难点
1. 文献内容较少，相比其他官方文档没有很丰富的内容
2. github中的资源也较少
3. 不清楚如何配置到项目中
如果根本不知道如何下手，身边有明白人，可以先问一下关键的点，像无头苍蝇一样浪费时间。

### 2.1.配置所需库
**Maven**
```
<dependency>
  <groupId>com.squareup.retrofit2</groupId>
  <artifactId>retrofit</artifactId>
  <version>3.1.0-SNAPSHOT</version>
</dependency>
```
这是文档中重要的内容，详细解释一下。
* Maven(依赖声明)
* groupId：表示库的开发者/组织
* artifactId：库的具体名称
* version：版本号
* SHAPSHOT表示是一个开发中的快照版本，生成环境不要使用SHAPSHOT版本，如果需要稳定版本，使用正式发布版本。

由上面的Maven可以配置出。

```kotlin
[versions]
retrofit2 = "3.0.0"

[libraries]
androidx-retrofit2 = { group = "com.squareup.retrofit2", name = "retrofit", version.ref = "retrofit2" }

implementation(libs.androidx.retrofit2)
```

此外还需要配置`converter-gson(转换器库)`
原因是：retrofit核心库的作用是提供网络请求的核心功能，但是此时请求到的数据不能被直接使用，需要使用converter-gson转换器库配合gson，将API返回的JSON数据自动转换为数据对象。converter-gson负责告诉retrofit如何使用gson来解析数据，gson负责真正的JSON与对象直接的转换。
**retrofit和converter-gson两个库版本要求一样。**

小tip
可以看出在构建文件中，是没有“-”的用“.”代替
```kotlin
implementation(libs.androidx.retrofit)
implementation(libs.androidx.converter.gson)

androidx-retrofit = { group = "com.squareup.retrofit2", name = "retrofit", version.ref = "retrofit" }
androidx-converter-gson = { group = "com.squareup.retrofit2", name = "converter-gson", version.ref = "retrofit" }
```

### 2.2.创建数据类
根据API相应格式定义数据类。下面是截取的片段，观察数据格式，即可创建出对应的data class，ApiResponse, NewsResult, NewsItem
```
{
  "code": 200,
  "msg": "success",
  "result": {
    "curpage": 1,
    "allnum": 10,
    "newslist": [
      {
        "id": "527cf27b57f79ba311f19b736c0f3eba",
        "ctime": "2025-09-11 00:00",
        "title": "硬核！原来这些纵横天地间的&quot;大国重器&quot;是天津制造",
        "description": "",
        "source": "中华国内",
        "picUrl": "https://img3.utuku.imgcdc.com/300x200/news/20250911/1ed640a8-8204-4b24-8f5e-c48af1caa791.jpg",
        "url": "https://news.china.com/domestic/945/20250911/48813922.html"
      },
```
**特别注意！！！**
**特别注意！
data class里面的字段名要和JSON中的字段名完全相同。
如果data class中是newsList，JSON中是newslist,也是无法匹配的
虽然正常的理解中变量名只是代称，但是Gson不知道这个代称对于JSON中的哪个字段，所以它只能使用字符串匹配的方式校验。**

### 2.3.创建API接口
定规API请求的规范，需要使用到一个关键字，`interface`
interface是Kotlin中的关键字，用于`定义接口`，接口是一种抽象类型。
在retrofit中接口通常用于定义API请求的规范，需要声明请求的函数，请求的类型、参数、返回值。**底层具体实现类由retrofit自动生成**。
* @GET中只放API的端点路径，不需要放完整的URL
* @Query的作用是添加URL查询参数，例如一个网址

例如这个网址：`https://whyta.cn/api/tx/guonei?key=d8c6d4c75ba0&num=10`
URL查询参数是“?”后面的用“&”分隔的。
由于可以定义出如下的接口
```kotlin
import retrofit2.http.GET
import retrofit2.http.Query

interface NewsApiService {
    @GET("api/tx/guonei")
    suspend fun getNewsList(
        @Query("key") key: String = "d8c6d4c75ba0",
        @Query("num") num: Int = 10
    ): ApiResponse
}
```

在刚刚写完@GET("api/tx/guonei")后会报一个错误`Expecting member declaration`，但是不用担心，只是因为缺少了必要的上下文，把下面的写完就好了。

`suspend`是kotlin`协程`的一个关键字，写了suspend关键字代表该函数是一个`挂起函数`，它可能会暂停当前的协程执行而不是阻塞线程，一般用于网络请求，数据库读写等**耗时操作**，并在它完成工作后**自动恢复**。

### 2.4.创建API接口服务实例
为什么叫`实例`呢，接口服务中Retrofit实例和kHttpClient实例的构建都是`建造者模式`，之前属于抽象的状态，在`object单例`中对需要的参数进行配置，从而得到一个不可变对象(所有字段都是final的)。
创建**OkHttpClient**实例，OkHttpClient是一个Http客户端库，Retrofit本身是一个接口层框架，负责将接口定义转换为网络请求，Retrofit通过依赖OkHttpClient来执行**实际的网络请求**。

之后创建retrofit实例，配置基本信息，也是建造者模式。

最后对接口进行实例，但是接口是不能直接实例出对象的(NewsApiService)，关键的地方是`retrofit.create(NewsApiService::class.java)`，retrofit.create会在运行时生成一个NewsApiService 接口的**实现类**，在这个实现类里面，通过你的注解，**转换为真实的HTTP请求逻辑**（比如拼接 URL、处理参数、调用 OkHttpClient 发请求）

#### 2.4.1.建造者模式
**建造者模式是一种创建型设计模式，它的主要目的是将一个对象的构建和表示分离，使得同样的构建过程可以创建不同的表示。**

需要建造者模式的原因
* 构造函数参数爆炸：现在需要创建一个复杂的对象，如果使用传统的构造函数的方式，需要重载极多的构造函数，并且其中许多参数可能是不必要的或需要默认值的，这会导致代码难以阅读和维护。
* Setter方法的不一致状态：如果使用setter方法，在对象完全构建出来之前，它可能处于一种部分初始化的状态，这不是线程安全的，也容易出错。
* 构建过程缺乏控制：无法强制客户端按照特定的顺序或步骤来构建对象。
建造者模式通过提供一个独立的`建造者`类来解决这些问题，由它来负责对象的逐步构建，最后返回一个完整的产品。

建造者模式通常包含一下四个角色：
1. 产品：要创建的复杂对象
2. 抽象建造者：为创建一个产品的各个部分指定抽象接口，通常包含构建各个部分的方法和一个返回最终产品的接口。（是具体建造者的抽象类）
3. 具体建造者：具体实现抽象建造者中的接口。
4. 指挥者（简单场景中可简化合并到客户端代码中）

```
Computer gamingComputer = new Computer.ComputerBuilder("Intel i9", "32GB")
                .setGraphicsCard("NVIDIA RTX 4080")
                .setStorage("1TB SSD")
                .setOs("Windows 11")
                .build(); // 最终调用 build 方法创建对象
```
如上代码，首先调用建造者构造函数提供必须参数，之后选用建造者接口，实现了同样的建造过程可以创建出不同的表示。最后调用build创建一个完整的对象。
### 2.5.发起网络请求并处理结果
在实际的界面（如 MainActivity）中使用这个配置好的 RetrofitClient 来发起网络请求并处理响应

## 3.理解 ViewModel + Flow 基本用法

## 4.了解 ViewModel + LiveData 基本用法

## 5.Demo
### 5.1.改造 TodoList：用 Room 保存任务数据


### 5.2.做一个「新闻阅读 App Demo」： 首页列表（通过 Retrofit 获取数据），点击条目进入详情页，收藏功能（用Room保存收藏状态）
通过Retrofit+Gson+OkHttpClient完成从网址中GET数据并转为数据类对象的操作，之后使用RecyclerView进行列表展示。

优化：现在就像面向过程的编程，把所有内容都写到了main()函数里面，让类符合单一职责原则，一个类负责一类任务，一个接口完成单一功能，转为高内聚低耦合，从而增强代码可。

## 6.遇到的问题
### 6.1.androidTestImplementation引入后无法使用库
androidTestImplementation 用于引入那些需要在 Android 设备上运行的测试依赖。需要注意对于主应用程序要使用的部分，要使用implementation，使用androidTestImplementation 在import引入库的时候，无法成功。可以通过Project -> 在左边列表搜索库名 的方式观察库是否成功载入。

### 6.2.数据类中picUrl是String类型，imageView不能直接使用picUrl赋值
可以使用Glide解决，Glide是Android图片加载和缓存库，可以帮助用简短的代码完成从各种来源加载图片并显示到ImageView中。

Glide的实用功能
```kotlin
Glide.with(holder.itemView.context)
    .load(currentItem.picUrl)
    .placeholder(R.drawable.ic_loading)    // 加载中显示的图片
    .error(R.drawable.ic_error)            // 加载失败显示的图片
    .override(300, 200)                    // 指定图片尺寸
    .centerCrop()                          // 图片裁剪方式
    .circleCrop()                          // 圆形图片
    .into(holder.image)
```
```kotlin
[versions]
glide = "4.16.0"

[libraries]
androidx-glide = { group = "com.github.bumptech.glide", name = "glide", version.ref = "glide" }

implementation(libs.androidx.glide)
```

## 7.收获经验/优化点
* 改架构然后监听状态，
* 格式化管理，new -> package
* 不要信任服务端的数据，进行空校验，或者赋值为可空，不然调用空对象，会崩要避免API字段返回null导致的解析崩溃


