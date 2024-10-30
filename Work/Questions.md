

### 一、注意事项

1. 时间转换，修改为98版本的

2. 参数设置为可修改的，开放可修改接口

3. 中文注释多的时候使用 `/**/`

   > 原因：如果使用//可能会导致编码时影响到注释后面的代码

4. 变量名：

   ```
   类成员变量：m_XX
   ```

5. 写代码符合单一原则：即同一个类只负责该对象相关的功能

   > 比如暴力破解类和时间转换类



### 二、项目框架

#### 2.1  数据传输

数据传输方式有protobuffer和Json格式，作为项目中的数据传输类型

##### 2.1.1 protobuffer

1. protobuffer的格式

   ```
   message xxx {
     // 字段规则：required -> 字段只能也必须出现 1 次
     // 字段规则：optional -> 字段可出现 0 次或1次
     // 字段规则：repeated -> 字段可出现任意多次（包括 0）
     // 类型：int32、int64、sint32、sint64、string、32-bit ....
     // 字段编号：0 ~ 536870911（除去 19000 到 19999 之间的数字）
     字段规则 类型 名称 = 字段编号;
   }
   ```

2. 接受时解序列化

   ```c++
   NetProtectC netProtectConfig {};				#NetProtectC是protbuf生成的class
   netProtectConfig.ParseFromString(msgData)
   ```

2. 传递时候的序列化(转字符串传输)

   ```c++
   ReportProtect report;				#ReportProtect是protbuf生成的class
   report.SerializeAsString()
   ```

3. 获取非重复非嵌套字段

   ```
   直接.加类型名()
   ```

4. 获取重复字段

   ```c++
   #遍历
   #获取重复次数：字段名_size()
   #获取第i个：processData.infolist(i)
   HmiToScan::VirusDataProcess processData {};
   if (!processData.ParseFromString(msgData)) {
        return false;
   }
   for (int i = 0; i < processData.infolist_size(); i++) {	 
       const HmiToScan::VirusDataProcess::VirusInfo &info = processData.infolist(i);
       if (info.action() == HmiToScan::VirusDataProcess_ProcessAction_TRUST_ADD) {
       m_pScan->virusScanProcessTrustAdd(info.data());
   ```

5. 获取非重复嵌套字段

   ```c++
   #点就完事
   scanNotify = commonC.notification().scan().status()     
   ```

6. 设置非重复非嵌套字段

   ```c++
   #set_字段名
   Person person;
   person.set_name("Alice");  // 设置名字
   person.set_age(30);        // 设置年龄	 	
   ```

7. 设置重复嵌套字段

   ```c++
   #先定义嵌套字段类型，再给该类型set数据
   ClientIsolationAreaInfo isoMsg{};
   ClientIsolationAreaInfo::InfectInfo* isoInfo = isoMsg.add_infects();
   isoInfo->set_infect_name(info.param.virusname);
   isoInfo->set_file_name(info.param.path);
   isoInfo->set_isolate_time(info.m_time);
   isoInfo->set_md5(info.param.md5);
   ```

8. mutable_

   ```c++
   // 获取指向嵌套 Address 消息的指针，并设置其字段。
   Address* addr = person.mutable_address();  
   addr->set_city("New York");
   addr->set_zip("10001");
   ```

9. add_和mutable\_的区别

   ```c++
   add_address()：向 repeated 字段中 新增一个地址。
   mutable_address(index)：获取并 修改指定索引处的地址。
   address(index)：只读访问指定索引处的地址。
   ```

10. 枚举类型直接通过下标_获取

    ```c++
    HmiToScan::VirusScan_ScanType_THREAT_CLEAN
    ```

11. protobuf的msg使用二进制类型byte和一个type，就可以根据不同的type的message进行解析

    > 详见jy_virus_scan_impl.cpp   158行



##### 2.1.2 Json

