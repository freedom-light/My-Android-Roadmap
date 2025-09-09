# 一.第一周内容

<img width="203" height="219" alt="image" src="https://github.com/user-attachments/assets/4c654f59-17eb-403d-8b52-f8f1d4f640b4" />

<img width="279" height="396" alt="image" src="https://github.com/user-attachments/assets/ba8dbf86-feaf-4585-92e2-e7b1bc41492c" /><img width="279" height="396" alt="image" src="https://github.com/user-attachments/assets/4ffadce4-6668-4e79-9272-5c3182fd5628" />

## 1.下载了Android Studio<img width="63" height="12" alt="image" src="https://github.com/user-attachments/assets/081a7ca1-256f-427c-984c-c9c1c2c2e5ea" />  

<img width="238" height="195" alt="image" src="https://github.com/user-attachments/assets/26de01eb-fac6-475f-9d55-4f026f75d86c" />

新建项目时选择Empty Views Activity，而不是Empty Activity（compose项目)

<img width="122" height="270" alt="image" src="https://github.com/user-attachments/assets/3c4e7908-e4d2-4d8c-a1e5-113627821fb3" />

## 2.对于Android项目结构与基本概念的理解
在创建新项目时选择empty activity，这里的activity与Qt中的QMainWindow类似，是负责展示UI，处理用户交互的地方。

Android 系统的 API 级别（API Level） 列表，每一个 API 级别对应着特定版本的 Android 操作系统(基于Linux的移动设备操作系统)
<img width="416" height="124" alt="image" src="https://github.com/user-attachments/assets/1e75eeae-ba8f-4fc7-bc58-44dc08bb3c65" />

这是 Android 项目中 构建脚本（Build Script）的两种 DSL（领域特定语言）选择，用于配置项目的构建逻辑
<img width="415" height="60" alt="image" src="https://github.com/user-attachments/assets/54ca0b91-6632-4c85-9248-83f1c4fd84f4" />

在选择完项目后，切换为Project Files视图模式
在gradle文件夹中的gradle-wrapper.properties文件中，将路径改为国内的镜像源，可以提高下载速度。
```kotlin
distributionUrl=https\://mirrors.cloud.tencent.com/gradle/gradle-8.13-bin.zip
```
修改完url之后点击大象图标重新同步一下<img width="33" height="60" alt="image" src="https://github.com/user-attachments/assets/82b81f6e-a887-4fab-a413-df1ba06846bb" />
在屏幕的右上角

<img width="289" height="214" alt="image" src="https://github.com/user-attachments/assets/4b6b618b-4867-489b-b4b8-6899400f1619" />
<img width="415" height="105" alt="image" src="https://github.com/user-attachments/assets/c3009afa-e41c-4911-a024-28992e7c208c" />

### 2.1. Android项目结构

#### Android Studio 提供两种主要视图：

|    视图类型     |          特点          |      使用场景      |
| :-------------: | :--------------------: | :----------------: |
| **Android视图** | 简化结构，隐藏无关文件 |      日常开发      |
| **Project视图** |      完整目录结构      | 项目配置和文件管理 |

日常开发常用 **Android 视图**。

#### Android 项目目录结构说明
<img width="279" height="396" alt="image" src="https://github.com/user-attachments/assets/d1d73cd5-bea5-4da4-872a-2191b95c29c2" />

##### `manifests/` 目录
- **仅包含 `AndroidManifest.xml`**（应用的核心配置文件），用于声明：
  - 应用包名、版本等基础信息
  - 所有组件
  - 应用权限（如联网、相机权限）
  - 应用元数据（图标、名称、主题等）
---
##### `kotlin/` + `java/` 目录
存放应用的**业务代码和测试代码**，是开发的核心区域。

1.  **主包目录**：`com.example.mycomposeapp`
    - 按**功能/模块**组织代码

2.  **测试包**：`com.example.mycomposeapp` (`androidTest`)
    - 存放 **Instrumentation 测试代码**
    - *需在 Android 设备 / 模拟器上运行*

