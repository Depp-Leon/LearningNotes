### 1. Git原理

git分为两个仓库，一个是`本地的`，一个是`远程的`。
git add .和git commit都是`针对你的本地仓库`。
git add做了什么，你可以简单理解是标记下那些文件要被提交到`本地仓库`。
git commit就是把你标记的文件提交到`本地仓库`
git pull是从远程仓库拉代码并merge到你的本地仓库，pull是两个命令的合（fetch和merge）

### 2. Git基本操作

1. 查看状态、包括所在分支、变更的文件信息

   ```
   git status
   ```

2. 展示版本信息

   ```
   git log
   
   commit e62ae9e1171a13a6a87c0149e00c448a5b73b4ce (HEAD -> feature_707_refactoring_jyd, origin/feature_707_refactoring, feature_707_refactoring)
   ```

      1. `HEAD -> feature_707_refactoring_jyd`

         - **HEAD** 是 Git 中的特殊指针，表示你**当前所在的工作位置**（当前检出的提交，**本地分支**）。
         - **-> feature_707_refactoring_jyd** 表示 HEAD 当前指向本地分支 feature_707_refactoring_jyd。
         - **含义**：你当前检出的分支是 feature_707_refactoring_jyd，工作目录和索引（staging area）基于这个分支的最新提交。

      2. `origin/feature_707_refactoring`

         - 这是一个**远程跟踪分支**（remote-tracking branch）与本地的同名分支建立了跟踪关系。

           > 即直接使用 `git push` 不用指定远程分支 会自动推送到该跟踪分支
           >
           > 如果其他分支使用 `git push -u` 指定了要跟踪的其他远端分支，则origin就会变更为那个分支

         - **origin** 是远程仓库的名称（默认是 origin，可以通过 `git remote -v` 查看）。

         - **含义**：这是你本地仓库中记录的远程分支 `origin/feature_707_refactoring` 的状态，通常通过 `git fetch` 更新。它表示远程仓库中这个分支的最新提交。

      3. `feature_707_refactoring`

         - 这是你本地的另一个分支 `feature_707_refactoring`，它也指向当前这个提交。

3. 查看日志

   ```
   git log 
   ```

4. 更新(同步)远端分支信息

   ```
   git fetch		# 更新远程分支信息，但不改变本地分支
   git pull  		# 从远程仓库拉代码并merge到你的本地仓库，pull是两个命令的合（fetch和merge）
   ```

5. 正常提交流程

   ```
   git add 
   git commit -m "fix/feat(): "
   git pull / git pull --rebase
   git push
   若有冲突，解决冲突
   git commit	#进入合并提交文件：ctrl+O 、 回车 、 ctrl+x
   git push	#提交，会有一条提交记录一条合并记录
   ```

   1. 关于提交流程有感

      1. 先git add . 再git pull下，如果pull下的文件对本次修改的文件有冲突，因为没提交，所以直接pull将会覆盖本次对该文件的修改(git会提示)。需要先提交commit才能pull。(因为这种情况下没提交，本地没有给你提供待解决的冲突)。这种情况下优点是gitlab中记录只有一次，少了merge记录，缺点是有冲突发现不了会被覆盖
      2. 先commit 再pull，本地提交记录会保存该次提交，pull时会将仓库提交记录拉取下来与本地记录比较，如果有相同文件的操作，那么会在本地文件中标记冲突，需要解决后重新add  commit 再push。优点是有冲突直接可以找到并解决，缺点是push后远端仓库记录中会有两次

   2. 为什么要使用rebase：同步远程分支更改并将本地未提交的更改重新应用在最新提交之上的命令。这样先`commit`再`pull`就不会多出来合并提交的记录了

      ```
      git pull --rebase 相当于执行下面两步
      1. git fetch：从远程仓库拉取最新的更改到本地。
      2. git rebase：将本地分支上的未提交更改重新应用在远程分支的最新提交之上。 #只pull的话是这步是merge
      ```

   3. 提交注释修改

      ```
      git commit --amend
      ```

   4. 提交多行注释

      ```
      git commit   #省略-m 进入nano文本编辑模式
      
      #在文本编辑后
      按下 Ctrl + O（保存文件）。
      按下 Enter 确认文件名（通常是默认的临时文件）。
      按下 Ctrl + X（退出编辑器）。
      ```

   5. 首次提交指定远端仓库

      ```
      git push -u origin feature		// 本地分支 feature，想推送到远程仓库 origin
      ```