1. 使用Json传递数据时，使用`dump()`函数转换为字符串

   ```c++
   //使用json传递数据
   std::string pass_timer = (m_alertTImer.status().lock()) ? "0" : std::to_string(m_alertTImer.timer());
   	nlohmann::json report = {
   		{"actionOp",std::to_string(m_op)},
   		{"protectProject","网络防护"},
   		{"protoType","SSH远程登陆协议"},
   		{"ipAddress",ip},
   		{"alert_time",pass_timer}
   	};
   	sendInfoToUI(report.dump());
   ```

2. 接受json数据

   ```c++
   void ZyHipsDialog::updateNetProDlg(const QByteArray &data){
   QJsonDocument doc = QJsonDocument::fromJson(data.data());
   QJsonObject rootObj = doc.object();
   int actionOP = rootObj.value("actionOp").toString().toInt();
   QString protectProject = rootObj.value("protectProject").toString();
   QString protoType = rootObj.value("protoType").toString();
   QString ipAddress = rootObj.value("ipAddress").toString();
   ```

##### 2.1.3 protobuffer和json的区别





#### 2.2 通信原理

1. IGeneralOperator有什么用，插件plugin**通信原理**？imessage和IPC？搞清楚IPC：上报中控、接收中控。发送UI，接收UI字段+流程

2. 客户端与中控之间的**交互**

   > 中控下发->需要使用onNewNotify根据下发key字段和message的protobuff获取指令
   >
   > > 在插件接受数据那块
   >
   > 中控上发->需要使用asynPort根据上发的key字段和需要封装的message的protobuff发送
   >
   > > 在功能上报

3. C++与UI之间的交互？SendToUI如何实现的



#### 2.3 插件

1. 模块->组件->包(deb)

2. plugins文件夹下面是一些**插件**模块

3. commen/utils文件夹下是一些常用的小插件(小功能，比如时间转换啥的)

4. **更新插件**分为软件更新和病毒库更新

   > calback(反馈)用来通信，与客户局端(前端界面)传递信息
   >
   > > update_callback_interface.h接口
   > >
   > > update_callback.h实现接口功能
   >
   > soft软件更新：
   >
   > > download:下载文件
   > >
   > > update_console_client更新用户界面

4. mingfei代码属于是让功能类继承CGeneralOperator,CGeneralOperator需要有参(IGeneralOperator)构造，然后在接口处初始化时直接让该功能类使用传来的IGeneralOperator进行构造。由于功能类继承于CGeneralOperator，所以可以使用其定义的函数进行与前端交互。

   > 接口类继承于CPluginHelper，该类继承于CPlugin和CGeneralOperator，所以接口类也可以使用插件类的函数和前端交互的函数

   kongbin代码属于是类的聚合，定义一个总类，里面包含功能类和CGeneralOperator(与前端交互类)实现功能互用

5. ZySingketon.h 实现了单例模式的模板类。

   ```
   class CZDFYManage : public CommonUtils::CSingleton<CZDFYManage>   #实现了单例
   ```

6. 插件对象，子对象可以当作父类传入

7. 普通日志输出不出来是因为上报日志的顺序不对，时间和类型的顺序搞反了

   详情日志输出不出来也是因为发送的第一个字段是时间而不是类型！

##### 2.3.1 扫描插件

关于隔离区数据上报与中控的逻辑：版本号初始可能为空，每次收到中控下发指令比较并保存该版本号，如果说版本号不同那么就上报所有隔离区数据

代码逻辑：

```
1. imp:插件，封装为插件形式启动、关闭、接受ui点击/中控 下发的信号key和msg。#里面包含task和flow两个成员对象，task初始化任务列表、flow作为功能核心
2. task_process: 工厂模式，任务类，当imp收到消息，则创建对应key的处理对象进行相应处理
3. flow_controller: 功能核心，task使用imp初始化的flow进行flow的初始话，并调用flow的处理函数进行处理
4. black： 黑名单模块
5. trust: 白名单(信任区)模块
6. isolate: 隔离区模块
7. scan_count: 查杀任务计数模块
8. app_scan: 扫描服务
8. post: 上报服务，(上报client_action)
# 4-9全部作为flow的成员对象进行任务处理调度
```



