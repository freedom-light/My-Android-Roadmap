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