6. 各种文件恢复

   ```
   git clean -f    #清除未跟踪的文件，clean用于从工作区删除这次新建的文件/文件夹
   git clean -fd	#清理未跟踪的文件和目录
   git clean -fdx	#只删除未跟踪的目录
   
   git checkout <修改的文件地址>  #从工作区恢复
   git restore <修改的文件地址>	#从工作区恢复，根据不同的版本选择这两句，看status提示	
   git checkout <commit-hash> -- <file-path>	# 单独给目标文件恢复到那次提交前的状态(已经add暂存过的)
   
   git reset <修改的文件地址>					#从暂存区恢复
   git restore --staged <修改的文件地址>		 #从暂存区恢复，根据不同的版本选择这两句，看status提示	
   ```

   > checkout还有切换分支的作用

7. 版本回溯

   1. `git checkout`:当你使用 `git checkout` 命令切换到历史上的某个提交时，你实际上是进入了一个“分离头指针”（detached HEAD）状态。此时你不再处于任何分支上，而是直接指向了一个具体的提交

      ```
      // 版本回溯
      git checkout <commit-hash> -- <file-path>#只是把本地恢复成提交记录的状态，但是不会修改版本历史
      git checkout ./	#修改到最新的状态
      
      // 恢复原始
      git checkout feature_707_refactoring  #切换到某个提交之后实际上进入分离指针状态，此时再切换回初始
      ```

   2. `git reset`： 三种模式

      ```
      // 版本回溯
      git reset <commit-hash>#重置到某次提交的状态，会影响版本历史、暂存区和工作目录。之前的记录都被清掉
      --soft：仅重置 HEAD指针指向，不修改工作目录和暂存区。适用于将提交回退，但保留修改，以便重新提交。
      		到目标提交记录之间的变更全部放在暂存区
      --mixed（默认）：重置 HEAD 和暂存区，但不修改工作目录。适用于将提交回退，同时将文件移出暂存区。
      		到目标提交记录之间的变更全部放在工作目录
      --hard：重置 HEAD、暂存区和工作目录。所有未提交的更改都会丢失，非常危险，但适用于完全回退到某个提交的状态。
      
      // 恢复原始
      1. git reflog  // 获取HEAD@{1}
      2. git reset --soft|mixed|hard  HEAD@{1}	// 选择之前回溯的类型
      ```

   3. 用于从一个分支中选择特定的提交并将其应用到当前分支

      ```
      git cherry-pick <commit-hash>
      ```

   4. 强制提交，当远端log与本地不同时

      ```
      git push --force		//强制将本地版本记录推送到远程仓库
      ```

8. 保存当前的工作进度

   ```
   git stash		#保存工作区跟踪和暂存区的所有文件,保存为一条记录形式
   git stash -u 	#保存工作区和暂存区的所有文件(包括未跟踪的，即新建的文件)
   
   git stash list	#列出所有的 stash 条目
   git stash show stash@{0}	#根据 stash 标识符查看其具体内容(具体保存的操作过的文件)
   git stash apply #恢复最新的 stash 内容并将它们应用到当前分支
   git stash apply stash@{1}	#恢复指定的 stash 内容并将它们应用到当前分支
   
   git stash drop stash@{0}	#删除特定的 stash 条目(将会把保存的文件记录都删除)
   git stash clear				#清空所有 stash 条目
   
   git stash pop	#恢复并删除 stash，当于 apply + drop 的组合
   // 如果恢复的时候有冲突，那么就手动处理冲突。而且stash记录不会被删除。实际上更改玩冲突就可以正常提交
   
   git fsck --lost-found		#列出最近删除的stash commit记录
   git stash apply <commit-hash>	#恢复到对应stash记录下
   ```

