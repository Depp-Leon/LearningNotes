1. 使用git

   ```
   git status    #查看状态、包括所在分支、变更的文件信息
   git checkout <修改的文件地址>  #将这个文件恢复之前从版本库的样子	
   ```

2. git push

   ```
   feat：新增功能
   示例：feat: add user authentication
   
   fix：修复 bug
   示例：fix: resolve login crash issue
   
   docs：文档相关的修改
   示例：docs: update API documentation
   
   style：格式（不影响功能的代码变动）
   示例：style: format code according to style guide
   
   refactor：重构代码（不增加功能或修复 bug）
   示例：refactor: simplify user login logic
   
   perf：优化性能
   示例：perf: improve database query speed
   
   test：添加测试代码或修复测试
   示例：test: add unit tests for payment processing
   
   chore：杂项（不属于功能或修复的修改）
   示例：chore: update build scripts
   
   revert：撤回之前的提交
   示例：revert: undo "feat: add user authentication"
   
   build：影响构建系统或依赖项的更改
   示例：build: update dependency versions
   
   ci：持续集成相关的更改
   示例：ci: update CI configuration
   ```

3. git如何确认与远端的差异

   > git pull是从远程仓库拉代码并merge到你的本地仓库，pull是两个命令的合（fetch和merge）

   ```
   git fetch：更新远程分支信息，但不改变本地分支。
   git status：显示是否有提交差异。
   git diff origin/main：详细查看与远程分支之间的代码差异。
   git log --oneline --graph --all：可视化查看分支和提交情况。
   ```

4. `git cherry-pick` 用于从一个分支中选择特定的提交（commit）并将其应用到当前分支

   ```
   git cherry-pick <commit-hash>
   ```

5. git先add再pull和commit后再pull

   ```
   操作	先 git add 再 git pull	先 git commit 再 git pull
   暂存的改动	不会直接参与 pull 的合并	已 commit 的改动会参与合并
   冲突处理	暂存改动留在暂存区，冲突解决后需重新 add/commit	冲突解决后需重新 add/commit
   合并完成	冲突解决完成后，你的暂存内容依然保留	合并时直接将提交内容与远程合并
   适用场景	适合准备提交但未完成工作时	适合已完成工作并准备同步远程仓库时
   ```

   > add暂存文件，存储这段时间改动的文件
   >
   > commit提交到本地仓库，本地仓库记录的有之前远端仓库的提交记录和自己的这次的提交记录
   >
   > push将会将本地仓库的记录传递给远端仓库，此时远端仓库的合并记录就是自己本地仓库的。(确保每次都是最新)
   >
   > pull将会拉取远端修改记录与本地做比较，从而会有冲突。解决完冲突完成本地合并（有冲突的文件如果已经add或者commit需要重新addcommit），再push远端将会有合并记录和commit记录

6. git add. git commit git pull git push的理解

   > pull是为了本地 commit 和远程commit 的对比记录,git 是按照文件的行数操作进行对比的,如果同时操作了某文件的同一行（在两个人操作同一个分支才会冲突）那么就会产生冲突,git 也会把这个冲突给标记出来,这个时候就需要先把和你冲突的那个人拉过来问问保留谁的代码,然后在 git add && git commit && git pull 这三连,再次 pull 一次是为了防止再你们协商的时候另一个人给又提交了一版东西,如果真发生了那流程重复一遍,通常没有冲突的时候就直接给你合并了,不会把你的代码给覆盖掉

   > 先commit不会出现代码覆盖或丢失的情况
   >
   > 不commit先pull将会提示pull失败的情况，根据提示的冲突修改才会pull成功
   >
   > 先commit再pull将会多一条merger记录，几个commit几个记录。先pull再commit只会有一个。
   >
   > 所以以后早中晚先pull等一些操作再说，只要不嫌麻烦，那么就能减少麻烦

   > 首先应该理解git的原理。
   > git分为两个仓库，一个是`本地的`，一个是`远程的`。
   > git add .和git commit都是`针对你的本地仓库`。
   > git add做了什么，你可以简单理解是标记下那些文件要被提交到`本地仓库`。
   > git commit就是把你标记的文件提交到`本地仓库`
   > git pull是从远程仓库拉代码并merge到你的本地仓库，pull是两个命令的合（fetch和merge）
   > 所以理论上pull在这两个之前之后都没什么问题

   > commit 后 push 之前需要 pull --rebase，然后再 push。pull 最好加 --rebase，可以将刚刚的 commit rebase 至远程最新的 commit，这样有时可避免直接 pull 造成的无用 merge 提交（因为 pull 等于 fetch && merge，如果远程提交比你本地提交新，就会产生 merge）。当然如果有冲突，是否加 --rebase 还都要手动解决冲突，然后再 push。

