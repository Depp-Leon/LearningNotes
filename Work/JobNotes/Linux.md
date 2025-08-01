### 1. linux

#### 1.1 快捷键

1. ubuntu快捷键

   ```
   ctrl + L		# 清屏快捷键等同于clear
   ctrl + A		# 移动到光标最左边
   ctrl + E		# 移动到光标最右边
   ctrl + C		# 中断当前命令(直接取消操作，后台无进程)
   ctrl + Z		# 暂停当前运行的程序并放入后台，按 fg 恢复(属于暂停操作，后台有进程)
   ```

2. ubuntu终端复制粘贴快捷键

   ```
    Ctrl+Shift+C
    Ctrl+Shift+V
    Ctrl+Shift+W		//关闭当前终端选项卡
    Ctrl+Shift+T		//在当前终端窗口中打开一个新的选项卡
    Ctrl + Page Down	//切换到下一个标签页
    Ctrl + Page Up		//切换到上一个标签页
    Alt + 数字		  //直接跳转到特定标签页
    
    Ctrl+Alt+T			//打开一个新的终端窗口
   ```

3. kaliLinux历史命令自动补全

   ```
   ctrl + E
   ```

   > Kali 默认使用的是 **Zsh**（Z Shell）作为终端 shell，而不是 Bas

4. Ubuntu文字输入法切换页：

   ```
   PageUp 和 PageDown快捷键
   ```

5. ubuntu进程关闭

   ```
   ctrl + z   #中断进程，后台还会有进程号
   ctrl + c   #取消进程，后台无该进程
   ```

6. 微信回车，同样适用于AI输入框：

   ```
   ctrl + enter 
   shift + enter 
   ```

   

#### 1.2 命令

1. linux下安装gdb

   ```
   apt-get update
   apt-get install  gdb	// 如果杀毒软件运行中将会报错
   ```
   
2. 虚拟机挂起重新开启后，如果显示"有线未托管"，右上角丢失有线连接选项时，解决办法：

   1. 终端重启网络服务

      ```bash
      sudo systemctl restart NetworkManager
      ```

   2. 如果方法1无效，可能是NetworkManager出现问题，使用方法二

      ```bash
      # 1. 进入下面文件，将第五行 managed=False 改为 managed=True
      sudo vim /etc/NetworkManager/NetworkManager.conf
      # 2. 删除NetworkManager配置
      sudo service NetworkManager stop
      sudo rm /var/lib/NetworkManager/NetworkManager.state 
      sudo service NetworkManager start 
      ```

3. kali开启ssh服务

   ```
   service ssh start
   ```

4. linux关于磁盘和内存的命令

   ```
   df -Th		# disk free 查看磁盘占用空间情况		T 显示文件系统类型 h 以人类可读形式(GB MB)
   lsblk		# 查看系统中所有磁盘设备及其信息
   
   free -h 	# 查看内存占用情况
   top			# 实时监控系统进程、CPU 和内存使用情况。
   netstat -ntlp | grep 3306	# 查看特定端口号
   ```

5. ubtuntu关于ssh远程连接的命令

   ```
   //1. 安装ssh，安装后其会自己启动
   sudo apt update
   sudo apt install openssh-server
   //2， 查看ssh状态
   sudo systemctl status ssh
   //3. 如果禁用，打开ufw防火墙
   sudo ufw allow ssh
   
   //4. 连接
   ssh leslie@192.168.3.222
   ```

6. 远程文件拷贝

   ```
   scp file.txt user@remote_host:/path/to/destination/
   scp JYNSAFED libJYVirusScan.so leslie@192.168.3.96:~/
   # file.txt 是本地文件。
   # user@remote_host 是远程主机用户名和地址。
   # /path/to/destination/ 是远程主机上的目标路径。
   
   scp -r <fileDocument>  user@remote_host:/path/to/destination/
   # 如果是文件夹需要加上-r参数
   ```

7. 麒麟系统上文件夹远程连接服务器：在文件夹索引上方输入`ftp://192.168.1.6`即可远程连接

   麒麟系统从远程文件夹拉取文件时，需要拉到左侧"桌面"文件夹下，不可以直接拉到桌面，否则报错

   麒麟系统实际上是linux的另一个发行版，与ubuntu的命令类似，安装时也使用dpkg

8. ubuntu给非root用户添加sudo权限

   ```
   #方式一：
   	# 为用户username添加sudo权限
   	sudo usermod -a -G sudo username
    
   	# 去除用户username的sudo权限
   	sudo usermod -G usergroup username
   
   #方式二：
   	sudo vi /etc/sudoers
   
   	#在root的下面增加下面这行
   	chenye    ALL=(ALL:ALL) ALL
   ```

9. linux授权指令

   1. `chown`命令用于更改文件或目录的所有者和/或所属组

      ```
      chown [OPTION] OWNER[:GROUP] FILE
      # sudo chown -R leslie normal_develop	
      # OWNER：新的所有者用户名或用户ID。
      # GROUP：新的组名或组ID（可选，如果不提供，则不会更改组）。
      # FILE：要修改的文件或目录。
      # OPTION：可选项，常用的选项有 -R（递归修改目录和其内容的所有者）
      ```

   2. `chmod`（change mode）命令用于更改文件或目录的权限。权限决定了谁可以读取、写入或执行文件

      ```
      chmod [OPTION] MODE FILE
      # MODE：设置文件权限的方式，可以使用符号模式或数字模式来指定权限。
      # FILE：要修改权限的文件或目录。
      # OPTION：常用选项包括 -R（递归修改目录及其内容的权限）
      
      chmod +x <file>		// 如果+前面不加所属人，那么默认是所有
      chomd -R 777/+x	<filedocument>   // 对目录加-R，则递归授权 
      ```

10. 关于授权

   ```
   chmod -R		// -R 递归授权，该文件夹下所有权限统一
   ```

   1. 配置文件（如 .ini）：644，仅拥有者可写。
   2. 可执行文件（如脚本、bin）：755，只允许拥有者写，其他用户只可执行。
   3. 库文件或静态资源（.so，resource，tools）：755，供程序访问，其他用户只读
   4. 动态生成或缓存目录（如隔离区或cache）：700 仅应用程序可读写。

   ```
   // 举例
   chmod 755 $jyn_install_path       # 安装目录
   chmod 644 $jyn_install_path/etc/*.ini  # 配置文件
   chmod 755 $jyn_install_path/bin/*      # 可执行文件
   chmod 700 $jyn_install_path/.quarzone/ -R # 隔离区
   ```

11. linux快速找之前命令的命令(以ps举例)

    ```
    !ps				// !后面加上命令，shell 会找到最近一次以 ps 开头的命令
    !?ps			// 查找包含 ps 的命令（不一定以 ps 开头）
    !!				// 重复执行上一次命令。
    
    history			// 查看历史命令记录
    !n				// 直接执行历史记录中编号为 n 的命令
    ```

12. 关于`ls`命令

   ```
   ls <文件夹>		// 不显示.开头的隐藏文件
   ls -A <文件夹>		// 显示除了. 和 .. 以外的所有文件包括隐藏文件
   ls -a <文件夹>		// 显示所有文件
   ```

11. ls命令下，s开头的是符号链接(软链接)

    ```
    ln -s /path/to/file mylink
    //此命令创建一个名为 mylink 的符号链接，指向 /path/to/file。
    ```

12. 一些linux命令

    ```
    dirname  <path>   // linux命令，用来从文件路径中提取出 目录部分，（即去掉文件名后只保留路径）。
    pwd 			  // linux命令，输出当前工作目录的绝对路径（即执行 pwd 时所在的路径）。
    whereis  <filename>	// whereis 命令用于快速查找指定命令的二进制文件、源代码文件和手册页的位置
    # whereis ls		// 查找ls命令所在的二进制文件位置
    find	<path> -name <> -type <>	// 在path路径下找符合条件的文件/文件夹
    which	<命令名>	// 用于查找命令的可执行文件的路径，只能查找 PATH环境变量中定义的目录
    ```

13. linux命令的参数

    ```
    mkdir -p   // -p 表示递归创建
    cp -rf	   // -r 表示递归	-f 表示强制覆盖
    rm -rf	   // -r 表示递归	-f 表示强制删除
    
    mv -f a.txt b.txt	// -f 表示对文件强制，如果 b.txt 已存在，a.txt 会被直接覆盖掉
    mv a b 			// a,b为目录，这样会将a移动到b里面作为b的子目录
    mv -T a b		// -T 表示对目录强制，把目录 a 移动并替换掉目录 b(b必须为空或者不存在)，而不会合并。相当于将a换为b的名
    ```