9. 查看修改的内容/记录

   ```
   git show <commit-hash>				 #查看某次提交修改内容
   git show <commit-hash> -- <file-path> #查看某次提交某个文件修改的详细内容
   
   git log -- <file-path>		#查看某个文件的历史改动记录
   git log -p -- <file-path> 	#查看文件在多个提交中的改动, log -p 相当于show
   ```

10. 查看和**比较**改动

    ```
    git diff		 	#比较工作目录中的文件和暂存区的文件之间的差异
    git diff --cached	#比较暂存区中的文件和最新提交之间的差异
    git diff commit1 commit2 #比较两个特定提交的差异
    git diff commit1 commit2 -- <file-path> #比较某个文件在两次提交之间的差异
    git diff [file]		#仅显示某些文件的改动
    git diff branch1 branch2	#比较两个分支的差异
    ```



### 3. 分支相关

1. 查看分支

   ```
   git branch			// 列出所有本地分支
   git branch -a		// 列出所有分支（包括远程分支）
   ```

2. 创建分支

   ```
   git branch <branch-name>	// 创建新分支
   git checkout -b <branch-name>	// 创建并切换到新分支
   ```

3. 重命名分支

   ```
   git branch -m <new-branch-name>		// 重命名当前分支
   ```

4. 切换分支

   ```
   git checkout <branch-name>	// 切换到已有分支
   git switch <branch-name>	// （Git 2.23+）以上版本可用
   ```

5. 删除分支

   ```
   git branch -d <branch-name>		// -d 表示安全删除本地分支，只有在分支已合并到其他分支时才会成功
   git branch -D <branch-name>		// 如果要强制删除（即使未合并）
   ```

6. 合并分支

   ```
   git merge <branch-name>	   // 将其他分支合并到当前分支
   
   // 如果有冲突，需要手动解决冲突，然后：
   git add <file>
   git commit
   
   git merge --abort		// 未解决冲突情况下，中止合并
   ```

7. 合并分支方式2

   ```
   git cherry-pick <commit-hash> // 适用于单条提交，可以避免产生合并记录
   ```

8. 变基(合并方式3)

   > 合并的一种方式，将目标分支的提交移动到当前分支新提交之前(移动到底部)

   ```
   git rebase <branch-name>	// 将当前分支的提交“移动”到目标分支的最新提交之上，保持线性历史
   								改变的是当前分支，比merge少了一个合并记录
   git rebase --continue		// 如果有冲突，解决后执行
   git rebase --abort			// 中止变基
   ```

9. 查看分支历史

   ```
   git log --graph --oneline --all
   ```

   1. `--graph` 显示分支合并关系。
   2. `--oneline` 简化每行输出。
   3. `--all` 显示所有分支。

10. 推送和拉取分支

    1. 推送本地分支

    ```
    git push origin <branch-name>	// 推送本地分支到远程仓库，origin为远端仓库名，<>分支名
    
    // 若首次推送时，则使用下面设置上游分支。
    git push --set-upstream origin <branch-name>
    git push -u origin <branch-name>				// 上句的简写
    // 设置上游分支之后，再次推送该分支的内容就可以只使用git push了
    ```

    2. 拉取远程分支

    ```
    git fetch origin			 // 同步远程分支信息
    git checkout <branch-name>	 // 如果本地没有该分支，Git 会自动创建并跟踪远程分支
    ```

11. 暂存未提交更改（与分支切换相关）

```
   git stash
   git stash pop		// 恢复暂存
```

11. merge 和 rebase 的区别：

    - merge 保留分支历史，生成合并提交。
    - rebase 重写历史，提交记录更线性，但会改变历史（谨慎用于已推送的分支）。

