**前提：本文记录使用AI Coding的经验，记录新人第一个接触AI Coding 碰到的任何问题**
注：第一次使用的是cursor

1. 安装时推荐配置：**IDE Layout**，IDE Layout 是真正的“VS Code 式”经典布局，Agent Layout 更偏向于聊天。
2. Cursor是一个完全独立的软件，不是Android Studio的插件，在使用的时候只需把本地项目文件夹用Cursor打开就行。
3. Agent的概念：Agent类似于一个生活在电脑里面的人，他可以和你对话，对你的需求进行任务拆分，并自己调用工具来完成任务。
4. 在使用的时候同时打开Cursor 和 Android Studio, 在Cursor里面进行修改代码，然后save, 代码就会同步到Android Studio那边
5. 在使用的时候，先用Plan模式详细表明自己的想法，并与Plan不断迭代方案，可以定义一句话作为Agent开始工作的标识，在最终方案确定好后，让Agent开始写代码。对于修改的代码可以全部撤销，或全部保留。
6. 在根目录下面新建两个文件夹和一个文件
   * 两个文件夹：[aiDevTasks], [tasks]第一个用于存放对ai的规则，第二个存放项目理解PRD和ai生成的task_PRD.md
   * 一个文件：[.cursorrules]存放AI的“全局配置文件”
7. 对于AI的标准要求的越详细，越严格越好，必要给他太多自我发挥的空间，不然它可能为了省力而偷懒，例如ai做导航栏，居然使用的时每次切换创建新的Fragment的方式。
8. 对于第 7 点的补充。作为掌舵人真的要考虑的越多越好，从Projects tab切换到项目内部时也是创建新的Fragment(晕倒)
9. 现在发现，ai会下意识通过打补丁的方式解决问题，不会从架构层面往长远考虑。
10. 对于添加功能来说，一个一个做，这样的话，做完一个采用一个（Keep All），如果有不满意的话可以撤销或重做。


# 沉淀文档
对于简单的报错信息：
```text
将下面的报错，转为成AI好理解的提示词，让AI解决BUG，要求不能通过“打补丁”的方式临时解决。
...(报错信息)
```
对于从figma设计稿得到矢量图绘制UI界面

```text
Figma Access Token:...(替换为自己的⚠️)
首页无图片:
https://www.figma.com/design/xxxxxxxxxx/V0.1?node-id=xxxx-xxxx&m=dev(替换为自己的⚠️)
...
使用我提供的 Figma Personal Access Token 和精确的 Dev Mode 链接，立即获取完整设计数据。
要求（一次性全部完成，禁止任何打补丁）
1. 使用 Figma REST API（Files + Images Export）完整读取当前 node-id 对应的 Frame
2. 提取以下所有内容（必须 100% 来自这个 Frame）：
   - 所有矢量图标的完整 SVG 路径数据
   - 每个控件的精确尺寸（width/height，单位 px → 转 dp）
   - 控件之间的精确间距（左右上下边距、对齐方式）
   - 层级关系、Auto Layout 约束、背景色、圆角、阴影
   - 文字的字体、大小、颜色、行高
3. 必须导出所有矢量图标为 Android VectorDrawable (.xml)：
   - 保留原始路径、fill、stroke、opacity
   - 文件名 = Figma 节点名小写下划线
   - 放入 res/drawable/
   - 禁止使用任何 Material Icons、系统图标、PNG、tint 替代
4. 根据提取的尺寸和约束关系，一次性完整生成的状态布局，像素级一模一样
5. 输出：
   - 所有导出的 drawable 文件名列表 + 对应 Figma 节点名(如果已经存在则跳过)
   - 完整的布局 XML（如果对应的布局已经存在则调整，否则创建）
   - 必要的 dimens.xml 更新
   - 真机验证结果：有图 ↔ 无图切换时页面高度无任何跳动
```
```text
现在你已经通过 MCP 成功加载了我的 Figma 设计稿。
从现在开始，你生成任何 UI 代码的唯一数据来源就是当前 MCP JSON，禁止任何凭记忆、凭感觉、凭“差不多”的行为。
具体要求（缺一不可，否则直接重做）：
1. 所有图标必须使用 Figma 中的原始组件实例（Instance），包括：
...
2. 必须导出为 Android Vector Drawable（XML），不能用 PNG，不能用 Compose ImageVector 手写
3. 必须 100% 保留 Figma 中的路径、填充颜色、stroke、布尔运算、opacity
4. 必须把这些图标保存到 res/drawable/ 下，文件名严格等于 Figma 节点名称（小写 + 下划线）
5. 所有图标大小必须严格等于 Figma 中的像素尺寸（24x24 就 24dp，20x20 就 20dp），不能写 wrap_content 或固定 24dp
6. 有任何无法导出并转换为Android Vector Drawable的图标，必须详细告知我！！！
```