#### 2.4 日志

1. 导入qlog库，LOG**日志**记录

2. 开启日志打印

   ```
   /opt/apps/chenxinSd/etc/LogConf.ini
   Debuglevel = 0;
   ```

3. 使用LOG打印日志

   ```
   #google::InitGoogleLogging(argv[0]);：初始化 glog 日志系统。
   
   LOG(INFO) << "This is an INFO message.";			#黑色
   LOG(WARNING) << "This is a WARNING message.";		#黄色
   LOG(ERROR) << "This is an ERROR message.";			#红色
   
   #google::ShutdownGoogleLogging();：在程序退出前清理并关闭日志系统。
   ```

   > **`LOG(INFO)`**：输出信息级别日志。**场景**：用于记录**正常的程序执行信息**。它表示程序按预期工作，没有异常情况。常用于打印一些程序流程、状态信息、统计数据等。
   >
   > **`LOG(WARNING)`**：输出警告信息。**场景**：用于记录**非致命的异常情况**或潜在问题。这些问题不会导致程序崩溃，但可能影响性能、用户体验，或者暗示更严重的问题可能即将发生。
   >
   > **`LOG(ERROR)`**：输出错误信息。**场景**：用于记录**严重的错误**，这些错误会导致程序某些功能失败，甚至崩溃。通常表示程序中无法忽略的错误。
   >
   > **`LOG(FATAL)`**：输出致命错误并终止程序。`LOG(FATAL)` 表示**不可恢复的错误**，程序在记录该日志后会**立即终止执行**。它用于捕捉那些使程序无法继续运行的严重错误，通常表示逻辑问题或异常情况无法修复。

   > 使用 **`LOG`** 宏时，不需要手动添加换行符。它会**自动在每条日志末尾添加换行符**，确保每一条日志信息在新行中打印。



#### 2.5 数据库

##### 2.5.1 SQlite

```
sqlite3 <数据库.db>
.tables   #查看所有表
SELECT * FROM TABLE    #类似于sql语句
.quit 
.exit	#退出
```



#### 2.6 编译

1. 详见**CMakelist**，打包so文件，即动态库



#### 2.7 调试

1. 使用"sudo gdb JYNSAFED"

2. 打断点：在目的函数名CJYNetProtectionPluginImpl::onNewNotify

3. 执行：r

4. 逐过程：n        /不能逐步骤，因为逐步骤会进入调用的所有函数

5. p 变量名：打印变量名的值，确认是否正确收到并解析

6. quit 退出

7. 当打印数据类型是原子类型时，比如atmoc_bool

   ```
   (gdb) p m_reportLog
   $11 = {_M_base = {static _S_alignment = 1, _M_i = true}}
   ```

   > 1. `_M_base` 和 `_M_i` 是 `std::atomic` 或其他标准库类型的内部表示。它们通常用于实现原子操作和存储状态。
   > 2. `_S_alignment` 是一个静态成员，表示这个类型在内存中的对齐要求。在这里，对齐为 `1` 字节，意味着该类型的对象在内存中必须以 1 字节对齐，这通常适用于 `bool` 类型。

8. 为什么启动有些服务需要sudo，有些不需要？

   > 需要sudo的话，这个服务一般涉及对文件/的操作等

9. gdb调试报错：

   > 1. 需要使用编译版本，如果直接使用的是deb安装下来的则使用不了gdb
   > 2. 哪部分需要调试，就将哪部分执行文件单独拿出来使用gdb执行，其他部分正常启动执行

   > 补充：后台使用systemctl start jyn*启动的是三个服务，CZ控制，ZDFY主动防御、safed安全三个。一个界面只展示两个，没有展示三个

10. 使用systemctl来启动某些服务

    > 有些可执行文件需要通过 **`systemctl`** 启动是因为它们作为 **服务（Service）** 或 **守护进程（Daemon）** 来运行，这种方式适用于在后台长期运行且需要稳定管理的程序