7. `git` 版本回溯

   ```
   git log 查看提交记录号
   git reset --soft <记录号> #--soft：保留所有内容在暂存区（你可以直接 git commit）
   						 #若不加，则直接回退，不保存暂存区内容
   git status 查看状态
   git checkout .   #丢弃工作区的改动，如果还需要这些改动不要执行这句
   ```

8. 关于`git clone`远程拉仓库

   ```
   使用git clone就可以直接把仓库拉到本地，并同步log，并把当前作为本地git仓库
   使用git clone git@时需要配公钥到github上
   使用git clone http@时不需要配公钥，根据仓库的类型可以允许克隆,但是需要在github上给用户权限才能push
   ```

9. 关于`git log`展示版本信息

   ```
   commit e62ae9e1171a13a6a87c0149e00c448a5b73b4ce (HEAD -> feature_707_refactoring, origin/feature_707_refactoring, feature_707_refactoring_jyd)
   ```

   1. `origin/feature_707_refactoring`

      - 这是一个**远程跟踪分支**（remote-tracking branch）与本地的同名分支建立了跟踪关系。

        > 即直接使用 `git push` 不用指定远程分支 会自动推送到该跟踪分支
        >
        > 如果其他分支使用 `git push -u` 指定了要跟踪的其他远端分支，则origin就会变更为那个分支

      - **origin** 是远程仓库的名称（默认是 origin，可以通过 `git remote -v` 查看）。

      - **含义**：这是你本地仓库中记录的远程分支 `origin/feature_707_refactoring` 的状态，通常通过 `git fetch` 更新。它表示远程仓库中这个分支的最新提交。

   2. `HEAD -> feature_707_refactoring_jyd`

      - **HEAD** 是 Git 中的特殊指针，表示你**当前所在的工作位置**（当前检出的提交）。
      - **-> feature_707_refactoring_jyd** 表示 HEAD 当前指向本地分支 feature_707_refactoring_jyd。
      - **含义**：你当前检出的分支是 feature_707_refactoring_jyd，工作目录和索引（staging area）基于这个分支的最新提交。

   3. `feature_707_refactoring`

      - 这是**本地分支**的名称

10. `git push`时如果检测到某次`commit`有`error`(比如有文件大于100M)，那么就需要本地版本回溯然后再解决问题，不然历史提交记录永远会保存这次的提交信息，导致后续永远`push`错误

11. github报错：`ssh: connect to host github.com port 22: Connection refused fatal`

    原因：`dns`被污染，导致解析`github.com`域名解析出来是本地`127.0.0.1`

    解决：在`host`文件中加入域名映射：`140.82.113.4 github.com`

