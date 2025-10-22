# 第9周：手势交互与矩阵变换
目标：为渲染的图形增加手势交互功能，通过矩阵变换实现**平移**和**缩放**操作。

核心任务
* 集成手势检测器
* 实现手势监听器
* 引入矩阵变换
* 更新着色器以支持矩阵
* 传递矩阵并刷新画面

## 集成手势检测器
Android中封装好了两种常用的手势检测器，
1. `GestureDetector` 是一个**通用型**手势检测器，它封装了复杂的 TouchEvent 处理逻辑，让你可以轻松地监听一些预设的、离散的手势事件。
   * 单击：打开项目、确认操作
   * 双击：放大或缩小内容（但这是离散的，不是连续的）
   * 长按：调出上下文菜单
   * 滚动/拖拽：滚动列表或移动视图
   * 快速滑动手势：翻页、删除等
2. `ScaleGestureDetector` 是一个**专精型**手势检测器，专门用于处理缩放（双指 pinch）手势。核心概念如下：
   * 缩放因子：`getScaleFactor()` 方法返回一个浮点数，表示相对于上一次事件的比例变化。例：1.2表达放大20%，0.8%表示缩小20%
   * 焦点：getFocusX() 和 getFocusY() 返回当前两个手指中心的坐标，通常围绕这个点进行缩放操作，用户体验更好。
   * 手势声明周期：onScaleBegin(手势开始) -> （多次）onScale(缩放进行中) -> onScaleEnd(手势结束)

在实际开发中，经常需要同时处理多种手势，例如一个图片浏览器可能需要：
* **单指拖拽**来移动图片
* **双指缩放**来放大缩小图片

这时，你就需要同时使用这两个检测器。

对于**平移**和**缩放**操作，在View中操作矩阵，之后将矩阵传入底层，底层直接进行矩阵乘法即可。不建议使用保存原始状态的方式。

对于对图片的操作，将屏幕数据同比转换为OpenGL数据，不要使用直接写定值的方式。
```kotlin
// 缩放手势监听器
private inner class ScaleListener: ScaleGestureDetector.SimpleOnScaleGestureListener(){
    override fun onScale(detector: ScaleGestureDetector): Boolean {
        val scaleFactor = detector.scaleFactor
        
        // 以手势焦点为中心进行缩放
        val focusX = detector.focusX
        val focusY = detector.focusY
        // 将屏幕焦点坐标转换为 OpenGL 坐标
        val glFocusX = (focusX / width) * 2 - 1
        val glFocusY = 1 - (focusY / height) * 2
        Log.d("MyGLSurfaceView", "${glFocusX}, ${glFocusY}")
        // 应用以焦点为中心的缩放
        applyScaleAroundPoint(scaleFactor, glFocusX, glFocusY)
        updateTransformation()
        return true
    }
}
```
```kotlin
// 平移手势监听器
private inner class GestureListener: GestureDetector.SimpleOnGestureListener(){
    override fun onScroll(
        e1: MotionEvent?,
        e2: MotionEvent,
        distanceX: Float,
        distanceY: Float
    ): Boolean {
        // 将屏幕"距离"转换为 OpenGL 坐标"距离"
        val glDistanceX = (distanceX / width) * 2
        val glDistanceY = (distanceY / height) * 2
        // 考虑当前缩放级别调整平移灵敏度
        val currentScale = getCurrentScale()
        val adjustedDistanceX = glDistanceX / currentScale
        val adjustedDistanceY = glDistanceY / currentScale
        // 使用 Matrix 进行平移
        Matrix.translateM(modelMatrix, 0, -adjustedDistanceX, adjustedDistanceY, 0f)
        updateTransformation()
        return true
    }
    override fun onDoubleTap(e: MotionEvent): Boolean {
        // 重置变换矩阵
        Matrix.setIdentityM(modelMatrix, 0)
        updateTransformation()
        return true
    }
}
```