11. <存疑>从安装包安装的软件，对其代码是不能进行gdb调试的，因为其是release模式而不是debug模式，所以不具备调试条件
12. 修改完JYNSAFED不仅需要把JYNSAFED转出，还需要把lib生成的动态库也转出，当gdb遇到找不到文件时就是找不到编译符号
13. 使用sudo或直接启动可执行文件时，使用ctrl+z关闭执行文件时记得使用ps查看并kill后台进程



#### 2.8 打包

1. 在/opt/apps/etc/plugin.conf 文件下添加插件配置信息，name字段、path字段就是打包的动态库文件、message字段为net_protect_c，收到处理的字段名为NetProtectC

2. EXPORT

   > **`PLUGIN_EXPORT=`**：
   >
   > - 它定义了一个宏 `PLUGIN_EXPORT`，其值为：
   >   `extern "C" __attribute__((visibility("default")))`。
   > - 在代码中，如果用到 `PLUGIN_EXPORT`，就会被替换为这个具体内容。
   >
   > **`extern "C"`**：
   >
   > - 用于 **C++** 代码，告诉编译器使用 **C 的符号命名规则（C linkage）**。
   > - 这是为了使编译后的符号能被 C 编译器或其他语言更容易找到（避免 C++ 名字修饰/mangling）。
   >
   > **`__attribute__((visibility("default")))`**：
   >
   > - 这是 **GCC/Clang 特定的语法**，用于控制符号的可见性。
   >
   > - `visibility("default")`
   >
   >    表示：
   >
   >   - 符号是默认可见的，可以从共享库外部访问。
   >   - 这是导出符号（如插件 API）时常用的设置。


##### 2.8.1 打包测试流程

1. 插件的打包配置

   ```
   1. 修改OutPut/JingyunSd_linux_2/BUILD_ROOT/opt/apps/chenxinsd/etc/plugins/plugins.conf
   2. 修改
   OutPut/JingyunSd_linux_2/py_make_packet/basic/new_packet.py，在plugins里面添加动态库so文件
   ```

2. git

   ```
   git pull		#保证最新、合并代码、解决冲突
   git add .
   git commit -m "xxx"
   git push
   ```

   > 是否commit之后才通过git status可以查看到跟远程仓库之间的更新程度？答：不是
   >
   > 1. fetch操作拉取远端提交记录
   > 2. pull操作包含fetch和merge操作

3. 系统打包环境：192.168.0.40 、ssh进入服务器（系统服务器将会执行打包脚本适配不同系统的环境包）在系统打包环境下 pull仓库，同步代码后进行编译。编译后执行脚本开始打包

   ```
   #1. SSH连接
   ssh work@192.168.0.40		
   zyinfo						#密码
   #2. 更新代码
   cd /home/work/workspace2/708/normal_develop/
   git pull
   #3. 编译代码
   make						
   #4. 在该文件夹下执行打包脚本，脚本将会打包不同的deb，并将包fetch-到公用环境192.168.1.6
   cd /home/work/workspace2/708/normal_develop/Output/bin2.0/py_make_packet  
   /opt/python-3.6.15/bin/python3.6 ./main.py -i NewPacket -n chenxinsd -v V708
   ```

4. 本地通过192.168.1.6(共用的包环境),拉取最新的包进行安装测试

   ```
   public share #账号密码
   cd Linux_Packets/text/common/deb/XXX.deb
   ```

5. 安装、卸载

   ```
   sudo dpkg -i xxx.deb		#安装
   sudo dpkg -P chenxinsd		#卸载
   ```

6. 中控(ip变更、要最新ip)账号密码

   ```
   admin	vsecure2020
   ```



#### 2.9 安装

1. qt版本build报错，重新安装qmake   

   > 实际上应该配环境变量，环境变量多了个'/'导致程序执行不出来qmake --version

   ```
   sudo apt-get install qt5-qmake qtbase5-dev
   ```