14. 使用 `tree` 命令可以直观显示文件夹的结构。

    ```
    tree /path/to/directory
    
    // ubuntu下安装tree命令
    sudo apt-get install tree
    ```

15. linux默认权限：**`umask` 是 Unix/Linux 系统上的用户掩码**，它决定了新建文件或文件夹的默认权限。

    在默认情况下：

    1. 文件夹的默认权限基准是 `777`。
    2. 文件的默认权限基准是 `666`。

    3. **实际权限 = 基准权限 & ~`umask`。**

    ```
    输入：umask
    输出：022
    
    // 所以实际上：
    - 文件夹权限：`777 & ~022 = 755`。
    - 文件权限：`666 & ~022 = 644`。
    ```

16. 查看linux系统架构

    ```
    uname -m 	// 会返回系统架构类型
    uname -a 	// 返回内核版本、主机名和架构
    
    lscpu		// 显示系统的CPU架构、核心数量、架构类型（x86_64、ARM等）、CPU型号等详细信息	
    ```

17. rpm包命令

    1. 安装命令

       ```
       rpm -i  // 安装全新包，仅用于安装尚未安装的软件包，如果系统中已存在同名软件包（即使版本不同），会报错（如“包已安装”）
       
       rpm -U // 升级已安装的软件包，或安装新软件包。
       ```

    2. 卸载命令

       ```
       rpm -e <包名>
       rpm -e chenxinsd
       ```

18. deb包命令

    1. 安装/升级

       ```
       dpkg -i 
       ```

    2. 卸载

       ```
       dpkg -P
       ```

    3. ubuntu下查看本地的包

       ```
       dpkg -l | grep <包名>
       ```

    4. `dpkg -b` 命令用于 **构建** 一个 Debian 包（`.deb` 文件），它将指定的目录或源文件打包为一个 `.deb` 文件。

       ```
       dpkg -b <目录> [<输出文件>]
       ```

    5. 解安装包

       ```
       dpkg-deb -x chenxinsd_7.0.0.8-24121304_c1.2110.2.1111_amd64.deb a
       ```

19. ubuntu下卸载使用包管理模式安装的包

    ```
    sudo apt remove --purge <包名>
    sudo apt autoremove
    	#remove：卸载软件包。
    	#--purge：同时删除与软件包相关的配置文件。
    	#autoremove：清理未使用的依赖项。
    ```

    

#### 1.3 安装包

1. rpm和deb包：RPM 和 DEB 是 Linux 系统上两种常见的软件包格式，它们主要与发行版和包管理工具相关，而不是直接与 CPU 架构挂钩。

   1. **RPM (Red Hat Package Manager)**:
      - 由 Red Hat 开发，广泛用于基于 Red Hat 的发行版，如 CentOS、Fedora、Rocky Linux 等。
      - 使用 `rpm` 命令管理，配合 `yum` 或 `dnf` 进行依赖解析和安装。
      - 文件扩展名为 `.rpm`。
   2. **DEB (Debian Package)**：
      - 由 Debian 项目开发，用于基于 Debian 的发行版，如 Ubuntu、Debian、Linux Mint 等。
      - 使用 `dpkg` 命令管理，配合 `apt` 或 `apt-get` 进行依赖解析和安装。
      - 文件扩展名为 `.deb`。
   3. **RPM 和 DEB 本身并不决定架构，它们只是“容器”(包管理工具)**，实际的二进制文件决定了架构（如 x86_64、ARM64）。甚至同一个架构可以同时打包成 RPM 和 DEB 两种格式，只是需要根据发行版的需求调整打包配置。
   4. 所以**最终还是要看系统支持哪种包管理工具**，比如打出来的deb包就只能支持deb的系统中(ubuntu等)才能安装。

2. `dpkg` 在使用 `-i` 选项**安装**软件包时，默认情况下会根据软件包的安装路径将文件安装到指定的目录（比如 `/opt/apps/`），如果软件包在 `.deb` 文件中指定了安装路径为 `/opt/apps/`，那么 `dpkg` 就会将文件放到该目录下。**卸载**时，**如果 `/opt/apps/` 目录下有该软件包的文件，`dpkg` 会删除这些文件**，但是如果该目录本身不为空（例如，可能有其他文件或目录），`dpkg` **不会删除非空的目录**。

   ```
   如何创建 .deb文件：
   1. 准备软件源文件：准备需要打包的源代码或二进制文件。
   2. 创建打包目录结构：通常使用 debian/ 目录来组织打包所需的文件。
   3. 编写控制文件：debian/control 文件用于描述软件包的元数据。
   4. 编写安装脚本：例如 postinst、prerm 等，用于指定安装和卸载过程中要执行的操作。
   5. 生成 .deb 包：使用 dpkg-deb 或 debuild 等命令将上述文件打包成 .deb 文件。
   ```

3. 在 `dpkg` 的软件包管理过程中，有几个关键的脚本文件，它们用于在包的安装、升级、卸载等过程中执行一些自定义操作。这些脚本是**控制脚本（control scripts）**的一部分，通常包含在 `.deb` 包中，并在相应的操作时执行。

   1. `preinst`（安装前脚本）
   2. `postinst`（安装后脚本）
   3. `prerm`（卸载前脚本）
   4. `postrm`（卸载后脚本）

   ```
   /var/lib/dpkg/info/chenxinsd.postrm		// 安装后的控制脚本放在在该目录下
   ```

4. **deb包**使用`dpkg`命令和`apt`工具来管理

   1. `dpkg`命令是 低级工具，直接操作 `deb` 包，类似 `rpm`，适合**离线安装**或手动管理。
   2. `apt / apt-get`通过软件源自动下载和安装包，类似 `yum`，自动解析依赖，适合**在线安装**

5. `apt`和`apt-get`的区别和联系：

   1. `apt` 和 `apt-get` 都是用于管理 Debian 系发行版（如 Ubuntu）软件包**(deb包)**的工具
   2. `apt-get`是传统的 Debian 包管理工具，存在时间更长，功能更全面
   3. `apt`整合了 `apt-get` 和 `apt-cache` 的常用功能，简化操作

6. **rpm包**使用`rpm`命令和`yum`工具来管理

   1. `rpm` 不自动解决依赖问题，安装时可能因缺少依赖而失败，常用于**离线安装**或管理单个 `RPM` 包
   2. `yum`（Yellowdog Updater Modified）是基于 `RPM` 的高级包管理工具，自动处理依赖关系
   3. `yum`可以从配置的软件源（repository）下载包，自动解决依赖，适合**在线安装**

7. deb包的四个脚本作用和执行顺序：

   1. preinst（预安装脚本）：在**包解压和安装文件之前**执行，用于准备安装环境或检查前提条件或**备份**

      > **注意**：此时包的文件尚未解压，因此脚本不能依赖包内的文件。

   2. 解压包并安装文件

   3. postinst（后安装脚本）：**在包解压并安装文件之后**执行，用于完成配置或初始化。

      > **注意**：此时包的文件已经可用，可以依赖它们进行操作，**适合执行安装脚本(完成配置、启动进程等)**

   4. prerm（预移除脚本）：在**包的文件被移除之前**执行，用于准备移除过程

      > **注意**：文件仍然存在，可以访问包的内容，**适合执行卸载脚本(停止进程、删除遗留文件等)**

   5. 包文件被移除，此时deb和rpm管理器会将管理的文件进行删除操作

   6. postrm（后移除脚本）：：在**包的文件被移除之后**执行，用于清理或完成移除

8. 当一个deb包从版本A升级到版本B，对应脚本执行顺序如下：

   1. A的 prerm：调用旧版本 A 的 prerm，参数为 upgrade 和新版本号，准备升级。

      > 此时在prerm中调用A的卸载脚本

   2. B的 preinst：调用新版本 B 的 preinst，参数为 upgrade 和旧版本号，检查升级条件。

      > 备份A的相关信息

   3. 解压新版本文件：dpkg 解压 B 的文件，覆盖 A 的文件。

   4. A的 postrm：调用旧版本 A 的 postrm，参数为 upgrade 和新版本号，清理旧版本

   5. B的 postinst：调用新版本 B 的 postinst，参数为 configure 和旧版本号，完成配置。

      > 执行B的安装脚本

