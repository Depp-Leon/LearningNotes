#### 序章：AI编程大模型

##### 1. AI编程的变革：

1. **IDE 智能提示**：代表了“字符级编程”的黄金时代，从 IntelliSense 到 TabNine，自动补全成为开发者的标配工具。
2. **Copilot 式补全**：开启了“函数级编程”时代。你只要写一句注释或函数名，它就能帮你补出整个实现逻辑，甚至包含 edge case 的处理。
3. **Codex 任务代理**：正式迈入“任务级编程”时代。你用自然语言分配一个任务，它能识别全局上下文，查阅依赖关系，主动完成整个功能开发并提交代码。



##### 2. AI coding细分方向

1. **AI-assisted Coding:** 以 **Cursor** 和 **GitHub Copilot** 为代表，它们是现有开发工作流的 “增强器”，致力于让专业开发者写代码更快、更爽。
2. **End-to-end Agent** 以 **Devin**、**Claude Code** 和 **Amp** 为代表，它们的目标是成为能独立完成任务的 “初级工程师”，将开发者从执行者提升为任务的分配者和审查者。Agent 同时也可能是作为合作者，特别是 Claude Code 这样 CLI based agent，我既可以和他 pair programming，也可以请他帮我干活。
3. **Vibe Coding / UGS:** 以 **v0** 和 **YouWare** 为代表，它们试图将代码的能力赋予非开发者，让他们通过自然语言创造应用和工具。



##### 3. 三款AI开发工具对比

> 这些AI开发工具都是建立在AI大模型基础上的，比如 Anthropic的Claude、OpenAi的ChatGPT（精通对话的模型）、OpenAi的CodeX（精通代码的模型）、Google的Gemini、马斯克的Grok等

1、**GitHub Copilot**

-  “AI 同伴编程”工具，主要是 **代码补全／建议**，嵌入 IDE（VS Code)、JetBrains 等）直接使用。

-  同时提供 Chat 功能、CLI 等扩展形式。 

-  面向大多数开发者，希望加速常规开发流程。

2、**Cursor**

- 是一个 “AI 优化的代码编辑器”（其实是基于 VS Code 或类似，但加了 AI 能力）——也许可以理解为 “编辑器＋AI” 而不仅是 “补全工具” 。 

  > 相当于Vscode + Copilot

- 对于开发者希望“更流畅地在编辑器里”被 AI 辅助的场景更强调。

3、**Claude Code**---**Coding agent(代码代理)**

- 出自 Anthropic，强调 “agentic coding assistant”——能够 **理解整个代码库**、执行命令、修改文件、提交代码、甚至跑测试。 

