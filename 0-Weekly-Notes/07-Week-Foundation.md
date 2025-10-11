# 第7周：EGL 环境与着色器基础
目标：掌握手动搭建 EGL 环境的方法，并学会使用 GLSL 着色器绘制基础二维几何图形，为后续图形渲染打下基础。

核心任务
* 创建自定义 SurfaceView 并实现渲染线程
* 手动搭建 EGL 环境
* 加载并使用 GLSL 着色器
* 定义几何图形数据
* 实现 drawFrame() 与按需渲染
* 管理 EGL 生命周期

## 1.创建自定义 SurfaceView 并实现渲染线程
### SurfaceView是什么，有什么用，为什么用它
SurfaceView 是 Android 视图系统中的一个特殊组件，它继承自 View。它的核心特点是 `拥有一个独立的绘图表面`，这个表面可以与主 UI 窗口分离，进行独立的渲染。

首先看看使用普通的`View`的局限性
* 主线程绘制：普通 View 的绘制（onDraw 方法）是在主线程（UI 线程）中执行的。
* 统一刷新：整个 View 树的更新是同步的。当系统触发绘制时，应用会先完成所有 View 的 onDraw 计算，然后才将最终结果统一提交给屏幕。这保证了 UI 的连贯性，但也带来了性能瓶颈。
问题场景：比如你要开发一个游戏，屏幕需要以 60FPS 的速率刷新。这意味着每 16 毫秒就需要完成一次所有画面的计算和绘制。如果游戏逻辑复杂、图形元素多，在主线程上进行这些繁重的计算和绘制，很容易导致掉帧、界面卡顿，甚至 ANR。

SurfaceView的工作原理和优势
SurfaceView 通过`双缓冲`和`独立表面`的机制来解决上述问题：
1. 独立的表面
   * `SurfaceView` 在 Z 轴上为自己开启了一个新的“层”，位于主窗口之下。它默认是透明的，你可以通过 `setZOrderOnTop` 等方法调整其层级。
2. 双缓冲机制
   * SurfaceView 在内部维护了一个双缓冲区。
     * 前端缓冲：当前正在屏幕上显示的内容。
     * 后端缓冲：下一帧即将要绘制的内容。
   * 开发者在一个独立的线程（通常是后台线程）里，在后端缓冲上执行所有的绘制操作（比如用 Canvas 画图）。当一帧绘制完成后，调用 unlockCanvasAndPost 方法，系统会交换前后端缓冲——刚刚绘制好的后端缓冲变成新的前端缓冲用于显示，而原来的前端缓冲则变成新的后端缓冲，准备接收下一帧的绘制。
3. 在非主线程绘制
   * 这是 SurfaceView 最核心的优势。因为它拥有独立的表面和双缓冲机制，所以繁重的绘制工作可以完全放在一个专门的“渲染线程”中进行。
   * 这样，主线程就可以被解放出来，专心处理用户交互（点击、滑动）、业务逻辑等，避免了因绘制任务繁重而导致的 UI 卡顿。
### 创建自定义线程
实现 SurfaceHolder.Callback → surfaceCreated() → 启动渲染线程 → surfaceChanged() → 渲染循环 → surfaceDestroyed() → 停止线程
```kotlin
class  MyGLSurfaceView(context: Context) : SurfaceView(context), SurfaceHolder.Callback {
    private var renderThread: Thread? = null
    private var isRunning = false
    init {
        holder.addCallback(this)
    }
    override fun surfaceCreated(holder: SurfaceHolder) {...}

    override fun surfaceChanged(holder: SurfaceHolder, format: Int, width: Int, height: Int) {...}

    override fun surfaceDestroyed(holder: SurfaceHolder)  {...}
}
```
类中两个变量的作用`renderThread`和`isRunning`.
* `isRunning`用于控制渲染线程的生命周期，`SurfaceView`的渲染线程需要与 Activity/Fragment 的生命周期同步
  * 当Activity进入后台时，必须停止渲染降低功耗
  * 当Activity进入前台时，需要开始重新渲染
* `renderThread`是承载OpenGL渲染任务的“工作线程”，使用可空类型是因为线程会在surface销毁时被释放。
通过SurfaceHolder注册回调的方式实现发布-订阅模式。因为Surface的生命周期是有系统层面管理的，通过回调机制，我注册回调，来信息了之后系统通知我，之后我执行自己的逻辑。

系统通知：Surface创建好了，就会执行你注册的回调函数，surfaceCreated内部的逻辑，你就可以进行渲染等操作了。

