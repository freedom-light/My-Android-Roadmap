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

|可组合项|
|:----|
|Text|
|Image|


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
如果未提供有关如何排列这多个元素的信息，它们会相互重叠，使元素难以观察。
`Column`函数可以用来垂直排列元素
可以使用 `Row` 水平排列各项，并使用 `Box` 堆叠元素。

Compose 使用了修饰符。通过修饰符，可以更改可组合项的大小、布局、外观，还可以添加高级互动，例如使元素可点击。可以将这些修饰符链接起来，以创建更丰富的可组合项。
### 1.3.Material Design
Material Design 就是一套让数字界面看起来更自然、用起来更直观的设计规则。例如通过阴影表示层次关系、通过动画提供视觉反馈、具有语义含义的色彩、规范的字号。

Compose 旨在支持 Material Design 原则。它的许多界面元素都原生支持 `Material Design 3` 及其界面元素的实现。

使用方式：
1. 使用项目中创建的 Material 主题，主题名可以通过`ui.theme -> Theme.kt`找到
2. `Surface` 来封装可组合函数。
这样一来，可组合项即可沿用应用主题中定义的样式，从而在整个应用中确保一致性。

`Material Design` 是围绕 `Color(颜色)`、`Typography(排版)`、`Shape(形状)` 这三大要素构建的。
* Color：可以在 `MaterialTheme.kt` 文件中将 `dynamicColor` 设置为 false，以更改此设置。否则Dynamic Color(动态取色)会自动从用户的壁纸中提取主色调。
* `MaterialTheme` 中提供了 `Material Typography` 样式，只需将其添加到可组合项中即可。
* 将消息正文封装在 `Surface` 可组合项中。这样即可自定义消息正文的形状、高度、内边距等..
启用深色主题
```kotlin
@Preview(
    uiMode = Configuration.UI_MODE_NIGHT_YES,
    showBackground = true,
    name = "Dark Mode"
)
```
### 1.4.列表和动画
#### 1.4.1.创建消息列表
使用 Compose 的 `LazyColumn` 和 `LazyRow`。这些可组合项只会呈现屏幕上显示的元素，因此，对于较长的列表，使用它们会非常高效。
#### 1.4.2.在展开消息时显示动画效果
为了存储此本地界面状态，需要跟踪消息是否已展开。为了跟踪这种状态变化，必须使用 `remember` 和 `mutableStateOf` 函数。

可组合函数可以使用 remember 将本地状态存储在内存中，并跟踪传递给 mutableStateOf 的值的变化。该值更新时，系统会自动重新绘制使用此状态的可组合项（及其子项）。这称为重组。

## 学习错误处理与 App 权限管理
## 掌握 Git 基本协作（如 Pull Request 流程）
## 了解日志系统（Timber）
