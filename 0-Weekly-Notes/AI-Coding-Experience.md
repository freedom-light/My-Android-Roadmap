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
### 对于简单的报错信息：
```text
将下面的报错，转为成AI好理解的提示词，让AI解决BUG，要求不能通过“打补丁”的方式临时解决。
...(报错信息)
```

### 对于从figma设计稿得到矢量图绘制UI界面
```text
Figma Access Token:...(替换为自己的⚠️)

首页无图片:
https://www.figma.com/design/xxxxxxxxxx/V0.1?node-id=xxxx-xxxx&m=dev(替换为自己的⚠️)
...(替换为自己的⚠️)

使用我提供的 Figma Personal Access Token 和精确的 Dev Mode 链接，立即获取完整设计数据。
要求（一次性全部完成，禁止任何打补丁）
【矢量图标导出规则 - 最严格部分，缺一不可】
1. 使用 Figma REST API（Files + Images Export）完整读取当前 node-id 对应的 Frame
2. 提取以下所有内容（必须 100% 来自这个 Frame）：
   - 所有矢量图标的完整 SVG 路径数据
   - 每个控件的精确尺寸（width/height，单位 px → 转 dp）
   - 控件之间的精确间距（左右上下边距、对齐方式）
   - 层级关系、Auto Layout 约束、背景色、圆角、阴影
   - 文字的字体、大小、颜色、行高
3. 必须导出所有矢量图标为 Android VectorDrawable (.xml)：
   - 保留原始路径、fill、stroke、opacity、布尔运算
   - 文件名 = Figma 节点名小写下划线
   - 放入 res/drawable/
   - 禁止使用任何 Material Icons、系统图标、PNG、tint 替代
4. 根据提取的尺寸和约束关系，一次性完整生成的状态布局，像素级一模一样
5. 输出：
   - 所有导出的 drawable 文件名列表 + 对应 Figma 节点名(如果已经存在则跳过)
   - 完整的布局 XML 文件（activity_xxx.xml 或 fragment_xxx.xml，如果已有同名布局则整体替换/调整）
   - 必要的 dimens.xml 更新
   - 真机验证结果：有图 ↔ 无图切换时页面高度无任何跳动

你现在通过 Figma MCP（或我提供的 Personal Access Token + Dev Mode 链接）已成功加载当前设计稿的完整 JSON 数据。
从此刻起，你生成任何 Android UI 代码的唯一来源就是这个 MCP JSON 或通过 Figma REST API 获取的精确数据。禁止任何凭记忆、凭经验、凭“差不多”、凭 Material Design 默认值的行为。一旦违反，直接重做全部。
- 使用 View 系统 XML 布局（ConstraintLayout 优先），像素级 100% 还原 Figma（包括有图/无图状态下高度零跳动）
- 所有图标使用 <ImageView android:src="@drawable/xxx" />，固定宽高 = Figma px 值（dp），无 wrap_content，无 scaleType，无 tint
- 尺寸/间距全部提取到 dimens.xml（已存在则复用，新建则添加）
- Auto Layout → 对应 ConstraintLayout 约束或 LinearLayout weights
- 背景、圆角、阴影全部用 XML drawable 或 view 属性实现
- 对应特别设置颜色的控件，清空主题系统默认色调干扰，app:backgroundTint="@null"
```

# OpenSpec 工作流程概览

## 基本流程（3 个阶段）
1. 提出变更提案 → 2. 获得批准 → 3. 实施变更

## 详细步骤

### 阶段 1：创建变更提案
当你想要添加功能或进行更改时：

**步骤 1.1：向 AI 提出需求**  
固定句式（中英文皆可）：
"I want to add [功能名称]. Please create an OpenSpec change proposal for this feature"  
例如：  
- "I want to add a brightness adjustment feature. Please create an OpenSpec change proposal"  
- "我想添加亮度调节功能，创建一个 OpenSpec 变更提案"

**步骤 1.2：AI 创建提案**  
AI 会按照 OpenSpec 格式创建提案，包括：  
- Description（描述）：功能说明  
- Motivation（动机）：为什么需要  
- Proposed Changes（变更内容）：具体改动  
- Technical Details（技术细节）：架构考虑、依赖等  
- Testing（测试）：如何测试  

**步骤 1.3：AI 对照项目规则检查**  
AI 会确保提案：  
- ✅ 遵循 .cursorrules 中的所有规则  
- ✅ 使用资源文件（无硬编码）  
- ✅ 遵循 RenderChain 架构（如果相关）  
- ✅ 使用 Room 进行数据持久化（如果相关）  
- ✅ 匹配 Figma 设计（如果 UI 相关）  
- ✅ 支持 Undo/Redo 和预设（如果适用）

### 阶段 2：审查和批准

**步骤 2.1：你审查提案**  
- 检查是否符合项目约定  
- 验证是否符合你的需求  
- 如有需要，要求澄清或修改  

**步骤 2.2：批准提案**  
如果满意，回复：  
"Go" 或 "执行"  

**重要**：在你说 "Go" 之前，AI 不会进行任何代码更改。

### 阶段 3：实施变更
获得批准后，AI 会：

**步骤 3.1：规划**  
- 如果是主要功能，在 tasks/tasks-项目管理功能.md 中创建任务列表  
- 将工作分解为子任务  
- 标记任务状态：[ ]（待处理）或 [x]（已完成）

**步骤 3.2：实施**  
- 遵循项目结构和约定  
- 按提案创建/更新文件  
- 确保所有资源在正确的文件中  
- 遵循架构模式

**步骤 3.3：验证**  
- 检查 lint 错误  
- 对照 Figma 设计验证（如果是 UI）  
- 测试功能  
- 更新任务列表的完成状态

### 阶段 4 归档
- 功能已全部实现，真机验证通过，无 lint 错误，无 Figma 偏差。
请更新任务状态为全部完成并归档。

## 压缩规则文本字数
规则文本随着使用不断迭代，会逐渐积累，可能会产生无效内容，通过以下内容进行定期压缩，保证AI高效的理解规则。
```text
（规则.md ⚠️⚠️⚠️规则内容）
对以上规则进行无损压缩，执行以下铁律：

1.必须 100% 保留全部规则原意，零遗漏、零歧义、零弱化。
2.只允许删冗余词、合并同义表达、用更短完全等价词替换。
3.若压缩后任何一条规则内容缺失或可产生歧义，立即废弃全部结果并重做。
4.目标：压缩量为10%-20%，每字必有用，AI 一扫即刻完美执行。
5.如果规则已经十分精简，不要刻意追求10%-20%压缩量，避免丢失内容。
6.更新原本的规则文档，我会逐行校验。

现在立即开始压缩。
```