3.  **测试包**：`com.example.mycomposeapp` (`test`)
    - 存放 **本地单元测试代码**
    - *在 JVM 上运行，不依赖 Android 系统*
    - **这个地方可以作为 Kotlin 的练习场**，编写任何纯 Kotlin 代码进行练习。
---
##### `res/` 目录（资源核心区）
管理**非代码资源**。

1. **`drawable/`**
    存放可绘制资源

2. **`layout/`**
    存放所有**界面布局文件**（`.xml` 文件）的地方，它是 Android 项目的核心部分。

3.  **`mipmap/`**
    存放启动图标资源

4.  **`values/`**
    存放常量资源

5.  **`xml/`**
   存放特殊配置文件

6.  **`res (generated)/`**
    *自动生成的资源目录*（由工具生成，**无需手动修改**）
---
创建layout文件夹（layout XML files）
* 右键res --> New --> XML --> Layout XML File

***使用Compose模式默认不会创建这个文件，因为Compose所有的UI都在代码中完成，就不用xml了***

|布局|描述|
|:-:|:-:|
|**LinearLayout**|线性排队|
|ConstraintLayout|通过约束“粘“定位置（首选推荐）|
|RelativeLayout|相对于其他视图定位|
|FrameLayout|所有视图堆叠在左上角|
|TableLayout|表格行和列|

### 2.2.Android 开发模式

|       模式               |              描述                       |                             特点                                       |     推荐阶段      |
|:-------------------------:|:--------------------------------------:|:-----------------------------------------------------------------------:|:----------------:|
| **传统 View 系统 (XML)** | 代码逻辑与UI布局分离，使用XML文件定义界面。         | 布局和逻辑解耦，可视化编辑，学习资源丰富，是理解Android UI架构的基础。 | **初学者阶段**   |
| **Compose (声明式)** | 所有UI都在代码（Kotlin）中完成，采用声明式编程范式。 | 开发效率高，代码更简洁，状态管理是核心概念，是现代Android开发的未来。   | **基础打牢之后** |

### 2.3.Android编译构建
Android Gradle插件（AGP）会执行一个叫做 “资源编译（AAPT2）” 的过程。
AAPT2会收集所有资源文件中的所有资源，为每个资源分配一个唯一的整形ID，然后自动生成一个名为R的类，生成的R类内部包含多个静态内部类，每个内部类对应一种资源类型。

| 资源类          | 描述                               |
|----------------|------------------------------------|
| R.layout       | 包含所有布局文件的 ID               |
| R.id           | 包含了所有在 XML 中通过 @+id/... 定义的视图 ID |
| R.string       | 包含了所有在 strings.xml 中定义的字符串 ID |
| R.drawable     | 包含了所有 drawable 资源的 ID       |
| R.color        | 包含了所有资源颜色的 ID             |

## 3.学习前准备（Android XML）
认识Android XML常见的关键字

| 关键字                | 作用                 | 类比 Qt                |
|----------------------|----------------------|-----------------------|
| match_parent         | 填满父容器           | QWidget::expanding    |
| wrap_content         | 包裹内容             | QWidget::minimumSize  |
| android:id           | 控件 ID(用于查找)    | objectName            |
| android:text         | 显示文本             | setText()             |
| android:layout_width  | 宽度                 | width                 |
| android:layout_height | 高度                 | height                |

在Android XML中 **`android:id="@+id/button"`**
- @：前缀符号, 表示告诉Android系统后面要引用一个资源了, 这个@符号是固定的，必须要有。
- +：	表示 ”创建” 符号，有”+”号表示如果R.id资源列表没有这个id, 就创建一个新的。
- id/button：资源类型和名称，id是资源类型的id（唯一的），button表示为控件指定的唯一名称。

在layout --> activity_main.xml中，像Qt一样，拖入控件会自动添加Android XML的代码。

