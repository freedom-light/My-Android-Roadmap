# 第 5 周：Jetpack Compose 与工程化
## 1.入门 Jetpack Compose（声明式 UI 思维）
### 1.1.可组合函数
Jetpack Compose 是围绕可组合函数构建的。只需描述应用界面的外观并提供数据依赖项，而不必关注界面的构建过程（初始化元素、将其附加到父项等）。如需创建可组合函数，只需将 @Composable 注解添加到函数名称中即可。
```kotlin
@Composable
fun Conversation(msgs: List<Message>){
    ...
}
```
`setContent` 块定义了 activity 的布局，我们会在其中调用可组合函数。可组合函数只能从其他可组合函数调用。
#### 1.1.1.在 Android Studio 中预览函数
借助 @Preview 注解，您可以在 Android Studio 中预览可组合函数，而无需构建应用并将其安装到 Android 设备或模拟器中。该注解只能用于不接受参数的可组合函数。预览带参数`fun A(val)`的可组合函数需要创建另一个函数`fun PreViewA()`，在`PreViewA`中传入适当的参数调用`A(val)`在 @Composable 上方添加 @Preview 注解。
```kotlin
@Preview
@Composable
fun PreViewA(){
    ...
}
```
### 1.2.布局
### 1.3.Material Design
### 1.4.列表和动画

`Material Design` 是围绕 `Color(颜色)`、`Typography(排版)`、`Shape(形状)` 这三大要素构建的。

## 学习错误处理与 App 权限管理
## 掌握 Git 基本协作（如 Pull Request 流程）
## 了解日志系统（Timber）
