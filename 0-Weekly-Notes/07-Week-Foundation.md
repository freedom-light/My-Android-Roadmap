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
// 获取OpenGL ES的渲染目标
eglDisplay = EGL14.eglGetDisplay(EGL14.EGL_DEFAULT_DISPLAY)
if(eglDisplay == EGL14.EGL_NO_DISPLAY){
    throw RuntimeException("unable to get EGL14 display")
}
```
2. 初始化显示设备，第一参数代表Major版本，第二个代表Minor版本。如果不关心版本号，传0或者null就可以了。初始化与 EGLDisplay 之间的连接：`eglInitialize()`
```kotlin
val version = IntArray(2)
if(!EGL14.eglInitialize(eglDisplay, version, 0, version, 1)){
    throw RuntimeException("EGL14 init error")
}
```
3. 下面我们进行配置选项，使用`eglChooseConfig()`方法,Android平台的配置代码如下：
```kotlin
// 参数的顺序没有要求，是按键值对的形式匹配的，EGL14.EGL_NONE要放在最后
val configAttribs = intArrayOf(
    EGL14.EGL_RED_SIZE, 8,
    EGL14.EGL_GREEN_SIZE, 8,
    EGL14.EGL_BLUE_SIZE, 8,
    EGL14.EGL_ALPHA_SIZE, 8,
    EGL14.EGL_DEPTH_SIZE, 0,
    EGL14.EGL_STENCIL_SIZE, 0,
    EGL14.EGL_RENDERABLE_TYPE, EGL14.EGL_OPENGL_ES2_BIT,
    EGL14.EGL_NONE
)
val configs = arrayOfNulls<EGLConfig>(1)
val numConfig = IntArray(1)
if(!EGL14.eglChooseConfig(
        eglDisplay,
        configAttribs, 0,
        configs, 0, 1,
        numConfig, 0))
{
    throw RuntimeException("eglChooseConfig error")
}
val config = configs[0] ?: throw RuntimeException("No EGLConfig found")
```
4. 接下来我们需要创建OpenGl的上下文环境 `EGLContext` 实例，这里值得留意的是，OpenGl的任何一条指令都是必须在自己的OpenGl上下文环境中运行，我们可以通过`eglCreateContext()`方法来构建上下文环境：`eglCreateContext`中的第三个参数可以传入一个`EGLContext`类型的变量，改变量的意义是可以与正在创建的上下文环境共享OpenGl资源，包括纹理ID,FrameBuffer以及其他Buffer资源。如果没有的话可以填写Null.
```kotlin
val contextAttribs = intArrayOf(
    EGL14.EGL_CONTEXT_CLIENT_VERSION, 2,
    EGL14.EGL_NONE
)
eglContext = EGL14.eglCreateContext(eglDisplay, config, EGL14.EGL_NO_CONTEXT, contextAttribs, 0)
if(eglContext == EGL14.EGL_NO_CONTEXT){
    throw RuntimeException("eglContext create error")
}
```
5. 通过上面四步,获取 OpenGl 上下文之后，说明EGL和OpenGl ES端的环境已经搭建完毕。下面的步骤是如何将EGL和设备屏幕连接起来，使用EGLSurface，通过EGL库提供eglCreateWindowSurface可以创建一个实际可以显示的surface.当然，如果需要离线的surface，我们可以通过`eglCreatePbufferSurface`创建。`eglCreateWindowSurface()`
```kotlin
val surfaceAttribs = intArrayOf(EGL14.EGL_NONE)
eglSurface = EGL14.eglCreateWindowSurface(eglDisplay, config, surfaceHolder.surface, surfaceAttribs, 0)
if(eglSurface == EGL14.EGL_NO_SURFACE){
    throw RuntimeException("eglSurfacae create error")
}
```
6. 通过上面的步骤，EGL的准备工作做好了，接下来的内容是如何在创建好的EGL环境下工作。首先OpenGl ES 的渲染必须新开一个线程，并为该线程绑定显示设备及上下文环境（Context）。因为前面有说过OpenGl指令必须要在其上下文环境中才能执行。所以我们首先要通过 `eglMakeCurrent()` 方法来绑定该线程的显示设备及上下文。
```kotlin
if(!EGL14.eglMakeCurrent(eglDisplay, eglSurface, eglSurface, eglContext)){
    throw RuntimeException("eglMakeCurrent error")
}
```
7. 当我们绑定完成之后，我们就可以进行`RenderLoop`循环了。EGL的工作模式是双缓冲模式。直到调用了`eglSwapBuffers`这条指令的时候，将前后台的FrameBuffer进行交换。
```kotlin