12. 常用步骤

    > 以A为master主分支、B为自己独立分支为例

    1. 本地创建自己的分支/使用远程其他分支
    2. 切换自己分支B(如果是远程分支则git自动创建并跟踪远程分支)
    3. 如果主分支A有其他人新提交的内容，需要切换到A 执行pull之后、再切换到B 执行合并A
    4. 开发并提交自己分支内容(如果要同步远端分支，则需要推送到远程) 
    5. 切换到主分支A，合并分支B(merge/rebase)

13. 关于git分支合并产生的一些问题：

    自己的分支提交之后，切换到主分支；

    1. 主分支pull最新提交之后，rebase子分支，解决冲突。切换到子分支，merge主分支。

       ```
       git checkout develop_jyd
       git add .
       git commit 
       git checkout develop
       git pull 
       git rebase debelop_jyd
       // 解决冲突
       git push --force	 // 强制提交
       git checkout develop_jyd
       git merge 	// 顺利合并，没有冲突
       ```

       优点：主分支基于子分支的提交为基底，子分支合并主分支的时候不需要再解决一次冲突。

       缺点：主分支与远端分支的历史记录出现区别，此时提交只能强制push

    2. 主分支pull最新提交之后，merge子分支，解决冲突，产生合并记录。此时要么切换到子分支合并重新解决一次冲突，要么删除掉子分支，新建子分支(推荐方式2)。

       ```
       // 前面部分与上面相同
       git merge develop_jyd
       // 解决冲突，提交产生合并记录
       // ps：如果使用git cherry-pick <commit-hash> 可以避免产生合并记录
       git push
       git branch -d develop_jyd
       git push origin --delete develop_jyd //删除掉远端的分支
       git checkout -b develop_jyd //新建子分支，同时切换到子分支
       git push origin develop_jyd //将新建子分支推送到远端
       ```

       优点：子分支不需要重新合并主分支，少一次解决冲突。

       缺点：会产生一条合并记录(如果只有一条提交记录，那么可以使用`cherry-pick`解决)

    

### 4. 仓库


1. 远程克隆仓库

   ```
   使用git clone就可以直接把仓库拉到本地，并同步log，并把当前作为本地git仓库
   使用git clone git@时需要配公钥到github上
   使用git clone http@时不需要配公钥，根据仓库的类型可以允许克隆,但是需要在github上给用户权限才能push
   ```

2. git信任本地仓库。当检测到当前用户可能不是该仓库的所有者时会不能执行命令

   ```
   git config --global --add safe.directory /home/leslie/ChenxinSpace/normal_develop
   ```

3. git设置身份信息

   ```
   git config --global user.name "jiayuandi"
   git config --global user.email "jiayuandi@v-secure.cn"
   git config --l
   ```

4. git设置免密

   ```
   ssh-keygen -t rsa -b 4096 -C "jiayuandi@v-secure.cn"	#生成公钥/私钥文件
   cat .ssh/id_rsa.pub				
   #复制公钥到git上
   ```

5. 为什么git需要输入密码：原因是使用了不同的认证方式（SSH vs HTTPS），当在远端配置了公钥后，可以直接使用SSH拉取，并且后续操作不需要密码。而使用HTTPS则每次都需要输入密码。

     > **SSH 认证**：如果你使用的是 SSH 方式（即 URL 以 `git@` 开头），Git 会使用你本地配置的 SSH 密钥来进行认证。如果你已经设置了 SSH 密钥并且将其添加到你的 Git 服务（如 GitHub）上，Git 就不需要每次都输入密码。
     >
     > **HTTPS 认证**：如果你使用的是 HTTPS 方式（即 URL 以 `https://` 开头），Git 会要求输入用户名和密码（或者 Git 服务使用的个人访问令牌 `token`）。如果没有缓存凭据或没有配置凭据助手，每次拉取时就会要求输入密码。

