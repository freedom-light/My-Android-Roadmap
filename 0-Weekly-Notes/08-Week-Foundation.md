# 第8周：纹理映射与图片展示
目标：学习将 Android 中的 Bitmap 对象作为纹理渲染到矩形上，并保持图片比例不变形，同时支持从相册选取图片。

核心任务
* 更新顶点数据与着色器以支持纹理
* 加载 Bitmap 并创建纹理
* 在 drawFrame() 中绑定并绘制纹理
* 保持图片原始比例
* 支持动态更新纹理	
* 从系统相册选取图片，并显示到纹理中

## 纹理的作用
如果想让图形看起来更真实，我们就必须有足够多的顶点，从而指定足够多的颜色。这将会产生很多额外开销，因为每个模型都会需求更多的顶点，每个顶点又需求一个颜色属性。

纹理是一个2D图片（甚至也有1D和3D的纹理），它可以用来添加物体的细节；你可以想象纹理是一张绘有砖块的纸，无缝折叠贴合到你的3D的房子上，这样你的房子看起来就像有砖墙外表了。因为我们可以在一张图片上插入非常多的细节，这样就可以让物体非常精细而不用指定额外的顶点。

图像源 (网络/磁盘/相机) -> 加载为 Bitmap (在内存中) -> 上传为 Texture (在显存中) -> GPU 使用 Texture 进行渲染

## HandlerThread
`HandlerThread`是Thread的一个子类，类似于`Handler+Thread`。HandlerThread自带一个`Looper`使得他可以通过消息队列来重复使用当前线程，节省资源开销。但是他也存在的问题就是，每一个任务都以队列的形式执行，导致如果有某一个任务执行时间过长的时候，后面的任务就只能等他执行完毕再执行。
 
他和普通线程的区别就是他在内部维护了一个looper(也正是这个原因所以他可以创建对应的Handler)，然后就会通过Handler的方式来通过Looper执行下一个任务。
由于内部的Looper是一个无限循环，所以当我们不再使用HandlerThread的时候就需要调用quit方法来终止他。

**内部原理** = Thread类 + Handler类机制，即：
* 通过继承Thread类，快速地创建1个带有Looper对象的新工作线程
* 通过封装Handler类，快速创建Handler & 与其他线程进行通信

使用步骤
```kotlin
// 步骤1：创建HandlerThread实例对象
// 传入参数 = 线程名字，作用 = 标记该线程
   HandlerThread mHandlerThread = new HandlerThread("handlerThread");

// 步骤2：启动线程
   mHandlerThread.start();

// 步骤3：创建工作线程Handler & 复写handleMessage（）
// 作用：关联HandlerThread的Looper对象、实现消息处理操作 & 与其他线程进行通信
// 注：消息处理操作（HandlerMessage（））的执行线程 = mHandlerThread所创建的工作线程中执行
  Handler workHandler = new Handler( handlerThread.getLooper() ) {
            @Override
            public boolean handleMessage(Message msg) {
                ...//消息处理
                return true;
            }
        });

// 步骤4：使用工作线程Handler向工作线程的消息队列发送消息
// 在工作线程中，当消息循环时取出对应消息 & 在工作线程执行相关操作
  // a. 定义要发送的消息
  Message msg = Message.obtain();
  msg.what = 2; //消息的标识
  msg.obj = "B"; // 消息的存放
  // b. 通过Handler发送消息到其绑定的消息队列
  workHandler.sendMessage(msg);

// 步骤5：结束线程，即停止线程的消息循环
  mHandlerThread.quit();
```
渲染线程渲染部分写成死循环导致消息无法正确传递

修改渲染逻辑，不再采用死循环等待信号触发的方式，将渲染消息也加入消息循环队列中。

## MVP矩阵变换
MVP矩阵变换是将3D空间中的顶点（Vertex）从模型空间（Model Space） 最终转换到裁剪空间（Clip Space） 的一系列矩阵乘法运算。这个过程的目的是为了确定3D物体在2D屏幕上的哪个位置、以何种大小和形态进行绘制。

“MVP”是三个变换阶段的缩写：
* M - 模型矩阵 施加在模型上的空间变换，包含平移变换（translateM）、旋转变换（rotateM）、对称变换（transposeM）、缩放变换（scaleM）
* V - 观察矩阵 施加在观测点上的变换，用于调整观测点位置、观测朝向、观测正方向
* P - 投影矩阵 透视变换，施加在视觉上的变换，用于调整模型的透视效果。投影矩阵不仅应用了透视/正交变形，还将坐标归一化到一个特定的范围内。在此范围外的顶点将被GPU丢弃或裁剪。
  * 透视投影：模拟人眼“近大远小”的效果。物体距离摄像机越远，看起来就越小。这是通过一个视锥体 来定义的，形成一个平截头体形状。
  * 正交投影：没有透视效果，物体的大小与其距离摄像机的远近无关。常用于CAD绘图、2D游戏或UI界面。

Clip Space Position = P * V * M * Model Space Position
* 将三个矩阵相乘，组合成一个 MVP 矩阵，然后与顶点坐标相乘
* 矩阵乘法的顺序是从右向左的。先进行模型变换（M），然后进行观察变换（V），最后进行投影变换（P）。
### MVP矩阵变换的使用(缩放变换)
1. 定义矩阵
```kotlin
private var matrixHandle: Int = 0  // 矩阵句柄

private val modelMatrix = FloatArray(16)       // 模型矩阵
private val viewMatrix = FloatArray(16)        // 观察矩阵，对应2D纹理来说可以不需要观察矩阵
private val projectionMatrix = FloatArray(16)  // 投影矩阵
private val mvpMatrix = FloatArray(16)         // 最终MVP矩阵
```
2. 初始化矩阵
```kotlin
matrixHandle = GLES20.glGetUniformLocation(program, "u_MVPMatrix")

Matrix.setIdentityM(modelMatrix, 0)
Matrix.setIdentityM(viewMatrix, 0)
Matrix.setIdentityM(projectionMatrix, 0)
Matrix.setIdentityM(mvpMatrix, 0)
```
3. 设置MVP矩阵  参数：位置句柄 数量 是否转置 矩阵数据 偏移量
```kotlin
GLES20.glUniformMatrix4fv(matrixHandle, 1, false, mvpMatrix, 0)
```
4. 设置矩阵
```kotlin
例：设置模型矩阵 设置正交投影矩阵
Matrix.scaleM(modelMatrix, 0, 1.0f, scaleY, 1.0f)
Matrix.scaleM(modelMatrix, 0, scaleX, 1.0f, 1.0f)

// 设置正交投影矩阵（适合2D渲染）
val left = -1.0f
val right = 1.0f
val bottom = -1.0f
val top = 1.0f
val near = -1.0f
val far = 1.0f
Matrix.orthoM(projectionMatrix, 0, left, right, bottom, top, near, far)
```
5. 计算MVP矩阵
```kotlin
Matrix.multiplyMM(mvpMatrix, 0, projectionMatrix, 0, modelMatrix, 0)
```