12. 关于git提交有感：

    1. 先git add . 再git pull下，如果pull下的文件对本次修改的文件有冲突，因为没提交，所以直接pull将会覆盖本次对该文件的修改(git会提示)。需要先提交commit才能pull。(因为这种情况下没提交，本地没有给你提供待解决的冲突)。这种情况下优点是gitlab中记录只有一次，少了merge记录，缺点是有冲突发现不了会被覆盖
    2. 先commit 再pull，本地提交记录会保存该次提交，pull时会将仓库提交记录拉取下来与本地记录比较，如果有相同文件的操作，那么会在本地文件中标记冲突，需要解决后重新add  commit 再push。优点是有冲突直接可以找到并解决，缺点是push后远端仓库记录中会有两次

13. 当使用`commit`注释打错了，可以使用如下命令，进入修改注释

    ```
    git commit --amend
    ```


14. `git stash`:保存当前的工作进度（暂存区和工作区的改动）,然后做其他的动作(切换分支、拉取等)。然后可以再恢复，恢复时直接在当前进度恢复。`git stash` 非常适用于**中断当前工作并稍后恢复**的场景

    > 当使用`stash`后，保存的文件将不会被`git`显示。即使用`git status`将提示无文件需要提交。
    >
    > 所以后面一定需要使用`stash apply`或者`stash pop`

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
    ```

15. 查看某次提交修改的详细内容

    ```
    git show <commit-hash>
    git show <commit-hash> -- <file-path> #查看某次提交某个文件修改的详细内容
    
    git log -- <file-path>		#查看某个文件的历史改动记录
    git log -p -- <file-path> 	#查看文件在多个提交中的改动, log -p 相当于show
    ```

16. `git checkout` 和`git restore`作用的区别

    1. `git checkout`的作用：1.切换分支；2.从某个分支或提交中恢复文件。

    ```
    git checkout ./ 		#作用是 将当前目录（./）下的所有文件还原为暂存区或最后一次提交的状态
    git checkout 分支名称	 #作用是 切换分支
    ```

    2. `git restore`的作用：还原未暂存(未add)的修改

    ```
    git restore ./			#还原未暂存的修改
    git restore file.txt	#还原特定文件
    ```

    3. `git reset`的作用：还原暂存的修改，退出暂存区

    ```
    git reset ./			#将暂存的文件撤出暂存区
    git reset <file>		#撤出特定文件
    ```

    4. `git restore --staged`的作用：取消暂存，同3

    ```
    git restore --staged <文件>
    ```

17. `git diff`: 可以查看和分析改动，包括查看两个文件之间、文件修改前后、暂存区中的文件和最新提交之间的差异

    ```
    git diff		 	#比较工作目录中的文件和暂存区的文件之间的差异
    git diff --cached	#比较暂存区中的文件和最新提交之间的差异
    git diff commit1 commit2 #比较两个特定提交的差异
    git diff commit1 commit2 -- <file-path> #比较某个文件在两次提交之间的差异
    git diff [file]		#仅显示某些文件的改动
    git diff branch1 branch2	#比较两个分支的差异
    ```

18. git信任本地仓库。当检测到当前用户可能不是该仓库的所有者时会不能执行命令

    ```
    git config --global --add safe.directory /home/leslie/ChenxinSpace/normal_develop
    ```

19. git设置身份信息

    ```
    git config --global user.name "jiayuandi"
    git config --global user.email "jiayuandi@v-secure.cn"
    git config --l
    ```

20. git设置免密

    ```
    ssh-keygen -t rsa -b 4096 -C "jiayuandi@v-secure.cn"
    cat .ssh/id_rsa.pub				
    #复制公钥到git上
    ```

21. git提交多行注释

    ```
    git commit   #省略-m 进入nano文本编辑模式
    
    #在文本编辑后
    按下 Ctrl + O（保存文件）。
    按下 Enter 确认文件名（通常是默认的临时文件）。
    按下 Ctrl + X（退出编辑器）。
    ```

22. git处理冲突

    ```
    git add . 
    git pull  #提示可能会覆盖缓存区
    git commit -m "" #先提交到本地
    git pull  #若有冲突则提示冲突，如无则拉取成功
    解决冲突
    git commit	#进入合并提交文件：ctrl+O 、 回车 、 ctrl+x
    git push	#提交，会有一条提交记录一条合并记录
    
    #如果想要只有一条记录，那么就使用stash缓存变更记录，待pull下来后再看哪块有问题自己改。不过麻烦
    ```

23. git恢复单独文件到之前某个版本

    ```
    git log -- <file-path>		#查看某个文件的历史改动记录
    git checkout <commit-hash> -- <file-path>	# 单独给目标文件恢复到那次提交前的状态(已经add暂存过的)
    ```

24. 关于`git checkout` 和 `git rese`t 的版本回溯

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
       --mixed（默认）：重置 HEAD 和暂存区，但不修改工作目录。适用于将提交回退，同时将文件移出暂存区。
       --hard：重置 HEAD、暂存区和工作目录。所有未提交的更改都会丢失，非常危险，但适用于完全回退到某个提交的状态。
       
       // 恢复原始
       1. git reflog  // 获取HEAD@{1}
       2. git reset --soft|mixed|hard  HEAD@{1}	// 选择之前回溯的类型
       ```