6. 关于公钥私钥

     1. **作用**：公钥和私钥是一对加密密钥，主要用于加密和身份验证。公钥是公开的，任何人都可以获取；而私钥是保密的，只有你自己拥有。

        > **公钥**：用于加密或验证签名。你将公钥放在服务器上（如 `authorized_keys` 中），然后当你尝试连接时，服务器会使用该公钥验证你的身份。
        >
        > **私钥**：用于解密或生成签名。你保存在本地，不应与他人共享

     2. 原则上**只要两个支持SSH协议都可以使用公钥和私钥来实现免密连接**；公钥和私钥是基于 SSH 协议的，与git并没有之间关系。只是连接git仓库使用到了公钥私钥，将本地公钥放入git仓库只是方便仓库认证本地的。

     3. **linux系统下的使用**：

        ```
        打开终端。
        运行相同的命令：ssh-keygen -t rsa -b 4096 -C "your_email@example.com"
        这将生成 RSA 密钥对，保存到 ~/.ssh/id_rsa 和 ~/.ssh/id_rsa.pub。
        将公钥 id_rsa.pub 内容复制到远程服务器的 ~/.ssh/authorized_keys 文件中。
        ```

     4. **windows系统下的使用**：

        ```
        打开 Git Bash 或者其他终端工具。
        运行命令：ssh-keygen -t rsa -b 4096 -C "your_email@example.com"
        这将生成一个 4096 位的 RSA 密钥对。
        默认情况下，密钥会保存在 ~/.ssh/id_rsa 和 ~/.ssh/id_rsa.pub 中。
        将公钥 id_rsa.pub 内容复制到远程服务器的 ~/.ssh/authorized_keys 文件中。
        ```

     5. 生成密钥**命令解析**

        ```
        ssh-keygen
        // 默认生成 ED25519 类型的密钥
        // 默认密钥长度为 256位（ED25519的固定长度）
        // 默认注释为 username@hostname（例如 jiayuandi@chenxin-jiayuandi）
        ```

        ```
        ssh-keygen -t rsa -b 4096 -C "jiayuandi@v-secure.cn" -f ~/.ssh/my_rsa_key
        // -t	指定生成 RSA 类型的密钥。
        // -b	指定密钥长度为 4096位。
        // -C	指定注释为 jiayuandi@v-secure.cn。
        // -f 	指定将密钥生成到目标路径。默认情况下，密钥会保存在用户主目录下的 ~/.ssh/id_rsa 和 ~/.ssh/id_rsa.pub 文件中。上面命令将生成 ~/.ssh/my_rsa_key（私钥）和 ~/.ssh/my_rsa_key.pub（公钥）
        ```

7. github仓库菜单功能：

   | 菜单          | 核心功能       | 典型场景             |
   | ------------- | -------------- | -------------------- |
   | Code          | 查看和管理代码 | 浏览文件、提交代码   |
   | Issues        | 问题跟踪       | 报告 Bug、讨论功能   |
   | Pull Requests | 代码审查与合并 | 提交新代码、团队协作 |
   | Actions       | 自动化工作流   | 测试、部署           |
   | Projects      | 项目管理       | 任务规划、可视化进度 |
   | Wiki          | 文档知识库     | 写指南、记录知识     |
   | Security      | 安全监控       | 检查漏洞、保护代码   |
   | Insights      | 数据分析       | 查看贡献、流量统计   |
   | Settings      | 仓库配置       | 权限管理、分支规则   |

   实际用途举例：

   ```
   Code: 放源代码和 README.md，告诉别人这是啥。
   Issues: 有人报告 “App 闪退”，你创建 Issue 跟踪。
   Pull Requests: 你修好闪退，提交 PR 给队友审查。
   Actions: 设置自动测试，每次推送跑 make test。
   Projects: 规划 “v1.0 发布”，列出任务。
   Wiki: 写 “安装步骤” 给用户看。
   Security: 更新有漏洞的库。
   Insights: 检查谁贡献最多。
   Settings: 设置只有你能直接推送到 main。
   ```

   常用：

   ```
   新手: 先用 Code、Issues 和 Wiki，够日常用了。
   团队: 加 Pull Requests 和 Projects 协作。
   高级: 用 Actions 自动化，Security 保安全。
   ```

