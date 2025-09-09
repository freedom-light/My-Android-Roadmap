# 一.第一周内容

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

Android Studio 提供两种主要视图：

| 视图类型 | 特点 | 使用场景 |
|---------|------|---------|
| **Android视图** | 简化结构，隐藏无关文件 | 日常开发 |
| **Project视图** | 完整目录结构 | 项目配置和文件管理 |

日常开发常用 **Android 视图**。