### 切换模式
在该界面的右上角，可以切换模式（编写Android XML / 拖动控件 / 混合）

<img width="203" height="219" alt="image" src="https://github.com/user-attachments/assets/64d7e6f4-3f46-4825-81b7-f2e4cdb210ea" />

### 以拖动的方式添加控件
点击这里即可开启以拖动的方式添加控件（自动添加对应的Android XML）

<img width="227" height="150" alt="image" src="https://github.com/user-attachments/assets/ee4b49e9-3ec4-4e41-8806-61b48611a9ba" />

## 4.小demo
### 4.1.实现一个页面，包含 Button + TextView，点击按钮后修改文字
加入控件后例如：Button和TextViex，可能看着相对位置是正确的，但是运行之后就会都跑到左上角（0, 0）位置。

原因是：使用了约束布局（ConstraintLayout），需要为每个控件设置约束条件，但是没有设置，Android Studio就会提醒。

<img width="415" height="29" alt="image" src="https://github.com/user-attachments/assets/f57fa782-be0c-4556-867d-33c6e83363b5" />

可以通过点击<img width="20" height="22" alt="image" src="https://github.com/user-attachments/assets/b6838797-afb1-4cb2-9f81-4da9a1f19e64" />
推断约束，自动分析当前放置的视图位置。

之后可以通过<img width="21" height="20" alt="image" src="https://github.com/user-attachments/assets/75aa8798-77c0-4c82-8fd9-0599d0500b27" />
清除当前布局中所有约束。

```kotlin
import android.widget.Button
import android.widget.TextView


// 添加以下代码：获取控件并设置点击事件
val button: Button = findViewById(R.id.button)  // 获取按钮
val textView: TextView = findViewById(R.id.textView)  // 获取TextView

// 设置按钮点击事件
button.setOnClickListener {
    // 点击按钮时修改TextView的文字
    textView.text = "你好！按钮被点击了！"
}
```
就可以成功点击按钮，修改文字了！

在MainActivity.kt中MainActivity继承于ComponentActivity()（应用的主界面载体），系统启动应用时，会自动创建MainActivity类实例，并调用其中的onCreate()方法，相当于程序的起点。
在Kotlin中，当函数的最后一个参数是lambda表达式时，可以将其放在括号的外面，并且函数名和lambda表达式之间不能有换行分隔，否则编译器无法正确识别这个一个函数调用。

<img width="416" height="155" alt="image" src="https://github.com/user-attachments/assets/d4fdf77e-ed2f-4bdc-9886-91cab2216318" />

### 4.2.做一个 待办清单 App（TodoList）Demo：支持新增、删除任务，数据保存到内存（无需数据库）
```kotlin
// 无需保存在数据库，通过成员遍历保存在数据库即可。
// 用于存储任务的列表（内存中）
private val taskList = mutableListOf<String>()
private lateinit var adapter: ArrayAdapter<String>
private lateinit var taskInput: EditText
private lateinit var taskListView: ListView
// lateinit 关键字代表延迟初始化，但需要确保在运行期通过findViewById找到实际的内存对象。
// 初始化视图
taskInput = findViewById(R.id.taskInput)
val addButton: Button = findViewById(R.id.addButton)
taskListView = findViewById(R.id.taskListView)

// 并且对按钮添加对应的事件，类似于Qt信号槽的绑定过程

// 添加任务按钮点击事件
addButton.setOnClickListener {
    addTask()
}
// 列表项长按删除事件
taskListView.setOnItemLongClickListener { _, _, position, _ ->
    removeTask(position)
    true // 返回true表示已处理事件
}
```

含义就是在addButton触发点击按钮事件后，调用addTask函数_, _, position, _ ->这个是Kotlin中lambda表达式的一种形式，可以通过_跳过不需要的

<img width="345" height="94" alt="image" src="https://github.com/user-attachments/assets/b0c03c55-3390-4458-b296-4a721ba6d3de" />