25. `git push --force` 将远端仓库修改为某一提交版本

    ```
    git reset <commit-hash>	// 本地恢复到某一版本(历史提交记录遭到修改)
    
    // 执行修改
    git add .
    git commit 
    
    git push --force		//强制将本地版本记录推送到远程仓库
    ```

26. `git pull --rebase`：同步远程分支更改并将本地未提交的更改重新应用在最新提交之上的命令。这样先`commit`再`pull`就不会多出来合并提交的记录了

    ```
    git pull --rebase 相当于执行下面两步
    1. git fetch：从远程仓库拉取最新的更改到本地。
    2. git rebase：将本地分支上的未提交更改重新应用在远程分支的最新提交之上。 #只pull的话是这步是merge
    ```

    **注意事项**：`git pull --rebase` 需要一个“干净”的工作目录（没有未提交的更改）。

    > 所以必须是git add .    -> git commit -> git pull -- rebase

    ```
    #如果本地有暂存的更改，但尚未提交，git pull --rebase 会提示：
    You have unstaged changes.
    Please commit or stash them before you rebase.
    #此时需要commit 或者 stash
    ```

    **工作流程**：

    1. 检查远程分支的最新提交（通过 `git fetch` 获取）。

    2. 将本地分支的提交临时保存为补丁（类似 `git stash`但不是。这是把`commit`记录给`stash`了）。

    3. 将本地分支重置为远程分支的最新提交。

    4. 依次应用之前保存的补丁到新的基点。

    5. 如果发生冲突，提示用户解决冲突并继续执行

       ```
       git status  # 查看冲突文件
       # 手动解决冲突
       git add <file>  # 标记已解决的文件
       git rebase --continue  # 继续 rebase
       ```

    **优点**：`--rebase` 将避免产生多余的合并提交（`merge commit`），保持分支历史记录的清晰和线性

27. `git rebase`命令：重新整理提交历史，将一个分支上的提交记录移动到另一个分支上(不仅适用于`master`和`branch`分支，也适用于远程分支和本地分支)

    ```
    git rebase [<upstream>] [<branch>]
    # <upstream>：要基于的分支（默认当前分支的上游分支）
    # <branch>： 要进行 rebase 的分支（默认为当前分支）
    # 将上游分支的commit记录移动到当前
    
    git rebase --continue  #发生冲突并解决冲突后继续执行rebase
    git rebase --abort	   #终止当前的 rebase 操作，并恢复到操作前的状态。
    git rebase --skip	   #跳过当前的补丁，即跳过正在导致冲突的提交，并继续应用后续的提交
    ```

28. git 清除未跟踪的文件

    ```
    git clean -f    #清除未跟踪的文件
    git clean -fd	#清理未跟踪的文件和目录
    git clean -fdx	#只删除未跟踪的目录
    git clean -n	#模拟清理（预览将删除哪些文件）
    ```