2. make编译时报错，qt版本有问题，实际5.6.1，但是实际使用显示大于5.10

   > 实际上是因为问题4，使用sudo命令执行使用的是5.12版本的qt

   ```
   QT_VERSION >= 0x050700    
   // 修改为：QT_VERSION >= 0x051000
   ```

3. 启动前端程序时报错：下载qmake时直接下载的是最新的，他会直接下载5.12对应的库。而我们要的是5.6对应的qmake

   > 实际上还是因为问题4

   ```
   sudo apt remove qt5-default qtbase5-dev
   ```

4. leslie用户下和root用户下执行qmake --version显示的是不同的版本，使用sudo执行cmake ..时，程序里面的命令都是使用的root权限(即默认的qmake版本)。所以会导致这种情况。

   ```
   #解决办法：更换文件权限，使用leslie用户下执行cmake ..即可
   ```

5. 开启流程：

   > 1. 前端界面：/opt/apps/chenxin/bin/JYNRRJJH2   ，必须用sudo启动(正常不应该是使用sudo)
   > 2. 后端中控+防护+主防：systremctl start jyn*     #JYNSAFED防护模块 、JYNGJCZ2中控模块、JYNZDFY主防模块



#### 2.10 项目管理

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

   > git pull 自动merge? 如果有冲突修改完冲突之后再add. commit才算是merge了？

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

6. git add. git commit git pull git push

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

7. git 版本回溯

   ```
   git log 查看提交记录号
   git reset --soft <记录号> #--soft：保留所有内容在暂存区（你可以直接 git commit）。不改动则不需要
   git status 查看状态
   git checkout .   #丢弃工作区的改动，如果还需要这些改动不要执行这句
   ```



#### 2.11 改BUG

1. 确定bug在哪个模块

2. 找到具体功能处，从最相关的地方开始往前/往后找

   > 技巧：1. 根据函数名ctrl+F在本cpp内依次找被调用的地方 2. 根据函数名在全局搜，看在别的cpp处调用

3. 若还找不到，查看该模块的实现架构，具体实现逻辑



### 三、技术问题

1. 停止暴力破解插件时，不仅要停止外层的检查日志循环，也要停止行的检测，所以需要两个停止变量。

2. 线程里面对成员变量的访问/操作要使用原子操作(即使用原子类型)

   ```c++
   std::atomic<bool> 
   std::atomic<bool>
   std::atomic<int>
   ```

3. 开启线程时，把两个线程分开，要`detach`而不要加`join`，`join`会导致第二个线程开启不来

4. `#if...#else...#endif...` 符合条件时**编译**，同`is else`

   ```
   #if condition1
   	int a = 10;
   #else condition2
   	int a = 12;
   #endif
   	std::cout << a << std::endl;
   ```

5. 宏定义判断：`#if defined(__linux__)` 和 `#endif` 是预处理指令，用于条件编译。在C++中，这些指令帮助控制代码的编译过程。具体而言：

   1. **条件编译**：`#if defined(__linux__)` 用于检查宏 `__linux__` 是否被定义。这通常是在 Linux 系统上编译代码时自动定义的宏。
   2. **编译特定部分**：如果在编译时检测到 `__linux__` 已定义，那么 `#if` 和 `#endif` 之间的代码会被编译。反之，如果不在 Linux 上编译（例如在 Windows 或其他操作系统上），那么这段代码将被忽略

6. C++多线程，传递类的非静态函数，方法一使用`lambda`，方法二使用`bind`

   > 原因：传递非静态函数需要指定对象指针

6. 线程使用`bind`绑定`this`指针

   ```
   m_checkConnectStateThread = std::thread(std::bind(&CZDFYHandlerMassage::CheckUiConnectState, this));
   ```

   > `std::bind` 是一个函数适配器，用于创建一个新的可调用对象（如函数、成员函数等），并绑定特定的参数。
   >
   > 在这里，`std::bind(&CZDFYHandlerMassage::CheckUiConnectState, this)` 的作用是创建一个绑定到当前对象 `this` 的 `CheckUiConnectState` 成员函数的可调用对象。
   >
   > `&CZDFYHandlerMassage::CheckUiConnectState` 指向成员函数的指针，`this` 是指向当前类实例的指针。

