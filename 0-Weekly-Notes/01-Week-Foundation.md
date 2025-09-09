# 一.第一周内容

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

| 视图类型 | 特点 | 使用场景 |
|---------|------|---------|
| **Android视图** | 简化结构，隐藏无关文件 | 日常开发 |
| **Project视图** | 完整目录结构 | 项目配置和文件管理 |

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