9. `.deb`安装包脚本的存放位置：`/var/lib/dpkg/info/`

10. **关于项目deb和rpm包的升级脚本执行顺序**：

    rpm和deb包对于升级的执行顺序不同：

    - **deb**：prerm（旧包）→ preinst（新包）→ 安装新包 → postinst（新包）。postrm 通常不执行，除非显式移除旧包。
    - **rpm**：%pre（新包）→ 安装新包 → %post（新包）→ %preun（旧包）→ %postun（旧包）。

11. deb包和rpm包打包区别：

    1. 相同点：都会用到pre post的四个脚本，只是deb显示指定、rpm会通过创建`/SPECS/jingyun.spec`文件里面指定
    2. rpm需要在spec文件中的`%file`字段显式的指定包管理文件、deb则自动根据打包目标路径下所有文件进行管理

#### 1.3 补充

1. DBus? 内核通信，获取界面锁定信息

   ```
   qdbus org.cdos.ScreenSaver /org/cdos/ScreenSaver org.cdos.ScreenSaver.GetActive
   
   // 第一个 org.cdos.ScreenSaver 是D-Bus 服务的名称，这个服务负责屏幕保护和锁定功能
   // 第二个/org/cdos/ScreenSaver 是对象路径，指向与锁屏功能相关的 D-Bus 对象。
   // 第三个org.cdos.ScreenSaver 是接口名称，定义了锁屏相关的函数和信号
   // "ActiveChanged"：是信号名称，此处即调用接口的函数和信号。
   
   #在linux下执行下面命令
   qdbus	// 获取qdbus提供的服务列表
   qdbus org.cdos.ScreenSaver	// 获取该服务下面的对象路径
   qdbus org.cdos.ScreenSaver /org/cdos/ScreenSaver	// 该对象提供的接口函数/信号
   qdbus org.cdos.ScreenSaver /org/cdos/ScreenSaver org.cdos.ScreenSaver.GetActive "参数"
   // 调用该函数
   
   qdbus-monitor --session "type='signal',interface='org.cdos.ScreenSaver',member='ActiveChanged'"	// 监听信号
   qdbus-send --session --dest=org.cdos.ScreenSaver --type=method_call --print-reply /org/cdos/ScreenSaver org.cdos.ScreenSaver.GetActive	// 另一种调用函数的方式
   ```

2. 动态库

   1. `LD_LIBRARY_PATH=$PWD` 这个命令与 Linux 系统中的动态链接库加载机制相关，它的作用是设置一个环境变量，指示系统在查找共享库时首先查找当前目录（由 `$PWD` 表示）

      ```c++
      LD_LIBRARY_PATH=$PWD ./ScreenStatusMonitor
      ```

      1. `LD_LIBRARY_PATH` 是一个环境变量，用于指定动态链接库的搜索路径。当你运行一个可执行程序时，程序会依赖一些共享库（例如 `.so` 文件），系统会根据该环境变量指定的路径来查找这些库。

      2. **设置环境变量**: `LD_LIBRARY_PATH=$PWD` 使得当前目录（`$PWD`）被加入到 `LD_LIBRARY_PATH` 中。

         > 这意味着系统会在当前目录中查找动态链接库，而不仅仅是默认的标准路径（如 `/lib` 或 `/usr/lib`）。
         >
         > LD是链接的意思。连接器(ld)

      3. 后面加上可执行程序，则表示可执行程序中所需要的动态库优先在设置的动态库搜索路径下查找。

   2. `ldd`命令：查看程序所依赖的动态库

      ```
      ldd    <file>
      // 显示文件的共享库依赖关系（即列出动态库）,用于检查程序或库依赖了哪些共享库
      ldd   -r  <file>
      // 显示未解析的符号及其依赖问题，列出缺失的符号或库,用于查找库依赖问题和缺失的符号
      ```

3. 在 Linux 中，`/proc` 是一个虚拟文件系统，它包含了系统和进程的详细信息。

     <img src="source/images/JobNotes/image-20250114095351601.png" alt="image-20250114095351601" style="zoom:80%;" />

     1. 每个运行中的进程都有一个与其进程 ID（**PID**）对应的目录 `/proc/<PID>`，其中包含了与该进程相关的所有信息。

     2. 查看进程的命令行及参数， 该文件(cmdline)包含了进程的完整命令行及其参数(**即上图CMD一列**)，以空字符（\0）分隔。

        ```
        cat /proc/<PID>/cmdline
        ```

     3. 进程目录下存放的文件

        ```
        /proc/%d/exe   //这个文件是一个符号链接（symlink），它指向进程 %d 所执行的二进制可执行文件。
        /proc/%d/status	//这个文件包含了进程 %d 的多种状态信息，包括内存使用、进程状态、PID 等信息
        /proc/%d/cmdline  //这个文件包含了进程 %d 启动时的命令行参数，即启动该进程时执行的命令以及所有传递给它的参数。
        ```

4. `tar`（Tape Archive）是一个用于将多个文件或目录打包成一个单独文件的工具，通常**用于存储和备份**。`tar` 生成的文件叫做 **tar包**，扩展名一般为 `.tar`。该工具本身不会对文件内容进行压缩，但可以与压缩工具（如 `gzip`、`bzip2` 等）一起使用，从而创建压缩的 `.tar.gz`、`.tar.bz2` 或 `.tar.xz` 等文件。

     使用tar包存放源码的原因：

       1. 代表项目已经封存了，不会再进行改动。便于版本控制。每次解压得到的文件和目录结构完全相同
       2. 时间等文件信息不会随着操作而更改
       3. 不会因为一些操作产生对应的缓存

5. `nautilus、caja、peony`是文件管理器，右键库是文件管理器的扩展库文件(`.so` 表示共享对象文件，即动态链接库)，比如下方是存放地址。这是 `Nautilus` 文件管理器的右键菜单扩展的一种实现方式，通常通过 C 或其他语言编写并编译为共享库，放在 `Nautilus` 的扩展目录中。

     ```
     /usr/lib/x86_64-linux-gnu/nautilus/extensions-3.0/libnautilus-gjyn.so
     ```
     
6. 关于linux系统磁盘分区：

   1. linux分区布局和原理：

      ```
      /dev/sda (40GB, 物理磁盘)
      ├── sda1 (主分区, 可能用于 /boot, 1GB)
      ├── sda2 (扩展分区, 初始 30GB, 包含逻辑分区)
      │   └── sda5 (逻辑分区, 初始 30GB, 系统分区, 挂载为 /)
      └── 未分配空间 (10GB, 物理扩容后新增)
      
      如果有多个物理磁盘
      /dev/sdb
      /dev/sdc  
      ```
      
         - 主分区：直接在磁盘上创建的分区，数量有限（传统 MBR 分区表最多支持 4 个主分区）。
      - 扩展分区：一种特殊的主分区，用于容纳多个逻辑分区。**扩展分区本身不存储数据**，而是作为一个容器，包含逻辑分区。MBR 分区表中最多只能有一个扩展分区（**占用一个主分区槽**）。
      - 逻辑分区；创建在扩展分区内的分区，用于实际存储数据（例如文件系统）。逻辑分区通常从 sda5 开始编号（因为 sda1 到 sda4 为主分区或扩展分区）。
      
   2. MBR（Master Boot Record分区表结构：

      - 磁盘最多支持 4 个主分区，或者 3 个主分区 + 1 个扩展分区。
      - 扩展分区是一个逻辑容器，**逻辑分区的边界必须在扩展分区范围内**。
      - 分区信息存储在磁盘的 MBR 区域，包括每个分区的起始扇区和大小。

      所以要想扩展磁盘容量，先把扩展分区扩充到最大，然后再将逻辑分区扩充。因为逻辑分区（sda5）是定义在扩展分区（sda2）内的，其起始和结束扇区必须在扩展分区的范围内。

7. 关于VMware的虚拟网络编辑器：

      1. 默认为三个网卡
         - VMnet0： 默认桥接模式，即虚拟机和宿主机在同一个子网下，虚拟机加入工作单位局域网
         - VMnet1：默认仅主机模式，即宿主机与虚拟机组成局域网，其他电脑不能访问该网络
         - VMnet8：默认为NAT模式，虚拟机通过宿主机访问外部网络（此时**宿主机起到路由器作用**），但外部网络不能访问虚拟机
      2. 当存在这几个虚拟网卡时，本地主机的网络连接里面就会显示这几个虚拟网卡
      3. 点击虚拟机(镜像)设置，可以设置这台虚拟机的网络连接模式，选择哪个模式就会通过上述的那个网卡进行联网