```
8. 在所有的操作都执行完之后，我们要销毁资源。特别注意，销毁资源必须在当前线程中进行。首先我们销毁显示设备（EGLSurface）,然后销毁上下文（EGLContext）,停止并释放线程，最后终止与EGLDisplay之间的链接。
```kotlin
EGL14.eglMakeCurrent(eglDisplay, EGL14.EGL_NO_SURFACE, EGL14.EGL_NO_SURFACE, EGL14.EGL_NO_CONTEXT)
EGL14.eglDestroySurface(eglDisplay, eglSurface)
EGL14.eglDestroyContext(eglDisplay, eglContext)
EGL14.eglTerminate(eglDisplay)
```
## 3.创建基础着色器
### GLSL是什么
GLSL（全称 OpenGL Shading Language）是一种专门为图形渲染编程设计的着色器语言，主要用于编写运行在 GPU（图形处理器）上的 “着色器程序”，控制图形渲染的各个阶段（如顶点变换、像素颜色计算等）。

GLSL使用类型限定符而不是通过读取和写入操作来管理输入和输出。着色器主要分为顶点着色器（Vertex Shader）和片段着色器（Fragment Shader）两部分。

在OpenGL程序中使用着色器有一套固定的流程，清楚每个函数是干什么的就好，不需要太纠结这个事情。

1. 顶点着色程序的源代码和片段着色程序的源代码分别写入到一个文件里（或字符数组）里面，一般顶点着色器源码文件后缀为.vert，片段着色器源码文件后缀为.frag；
2. 使用glCreateshader()分别创建一个顶点着色器对象和一个片段着色器对象；
3. 使用glShaderSource()分别将顶点/片段着色程序的源代码字符数组绑定到顶点/片段着色器对象上；
4. 使用glCompileShader()分别编译顶点着色器和片段着色器对象（最好检查一下编译的成功与否）；
5. 使用glCreaterProgram()创建一个着色程序对象；
6. 使用glAttachShader()将顶点和片段着色器对象附件到需要着色的程序对象上；
7. 使用glLinkProgram()分别将顶点和片段着色器和着色程序执行链接生成一个可执行程序（最好检查一下链接的成功与否）；
8. 使用glUseProgram()将OpenGL渲染管道切换到着色器模式，并使用当前的着色器进行渲染；

```kotlin
// OpenGL着色器管理工具类
object ShaderHelper {
    fun compileVertexShader(context: Context, resourceId: Int): Int{
        return compileShader(GLES20.GL_VERTEX_SHADER, readShaderSource(context, resourceId))
    }
    fun compileFragmentShader(context: Context, resourceId: Int): Int{
        return compileShader(GLES20.GL_FRAGMENT_SHADER, readShaderSource(context, resourceId))
    }

    // 读取着色器源文件的工具函数
    private fun readShaderSource(context: Context, resourceId: Int): String{
        // 通过上下文，根据资源文件ID打开对应的源文件
        // use函数，进行资源自动管理，inputStream代表打开的输入流
        return context.resources.openRawResource(resourceId).use { inputStream ->
            // it - lambad表达式的默认参数名，这里指BufferedReader实例
            inputStream.bufferedReader().use { it.readText() }
        }
    }

    // 编译着色器
    private fun compileShader(type: Int, shaderCode: String): Int{
        // 创建着色器对象
        // 参数type 指定要创建的着色器类型（顶点着色器/片段着色器）
        val shader = GLES20.glCreateShader(type)

        // 加载着色器源代码
        GLES20.glShaderSource(shader, shaderCode)
        // 编译着色器
        GLES20.glCompileShader(shader)

        //检查编译状态
        val compileStatus = IntArray(1)
        GLES20.glGetShaderiv(shader, GLES20.GL_COMPILE_STATUS, compileStatus, 0)
        // 错误处理
        if(compileStatus[0] == 0){
            throw RuntimeException("Shader compile error: ${GLES20.glGetShaderInfoLog(shader)}")
        }

        // 返回着色器句柄
        return shader
    }

