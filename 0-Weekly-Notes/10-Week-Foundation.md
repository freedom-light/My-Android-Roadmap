# 第10周：滤镜渲染与综合应用
目标: 整合前三周内容，掌握通过片段着色器实现基础滤镜以及基于 LUT 的色彩映射滤镜，并完成综合 Demo。

核心任务
* LUT（Look-Up Table）滤镜实现
* 滤镜切换与交互
* 综合练习与总结

## LUT(LookUpTable)
LUT全称LookUpTable，也称为颜色查找表，它代表的是一种映射关系，通过LUT可以将输入的像素数组通过映射关系转换输出成另外的像素数组。
将3DLUT转为2D纹理，一个3DLUT可以看作一个三维的网格，常见的为`64x64x64`，通过`平铺式布局`将3D网格拍成2D纹理，具体做法为：固定蓝色通道的值，那么剩下的`R`和`G`就形成了一个2D的平面，这个平面也称为`Quad`。
多个这样的切片(Quad)整齐的排列在2D纹理中，形成一个大网格，8个切片一排，8个切片一列，这样就组成了一个 512x512 （64*8 = 512）的2D纹理，其中包含了 64 个切片。
### 三线形插值
当我们进行颜色查找时，我们拥有的rgb是连续的，但在LUT纹理的单元格是离散的，因此需要进行三线形插值来使得到的颜色映射尽量的平滑和连续。三线形插值可以分解为：
* 在同一个切片内进行双线性插值(R和G方向)
* 在相邻的两个切片之间进行线性插值(B方向)
### 3DLUT转为2D纹理全过程
1. 解析 .cube 文件，获取 3D LUT 数据
2. 创建2D Bitmap
   * 确定2D纹理大小
   * 创建空的Bitmap
3. 将3DLUT数据映射到2DBitmap
4. 将Bitmap上传到GPU成为OpenGL ES 2D 纹理
   * 生成纹理ID
   * 绑定纹理
   * 设置纹理参数
   * 将Bitmap数据上传到纹理
   * 回收Bitmap, **一旦数据上传到 GPU，Bitmap 对象就不再需要了**
   * 返回纹理ID

#### 通过保存bitmap使后续需要加载纹理时提高效率
1. 定义缓存路径
```kotlin
private const val CACHE_DIR_NAME = "lut_bitmaps" // 在应用缓存目录下的子目录名称
```
2. 生成缓存文件名
```kotlin
val cacheDir = File(context.cacheDir, CACHE_DIR_NAME)
if (!cacheDir.exists()) {
    cacheDir.mkdirs() // 创建缓存目录
}
// 将 .cube 后缀替换为 .png 作为缓存文件名
// Kotlin 字符串扩展函数，用于替换字符串中的子串
val cacheFileName = originalFileName.replace(".cube", ".png", ignoreCase = true)
return File(cacheDir, cacheFileName)
```
3. 检查缓存
   * 存在：从**硬盘**加载Bitmap,之后直接转为OpenGL纹理
   * 不存在：走正常的逻辑，解析.cube文件，生成Bitmap
4. 保存缓存：将生成的Bitmap保存在硬盘中，方便下次使用
```kotlin
val cacheFile = getCacheFile(context, originalFileName)
// 将 bitmap 的像素数据以 PNG 格式写入到 out (即 cacheFile) 中
try {                          //use 块执行完毕后资源 (FileOutputStream) 会被自动关闭
    FileOutputStream(cacheFile).use { out ->
        // 使用PNG格式保存，无损压缩
        // 用于将 Bitmap 的图像数据压缩并写入到指定的输出流中
        bitmap.compress(Bitmap.CompressFormat.PNG, 100, out)
        Log.d(TAG, "Bitmap saved to cache: ${cacheFile.absolutePath}")
    }
} catch (e: Exception) {
    Log.e(TAG, "Failed to save bitmap to cache: ${cacheFile.absolutePath}", e)
}
```