8. 关于VMware开启虚拟机失败的问题：

      1. 提示无权限、未能将管道连接到虚拟机，系统找不到指定的文件：
         - 切换为管理员模式启动Vmware，尝试启动
         - 会提示缺少某个dll库，此时去[免费下载缺失的 DLL 文件 | DLL‑files.com](https://cn.dll-files.com/)去下载所需的dll文件，并移动到`c:windows/system32`下
      2. 虚拟网络编辑器中缺少vmnet01，切换不了桥接模式
         - 卸载Vmware，并使用cclean删除注册表
         - 重新下载安装，使用普通用户打开Vmware，打开虚拟网络编辑器，点击使用管理员权限更改，此时VMware01就会出来
      3. 避免出现问题，**虚拟机要选择挂起，避免关机**。因为如果出现虚拟机未完全关机就关闭VMware可能就会导致上述情况

### 2. VScode

1. `BetterComments`注释插件

   ```vscode
   #自带五种注释：
   
   // ! 红色的高亮注释
   // ? 蓝色的高亮注释
   // * 绿色的高亮注释
   // todo 橙色的高亮注释
   // // 灰色带删除线的注释
   // 普通的注释
   
   
   /**
     // ! 红色的高亮注释
     // ? 蓝色的高亮注释
     // * 绿色的高亮注释
     // todo 橙色的高亮注释
     // // 灰色带删除线的注释
   */
   ```

   ```
   #手动添加注释格式：
    打开VScode的settings.json文件，添加高亮注释或者修改注释颜色
   ```

2. vscode导入头文件爆红的解决方式

   > 删除 `.cache/vscode-cpptools` 清理磁盘空间时导致 VS Code 找不到头文件

   1. 重启 VS Code (`Ctrl + Shift + P -> Reload Window`)
   2. 重新安装 C/C++ 插件
   3. 手动刷新 IntelliSense (`Ctrl+Shift+P -> Rescan Workspace` 、`Reset IntelliSense Database`)
   4. 检查 `c_cpp_properties.json` 是否正确
   5. 删除 `.vscode` 目录并重建

3. VScode快捷键：

   ```
   ctrl+p			#快速打开(Go to File..)
   				输入 @：显示当前文件的大纲（函数、类等），可模糊跳转到符号。等价于(ctrl+shift+O)
   				输入 #：在当前文件内搜索关键字。
   				输入 :：跳转到指定行号。等价于(ctrl+g)
   				输入 >: 输入可执行命令。等价于(ctrl+shift+p)
   Ctrl+Shift+P	#用于打开命令面板，你可以通过它运行任何命令、设置或插件功能
   ctrl+g			#跳转某一行
   
   ctrl+o			#打开文件，可以从磁盘中选择文件
   ctrl+shift+O	#当前文件中查找函数,可以模式匹配
   
   ctrl+f 			#当前文件中查找
   ctrl+shift+f	#全局查找
   
   ctrl+左键	   	   #进入函数 或者 F12
   alt+leftarrow(左箭头)  #修改后的返回函数快捷键
   
   选中代码->`ctrl+[`  / `ctrl+]`  #代码整体缩进或者右移：
   ctrl + ` (esc下面那个) 			#打开终端
   ctrl+,						  #打开setting设置
   ctrl+k   ctrl+s				  #设置快捷键
   
   ```

4. **VScode编辑器配置**

     VSCode是微软出的一款轻量级编辑器，它本身**只是一款文本编辑器**而已，并**不是一个集成开发环境(IDE)**，几乎所有功能都是以插件扩展的形式所存在的。因此，我们想用它编程，不只是把vscode下载下来就行，还需要**安装对应编程语言的扩展**以及**相应的编译器**

     1. C++编译器：

        > - Linux用**gcc**
        > - Windows 用**msvc**(visual studio使用)，**minGW**(Windows平台上的GNU工具集合)
        > - ios/mac 用**clang**(也可以在window下使用)

     2. 关于VScode下的`.vscode`文件夹

        1. `c_cpp_properties.json`
           - **作用**: 用于配置 C/C++ 项目的 IntelliSense（自动补全、语法高亮等）功能。
           
           - **主要配置项**:
             - `includePath`: 指定包含头文件的路径。
             
               > 如果头文件爆红，在这里看是不是头文件遗漏了
               >
               > **表示递归包含项目根目录下的所有子目录及其文件
             
             - `defines`: 定义预处理器宏。
             
               > 动态库预处理宏放这里就不会爆红了
             
             - `compilerPath`: 配置编译器的路径，影响 IntelliSense 的工作方式。
             
             - `intelliSenseMode`: 设置 IntelliSense 的模式，例如 `gcc`、`clang` 或 `msvc`。
             
             ```json
             {
                 "configurations": [
                     {
                         "name": "Linux",
                         "includePath": [
                             "${workspaceFolder}/**",
                             "${workspaceFolder}/include/common/**"
                      ],
                         "defines": [
                             "PLUGIN_EXPORT=extern \"C\" __attribute__((visibility(\"default\")))"
                         ],
                         "compilerPath": "/usr/bin/gcc",
                         "cStandard": "c17",
                         "cppStandard": "gnu++14",
                         "intelliSenseMode": "linux-gcc-x64"
                     }
                 ],
                 "version": 4
             }
             ```
        2. `settings.json`
           - **作用**: 存储工作区的特定设置，覆盖全局设置。用于调整 VS Code 编辑器的行为和外观，或者为当前项目定制工作环境。
           
             > clang-format配置在这里面
           
           - **主要配置项**:
             
             - `files.associations`：用于将文件类型与特定语言进行关联。它允许你指定哪些文件扩展名对应于哪些语言。
        3. `launch.json`
           
           - **作用**: 配置调试(**debug**)环境。它定义了如何启动调试会话，指明要运行的程序、调试器的配置、环境变量等。
           - **常见字段**:
             - `program`: 要调试的程序的路径。
             - `args`: 启动程序时传递的命令行参数。
             - `stopAtEntry`: 设置是否在程序入口处暂停。
             - `cwd`: 设置工作目录。
        4. `tasks.json`
           - **作用**: 用于配置构建(**build**)和执行(**run**)任务。你可以通过这个文件配置项目的构建、测试等自动化任务，并通过 VS Code 的任务系统来运行。
           - **常见字段**:
             - `label`: 任务的名称，通常在任务面板中显示。
             - `type`: 任务的类型（例如，shell、process 等）。
             - `command`: 执行的命令。
             - `args`: 命令的参数。
             - `group`: 将任务分类，便于管理。
        
        > `launch.json`和`tasks.json`主要用来在vscode这个编辑器中配置调试（debug）和 构建（build）、执行（run）信息。**如果不需要在vscode中调试和执行，那么不需要撰写这两个文件**。比如使用ssh远程连接linux下的文件，只需要保证在linux下有关于C++的编译器和调试器gdb以及项目构建使用的cmake。

5. **vscode远程开发**，这种模式完美实现了“本地编辑，远程编译调试”，尤其适合跨平台开发或依赖Linux环境的场景

     1. 安装vscode

     2. 安装`Remote - SSH`插件

        ```
        1. Remote-SSH: Connect to Host.. || 选择“Add New SSH Host...”	添加新的ssh远程主机		(格式为user@192.168.1.10)
        2. 选择添加ssh配置到本地(选第一个主机本地用户的.ssh配置)
        3. 再次Remote-SSH: Connect to Host..	|| 选择刚刚添加的主机并输入密码等待连接成功
        4. 会提示选择一个目录作为工作区
        ```

     3. 首次登录后，VS Code会自动弹出一个新的窗口用于远程工作，并且会自动在远程主机上安装VS Code server。

     4. 在vscode安装其他插件，比如C/C++拓展...

6. 关于本地vscode下载拓展到远端

     当你通过 Remote-SSH 连接到 Linux 服务器时，VS Code 会检测当前的工作环境（即远程服务器）。如果你在扩展市场中安装一个扩展，VS Code 可能会判断这个扩展需要与远程环境交互（比如需要访问服务器上的工具或文件系统），于是自动将其安装到远端(并由服务器端的 VS Code Server 管理)，而不是本地。

7. **clang-format代码的自动格式化工具**

     **项目文件操作**：

     1. ubuntu安装clang-format

        ```
        sudo apt install clang-format
        ```

     2. 确保在<u>项目根目录下</u>创建`.clang-format`配置文件，里面实现配置

     **VScode中操作**：

     1. 下载Clang-Format扩展插件

     2. 在VScode的.vscode/settings.json中添加配置

        ```
        "[cpp]": {
            "editor.defaultFormatter": "xaver.clang-format",
            "editor.formatOnSave": true
        }
        ```

     **实际使用**：

     ​	配置完成后VSCode 在保存 C/C++ 文件时自动运行 `clang-format`，会在项目根目录下找clang-format文件

8. 关于**koroFileHeader注释插件**：

     1. 在全局`setting.json`(.vscode下)中添加头注释和函数注释的配置

        ```
        {
            "fileheader.customMade": { // 此为头部注释     
                "Author": "xx", // 文件作者
                "Date": "Do not edit", // 文件创建时间
                "LastEditors": "Do not edit", // 最后编辑文件的人
                "LastEditTime": "Do not edit", // 文件最后编辑时间
                "Description": "", // 文件描述
                "FilePath": "Do not edit", // 文件在项目中的相对路径 自动更新  
            },
            "fileheader.cursorMode": { //此为函数注释
                "description":"", // 功能描述
                "param": "", // 传入参数
                "return": "", // 返回结果
                "author":"wuyue.nan" // 函数作者
            }
        }
          // 不填写对应属性即关闭对应功能
        ```

     2. 使用快捷键：

        ```
        ctrl+win+i   // 头	
        ctrl+atl+t   // 函数
        win + y		 // 注释跳到下一行
        ```

        > 在函数声明那行按快捷键，如果函数声明占多行需要**全选**

9. 关于vscode使用插件有感

     1. 下载安装，通过**wiki**(通常是github链接)使用文档学习如何使用
     2. 很多插件都具有默认配置，可以直接安装并使用；有些插件需要通过 **`settings.json`** 文件来修改插件的行为。
     3. 默认的快捷键如果不生效，可能是操作系统使用了该快捷键，需要在vscode中修改快捷键

10. **vscode做的事情只是替你开了一个终端，然后把这条命令在终端里执行了一下**

      我再次强调，这和c_cpp_properties.json文件里的includePath完全没有任何关系。includePath告诉了vscode你的头文件位置，从而使得vscode能够提供正确的代码提示。但编译器并不理会这一点，编译器只会从你给它的命令行参数里找用户头文件。编译器需要的信息你要在`-l`命令行参数里提供给编译器

11. **Vscode的AI辅助之Github Copilot**：用于AI编程、并且提供AI交互接口

      > 类似的还有**Windsurf**、  **Cursor** 、 **通义灵码**

      **插件使用方法**：

      1. 点击上方图标、登录github账号即可免费使用
      2. 写代码时，提供ai代码，按两下tab 即可自动补全 
      3. 打开交互接口快捷键为 ctrl + alt + i  /  ctrl + shirft + i

      **使用技巧**：

      1. 按两下 tab 代码自动补全：
         1. 正在写代码时
         2. 写完注释(注释中也可以写这段代码要实现的逻辑)，另起一行就会给出这段代码的AI生成
      2. 解释分析代码：
         1. 选中代码-》右键选择copilot-》点击Explain，即会分析这段代码并在右侧窗口弹出分析
         2. 选中代码-》右键选择copilot-》点击Edit Inline chat，弹出一个命令输入框，此时输入"按行解释"、"每行增加注释"等操作

      

### 3. bash脚本

1. 关于bash脚本中局部变量和全局变量

   - `local` 用来声明局部变量的关键字

   ```bash
foo() {
       var=123   # 没有local
   }
   foo
   echo $var    # 输出123，说明var是全局变量
   
   bar() {
       local var=456
   }
   bar
   echo $var    # 什么都不输出，var没有污染全局
   ```
   
2. bash脚本中启动程序的两种方式：

   1. 使用`exec`启动(将要启动的程序放在当前进程环境中，之前的进程自动删除)：exec 是一个 Bash 内置命令，用于**替换当前 shell 进程**，执行指定的命令或程序。执行 exec 后，当前脚本的进程会被新命令完全替换，脚本后续的代码不会执行。

      > 当你运行 exec command，当前 shell 的进程 ID（PID）保持不变，但进程的内容（代码、环境等）被替换为 command。

      ```bash
      exec $BASH_PATH/bin/JYNRJJH2 "$RJJH_PARA" -platform xcb "$typewriting" > /tmp/jy_rjjh.log 2>&1
      ```

   2. 使用 `&` 转为后台启动(即启动一个新进程)：& 是一个 Bash 语法，用于将命令放入**后台运行**，允许当前脚本继续执行后续代码。它不会替换当前进程，而是创建一个新的子进程。

      > 当你运行 command &，Bash 会创建一个子进程来执行 command，子进程的 PID 不同于当前脚本的 PID。

      ```bash
      "$BASH_PATH/bin/IYNRJJH2" "$RJJH_PARA" -platform xcb "$typewriting" > /tmp/jy_rjjh.log 2>&1 &
      
      exit 1 	# 退出当前进程
      ```

3. bash脚本中是否对变量名使用双引号（如 `"$var"` 或 `$var`）

   **基本原则**：

   1. **使用双引号（"$var"）**：在大多数情况下，建议始终使用双引号引用变量，以<u>防止变量未定义或包含空格等特殊字符时导致的语法错误或意外行为</u>。
   2. **不使用双引号（$var）**：在少数特定场景下，变量值明确且<u>不含特殊字符（如空格、换行符等）</u>，可以省略双引号，但仍需谨慎。

   为了防止变量未定义或者包含特殊字符导致解析错误，建议使用双引号的场景:

   1. 在 `if [ ... ]` 或 `test` 命令中，变量必须用双引号引用，以避免变量未定义或为空时的语法错误。

   2. 当变量用于**字符串比较**、拼接或作为命令参数时，使用双引号防止空格或特殊字符导致的解析错误。

   3. 将变量**作为命令参数**传递时，使用双引号确保变量值作为一个整体传递。

      > 不加双引号时，变量值中的空格会导致参数被分割。

   可以不使用双引号的场景(使用双引号也没影响)：

   1. 在变量赋值时，**右侧的变量**不需要双引号，因为 Bash 不会对赋值右侧进行单词拆分。

      ```
      var1="hello"
      var2=$var1           # 正确：var2 赋值为 "hello"
      echo "$var2"         # 输出 "hello"
      ```

   2. **算术运算中**：在 Bash 的算术表达式（`(( ... ))` 或 `$(( ... ))`）中，变量不需要双引号，因为算术上下文会将变量解析为数值。

   3. **case 语句的模式匹配**：变量用于模式匹配时通常不需要双引号，因为 case 会将变量值作为一个整体处理。

4. bash中：test语句两种方式，对应处理的文件路径格式不同

   ```bash
   # 使用test，文件路径不需要加引号
   if test -e $desktop_path/JYNRJJH2.desktop ; then        
   		rm $desktop_path/JYNRJJH2.desktop
   fi
   
   # 不使用test，文件路径需要加引号
   if [ -e $desktop_path/"JYNRJJH2.desktop" ]; then
           rm -rf $desktop_path/"JYNRJJH2.desktop"
   fi
   ```

5. `soft_name=${jyn_install_path%/}`：**功能**：移除变量 jyn_install_path 末尾的斜杠 /（如果存在）。

   `${jyn_install_path%/}` 是一种参数扩展形式，**`%` 表示从字符串末尾移除匹配的模式。**

   **`/` 是要匹配的模式，`%/` 表示移除末尾的单个 `/`（如果有的话）。**

6. `soft_name=${soft_name##*/}`：**功能**：从 `soft_name` 中移除最长匹配的 `*/`（即路径部分），提取最后一个路径组件（文件名或目录名）。

   `${soft_name##*/}` 是一种参数扩展形式，**`##` 表示从字符串开头移除最长匹配的模式。**

   **`*/` 是要匹配的模式，表示任意字符（`*`）后跟一个 `/`。**

7. bash的选择分支语法(`if..else`..、  `case`)

   ```bash
   case "$variable" in
       pattern1)
           # 匹配 pattern1 时执行的代码
           ;;
       pattern2)
           # 匹配 pattern2 时执行的代码
           ;;
       *)
           # 可选，默认分支（未匹配任何模式时）
           ;;
   esac
   ```

8. bash脚本括号之间的区别()  (()) [ ] [[ ]]

   | 括号类型 | 主要用途                | 空格要求 | 变量引用是否需引号 | 特殊功能                       | POSIX 兼容性 |
   | -------- | ----------------------- | -------- | ------------------ | ------------------------------ | ------------ |
   | ( )      | 子shell、命令分组、函数 | 无       | 无需               | 创建独立环境、命令分组         | 是           |
   | (())     | 算术运算、数值比较      | 无       | 无需               | C 风格运算、数值条件测试       | 否           |
   | [ ]      | 条件测试                | 需要     | 需要               | 基础测试（文件、字符串、数值） | 是           |
   | [[ ]]    | 增强条件测试            | 需要     | 不需要             | 模式匹配、正则、逻辑运算       | 否           |

9. 关于`shell`脚本中的`test`命令

   1. 内置的`test`命令，常作用于检查文件、字符串和整数的条件，并返回退出状态码（**0 表示条件为真，1 表示条件为假**）。它通常用于 Shell 脚本中的条件判断。

   2. 有两种表现形式：

      ```bash
      1. 正常表示：
      if test -f /etc/passwd; then
          echo "文件存在且为普通文件"
      fi
      
      2. 简写：
      // 文件
      if [ -f /etc/passwd ]; then
          echo "文件存在且为普通文件"
      fi
      // 字符串
      str="hello"
      if [ -n "$str" ]; then
          echo "字符串非空"
      fi
      // 数字
      num=5
      if [ "$num" -gt 3 ]; then
          echo "num 大于 3"
      fi
      
      3. 常用的简写
      [ -e 文件路径 ]			// -e:如果文件或者目录存在返回true
      if [ "$var" -eq 10 ];	// -eq: 如果是两个参数的比较，option放中间
      if [ -z "$res" ] //-z：检查字符串的长度是否为零。如果是零，则条件成立，返回 true。
      if [ -d $jyn_install_path/bin ]	//-d: 用于检查路径是否存在且是一个目录。
      ```

   | 数值相关 | 含义                                     | 示例                       |
   | -------- | ---------------------------------------- | -------------------------- |
   | `-eq`    | 测试两个整数是否相等。                   | `test "$num1" -eq "$num2"` |
   | `-ne`    | 测试两个整数是否不相等。                 | `test "$num1" -ne "$num2"` |
   | `-gt`    | 测试第一个整数是否大于第二个整数。       | `test "$num1" -gt "$num2"` |
   | `-lt`    | 测试第一个整数是否小于第二个整数。       | `test "$num1" -lt "$num2"` |
   | `-ge`    | 测试第一个整数是否大于或等于第二个整数。 | `test "$num1" -ge "$num2"` |
   | `-le`    | 测试第一个整数是否小于或等于第二个整数。 | `test "$num1" -le "$num2"` |

   | 字符串相关 | 含义                       | 示例                      |
   | ---------- | -------------------------- | ------------------------- |
   | `-z`       | 测试字符串是否为空。       | `test -z "$my_var"`       |
   | `-n`       | 测试字符串是否非空。       | `test -n "$my_var"`       |
   | `=`        | 测试两个字符串是否相等。   | `test "$str1" = "$str2"`  |
   | `!=`       | 测试两个字符串是否不相等。 | `test "$str1" != "$str2"` |

   | 文件相关 | 含义                                               | 示例                         |
   | -------- | -------------------------------------------------- | ---------------------------- |
   | `-e`     | 测试文件是否存在（无论是文件还是目录）。           | `test -e /path/to/file`      |
   | `-f`     | 测试文件是否为普通文件（不是目录或其他特殊文件）。 | `test -f /path/to/file`      |
   | `-d`     | 测试是否为目录。                                   | `test -d /path/to/directory` |
   | `-r`     | 测试文件是否可读。                                 | `test -r /path/to/file`      |
   | `-w`     | 测试文件是否可写。                                 | `test -w /path/to/file`      |
   | `-x`     | 测试文件是否可执行。                               | `test -x /path/to/file`      |
   | `-s`     | 测试文件是否非空（大小大于 0）。                   | `test -s /path/to/file`      |

10. `shell`的`bash`对字符串的操作

   ```bash
   desktop_path=${desktop_path%/*}/桌面	// 删除路径中的最后部分 并替换为“桌面”
   技巧：
   1. % 是 Bash 字符串处理的一种操作符，用来删除从右侧开始匹配模式的部分。
   2. /* 表示“从最后一个斜杠（/）开始的部分”。
   ```

11. bash脚本获取命令输出的方式

   12. 反引号(``)

       ```
       res=`ls -A /opt/apps`
       ```

   13. $() 命令替换

       ```
       res=$(ls -A /opt/apps)
       ```

   > 在 Bash 脚本中，给变量赋值时，**等号两边不能有空格**，它必须紧挨着变量名和变量值。

11. bash脚本

    ```bash
    timeout 5 peony -q 
    		if [ $? -eq 124 ]; then
    			echo "右键库刷新失败，尝试手动重启"
    			pkill peony
    			peony &
    		fi
    ```

    1. `timeout`： 限制命令的执行时间

    2. `$?` 是 Bash 脚本中一个特殊变量，表示上一个命令的退出状态码（exit status）。

       - 0：表示命令成功执行。

       - 非 0：表示命令失败或被中断，具体值取决于命令的定义。

         > 比如 `timeout` 本身的退出状态码是 124， 表示命令被 timeout 主动杀掉+

12. `xargs`命令:`xargs` 的主要作用是将标准输入中的数据（如从 `find`、`echo` 或其他命令生成的数据）转换为命令行参数，并执行指定的命令。

    ```
    xargs [OPTION] [COMMAND]
    
    [OPTION]:
    1. -i / --replace:使用占位符 {} 来指定命令中的位置，在每次执行时将标准输入的每一行替换到 {} 的位置。
    echo "file1.txt file2.txt" | xargs -i mv {} /path/to/destination/
    2. -n:指定每次调用命令时传递的参数个数
    echo "file1.txt file2.txt file3.txt file4.txt" | xargs -n 2 rm	//只传递前两个
    3. -p:在执行每个命令之前，xargs 会提示用户确认。用户需要输入 y 或 n 来决定是否执行命令。
    ```

13. while循环

    ```bash
    while true; do
        if pgrep -x "JYNRJJH2" > /dev/null; then
            break
        fi
    done
    ```

14. 删除某个文件夹下所有带"="的文件

    ```bash
    find "$jyn_install_path/bin" -type f -name "*=*" -exec rm -f {} \;
    ```

    1. `-exec`: 是 `find` 的一个选项，允许对查找结果（匹配的文件）执行指定的命令

       > 它比管道（|）更直接，因为管道需要额外的工具（如 xargs），而 -exec 是 find 的内置功能，处理文件名中的特殊字符（如空格、换行）更可靠。

    2. `{}` : 是 `find` 的占位符，代表每个匹配文件的完整路径

    3. `\;` : 是 `-exec` 的结束标志，之所以用 `\` 转义分号，是因为分号（`;`）在 Bash 中是命令分隔符（表示一条命令结束），而`-exec` 后面可能跟其他 `find` 选项（如 `-type d`），

15. 管道(`|`) 和 `-exec`

    1. `|`（管道）：是 `Shell` 的通用功能，依赖输出输入流，适用于几乎所有命令，用于**将一个命令的输出（通常是标准输出 stdout）传递给另一个命令作为输入（标准输入 stdin）**，数据以文本形式在命令之间传递

    2. `-exec` ：是 `find` 命令的**内置选项**，用于对 `find` 找到的**每个**文件路径直接执行指定的命令

       > `-exec ... \;`：逐个文件执行（如多次调用 `rm -f`），效率较低。
       >
       > `-exec ... +`：批量执行，效率接近管道加 `xargs`。

16. 关于`xargs`：`xargs` 是一个非常强大且常用的 Unix/Linux 命令行工具，它的主要功能是**将标准输入（通常来自管道）转换为命令行参数，然后传递给指定的命令执行**。

   > xargs 是 "execute arguments"（执行参数）的缩写

   **为什么要使用`xargs`：**

   有些命令（比如 rm、ls）需要直接接收文件路径作为参数，而不是从标准输入读取数据。

   ```bash
   echo "file1.txt file2.txt" | rm    	#这会失败，因为 rm 不处理管道输入
   echo "file1.txt file2.txt" | xargs rm   # 成功，会将 "file1.txt file2.txt" 转为 rm file1.txt file2.txt
   ```

   **基本用法：**

   ```bash
   xargs [选项] [命令]
   
   # 输入：从标准输入读取数据（默认以空格、换行符分割）。
   # 命令：将输入转为参数，附加到指定的命令后执行。
   # 默认行为：如果不指定命令，默认是 /bin/echo。
   ```

   1. 选项：

      **-n 数字**：每次传递的最大参数个数

      ```bash
      echo "a b c d" | xargs -n 2 echo	# 每次将两个参数传递给echo
      ```

      **-I 占位符**：指定占位符(不只局限于`{}`，任何符号都可以)，替换输入

      ```bash
      echo "file1.txt file2.txt" | xargs -I {} echo "处理: {}"	
      # -I {} 定义 {} 作为占位符，每行输入替换 {}，逐个执行。
      ```

   2. 

11. 关于占位符 `{}`：大多数命令（如 ls、grep、rm）不会自动解析 `{}` 为占位符，除非配合 `find` 或 `xargs` 使用。

    1. `{}` 在 find -exec 中：是专属占位符，由 `find` 处理。
    2. `{}` 在管道中：默认无特殊含义，除非用 `xargs -I {}` 或类似工具明确定义。

12. bash脚本语法

    ```
    exec $BASH_PATH/bin/JYNRJJH2 > /tmp/jy_rjjh.log 2>&1
    ```

    1. `>`是输出重定向符号，将**标准输出**重定向到目的文件中

    2. `2>&1`是一种文件描述符重定向的写法，用于将**标准错误**（stderr）重定向到**标准输出**（stdout）的位置

       > 2 表示标准错误（stderr）。  \>&1 表示将标准错误重定向到标准输出的目标

    **背景**：

    在 Linux/Unix 系统中，每个进程默认有三个文件描述符：

    - **0 - 标准输入（stdin）**：通常是键盘输入。
    - **1 - 标准输出（stdout）**：程序的正常输出，通常显示在终端。
    - **2 - 标准错误（stderr）**：程序的错误信息，也通常显示在终端，但独立于 stdout。

    **2>&1 的含义**

    - **2**：表示标准错误（stderr）。
    - **>**：表示重定向。
    - **&1**：表示将输出重定向到文件描述符 1（标准输出）的位置。
    - **整体含义**：将标准错误（stderr）的输出重定向到标准输出（stdout）当前指向的地方。

13. Bash脚本中的 `$@`表示**脚本或函数接收到的所有位置参数（positional parameters）**，并且以**单独引用的方式**传递这些参数。

    1. **位置参数**是指脚本或函数在调用时传入的参数，通常用 $1、$2、$3 等表示（分别对应第一个、第二个、第三个参数）。
    2. `$@` 表示所有位置参数的集合，从 $1 开始，依次包含 $2、$3 等。
    3. **关键特性**：`$@` 会将每个参数视为单独的字符串，保留参数之间的分隔（即使参数中包含空格）。

### 4. 操作系统

1. 文件的 **MD5** 值是通过对文件内容进行哈希计算得到的，MD5（Message Digest Algorithm 5）是一种广泛使用的哈希算法，它将任意长度的输入（如文件内容）转化为一个固定长度的哈希值（128位，通常表示为32个字符的十六进制字符串）。

2. **测试版**（Beta）：更新稍快，可能有新的功能，使用上不一定很稳定，适合喜欢尝鲜的使用者使用。

3. GNU、GCC、G++

     1. **GNU** 是一个自由软件项目，旨在提供一个类似 Unix 的自由操作系统，包含了许多工具和库

        **GNU 项目内容**：GNU 项目包括许多重要的组件，如：

        - **GNU 操作系统的核心部分**（尽管 Linux 核心也非常流行，许多人将 GNU 与 Linux 操作系统一起使用，称为 GNU/Linux）。
        - **GNU 工具链**（如编译器、调试器、构建工具等）。
        - **GNU 编程语言库**，例如 C 库（glibc）等。

     2. **GCC**（GNU Compiler Collection）：是 GNU 项目的一部分，是一个开源的编译器集合，用于将源代码（如 C、C++、Fortran、Ada 等）编译为机器码。

     3. **G++**：是 **GCC** 中专门用于编译 **C++** 源代码的前端编译器。G++ 实际上是 GCC 的一部分，用于支持 C++ 语言的编译

4. GNU/Linux:

   1. GNU 是什么？
      - GNU 是 "GNU's Not Unix" 的递归缩写。
      - 创建一个完全自由的类 Unix 操作系统，替代当时的专有 Unix 系统
      - GNU 项目开发了一系列自由软件工具，包括**编译器**: GCC 、**核心工具**: Bash 、**其他**: Make、GDB
   2. Linux 是什么？
      - Linux 是一个操作系统内核。
      - 旨在打造一个类似 Minix 的自由内核，后来发展为功能强大的操作系统内核。
      - Linux 本身**不包括**用户空间工具（如 shell、编译器等），需要配合其他软件才能成为完整的操作系统。
   3. 两者关系：
      - GNU 项目有了大量用户空间工具，但缺少内核（GNU 的内核 Hurd 开发缓慢，至今未广泛使用），Linux 提供了内核，但没有用户空间工具。两者结合后，形成了一个完整的操作系统：**GNU/Linux**
      - 常见的“Linux 发行版”（如 Ubuntu、Debian、Fedora）本质上是 GNU 工具链 + Linux 内核的组合，再加上发行版特定的软件包管理系统和其他工具。例如，运行 ls（来自 GNU Coreutils）或用 GCC 编译程序时，底层依赖 Linux 内核调度资源

5. 什么是POSIX系统：POSIX 系统是指完全或部分实现了 POSIX 标准的操作系统。简单来说，它是**能够支持 POSIX 定义的接口和行为的系统**，程序员可以在这些系统上使用 POSIX API 编写程序，且代码具有较高的可移植性

   **起源**：

   - POSIX 诞生于 1980 年代，当时不同的 Unix 系统（如 System V 和 BSD）有各自的 API，导致软件在不同系统间的移植性很差。
   - 为了解决这个问题，IEEE 提出了 POSIX 标准，旨在定义一套统一的接口规范，使程序员可以在遵循 POSIX 的系统上编写可移植的代码

   **核心内容**：

   - POSIX 标准主要定义了操作系统提供的 **系统调用**、**库函数** 和 **命令行工具** 的行为。
   - 包括文件操作（如 `open、read`）、进程管理（如 `fork、exec`）、线程（POSIX Threads，如 `pthread_create`）、信号处理（如 `signal`）等。
   - 还包括标准化的 shell（POSIX Shell）和常用工具（如 `ls、grep`）。

6. 堆和栈

   奠基者们发现，使用内存主要分两种形式， 大块的、偶而的分配/使用内存，和小块的、频繁的分配/使用内存。

   1. **堆使用“软”的方式进行管理**。比如，你调用GNU函数`malloc`，就能得到一块内存。当然，这块内存也可能很小，小到一个字节。但如果你真的频繁调用`malloc/free`，分配、释放内存，碎不碎片的先不说，效率一定是不高的。

      > 这里“软”的方式，是指CPU中没有专门组件、模块或指令，对应malloc/free这样的内存管理函数。

   2. **栈走软、硬结合的方式提高性能**。在CPU中，内置一部分集成电路，对外提供两条指令: push , pop，和两个寄存器，rbp, rsp，再进行一些“软”方面的约定，把“分配一个内存变量”，变成一个非常简单的操作。有了push/pop，和 rbp/rsp，CPU又不断在这个基础上进行优化。现在push/pop的执行只要一个周期（但涉及内存操作，完成时间仍需要看内存操作的完成速度，不过，相比普通的内存操作，CPU在这方面仍有优化），而且，程序的局部变量可以通过rbp寄存器进行，也简单了许多，并且方便CPU进行预取等优化。

7. AMD64、ARM64、X86_64等之间是什么关系

   1. X86_64：全称是 **x86-64**，也被称为 **AMD64**（因为 AMD 首先提出了这一扩展），后来被 Intel 采纳
   2. ARM64：全称是 **AArch64**，是 ARM 架构的 64 位版本，由 ARM 公司开发。与 x86_64/AMD64 完全不同，ARM64 使用的是 ARMv8-A 指令集，设计注重<u>低功耗和高效率</u>。常见于移动设备（如智能手机、平板）、嵌入式系统以及近年来的服务器（如 AWS Graviton 处理器）和桌面设备（如 Apple M 系列芯片）。

   > AMD64（即 x86_64）、ARM64 等只是不同的架构，适用于不同的设备，彼此之间通常不能直接通用（除非通过模拟器或翻译层）

8. 什么是架构：

   “架构”在计算机领域通常指的是 **指令集架构（Instruction Set Architecture, ISA）**，它是 CPU（中央处理器）的设计蓝图，定义了处理器能够理解和执行的指令集、寄存器、内存访问方式等。它是硬件和软件之间的桥梁，决定了软件如何与硬件交互。

   > 二进制代码最终由指令集系统转为对硬件操作的指令。指令是cpu连接硬件和软件之间的语言

9. 两种指令集的设计理念：

   **CISC（复杂指令集计算）和 RISC（精简指令集计算）本质上是指令集设计的两种理念或哲学**，而不是具体的架构。**不同的 CPU 架构（如 AMD64、ARM64）在设计时会选择其中一种理念作为基础，并根据具体需求和目标进行实现和优化。**

   | 特性           | CISC (如 x86_64)           | RISC (如 ARM64)              |
   | -------------- | -------------------------- | ---------------------------- |
   | **指令复杂度** | 高，功能复杂               | 低，简单统一                 |
   | **指令长度**   | 可变                       | 固定                         |
   | **执行效率**   | 单指令慢，但指令少         | 单指令快，但指令多           |
   | **硬件设计**   | 复杂，解码开销大           | 简单，适合流水线             |
   | **功耗**       | 较高                       | 较低                         |
   | **典型应用**   | 桌面、服务器（高性能计算） | 移动设备、嵌入式系统、服务器 |

10. 关于架构和系统：架构定义了计算机使用的指令集，是计算机软件和硬件通信的桥梁。**有了架构，才可以在架构上安装操作系统**。操作系统用来管理和分配计算机系统资源的；

    常见的架构：

    | 架构              | 指令集 | 主要应用场景           | 优点               | 缺点           |
    | ----------------- | ------ | ---------------------- | ------------------ | -------------- |
    | X64               | CISC   | PC、服务器             | 兼容性强，生态成熟 | 功耗高，复杂   |
    | ARM64             | RISC   | 移动设备、服务器       | 低功耗，灵活性强   | 桌面生态较新   |
    | SW64(神威)        | RISC   | 超算、国产设备         | 高性能，自主可控   | 生态封闭       |
    | MIPS64            | RISC   | 嵌入式、路由器、游戏机 | 简洁高效           | 市场萎缩       |
    | LoongArch64(龙芯) | RISC   | 国产 PC、服务器        | 自主研发，潜力大   | **生态发展中** |

    常见操作系统：window，linux(基于linux内核的操作系统)，uos(统信软件开发)，麒麟(kilin OS)，中科方德等

11. 操作系统、架构、指令集、内核之间的关系

    1. 操作系统内核

       1. **Windows** 操作系统由 **Microsoft** 开发，使用的是 **Windows NT 内核**。Windows NT 内核设计是为了支持 **多任务操作**、**多用户**、**硬件抽象**、**安全性** 和 **网络功能** 等。

       2. **macOS** 使用的是 **XNU 内核**。macOS和Windows NT都是**混合内核**，结合了微内核的模块化和宏内核的高效执行。

       3. **Linux 内核** 是一个 **单内核（Monolithic Kernel）**，所有核心功能（如进程管理、内存管理、硬件控制等）都集中在内核中。Linux 内核本身并不是一个完整的操作系统，通常它与各种用户空间工具、库和应用程序结合在一起，形成一个完整的操作系统，如 **Ubuntu**、**Debian**、**CentOS** 等。

          | 特性           | **Windows**                         | **macOS**                            | **Linux**                              |
          | -------------- | ----------------------------------- | ------------------------------------ | -------------------------------------- |
          | **内核**       | Windows NT 内核                     | XNU 内核（基于 Mach 和 BSD）         | Linux 内核（基于 monolithic）          |
          | **内核类型**   | 混合内核                            | 混合内核                             | 单内核（Monolithic）                   |
          | **核心组件**   | 设备驱动、文件系统、网络堆栈等      | BSD 用户空间 + Mach 微内核           | 文件系统、网络、硬件管理等（核心模块） |
          | **开源**       | 否                                  | 否                                   | 是（完全开源）                         |
          | **用户界面**   | 图形化界面，Windows Shell（命令行） | 图形化界面，Terminal（命令行）       | 图形化界面（如 GNOME、KDE 等）或 CLI   |
          | **硬件兼容性** | 广泛的硬件支持，特别是 x86/x64 架构 | 主要支持 Apple 自家硬件（Intel、M1） | 支持广泛的硬件平台（x86、x64、ARM 等） |
          | **市场定位**   | 面向个人用户、企业、游戏等          | 面向创意工作者、开发者等             | 面向开发者、服务器、嵌入式设备等       |

    2. 基于内核发行的版本：**麒麟操作系统**和**统信操作系统** 是基于 Linux 内核的国产自主研发的操作系统，类似于 **Ubuntu** 和 **Red Hat**

    3. **Linux** 是操作系统的核心，而 **Ubuntu** 和 **Red Hat** 是两个具体的操作系统发行版，它们在 Linux 内核基础上进行了不同的定制和改进。

    4. 操作系统的 **架构支持** 是指它能在某种**硬件架构**上运行。常见的硬件架构有 **x86**、**x64**、**ARM** 等，而不同的操作系统会有不同的架构支持。

       1. **x86**：指的是基于 **32 位** 架构的处理器和操作系统

       2. **x64**（也叫 x86_64 或 AMD64）：指的是基于 **64 位** 架构的处理器和操作系统。

          > cpu同时处理的最大位数

       3. **ARM** 架构主要用于移动设备、嵌入式系统以及越来越多的服务器和工作站。ARM 是一种低功耗的架构，广泛用于手机、平板、路由器、嵌入式设备以及一些专用的服务器系统。

    5. **硬件架构**和**指令集架构**的关系：

       1. **硬件架构** 指的是计算机硬件的整体设计，包括 CPU（中央处理单元）、内存、I/O 设备等所有硬件组件。硬件架构描述了计算机系统各个组件如何相互连接与交互。

       2. **指令集架构（ISA）**  则是描述 CPU 如何与软件进行交互的规范，也就是说，它定义了 CPU 可以执行的指令集（例如加法、乘法、数据加载等指令）和这些指令的行为。ISA 规定了 CPU 如何执行运算、存储数据、管理程序等。

       3. **硬件架构 是计算机硬件的具体实现，而 指令集架构（ISA） 是软件与硬件之间的接口**，指令集架构定义了 CPU 可以执行的指令，进而影响硬件设计

       4. 硬件架构与指令集架构（如 RISC 和 CISC）之间的关系体现在 CPU 的设计上，具体来说，指令集架构（ISA）规定了硬件需要支持的指令集，而硬件架构则实现了这些指令的执行。

          如**RISC 架构** 更加注重简化硬件设计。**ARM** 和 **RISC-V** 都是基于 RISC 架构的处理器，硬件设计相对简单，通常能够达到较低的功耗和较高的性能。

          **CISC 架构** 的设计理念更为复杂，因为每条指令可能包含多个操作，硬件需要更多的解码器和执行单元来处理这些复杂的指令。**x86** 是典型的 CISC 架构，它拥有庞大的指令集，处理器需要支持各种复杂的指令解析和执行机制。

12. “GC” 通常是 **Garbage Collection（垃圾回收）** 的缩写： 垃圾回收是一种**自动内存管理机制**，由运行时环境（如虚拟机或解释器）负责，旨在识别和回收程序中不再使用的内存（即“垃圾”），从而避免内存泄漏并简化开发者的内存管理负担。

    - **手动内存管理**：在像 C 或 C++ 这样的语言中，开发者需要显式地分配内存（malloc 或 new）和释放内存（free 或 delete）。如果忘记释放，就会导致内存泄漏。
    - **垃圾回收**：在支持 GC 的语言（如 Java、Python、C#、JavaScript）中，开发者无需手动释放内存，垃圾回收器会自动检测并清理无用的对象。
13. 逻辑处理器：CPU 逻辑处理器（Logical Processor）是指**操作系统能够识别和调度的基本计算单元**，它是 CPU 核心（Core）与超线程技术（Hyper-Threading，简称 HT）结合后的产物。逻辑处理器数量通常决定了一个 CPU 在操作系统中表现为多少个“处理器”，直接影响多任务处理和并行计算能力。