7. 什么时候需要重新执行cmake和make

   > 当更改了cmakelist中的库依赖、文件路径等需要重新对项目进行cmake
   >
   > 若只修改了代码并没有改变项目架构，那么只需要执行make编译就可

7. Cmakelost添加头文件路径时，要指定到最终的文件夹下，不然还是找不到头文件。(它只会在你指定的那个而文件夹下面找头文件)

8. vscode导入头文件爆红的解决方式

   > 按atrl+shirft+p打开cppconfigure，手动添加头文件路径

9. `ifstream` 和 `ofstream` 对象在它们的作用域结束时，都会自动释放资源，包括关闭文件流。

   > 这是因为 C++ 中的文件流类（如 `ifstream` 和 `ofstream`）遵循 RAII（Resource Acquisition Is Initialization）的原则。RAII 确保当对象超出其作用域时，自动调用其析构函数来释放资源，包括关闭文件流。

10. 使用迭代器遍历时，`first`和`second`是成员变量而不是函数，不需要加括号

    ```c++
    for(auto it = time_map.begin();it != time_map.end(); ){
    		if(now - (*it).first > m_detectionTime){
    			m_failureCounts[ip] -= (*it).second;
    			it = time_map.erase(it);
    		}else{
    			it++;
    		}
    	}
    ```

11. `const static` 成员变量，在类外边初始化时(`static`只能在类外)，需要加上`const`关键字

12. `const`成员变量只能使用初始化列表进行赋值

13. `const map`类型不可以使用`operator[]`进行取值，因为如果不存在会造成修改

14. g++编译命令（不使用cmake）

    > g++ check_log.cpp main.cpp -o checklog  -lpthread     	//链接线程库
    >
    > g++ check_net.cpp main.cpp -o checknet -lpcap			 //链接libcap库

15. 枚举类型，只需要包含该头文件就可以直接使用枚举变量，不需要像类成员或者命名空间那样使用`::`来获取

16. **`QString::fromUtf8(dataType)`**:使用 `fromUtf8` 将 `dataType` 从 `QByteArray` 转换为 `QString`。

17. `QJsonDocument::fromJson(data.data())`：

    > **`data.data()`**：
    >
    > - `data` 是一个 `QByteArray` 对象，包含要解析的 JSON 数据。通过调用 `data()`，你获取一个指向 `data` 内部字节数组的 `const char*` 指针，方便传递给 `fromJson()` 方法。
    >
    > **`QJsonDocument::fromJson()`**：
    >
    > - 这个静态方法用于创建 `QJsonDocument` 对象，并解析传入的 JSON 数据。如果 JSON 数据格式正确，它将返回一个有效的 `QJsonDocument` 对象；

18. 问题：make的时候报`moc_xx.cpp`文件`had not been declared`

    > 原因：复制root_partition_protection_plugin.h时没有修改头文件防护编码，导致有两个头文件包含一样的防重复编码，从而错误

    ```
    #ifndef FEATURE_707_ROOT_PARTITION_SPACE_CHECK_H
    #define FEATURE_707_ROOT_PARTITION_SPACE_CHECK_H
    #endif
    ```

    > 解决思路：

    1. git status 查看仓库文件修改情况
    2. git checkout <文件名> 切换未修改前的文件（丢掉本次改动）
    3. 重新build最里面的cmakelist，发现是该文件的错误，由于只新加了新头文件，所以查看新文件发现错误

19. try catch：当代码中涉及文件的操作必须加上trycatch，避免因为文件相关的问题导致整个程序崩溃

    ```
    throw std::runtime_error("Unable to open files:");  #抛出运行时错误
    ```

20. 使用JSON传递string数据，不能直接解JSON数据的时候`toInt()`,必须先`toString()`再`toInt()`;

21. 远程复制/传输文件

    ```
    scp JYNSAFED libJYVirusScan.so leslie@192.168.3.96:~/
    ```

