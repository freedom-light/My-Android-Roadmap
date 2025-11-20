**前提：本文记录使用AI Coding的经验，记录新人第一个接触AI Coding 碰到的任何问题**
注：第一次使用的是cursor

### 安装阶段
1. 安装时推荐配置：**IDE Layout**IDE Layout 是真正的“VS Code 式”经典布局，Agent Layout 更偏向于聊天。
2. Cursor是一个完全独立的软件，不是Android Studio的插件，在使用的时候只需把本地项目文件夹用Cursor打开就行。
3. Agent的概念：Agent类似于一个生活在电脑里面的人，他可以和你对话，对你的需求进行任务拆分，并自己调用工具来完成任务。
4. 在使用的时候同时打开Cursor 和 Android Studio, 在Cursor里面进行修改代码，然后save, 代码就会同步到Android Studio那边
5. 在使用的时候，先用Plan模式详细表明自己的想法，并与Plan不断迭代方案，可以定义一句话作为Agent开始工作的标识，在最终方案确定好后，让Agent开始写代码。对于修改的代码可以全部撤销，或全部保留。
6. 在根目录下面新建两个文件夹和一个文件
   * 两个文件夹：[aiDevTasks], [tasks]第一个用于存放对ai的规则，第二个存放项目理解PRD和ai生成的task_PRD.md
   * 一个文件：[.cursorrules]存在AI的“全局配置文件”
7. 对于AI的标准要求的越详细，越严格越好，必要给他太多自我发挥的空间，不然它可能为了省力而偷懒，例如ai做导航栏，居然使用的时每次切换创建新的Fragment的方式。
8. 对于第 7 点的补充。作为掌舵人真的要考虑的越多越好，从Projects tab切换到项目内部时也是创建新的Fragment(晕倒)
9. 现在发现，ai会下意识通过打补丁的方式解决问题，不会从架构层面往长远考虑。