29. `git stash`找到删除的记录

    ```
    git fsck --lost-found		#列出最近删除的stash commit记录
    
    git stash apply <commit-hash>	#恢复到对应stash记录下
    ```

30. 为什么git需要输入密码：原因是使用了不同的认证方式（SSH vs HTTPS），当在远端配置了公钥后，可以直接使用SSH拉取，并且后续操作不需要密码。而使用HTTPS则每次都需要输入密码。

      > **SSH 认证**：如果你使用的是 SSH 方式（即 URL 以 `git@` 开头），Git 会使用你本地配置的 SSH 密钥来进行认证。如果你已经设置了 SSH 密钥并且将其添加到你的 Git 服务（如 GitHub）上，Git 就不需要每次都输入密码。
      >
      > **HTTPS 认证**：如果你使用的是 HTTPS 方式（即 URL 以 `https://` 开头），Git 会要求输入用户名和密码（或者 Git 服务使用的个人访问令牌 `token`）。如果没有缓存凭据或没有配置凭据助手，每次拉取时就会要求输入密码。

31. github仓库菜单功能：

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

32. 关于公钥私钥

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
         ssh-keygen -t rsa -b 4096 -C "jiayuandi@v-secure.cn -f ~/.ssh/my_rsa_key
         // -t	指定生成 RSA 类型的密钥。
         // -b	指定密钥长度为 4096位。
         // -C	指定注释为 jiayuandi@v-secure.cn。
         // -f 	指定将密钥生成到目标路径。默认情况下，密钥会保存在用户主目录下的 ~/.ssh/id_rsa 和 ~/.ssh/id_rsa.pub 文件中。上面命令将生成 ~/.ssh/my_rsa_key（私钥）和 ~/.ssh/my_rsa_key.pub（公钥）
         ```

33. 关于github需要使用2FA来进行二次认证

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
    
34. 分支相关操作：

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

    3. 切换分支

       ```
       git checkout <branch-name>	// 切换到已有分支
       git switch <branch-name>	// （Git 2.23+）以上版本可用
       ```

    4. 删除分支

       ```
       git branch -d <branch-name>		// -d 表示安全删除本地分支，只有在分支已合并到其他分支时才会成功
       git branch -D <branch-name>		// 如果要强制删除（即使未合并）
       ```

    5. 合并分支

       ```
       git merge <branch-name>	   // 将其他分支合并到当前分支
       
       // 如果有冲突，需要手动解决冲突，然后：
       git add <file>
       git commit
       
       git merge --abort		// 未解决冲突情况下，中止合并
       ```

    6. 重命名分支

       ```
       git branch -m <new-branch-name>		// 重命名当前分支
       ```

    7. 查看分支历史

       ```
       git log --graph --oneline --all
       ```

       1. `--graph` 显示分支合并关系。
       2. `--oneline` 简化每行输出。
       3. `--all` 显示所有分支。

    8. 推送和拉取分支

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

    9. 暂存未提交更改（与分支切换相关）

       ```
       git stash
       git stash pop		// 恢复暂存
       ```

    10. 变基

       ```
       git rebase <branch-name>	// 将当前分支的提交“移动”到目标分支的最新提交之上，保持线性历史
       
       git rebase --continue		// 如果有冲突，解决后执行
       git rebase --abort			// 中止变基
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

35. `git push` 的 `-u` 参数：

    ```
    git push -u origin feature		// 本地分支 feature，想推送到远程仓库 origin
    ```

    1. `-u` 是 `--set-upstream` 的缩写，用于在**推送分支时设置上游跟踪关系**。
    2. 它简化了后续的 `git push` 和 `git pull` 操作，使 Git 知道默认的远程目标。
    3. 常见用法是**首次推送新分支**时，例如 `git push -u origin <branch>`。