22. 前置声明类：允许在类或结构体的定义之前告诉编译器某个类型的存在

    ```
    class B; // 前置声明
    class A {
    public:
        void setB(B* b); // 只需要知道 B 的存在
    };
    class B {
    public:
        void doSomething();
    };
    ```

23. protobuf使用枚举字段、使用（取值、赋值）

    ```
    HmiToScan::VirusScan_ScanType_THREAT_CLEAN
    ```

24. protobuf的msg使用二进制类型byte和一个type，就可以根据不同的type的message进行解析

25. 关于QStackedWidget和QTabWidget的区别

    > 1. QStackedWidget不提供直接的页面切换界面（没有标签栏）。QTabWidget提供带标签的界面，每个标签对应一个页面。
    > 2. QStackedWidget必须通过代码手动切换页面 (`setCurrentIndex()` 或 `setCurrentWidget()`)。QTabWidget用户可以通过点击标签页直接切换页面。

28. `exec()`和`show()`的区别

    > 1. **`exec()`**：以 **模态对话框** 的方式显示，阻塞主线程，用户必须关闭对话框后才能继续与主程序交互。
    > 2. **`show()`**：以 **非模态** 方式显示，不阻塞主线程，用户可以同时操作其他窗



# 未完成笔记

#### 1. 项目部分

1. 项目通信部分逻辑是什么

2. C++与UI之间的交互？SendToUI如何实现的

3. 项目的架构

4. 隔离区、信任区页面bug

   > FramelessWindow页面的主框架、主页面。里面包含了点击隔离信任区后的槽函数，展示对应dialog
   >
   > TableOfContents页码单独分离出来实现调页

#### 2. 代码部分

1. fork分离进程  、  daemon守护进程

   > 1. 父子进程从 `fork()` 之后的代码开始**并行运行**
   > 2. daemon用于将当前进程变成 **守护进程**，即后台独立运行且与控制终端断开的进程

   ```c++
   void executeCommand(const std::string &command)
       {
           pid_t pid = fork();
           if (pid < 0) {
               // 错误处理
               LOG(ERROR) << "Fork failed.";
               return;
           } else if (pid > 0) {
               // 主进程：可以直接返回，不需要执行任何操作
               std::cout << "主进程！" << pid << std::endl;
               return;
           } else {
               // 子进程
               becomeDaemon();
               // 分解命令为字符串数组
               const char *argv[] = {"/bin/sh", "-c", command.c_str(), nullptr};
               // 睡眠1秒
               std::this_thread::sleep_for(std::chrono::seconds(1));
               // 执行命令
               if (execve("/bin/sh", (char *const *)argv, environ) == -1) {
                   LOG(INFO) << "Software uninstallation, error: execution error.";
               }
                exit(0);
           }
       }
   ```

   ```c++
   bool becomeDaemon()
       {
           if (chdir("/") == -1) {
               return false;
           }
   
           if (daemon(0, 0) != 0) {
               std::cerr << "Failed to daemonize." << std::endl;
               return false;
           }
   
           std::cout << "Daemon started." << std::endl;
           return true;
       }
   
   ```

   > ```
   > #include <unistd.h>
   > 
   > int daemon(int nochdir, int noclose);
   > ```
   >
   > **`nochdir`**：
   >
   > - 如果为 **0**，将工作目录更改为根目录 (`/`)。
   > - 如果为 **1**，则保持当前工作目录不变。
   >
   > **`noclose`**：
   >
   > - 如果为 **0**，则关闭标准输入 (`stdin`)、标准输出 (`stdout`)、标准错误 (`stderr`)，并将它们重定向到 `/dev/null`。
   > - 如果为 **1**，保持标准输入/输出/错误流不变。

2. md5:MD5（Message-Digest Algorithm 5）是一种广泛使用的加密哈希函数，能够将任意长度的输入数据（通常称为消息）转换为固定长度的输出，具体来说是 128 位（16 字节）长的哈希值。