- 定位于更高级、自动化程度更高的开发辅助工具，不只是补全，而是 “让 AI 做一部分开发流程”

  > 属于命令行编程工具(**CLI工具**）：CLI 是 **C**ommand **L**ine **I**nterface，**命令行接口**
  >
  > **CodeX**同样也是open AI开发的CLI工具：Codex模型是引擎，用於代碼翻譯，而GitHub Co-Pilot則是建立在Codex模型之上的一個服務，使代碼編寫更加容易



##### 4. AI 大模型的一些概念

1. API： **Application Programming Interface**（应用程序编程接口），API 允许两个不相关的软件系统，按照一种约定的“协议”进行对话

   > 比如知乎选择qq登录，知乎调用qq的API接口请求验证身份

2. API key：API密钥，一串唯一的字符作为你的**身份标识**，当调用API时，API key证明有权利使用服务，并开始计费

3. AI语境下的Token：AI 阅读和计算的基本单位/**计费单位**，一个 Token 大约等于 4 个字符或 0.75 个单词，一个汉字通常占用 1 到 2 个 Token，这取决于具体的模型分词算法。

4. 计算机安全语境下的Token：登录网页时也常听到“Token”。这时的 Token 指的是**临时身份证**

   > 你输入账号密码登录。
   >
   > 服务器验证成功，发给你一个 **Token**。
   >
   > 在接下来的 24 小时内，你访问任何页面只需出示这个 Token，不需要重复输密码。

5. 从研发和应用的角度，分为不同的大模型

   > 1. 聊天大模型 (Chat LLMs) —— “全能翻译官 & 创意作家”，比如Chat-gpt、gemini2.0等
   > 2. 代码大模型 (Code LLMs) ，这些模型在训练时“读”了海量的 GitHub 开源库和编程文档，比如：Claude 3.5 
   > 3. 多模态大模型 (Multimodal Models），这类模型不再局限于文字，它们能“看”图、“听”声音，
   > 4. 推理大模型 (Reasoning/O1-style Models)，解决极难的数学题、复杂的逻辑推理、物理化学竞赛题，比如OpenAI o1



##### 5. AI工具、API key、大模型之间的关系

| **组成部分**    | **属性**   | **类似生活中的...**    | **如果没有它会怎样？**             |
| --------------- | ---------- | ---------------------- | ---------------------------------- |
| **Claude Code** | 客户端工具 | 自动驾驶系统           | 你只能手动在网页上复制粘贴代码     |
| **API Key**     | 身份凭证   | 绑定了银行卡的感应钥匙 | 即使有车有引擎，你也发动不了       |
| **大模型**      | 智能核心   | 跑车发动机             | 工具变成了“空壳”，无法理解你的意图 |

1. 各种AI网页聊天不需要API key，但是开发工具(Claude Code)必须配置key

   > 1. 权限不同。网页版AI是在“沙盒”中聊天不能直接读取本地文件，而Claude Code是在本地中操作文件
   > 2. 计费逻辑不同。前者包月模式，后者通常要读取整个项目代码，耗费token巨大
   > 3. 模型切换。后者可以配置不同情况下使用不同的大模型

2. 三者协作流程

   > 1. **本地扫描**：**Claude Code** 扫描你的本地文件夹，读取相关的 `.py` 或 `.js` 文件。
   > 2. **打包请求**：它把你的要求和代码片段打包成一个数据包，并在信封上贴上你的 **API Key**。
   > 3. **云端思考**：数据包通过 **API** 通道传给 **Claude 大模型**。服务器验证 Key 有效且有余额。
   > 4. **返回指令**：**大模型**生成具体的代码修改指令，传回你的电脑。
   > 5. **落地执行**：**Claude Code** 接收到指令，直接修改你硬盘上的文件。



#### 一、安装过程

1. 安装nodejs

   > 1. 确保安装的是18.0版本以后的，需要通过npm进行网络下载
   > 2. 如果不用root，而是普通用户，建议使用第二种安装，避免ccr因为权限问题启动时找不到模型

   ```
   #1
   curl -fsSL https://deb.nodesource.com/setup_lts.x | sudo bash -
   sudo apt-get install -y nodejs
   node --version
   
   #2 
   curl -o- https://raw.githubusercontent.com/nvm-sh/nvm/v0.39.7/install.sh | bash
   source ~/.bashrc
   nvm install --lts    # 安装长期支持版
   nvm use --lts        # 启用它
   ```

2. 安装Claude Code

   ```
   npm install -g @anthropic-ai/claude-code
   ```

3. 安装Claude Code Router

   > 官方的CC只允许链接其Anthropic 自己的服务器，且禁止大陆访问。所以使用该router可以把 Claude Code 的请求拦截下来，转换成 OpenAI 兼容格式。所以可以使用其他大模型来回复该请求。
   >
   > CCR：[GitHub - musistudio/claude-code-router: Use Claude Code as the foundation for coding infrastructure, allowing you to decide how to interact with the model while enjoying updates from Anthropic.](https://github.com/musistudio/claude-code-router)

   ```
   npm install -g @musistudio/claude-code-router
   ```

4. 配置CCR，创建`~/.claude-code-router/config.json`配置文件

   ```
   {
     "APIKEY": "af608f4e26234b8b9a60c9fdfa0a5a82.41rRFgDWFbVxnQuw",
     "LOG": true,
     "API_TIMEOUT_MS": 600000,
     "NON_INTERACTIVE_MODE": false,
     "Providers": [
       {
         "name": "openrouter",
         "api_base_url": "https://open.bigmodel.cn/api/paas/v4/chat/completions",
         "api_key": "af608f4e26234b8b9a60c9fdfa0a5a82.41rRFgDWFbVxnQuw",
         "models": [
           "glm-4.5-air"
         ],
         "transformer": {
           "use": [
             "openrouter"
           ]
         }
       }
     ],
     "Router": {
       "default": "openrouter,glm-4.5-air"
     }
   }
   ```

5. 申请API KEY

   > 可以使用付费大模型，此处使用质谱，注册即送token
   >
   > zhipu：[智谱AI开放平台](https://bigmodel.cn/console/overview)

   ```
   1. 添加API key
   2. 复制API key，填充到上述CCR配置文件中，并根据模型，更改对应的字段
   ```

#### 二、使用

1. 只使用Claude Code(不使用CCR的情况下)，在目标项目下启动

   ```
   #Mac和Linux 环境变量
   export ANTHROPIC_BASE_URL="连接点url"
   export ANTHROPIC_AUTH_TOKEN="你的key"
   cd your-project-folder
   claude
   
   #避免每次启动都要配置环境变量，可以将其写入文件中
   echo -e '\n export ANTHROPIC_AUTH_TOKEN=你的key' >> ~/.bash_profile
   echo -e '\n export ANTHROPIC_BASE_URL=连接点url' >> ~/.bash_profile
   echo -e '\n export ANTHROPIC_AUTH_TOKEN=你的key' >> ~/.bashrc
   echo -e '\n export ANTHROPIC_BASE_URL=连接点url' >> ~/.bashrc
   echo -e '\n export ANTHROPIC_AUTH_TOKEN=你的key' >> ~/.zshrc
   echo -e '\n export ANTHROPIC_BASE_URL=连接点url' >> ~/.zshrc
   
   #重启终端后直接执行
   cd your-project-folder
   claude
   ```

   

2. 使用CCR情况下，在目标项目下启动

   > 1. 若ccr start启动后立马退出，则为其端口3456被占用，杀掉对应进程即可
   >
   > 2. 若cc code启动报网络链接错误，则需要在文件`~/.claude.json`增加一行
   >
   >    ```
   >    "hasCompletedOnboarding": true
   >    ```

   ```
   #配置~/.claude-code-router/config.json文件，填充url和key
   #注：修改完配置后需要重启CCR服务
   
   cd your-project-folder
   ccr mode		// 查看模型
   ccr start		// 启动ccr
   ccr code		// 启动cc
   ```

#### 三、CC教程

> [Claude Code 最佳实践和使用技巧  | Claude Code中文网 - 国内镜像站 | 免费使用指南教程](https://www.claude-cn.org/posts/Claude-code-best-practices-tips.html)
>
> [【Claude Code入门教程】CLAUDE.md完整解析与实战示例_Claude Code安装配置全流程与API代理使用指南 - whatai - 博客园](https://www.cnblogs.com/whatai/p/19142921)

1. `/init` 

   初始化项目配置，会分析整个项目，根目录下生成Claude.md分析文档

2. `/config`

   查看/修改配置

3. `/model`

   切换模型

4. `/cost`

   查看当前会话或累计token使用费用情况

5. `/clear`

   长会话中，Claude 的上下文窗口可能被无关对话、文件内容和命令填满，影响性能甚至分散注意力。每完成一个任务，建议用 ‎⁠/clear⁠ 重置上下文窗口。

6. 大致流程为

   ```
   1. 启动 Claude：claude
   2. 提出需求：> 给这个项目添加一个用户登录注册功能
   3. Claude 会扫描 CLAUDE.md、项目文件，列出一个 plan（子任务清单），可能先让你确认 plan
   4. 你确认 plan 后，Claude 会依次对文件做更改（生成、修改、删除等），在每步修改前询问你是否接受
   5. 最后 Claude 会生成提交、PR、commit message 等
   6. 你可以接着让 Claude 审查改动、写文档、重构、修 bug 等
   ```


#### 四、最佳实践

[Claude Code最佳实践（Claude Code: Best practices）  | Claude Code中文网 - 国内镜像站 | 免费使用指南教程](https://www.claude-cn.org/posts/Claude-code-best-practices.html#e-代码库问答-)

##### 1. 代码库问答

新成员入职时，可用 Claude Code 学习和探索代码库。你可以像和项目工程师结对编程时一样向 Claude 提问。Claude 会自动搜索代码库，比如

> - 日志是如何工作的？
> - 如何新增 API 端点？
> - foo.rs⁠ 第 134 行的 ‎⁠async move { ... }⁠ 有什么作用？
> - CustomerOnboardingFlowImpl⁠ 处理了哪些边界情况？
> - 为什么第 333 行调用 ‎⁠foo()⁠ 而不是 ‎⁠bar()⁠？
> - baz.py⁠ 第 334 行在 Java 中的等价实现是什么？

​		

##### 2. Claude操作git

1. 搜索 git 历史，回答“v1.2.3 包含了哪些更改？”、“谁负责这个功能？”、“这个 API 为什么这样设计？”等问题。建议明确让 Claude 查 git 历史。
2. 编写提交信息。Claude 会自动查看你的更改和最近历史，生成包含所有相关上下文的提交信息。
3. 处理复杂 git 操作，如回滚文件、解决 rebase 冲突、比较和移植补丁。



##### 3. 提供图片

1. 拖拽图片到输入框
2. 提供图片文件地址



##### 4. 提供URL

在提示中粘贴具体 URL，Claude 会自动拉取和阅读。为避免同一域名反复请求权限（如 docs.foo.com），用 ‎⁠/permissions⁠ 添加域名到允许列表。



##### 5. 多Claude

简单高效的方法是让一个 Claude 写代码，另一个审查或测试。类似多工程师协作，分开上下文有时更好：

> 1. 用 Claude 写代码
> 2. 用 ‎⁠/clear⁠ 或在另一个终端启动第二个 Claude
> 3. 让第二个 Claude 审查第一个 Claude 的工作
> 4. 再启动一个 Claude（或再次 ‎⁠/clear⁠），读取代码和审查反馈
> 5. 让这个 Claude 根据反馈修改代码

你也可以让一个 Claude 写测试，另一个写通过测试的代码。甚至可以让 Claude 实例间通过草稿本交流，指定谁写谁读。

这种分工往往比单 Claude 处理所有任务效果更好