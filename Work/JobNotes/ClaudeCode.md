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

   > 1. 若ccr start启动后立马退出，则为其端口3456被占用，去掉对应进程即可
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

5. 大致流程为

   ```
   1. 启动 Claude：claude
   2. 提出需求：> 给这个项目添加一个用户登录注册功能
   3. Claude 会扫描 CLAUDE.md、项目文件，列出一个 plan（子任务清单），可能先让你确认 plan
   4. 你确认 plan 后，Claude 会依次对文件做更改（生成、修改、删除等），在每步修改前询问你是否接受
   5. 最后 Claude 会生成提交、PR、commit message 等
   6. 你可以接着让 Claude 审查改动、写文档、重构、修 bug 等
   ```

   