    // 链接着色器 将编译好的顶点着色器和片段着色器链接成一个完整的GPU可执行程序
    fun linkProgram(vertexShader: Int, fragmentShader: Int): Int{
        // 创建着色器对象
        val program = GLES20.glCreateProgram()
        // 附加顶点着色器及片段着色器
        GLES20.glAttachShader(program, vertexShader)
        GLES20.glAttachShader(program, fragmentShader)

        // 链接程序
        GLES20.glLinkProgram(program)
        // 检查链接状态
        val linkStatus = IntArray(1)
        GLES20.glGetProgramiv(program, GLES20.GL_LINK_STATUS, linkStatus, 0)

        //错误处理
        if(linkStatus[0] == 0){
            GLES20.glDeleteProgram(program)
            throw RuntimeException("Program linking failed: ${GLES20.glGetProgramInfoLog(program)}")
        }

        // 返回程序句柄
        return program
    }
}
```

private fun readShaderSource(context: Context, resourceId: Int): String
说一下context参数，Android上下文参数，用于访问应用资源。resourceId资源ID参数，指向raw资源目录中的着色器文件。
### GLES是什么
`GLES` 是 OpenGL for Embedded Systems 的缩写，通常直接叫做 `OpenGL ES`。一个专门为嵌入式设备设计的2D/3D图形渲染API，它是桌面版OpenGL的简化，轻量级版本，使其更适合移动设备有限的资源。

## 4.定义几何图形和绘制
### 简单了解计算机图形学(也称电脑图形学)
简单地说，计算机图形学的主要研究内容就是研究如何在计算机中表示图形、以及利用计算机进行图形的计算、处理和显示的相关原理与算法。图形通常由点、线、面、体等几何元素和灰度、色彩、线型、线宽等非几何属性组成。从处理技术上来看，图形主要分为两类，一类是基于线条信息表示的，如工程图、等高线地图、曲面的线框图等，另一类是明暗图，也就是通常所说的真实感图形。
### 顶点数据
`顶点`：顶点是存储一系列基本绘图所需属性的基本元素，例如二维或三维空间中的点、或曲面上的多个点。在着色器中，与顶点相连的元素称为图元，图元内部应上色的区域称为片段，顶点的集合称为顶点组或顶点数组。[1]而在OpenGL中，顶点默认会包含位置、法向量、颜色、第二色彩、纹理坐标等属性[2]，而其可以透过着色器编程添加更多属性。

计算机图形学中的顶点是一个包含了所有必要信息的`数据结构`，这些信息用于最终在屏幕上生成一个像素(片段)，一个顶点通常包含以下信息(不限于此)
1. 位置：这是从几何学中继承来的核心属性。它是一个3D或2D坐标 (x, y, z)，定义了顶点在空间中的位置。这是唯一必需的属性。
2. 颜色：这个顶点本身的颜色值 (r, g, b, a)。在光栅化过程中，顶点之间的颜色会进行平滑插值，从而产生渐变效果。
3. 纹理坐标：这是一个2D坐标 (u, v)，它像一个`地图标记`，告诉GPU这个顶点对应到纹理图片（比如墙纸、皮肤）上的哪个位置。这样，GPU才能把图片正确地“贴”到3D模型上。
4. 法向量：这是一个方向向量 (nx, ny, nz)，它垂直于顶点所在的表面。这是实现光照计算的关键。光照模型（如Phong模型）需要用法向量来计算光线与表面的夹角，从而决定该顶点应该有多亮。
5. 切线/副切线：用于更复杂的纹理映射（如法线贴图），它们与法向量一起构成一个坐标系，用于在曲面 correctly 地定位纹理方向。
6. 骨骼权重/索引：用于角色动画。它定义了该顶点受到哪些`骨骼`的影响，以及影响的程度有多大。
7. 自定义属性：通过着色器编程，你完全可以定义任何你需要的顶点属性，比如顶点ID、速度、温度等等，用于实现特定的视觉效果。

GPU拿到一大堆这样的`顶点指令`(顶点着色器处理)，然后根据它们组装出三角形(图形装配)，最后在每个三角形内部（光栅化），根据三个顶点的各种属性（颜色、纹理坐标等）进行插值，为每一个小片段（像素）计算出最终的颜色（片段着色器处理），最终渲染出图像。

顶点中需要这么多信息的原因：GRU的渲染流程是基于顶点的，但最终效果是体现在片段(像素)上的。

### 为什么GPU将图像转为三角形的组合来处理
首先简单来说：三角形是构成所有其他复杂图形最简单，最可靠，最高效的原子单位。
1. 可靠性：三角形是`永远平坦`的。数学原理：在三维空间中，三个点永远且只能确定一个唯一的平面。你无法让一个三角形是“扭曲”的。三角形消除了不确定性。GPU可以确信地计算三角形内部每个像素的颜色、光照和纹理，因为它知道这是一个完美的平面。
2. 高效性：GPU是大规模并行处理器，有成千上万个核心。为了达到极致性能，它希望处理的任务尽可能统一和简单。如果GPU需要同时处理三角形、矩形、五边形、圆形的绘制指令，其内部电路会变得极其复杂和低效。将一切归为三角形，处理的全流程都只为处理三角形而高度优化，效率极高。
3. 通用性：三角形是`原子图形`
   * 多边形分解：任何复杂的多边形（无论有多少条边）都可以被分解成一系列三角形，这个过程称为三角剖分。
   * 曲面近似：像球体、圆柱体、角色模型这样的曲面，也可以用无数个微小的三角形网格来近似模拟。三角形越多，模型表面就越光滑。
