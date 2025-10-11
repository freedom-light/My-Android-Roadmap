# 第7周：EGL 环境与着色器基础
目标：掌握手动搭建 EGL 环境的方法，并学会使用 GLSL 着色器绘制基础二维几何图形，为后续图形渲染打下基础。

核心任务
* 创建自定义 SurfaceView 并实现渲染线程
* 手动搭建 EGL 环境
* 加载并使用 GLSL 着色器
* 定义几何图形数据
* 实现 drawFrame() 与按需渲染
* 管理 EGL 生命周期

## 创建自定义 SurfaceView 并实现渲染线程
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
### 创建自定义线程(1)
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