## 2.手动搭建 EGL 环境
### EGL是什么，OpenGL、EGL与SurfaceView的关系
首先EGL是什么，简单说它是一个用来给OpenGL ES提供绘制界面的接口。

EGL是渲染API(如OpenGL, OpenGL ES, OpenVG)和本地窗口系统之间的接口。它处理图形上下文管理，表面/缓冲区创建，绑定和渲染同步，并使用其他Khronos API实现高性能，加速，混合模式2D和3D渲染OpenGL / OpenGL ES渲染客户端API OpenVG渲染客户端API原生平台窗口系统。

然后来说一下三者的关系：
* OpenGL：负责画什么
  * 角色：绘图API / 绘图引擎
  * 职责：它是一套跨语言的、跨平台的编程接口标准，用于绘制2D和3D矢量图形。它只管“画”，但不管“画在哪里”。
* EGL：负责在哪里画
  * 角色：平台中间件 / 桥梁
  * 职责：
    1. 获取显示设备
    2. 创建绘图表面：创建一个供OpenGL绘制的“画布”。
    3. 创建OpenGL上下文
    4. 绑定：将OpenGL上下文与绘图表面进行绑定。
  * 比喻：EGL就像一个艺术经纪人。他负责为画家（OpenGL）准备画布、安排画廊、处理所有与场地相关的杂事，让画家可以心无旁骛地作画。
* SurfaceView：负责提供一个“画布”
  * UI组件 / 画布的提供者
  * SurfaceView是Android系统提供的一个特殊View。它的核心特点是拥有自己独立的绘图表面。

下面是渲染线程类的实现，EGL的三个相关变量就是上面提到的`显示设备`,`绘图表面`,`OpenGL上下文`
```kotlin
class GLRenderThread(private val surfaceHolder: SurfaceHolder) : Runnable {
    
    private var isRunning = false
    private var thread: Thread? = null
    
    // EGL 相关变量
    private var eglDisplay: EGLDisplay? = null
    private var eglContext: EGLContext? = null
    private var eglSurface: EGLSurface? = null
}
```
### EGL的基本使用步骤
1. 首先我们需要知道绘制内容的目标在哪里，`EGLDisplayer`是一个封装系统屏幕的数据类型，通常通过`eglGetDisplay`方法来返回`EGLDisplay`作为OpenGl ES的渲染目标，`eglGetDisplay()`
```kotlin

```
2. 初始化显示设备，第一参数代表Major版本，第二个代表Minor版本。如果不关心版本号，传0或者null就可以了。初始化与 EGLDisplay 之间的连接：`eglInitialize()`
```kotlin

```
3. 下面我们进行配置选项，使用`eglChooseConfig()`方法,Android平台的配置代码如下：
```kotlin

```
4. 接下来我们需要创建OpenGl的上下文环境 `EGLContext` 实例，这里值得留意的是，OpenGl的任何一条指令都是必须在自己的OpenGl上下文环境中运行，我们可以通过`eglCreateContext()`方法来构建上下文环境：`eglCreateContext`中的第三个参数可以传入一个`EGLContext`类型的变量，改变量的意义是可以与正在创建的上下文环境共享OpenGl资源，包括纹理ID,FrameBuffer以及其他Buffer资源。如果没有的话可以填写Null.
```kotlin

```
5. 通过上面四步,获取 OpenGl 上下文之后，说明EGL和OpenGl ES端的环境已经搭建完毕。下面的步骤是如何将EGL和设备屏幕连接起来，使用EGLSurface，通过EGL库提供eglCreateWindowSurface可以创建一个实际可以显示的surface.当然，如果需要离线的surface，我们可以通过`eglCreatePbufferSurface`创建。`eglCreateWindowSurface()`
```kotlin

```
6. 通过上面的步骤，EGL的准备工作做好了，接下来的内容是如何在创建好的EGL环境下工作。首先OpenGl ES 的渲染必须新开一个线程，并为该线程绑定显示设备及上下文环境（Context）。因为前面有说过OpenGl指令必须要在其上下文环境中才能执行。所以我们首先要通过 `eglMakeCurrent()` 方法来绑定该线程的显示设备及上下文。
```kotlin

```
7. 当我们绑定完成之后，我们就可以进行`RenderLoop`循环了。EGL的工作模式是双缓冲模式。直到调用了`eglSwapBuffers`这条指令的时候，将前后台的FrameBuffer进行交换。
```kotlin

```
8. 在所有的操作都执行完之后，我们要销毁资源。特别注意，销毁资源必须在当前线程中进行。首先我们销毁显示设备（EGLSurface）,然后销毁上下文（EGLContext）,停止并释放线程，最后终止与EGLDisplay之间的链接。
```kotlin

```