8. 关于github需要使用2FA来进行二次认证

     1. Authenticator 是什么工具？

        `Authenticator` 是一种多因素认证（2FA）工具，用于增强账户的安全性。它通过生成**一次性验证码**（TOTP, Time-based One-Time Password）来确保登录过程的安全。

        > `Authenticator` 还有一个功能就是微软的密码和地址同步，只要登陆上微软账号就可以将该账号的密码同步到手机设置
        >
        > 如果使用不了2FA的情况下，使用github-recovery-codes.txt这里面的恢复码进行登录

     2. QR 码是什么？

        `QR 码`（Quick Response Code）是一种二维条形码，可以快速存储和读取信息。

        当你启用双因素认证（2FA）时，服务提供商通常会提供一个 **QR 码**，该二维码包含了生成一次性密码所需的密钥。

        使用 Authenticator 应用扫描这个二维码后，应用就能开始生成与该服务匹配的动态验证码。这个二维码通常**只会在初次设置 2FA 时显示一次**。

     3. 2FA（双因素认证）是什么？

        **2FA（Two-Factor Authentication，双因素认证）** 是一种安全机制，它要求用户提供两种不同类型的认证信息才能成功登录账户。即**知识因素**(pin码或者密码)、**持有因素**(用户持有的设备)。

        国内常见的如：短信验证码、面容识别等；Authenticator通过基于时间生成的验证码。



### 5.  常见问题

1. 关于`git log`展示版本信息`git push`时如果检测到某次`commit`有`error`(比如有文件大于100M)，那么就需要本地版本回溯然后再解决问题，不然历史提交记录永远会保存这次的提交信息，导致后续永远`push`错误

2. github报错：`ssh: connect to host github.com port 22: Connection refused fatal`

   原因：`dns`被污染，导致解析`github.com`域名解析出来是本地`127.0.0.1`

   解决：在`host`文件中加入域名映射：`140.82.113.4 github.com`

3. git tatus 时出现下面的情况，而且怎么也restore不掉

   ```
     修改：     common/zav/zav_lib (新提交)
   ```

   **原因**：common/zav/zav_lib是一个子模块(子git)、子模块内有改动、新提交、变更了分支

   ```
   git submodule status		// 查看子模块状态
   ```

   **解决办法**：

   1. 进入子模块内，使其git status展示到最新

   2. 再退出到主模块，使用git status查看情况，如果还有这种现象，执行下面代码

      ```
      git submodule update --init
      ```

      > 初始化子模块 (--init)：--init 参数会初始化子模块，即读取 .gitmodules 文件中的配置信息（如子模块的路径和 URL），并为每个子模块在本地仓库中创建必要的配置。
      >
      > 更新子模块 (update)：在初始化子模块后，update 会从子模块的远程仓库拉取指定的版本（通常是 .gitmodules 文件中记录的提交哈希值）的内容，填充到对应的子模块目录中

4. Git，执行了错误的命令，如何回退：

   案例：使用了`git pull --rebase`，解决完冲突，使用失误，执行了`git rebase --skip`,导致本次提交的内容全部消失

   解决：使用 `git reflog`：**Git 所有操作都会记录在 reflog 中，包括 rebase、skip、continue 等。**

   ```
   // 1. 查看reflog： 
   git reflog
   
   // 2. 找到想回退的那句操作，将对应行复制其头（比如HEAD@{2}），然后执行
   git reset --hard HEAD@{2}
   ```
   
5. windows使用git相关命令，由于本地用户名为中文名导致的验证不了本地私钥，可以创建一个英文用户的路径并复制私钥到其目录下，再临时指定当前HOME的环境变量

   ```
   # 将~/用户/.ssh/rsa私钥复制到下面的路径下
   $env:HOME="C:\Users\leslie\Documents\my_ssh_home"		// 临时指定
   ```

   