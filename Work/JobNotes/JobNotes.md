

## 一、注意事项

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

6. SDK版本：SDK版本是指开发者在开发应用时所使用的SDK的版本号。每个SDK版本都有其特定的功能和API

7. 一些常用单词

   ```
   1. parameter/param		参数
   ```

   

## 二、项目框架

### 2.1 客户端层次

1. 客户端整体框架层次分布

   ```
   1. UI界面层
   2. UI moudle层
   3. 插件层/助手层(safed)：功能的主要逻辑，里面包含了组件和插件，负责与UI和服务端(中控)之间的信息传输、不同插件(后台功能)的实现
   4. ZDFY和GJCZ层：ZDFY主要负责系统防护和数据防护；GJCZ主要是和扫描有关的；两个都是开启系统后就在后台启动。记录的数据通过插件层zdfy和gjcz的中转到达UI和服务端
   ```

2. 组件(`modules/component`)主要是一些共用的功能模块，插件(`plugins`)是单独的可以增加、去掉的模块。比如不同的插件都可以使用组件里的认证、防护日志、任务管理、定时任务、更新等

3. `include`文件夹里面主要包含的是定义的头文件，把复杂的头文件/接口摘出来

4. `common`文件夹里面主要包含的是整个项目封装好的工具类，比如`nlohman`(`json`文件)、`thread`(线程)、`utils`(工具包，各种时间、类型转换)、`epoll`(`io`相关)

5. 组件实现方式与插件类似，都是接口`impl`和实现分离。



### 2.2  数据传输

数据传输方式有`protobuffer`和`Json`格式，作为项目中的数据传输类型

##### 2.2.1 protobuffer

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
   mutable_address(index)：获取并修改指定索引处的地址。
   address(index)：只读访问指定索引处的地址。
   ```

10. 枚举类型直接通过下标_获取

    ```c++
    HmiToScan::VirusScan_ScanType_THREAT_CLEAN
    ```

11. protobuf的msg使用二进制类型byte和一个type，就可以根据不同的type的message进行解析

    > 详见jy_virus_scan_impl.cpp   158行



##### 2.2.2 Json

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

##### 2.2.3 protobuffer和json的区别





### 2.3 通信原理

1. IGeneralOperator有什么用，插件plugin**通信原理**？imessage和IPC？搞清楚IPC：上报中控、接收中控。发送UI，接收UI字段+流程

2. 客户端与中控之间的**交互**

   ```
   中控下发->需要使用onNewNotify根据下发key字段和message的protobuff获取指令
   > 在插件接受数据那块
   中控上发->需要使用asynPort根据上发的key字段和需要封装的message的protobuff发送
   > 在功能上报
   ```
   
3. C++与UI之间的交互？SendToUI如何实现的



### 2.4 插件层/助手

#### 2.4.1 通述

1. 组件(`modules/component`)主要是一些共用的功能模块，插件(`plugins`)是单独的可以增加、去掉的模块。比如不同的插件都可以使用组件里的认证、防护日志、任务管理、定时任务、更新等

2. mingfei代码属于是让功能类继承CGeneralOperator,CGeneralOperator需要有参(IGeneralOperator)构造，然后在接口处初始化时直接让该功能类使用传来的IGeneralOperator进行构造。由于功能类继承于CGeneralOperator，所以可以使用其定义的函数进行与前端交互。

   > 接口类继承于CPluginHelper，该类继承于CPlugin和CGeneralOperator，所以接口类也可以使用插件类的函数和前端交互的函数

   kongbin代码属于是类的聚合，定义一个总类，里面包含功能类和CGeneralOperator(与前端交互类)实现功能互用

3. `ZySingketon.h` 实现了单例模式的模板类。

   ```
   class CZDFYManage : public CommonUtils::CSingleton<CZDFYManage>   #实现了单例
   ```

4. 插件对象，子类可以当作父类传入

5. 组件`component`里面实现了多个组件，通过`general_operator_impl`定义创建不同的`shared`类型的组件指针

6. 组件/插件开启时使用线程

   ```
   JYThread::autoRun(std::bind(&CTaskManagerComponentImpl::taskQueueDistribution, this));
   ```



#### 2.4.2 运转核心/core

##### 1. 组成部分

```
1. 组件管理component_manager:包括创建所有组件接口、启动接口
2. 插件管理plugin_manager:包括加载conf文件配置、启动插件、注册消息订阅
3. 插件配置plugin_config:负责从conf文件中读取插件配置
4. 消息订阅message_center:消息的订阅和分发(组件和插件都使用)
5. 核心接口core_impl:将main函数的功能封装为接口，包括启动组件和插件等功能
6. 帮助类bussiness_helper:查看加载模块以及执行命令行等操作
7. main:safed执行主函数
```

##### 2. 插件管理`plugin_manager`

```
loadPlugin：根据plugin.conf加载插件到map中
enablePlugin：遍历map，依次启动插件的init和start，并注册消息订阅，触发消息时执行publishPlugin
unloadPlugin：根据插件名称卸载插件
disablePlugin: 停止所有插件，启动插件的stop和release
publishPlugin：找到对应插件，执行对应插件的onNewNotify操作
```

##### 3. 消息队列/message_center

用于消息的订阅和发布，提前订阅好(subscribe)，就可以根据key执行发布(publish)；比如任务队列中，从任务队列中获取到任务，将key和value封装为Buddle，通过publish从消息处理队列中(m_handlers)执行绑定过的函数(包括向组件(terminal_detatil)中执行中控下发信息填写命令、向插件(Scan)中执行扫描任务等)。

1. 实现：封装一个任务队列，里面保存的是处理函数。发布时使用线程池多线程执行

```
#MessageHandler 是一个可以接受指向 IBundle 类型的指针 pBundle 的函数的类型别名
using MessageHandler = std::function<void(IBundle *pBundle)>;
#第二个参数是一个处理消息的回调函数，类型为之前定义的 MessageHandler
virtual void subscribe(const std::string &messageType, MessageHandler handler) = 0;

#消息队列，使用时用多线程并行处理
std::unordered_multimap<std::string, MessageHandler> m_handlers;
```

2. 订阅：提前将key和处理函数加入消息队列中

```
#subsrcibe实现：
void CMessageCenter::subscribe(const std::string &messageType, MessageHandler handler)
{
    std::lock_guard<std::mutex> lock(m_mutex);
    m_handlers.insert({messageType, handler});
}

#实际订阅使用时可以用lambda
m_pMessageCenter->subscribe("REPORT_USER_INFO", [this](IBundle *pBundle) {
        registerResetRequest(pBundle);
});
```

3. 发布：需要发送时，根据key，执行key对应的处理函数

```
#m_Pool是线程池，在message_center初始化时就创建好了。把处理函数放到线程池
m_pPool->submit([this, pBundle]() {
        std::string messageType = BundleHelper::getBundleAString(pBundle, JYMessageBundleKey::JYMessageType, "");
        if (messageType.empty()) {
            return;
        }
        std::lock_guard<std::mutex> lock(m_mutex);
        const auto &range = m_handlers.equal_range(messageType);
        for (auto it = range.first; it != range.second; ++it) {
            it->second(pBundle);
        }
        pBundle->release();
    });
```



#### 2.4.3 组件/component

##### 1. 组成部分

1. 组件接口/抽象：

   ```
   1. CGeneralOperatorImp: 初始化创建不同组件的功能对象(非组件接口impl)；所有组件的接口使用组件功能类时都通过该类获取到功能对象
   
   2. CComponentInterface: 
   	组件的总接口CComponentInterface，继承于IComponentInterface(接口类，定义虚函数初始化、开启、停止、释放等);所有的组件impl都继承于这个类。
   	初始化时需要CGeneralOperatorImpl和IMessageCenter
       自己定义模板返回创建组件接口impl对象、sendToUi向UI层发送消息这两个函数
   
   3. CGeneralOperatorAdapter:
   	CGeneralOperatorAdapter继承于IGeneralOperator(运转通信总类)，要实现各类通信包括向UI层、中控发送消息等。
   	初始化函数中接收CGeneralOperatorImpl对象为自己的不同组件对象赋值。重写IGeneralOperator的所有函数供插件使用
   ```

2. 组件和插件之间的交互：

   ```
   #插件启动时pluginInit需要接收IGeneralOperator(实际传输CGeneralOperatorAdapter)来实现使用组件的所有功能
   #coreImpl: init时将组件和插件都初始化，插件使用组件初始化后的CGeneralOperatorAdapter
   #插件继承CPluginHelper，该类继承于IPlugin(插件接口类)和CgeneralOperator(组件所有功能类，该类继承IGeneralOperator，并需要一个IGeneralOperator对象初始化，实际给插件初始化传输的是实际传输CGeneralOperatorAdapter)
   ```

3. 组件成员

   ```
   ctrl_center组件，实现向中控的信息上报
   IPC组件(local_ser)：实现sendToUi，externalConnectState连接状态
   protect_logger组件：实现UI层面的日志上报
   authorize组件：实现各种认证、版本信息、病毒库信息、引擎信息等
   timed_tasks组件：实现定时任务
   task_manager组件：实现任务队列
   upgrade组件：实现软件和病毒库更新。没有在general_Operator_impl中定义。
   terminal_details组件：接收中控下发的填写用户信息、更新用户信息、授权、密码文件等功能。
   	1. 在init初始化时就通过messageCenter订阅，待任务队列到达他们时再pulish
   	2. registerResetRequest，接收到中控消息，任务队列中执行插入任务
   	3. registerResetResponse，收到UI层传来完成信息，任务队列中关闭掉任务
   ```

   

##### 2. 更新组件/upgrade

更新组件分为软件更新和病毒库更新

1. `calback`(反馈)用来通信，与客户局端(前端界面)传递信息

   ```
   update_callback_interface.h接口
   update_callback.h实现接口功能
   ```

2. `soft`软件更新

   ```
   download:下载文件
   update_console_client更新用户界面
   ```

3. `virus`病毒库更新



##### 3. 任务队列/task_manager

所有中控下发的扫描任务都通过`Scan`插件接收，放到(`task_manager`组件)任务队列，若没有正在执行的任务时直接执行，否则等待`task_manager`组件根据队列给`Scan`插件下发任务。

```
1. 涉及component下面的task_manager组件
2. impl定义组件的接口，包括初始化、任务分发(publish)等
3. task_manager组件：
	1. TaskManagerConfigInfo 优先级任务队列配置信息
	2. CTaskManagerConfig 管理配置信息，包括从config文件下加载、map存储、匹配
	3. TaskManagerInfo	任务的信息
	4. CTaskManager	管理任务队列，通过key找任务
4. 下发执行：task_manager_component_impl:任务管理组件的接口，包括start、stop、等，启动时会开启一个线程(obtainDistributionTask)，每秒遍历任务队列，将end字段的任务移除队列，如果没有正在执行的任务，那就从task_manager的队列中取出任务发送(publish)给Scan插件(/通过MessageCenter发送)。
5. 接收入队：Scan插件：接受中控的下发请求信息(插件中的onMessageNotify)，并尝试交付给m_pTaskManager的taskExecutionBegins函数进行任务执行，有过有正在执行的任务，那么清除低优先级任务后添加到队列中，如果没有就直接返回true开启执行
6. 执行扫描：在scan插件中：impl接口类中继承了CPluginHelper(其继承于Cplugin，实现插件的初始化、开启、结束等)，init初始化插件需要传入IGeneralOperator通信父类，而组件的CGeneralOperatorAdapter继承于IGeneralOperator，所以可以传入子类代替父类，从而在scan插件接受到执行扫描任务时(通过onNewNotify)通过scan_task任务分发，通过scan_flow执行不同的scan任务，并检测该任务是否正在运行(退出不执行扫描)、是否成功结束(将任务状态字段改为end)
```



#### 2.4.4 插件/plugin

##### 1. 扫描插件/scan

关于隔离区数据上报与中控的逻辑：版本号初始可能为空，每次收到中控下发指令比较并保存该版本号，如果说版本号不同那么就上报所有隔离区数据

扫描插件逻辑：

```
1. imp:插件，封装为插件形式启动、关闭、接受ui点击/中控 下发的信号key和msg。#里面包含task和flow两个成员对象，task初始化任务列表、flow作为功能核心
2. task_process: 工厂模式，任务类，当imp收到消息，则创建对应key的处理对象进行相应处理
3. flow_controller: 功能核心，task使用imp初始化的flow进行flow的初始话，并调用flow的处理函数进行处理
4. black： 黑名单模块
5. trust: 白名单(信任区)模块
6. isolate: 隔离区模块
7. scan_count: 查杀的状态、任务计数等信息
8. app_scan: 扫描服务，根据扫描文件路径进行扫描
8. post: 上报服务，(上报client_action到中控)
# 4-9全部作为flow的成员对象进行任务处理调度
```



### 2.5 日志

1. 导入qlog库，使用LOG日志记录

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



### 2.6 数据库

##### 2.6.1 SQlite

1. sqlite的常用命令

   ```
   sqlite3 <数据库.db>
   .tables   #查看所有表
   SELECT * FROM TABLE    #类似于sql语句
   .quit 
   .exit	#退出
   ```

2. 项目中sqlite的路径

   ```
   /opt/apps/chenxinsd/cache/xxx.db
   ```

   

### 2.7 编译

1. 详见**CMakelist**，打包so文件，即动态库



### 2.8 调试

1. 使用"sudo gdb JYNSAFED"

2. 打断点：在目的函数名 b CJYNetProtectionPluginImpl::onNewNotify

3. 执行：r

4. 逐过程：n        /不能逐步骤s，因为逐步骤会进入调用的所有函数

5. p 变量名：打印变量名的值，确认是否正确收到并解析

6. bt：查看栈调用情况，如果程序崩溃可以通过栈查看那个地方有问题

7. quit ：退出

8. c/C：可以跳过当前断点要操作的内容，继续执行

9. 使用GDB调试：遇到某个函数执行，但是又不知道是那部分在调用这个函数时，使用`gdb`在这个函数打上断点，执行到这个函数时，执行`bt`查看栈，就可以看到调用函数一层一层的顺序了。

10. 当打印数据类型是原子类型时，比如atmoc_bool

   ```
   (gdb) p m_reportLog
   $11 = {_M_base = {static _S_alignment = 1, _M_i = true}}
   ```

   > 1. `_M_base` 和 `_M_i` 是 `std::atomic` 或其他标准库类型的内部表示。它们通常用于实现原子操作和存储状态。
   > 2. `_S_alignment` 是一个静态成员，表示这个类型在内存中的对齐要求。在这里，对齐为 `1` 字节，意味着该类型的对象在内存中必须以 1 字节对齐，这通常适用于 `bool` 类型。

11. 为什么启动有些服务需要sudo，有些不需要？

    > 需要sudo的话，这个服务一般涉及对文件/的操作等

12. gdb调试报错：

   > 1. 是否编译的依赖的so没有导过去
   > 2. **哪部分需要调试，就将哪部分执行文件单独拿出来使用gdb执行，其他部分正常启动执行**

   > 补充：后台使用systemctl start jyn*启动的是三个服务，CZ控制，ZDFY主动防御、safed安全三个。一个界面只展示两个，没有展示三个

11. 使用systemctl来启动某些服务

    > 有些可执行文件需要通过 **`systemctl`** 启动是因为它们作为 **服务（Service）** 或 **守护进程（Daemon）** 来运行，这种方式适用于在后台长期运行且需要稳定管理的程序

12. 修改完JYNSAFED不仅需要把JYNSAFED转出，还需要把lib生成的动态库也转出，当gdb遇到找不到文件时就是找不到编译符号

13. 使用sudo或直接启动可执行文件时，使用ctrl+z关闭执行文件时记得使用ps查看并kill后台进程



### 2.9 打包

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

3. 系统打包环境：`192.168.0.40` 、ssh进入服务器（系统服务器将会执行打包脚本适配不同系统的环境包）在系统打包环境下 pull仓库，同步代码后进行编译。编译后执行脚本开始打包

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

4. 本地通过`192.168.1.6`(共用的包环境),拉取最新的包进行安装测试

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
   admin	vsecure2020		#中控账号密码
   ```



### 2.10 安装

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

   1. 前端界面：/opt/apps/chenxin/bin/JYNRRJJH2   ，必须用sudo启动(正常不应该是使用sudo)

   2. 后端中控+防护+主防：systremctl start jyn*     #JYNSAFED防护模块 、JYNGJCZ2中控模块、JYNZDFY主防模块

      ```
      #主动启动服务，适用于调试，看日志
      cd /opt/apps/chenxinsd/bin
      ./JYNRJJH2
      sudo ./JYNSAFED
      sudo ./JYNGJCZ show 
      ```

      



### 2.11 项目管理

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

7. git 版本回溯

   ```
   git log 查看提交记录号
   git reset --soft <记录号> #--soft：保留所有内容在暂存区（你可以直接 git commit）。不改动则不需要
   git status 查看状态
   git checkout .   #丢弃工作区的改动，如果还需要这些改动不要执行这句
   ```

8. 关于git clone远程拉仓库

   ```
   使用git clone就可以直接把仓库拉到本地，并同步log，并把当前作为本地git仓库
   使用git clone git@时需要配公钥到github上
   使用git clone http@时不需要配公钥，根据仓库的类型可以允许克隆,但是需要在github上给用户权限才能push
   ```

9. 关于git log展示版本信息

   ```
   (origin/master, origin/HEAD) 说明：
   	远程仓库的 HEAD 当前指向 master 分支。
   	origin/master 是远程仓库 origin 上的 master 分支的最新提交。
   (HEAD -> master) 表示：
   	当前 HEAD 指向本地的 master 分支，意味着你正在 master 分支上工作。
   	当你提交新的更改时，master 分支的指针会随之移动到新的提交，而 HEAD 也会继续跟随指向 master 分支。
   ```

10. git push时如果检测到某次commit有error(比如有文件大于100M)，那么就需要本地版本回溯然后再解决问题，不然历史提交记录永远会保存这次的提交信息，导致后续永远push错误

11. github报错：`ssh: connect to host github.com port 22: Connection refused fatal`

    原因：`dns`被污染，导致解析`github.com`域名解析出来是本地`127.0.0.1`

    解决：在`host`文件中加入域名映射：`140.82.113.4 github.com

### 2.12 改BUG

##### 2.12.1 技巧

1. 确定bug在哪个模块

2. 找到具体功能处，从最相关的地方开始往前/往后找

   > 技巧：1. 根据函数名ctrl+F在本cpp内依次找被调用的地方 2. 根据函数名在全局搜，看在别的cpp处调用

3. 若还找不到，查看该模块的实现架构，具体实现逻辑



##### 2.12.2 具体bug具体分析

1. 隔离区、信任区页面bug

   `FramelessWindow`页面的主框架、主页面。里面包含了点击隔离信任区后的槽函数，展示对应`dialog`
   
   `TableOfContents`页码单独分离出来实现调页

2. 设置界面bug

   **思路**：将ui传来的配置保存下来。如果点击取消，就不会保存这次的配置，那么下次打开就还是之前的

   **cpp逻辑剖析**：

   ```
   1. 槽函数updateXXX：ui界面点击保存后会触发该槽函数，将传来的结构体转为protobuffer，再将该protobuffer传递(send)给IPC进行转发给对应处理模块进行后续处理
   2. fromxxx：负责将ui传来的结构体转换未protobuffer
   3. toxxx：负责将服务端/后台新的配置信息从protobuffer转为对象结构体，并将结构体emit发出供settingDialog接受响应
   4. 信号函数sigXXX：将新的配置信息发送，供setting/trustAndIso/界面接收
   5. send和recive都是通过类似于插件的模型moudle进行对其他模块的数据发送和接受(通过key字段和TerminalConfigSeesion的protobuffer字段)。
   #接收时会根据TerminalConfigSeesion的type字段区分，再根据info的key字段区分不同的模块的要emit信息，再调用不同的emit
   ```

   **解决**：将每次下发或者保存的配置保存下来，在界面打开`settingDialog`的时候统一`emit`一遍信号更改设置界面

3. 关于扫描多任务下发bug

   **思路**：查看是否加入队列，如果加入队列，那么看分发消息的情况

   **解决**：

   1. 发现是有扫描任务插入队列后就会有一次关闭操作。实际上由于任务队列排队机制，当有扫描正在执行，第二次的扫描只会进行排队，而不会进入scan插件执行扫描
   2. `taskExecutionBegins`里面插入队列返回值报错，插入执行任务应该返回true执行scan，插入排队任务应返回false。

4. 关于信息填写弹窗bug:

   **要求**：中控下发信息填写任务，只有一个弹窗，当下发多个任务只执行最后的任务

   **原因**：publish时，由于对应字段的data为空，导致第二次弹窗失败。且由于任务队列是map，不能同时存在多个相同key的任务

   **解决**：

   1. 修改任务队列为mutilmap，并使用锁
   2. 修改了taskBegain处的逻辑，一个任务、多个任务如何处理
   3. 第二次从队列中取出并通过MessageCenter向terminal_datils发送Buddle时，不给Buddle赋值data，只赋type



## 三、技术问题

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

27. 关于`QStackedWidget`和`QTabWidget`的区别

    1. `QStackedWidget`不提供直接的页面切换界面（没有标签栏）。`QTabWidget`提供带标签的界面，每个标签对应一个页面。
    2. `QStackedWidget`必须通过代码手动切换页面 (`setCurrentIndex()` 或 `setCurrentWidget()`)。`QTabWidget`用户可以通过点击标签页直接切换页面。

28. `exec()`和`show()`的区别

    1. **`exec()`**：以 **模态对话框** 的方式显示，阻塞主线程，用户必须关闭对话框后才能继续与主程序交互。
    2. **`show()`**：以 **非模态** 方式显示，不阻塞主线程，用户可以同时操作其他窗

29. `fork`分离进程  、  `daemon`守护进程

    1. 父子进程从 `fork()` 之后的代码开始**并行运行**
    2. `daemon`用于将当前进程变成 **守护进程**，即后台独立运行且与控制终端断开的进程

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

    ```
    #include <unistd.h>
    
    int daemon(int nochdir, int noclose);
    ```

    > **`nochdir`**：
    > 
    > - 如果为 **0**，将工作目录更改为根目录 (`/`)。
    > - 如果为 **1**，则保持当前工作目录不变。
    > 
    >**`noclose`**：
    > 
    >- 如果为 **0**，则关闭标准输入 (`stdin`)、标准输出 (`stdout`)、标准错误 (`stderr`)，并将它们重定向到 `/dev/null`。
    > - 如果为 **1**，保持标准输入/输出/错误流不变。

30. 关于EXPORT

    ```
    #cmakelist中定义：
    target_compile_definitions(${BIN_NAME} PRIVATE "PLUGIN_EXPORT=extern \"C\" __attribute__((visibility(\"default\")))")
    #cpp中：
    PLUGIN_EXPORT std::unique_ptr<IPlugin> createPlugin()
    {
        return std::unique_ptr<CJYNetProtectionPluginImpl>(new CJYNetProtectionPluginImpl());
    }
    ```

    1. **`PLUGIN_EXPORT=`**：
       - 它定义了一个宏 `PLUGIN_EXPORT`，其值为：
         `extern "C" __attribute__((visibility("default")))`。
       - 在代码中，如果用到 `PLUGIN_EXPORT`，就会被替换为这个具体内容。

    2. **`extern "C"`**：
       - 用于 **C++** 代码，告诉编译器使用 **C 的符号命名规则（C linkage）**。
       - 这是为了使编译后的符号能被 C 编译器或其他语言更容易找到（避免 C++ 名字修饰/mangling）。

    3. **`__attribute__((visibility("default")))`**：

       - 这是 **GCC/Clang 特定的语法**，用于控制符号的可见性。

       - `visibility("default")`

         表示：

         - 符号是默认可见的，可以从共享库外部访问。
         - 这是导出符号（如插件 API）时常用的设置。

31. Json格式的是vector吗：是的

    ```
    #假设jsonData对象为
    "tasks": [
        {"task_id": 1, "task_name": "Task One"},
        {"task_id": 2, "task_name": "Task Two"}
      ]
    
    std::vector<nlohmann::json> tasks = jsonData["tasks"];
    #每一个vector就是{"task_id": 1, "task_name": "Task One"},
    #遍历 tasks vector，输出每个任务的内容
    for (const auto& task : tasks) {
       std::cout << "Task ID: " << task["task_id"] << ", Task Name: " << task["task_name"] << std::endl;
    }
    ```

32. `using` 定义函数签名和回调函数

    ```
    using MessageHandler = std::function<void(IBundle *pBundle)>;
    #MessageHandler 是一个可以接受指向 IBundle 类型的指针 pBundle 的函数的类型别名
    ```

33. `unordered_multimap`的`equal_range`函数

    ```
    std::pair<iterator, iterator> equal_range(const Key& key);
    #key：要查找的键。
    #返回一个 std::pair：
    	#第一个元素是指向第一个匹配元素的迭代器
    	#第二个元素是指向第一个不匹配元素的迭代器（即结束位置）。
    ```

34. 容器(`multimap`)加锁

    ```
    #map和multimap使用的都是<map>头文件
    ```

35. 关于文件的读写，如果写文件时文件名不存在，那么会创建一个。如果读文件时文件名不存在，那么会打开失败

    ```
    std::ofstream file(filePath, std::ios::out | std::ios::trunc);		#写文件
    std::ifstream file(filePath);										#读文件
    file.is_open()			#检测是否成功打开
    file >> strValue;		#读文件，只针对ifstream类型
    file << Base64Util::Base64Encode(strValue);		#写文件，只针对ofstream类型
    ```

36. 关于protobuffer的枚举类型指针，反向获取枚举类型名

    ```c++
    #获取枚举类型描述符
    const google::protobuf::EnumDescriptor *descriptor = HmiToScan::VirusScan_ScanType_descriptor();
    #通过枚举数值反向找该数值对应描述符
    const google::protobuf::EnumValueDescriptor *enumValueDesc = descriptor->FindValueByNumber(static_cast<int>(scanMessage.scan_type()));、
    #通过name()函数得到string类型的类型
    enumValueDesc->name()
    ```

37. protobuffer打印出来为空

    这可能是由于 `SerializeAsString()` 方法默认序列化到二进制格式，直接打印到日志时会因为它是不可读的二进制数据而显示为空，实际进行protobuffer转换是可以输出的。

38. `static_assert(std::is_base_of<IComponentInterface, T>::value, "T must derive from IComponentInterface");`

    >这条语句的作用是，**在编译时**强制要求 `T` 必须是 `IComponentInterface` 的派生类;如果 `T` 不是 `IComponentInterface` 的派生类，编译器会抛出错误，错误信息为 `T must derive from IComponentInterface`

39. 组件接口父类分析

    ```c++
    《component_interface》
    template<typename T>
    # 返回类型为IComponentInterface的共享指针
    static std::shared_ptr<IComponentInterface> createComponent(
        std::shared_ptr<CGeneralOperatorImpl> pGeneralOpImpl,
        std::shared_ptr<IMessageCenter> pMessageCenter)
    {	
       	#断言，确保T属于IComponentInterface的子类
        static_assert(std::is_base_of<IComponentInterface, T>::value, "T must derive from IComponentInterface");
        #返回使用子类构造初始化的共享指针
        return std::make_shared<T>(pGeneralOpImpl, pMessageCenter);
    }
    ```

    







# 未完成笔记

#### 1. 项目部分

1. 项目通信部分逻辑是什么?

   > 看safed->core->plugin/message相关的cpp

2. 助手与UI之间的交互？助手与服务端的交互都是如何实现的

3. 执行文件(SAFED ZDFY GZCZ)和包(.so)的分布情况

4. plugin.conf的message的Key，在哪个地方初始化。如果key没有卸载config文件里面会如何

   ```
   #下面两句如何实现的?
   std::string msgType = BundleHelper::getBundleAString(pIn, JYMessageBundleKey::JYMessageType, "");
   std::string msgData = BundleHelper::getBundleBinary4String(pIn, JYMessageBundleKey::JYMessageBinValue, "");
   ```

5. 线程类(`ThreadWrapper`)是如何实现的？如何通过集成该类就可以实现线程的功能？线程池如何运转

6. 查看`.clang-format`如何设置，规格化工具

7. 插件、组件之间通信逻辑，画图梳理。总结关系

8. 组件之间通信都是靠`message_center`的发布-订阅模式进行传递消息并执行相应操作。给组件传递消息是通过`plugin_manager`的注册消息订阅以及对回调时对不同插件调用`onNewNotify`函数

9. 关于强力查杀bug

   **分析**：

   1. 使用`FULL_SCAN`字段下发任务，具体实现在`FULL_SCAN`任务处理那块找

      ```
      string strong_sign = 5; // 强力查杀标识，兼容老版本客户端，老版本执行全盘查杀，新版本解析此标识，1为强力查杀，0为全盘查杀
      ```

   2. `ScanFullTask`初始化时会`detach`一个线程：读取强力查杀留下的标记文件、如果含有强力查杀标记，那么就执行一次全盘扫描。

      `task`函数：由于全盘和强力是同一个param，对其执行全盘扫描，并判断type是否是强力查杀，如果是那么将本次参数记录到`conf`文件中，供重启后执行扫描使用

   3. 扫描最终执行的是给告警处置模块处理了？(通过gjcz的IPC)

   **未完成部分**：

   1. 没有进行病毒库升级                    -----需要创建升级任务放入队列 
   2. 是否不允许终端暂停或停止、需要验证        ----------UI层实现
   3. 扫描是否加入信任区文件          ---------应该是，因为全盘扫描从 /根目录下面同一扫
   4. 发现病毒如何自动清除          ------auto_remove | auto_clear 字段 或者在HIpDip的界面处理逻辑上
   5. 如何清除病毒后实现重启       ----------UI层实现

   **解决思路**：当接收到强力查杀任务key后，往任务队列中插入病毒库升级任务和强力查杀任务进行排队，等到病毒库升级成功后再执行强力查杀任务。

   1. 优先级：查看配置文件是否有病毒库升级任务
   2. upgrade中是否像terminal_details一样，先订阅key和处理逻辑。方便任务队列调用
   3. 在接收到强力查杀任务后，启动病毒库升级任务(需自己实现)，待其完成后实现强力查杀

   **解决**：

   1. 在强力查杀任务处，使用Task_managerBegain，传入key为VIRUS_LIB_UPGRADE ，value为UpgradeParam（其中类型为bool silent。0为不静默）

   2. gdb调试，看卡哪里了


   **11.7新问题**：

   1. 病毒升级卡在publish的锁那块，必须等强力扫描加入任务队列，才会解锁执行升级对应函数。强力扫描状态为begin，开始功能；病毒重新提交任务，状态为init，也开始功能。导致同步进行。
      2. 原因：执行publish时对发布队列进行了加锁，所以当病毒升级任务需要publish时也需要锁，最终造成了死锁的情况。

**11.8新问题解决思路**

1. 使用句柄，把病毒升级的那部分功能摘出来，重新封装一个cpp，跟task_manager一样，单纯的功能类。不同的是需要使用CGeneralOperatorAdapter对象shared指针，获取句柄，升级前执行UI弹窗，升级后执行版本号升级。
2. CGeneralOperatorImpl加入新对象。在CGeneralOperatorAdapter中加入新对象并加入新的执行病毒库升级函数
3. 修改相应部分，到强力扫描插件处，直接执行该功能函数(这个不是多线程)，然后执行强力查杀。如果是多线程，那么就在执行病毒库升级前加入任务队列begain状态。

**11.9问题**

1. operator使用句柄时，没有接收指针类型的拷贝构造。

#### 2. 代码部分

1. md5:MD5（Message-Digest Algorithm 5）是一种广泛使用的加密哈希函数，能够将任意长度的输入数据（通常称为消息）转换为固定长度的输出，具体来说是 128 位（16 字节）长的哈希值。

2. `enum class`是C++11的强枚举类型

   ```
   enum class ThreadMode {
           InnerMode,
           NormalMode
   };
   #1. enum class 定义的枚举成员受限于枚举类型的作用域:必须用 ThreadMode::InnerMode 才能访问到枚举类型
   #2. enum class 定义的枚举类型不会隐式转换为整数类型，需要显式转换
   	如：ThreadMode mode = ThreadMode::InnerMode;
   		int modeValue = static_cast<int>(mode);
   #3. 可以指定底层类型，比如 enum class ThreadMode : int，表示 ThreadMode 使用 int 作为底层类型。默认底层类型是 int，也可以指定为其他整数类型（如 char、unsigned int 等）。
   ```

3. 对于原子类型，线程安全的方式读取该值使用`load()`函数;

   `load()` 函数的作用是读取原子变量的当前值并返回。在多线程环境中，使用 `load()` 可以确保读取的值是线程安全的，即便其他线程可能同时在修改该变量。

4. C++标准库的承诺-未来机制`std::promise`和 `std::future`，用于在线程之间传递值或状态，尤其是在异步编程中协助传递结果。实现在两个线程间同步任务完成状态

   ```
   #使用方式，没有返回值的
   std::promise<void> _terminationPromise;
   std::future<void> _terminationFuture;
   ```

   ```c++
   //案例：
   #include <iostream>
   #include <thread>
   #include <future>
   
   void task(std::promise<void> taskPromise) {
       std::cout << "Task is running..." << std::endl;
       // 模拟一些工作
       std::this_thread::sleep_for(std::chrono::seconds(2));
       // 设置任务完成状态
       taskPromise.set_value();
       std::cout << "Task has completed." << std::endl;
   }
   
   int main() {
       // 创建一个 promise
       std::promise<void> taskPromise;
       // 从 promise 获得对应的 future
       std::future<void> taskFuture = taskPromise.get_future();
   
       // 启动一个线程执行任务，并将 promise 传递进去
       std::thread t(task, std::move(taskPromise));
   
       // 等待任务完成
       taskFuture.get();  // 阻塞直到 taskPromise 设置了值
       std::cout << "Main thread detected task completion." << std::endl;
   
       t.join();
       return 0;
   }
   
   ```

   ```
   #输出结果
   Task is running...
   Task has completed.
   Main thread detected task completion.
   ```

5. 关于`std::nothrow` 

   ```
   int* p = new(std::nothrow) int[10000000000]; // 请求分配大量内存
   ```

   `std::nothrow` 是 C++ 标准库中的一个常量，用于控制内存分配失败时的行为。通常在使用 `new` 操作符进行内存分配时，分配失败会抛出 `std::bad_alloc` 异常。而使用 `std::nothrow`，可以使 `new` 操作符在分配失败时返回 `nullptr`，而不是抛出异常。

6. 关于回调函数：回调函数（Callback Function）是一种通过函数指针、函数对象或 lambda 表达式传递到另一函数中的 **可调用对象**。回调函数可以理解为一个“回去调用”的函数，即某个函数 `A` 把函数 `B` 的地址传递给另一函数 `C`，然后 `C` 在适当的时机调用 `B`。

   > 个人理解就是把函数指针/lambda/函数对象作为函数参数，在该函数内部可以执行传来的作为参数的函数

7. 关于发布-订阅模式：使用了回调函数机制，订阅方使用`subscribe`将执行函数注册，插入到订阅-发布类的map中，当调用`publish`时，就会将执行函数发布/执行

   1. 发布-订阅(message_center)类

   ```c++
   #发布-订阅类
   #include <iostream>
   #include <functional>
   #include <vector>
   #include <string>
   class Publisher {
   public:
       // 定义回调类型，接受字符串作为参数
       using Callback = std::function<void(const std::string&)>;
   
       // 订阅事件（注册回调）
       void subscribe(const Callback& callback) {
           callbacks_.push_back(callback);
       }
   
       // 发布事件（调用所有已注册的回调）
       void publish(const std::string& message) {
           for (const auto& callback : callbacks_) {
               callback(message); // 执行回调，传递消息
           }
       }
   private:
       std::vector<Callback> callbacks_; // 存储所有的回调
   };
   ```

   2. 注册订阅

   ```
   m_pMessageCenter->subscribe("REPORT_USER_INFO", [this](IBundle *pBundle) {
           registerResetRequest(pBundle);
   });
   ```

   回调函数可以通过 **闭包** 或 **绑定对象的成员函数** 来捕获并访问订阅者类的成员变量，`std::bind` 或 lambda 表达式可以确保回调在被调用时持有 `this` 指针，从而能够访问订阅者类的成员变量。

   > 类似于在类中开启线程，需要使用bind或者lambda传递this指针

   3. 发布

   ```
   m_pMessageCenter->publish(pPublishBundle);
   ```

   只需要传递`map`中的`key`，就会根据`key`找到对应回调函数并执行

8. 闭包：闭包是 **函数和它的捕获上下文的组合**，这个上下文允许闭包函数在被调用时访问定义它时所在作用域中的变量，即便该作用域已超出生命周期。

   举例：在 C++ 中，lambda 表达式可以捕获当前作用域中的变量，因此 lambda 可以作为闭包的一个具体实现。

   1. 按值捕获 (`[=]`)：复制变量值到闭包中。
   2. 按引用捕获 (`[&]`)：捕获变量的引用，使得闭包可以访问和修改原变量。
   3. 捕获 `this` 指针 (`[this]`)：捕获 lambda 所在对象的 `this` 指针，以便在 lambda 中访问对象的成员。

9. 句柄：**句柄（handle）** 是一种特殊的指针或引用，通常指一个**可以访问或操作其他对象的“引用”或“指针”**,用来标识和管理资源的访问。句柄通常不直接提供对资源的底层访问，句柄广泛用于资源管理，比如文件、内存、线程、网络连接等。句柄本质上是一种间接引用，指向一个资源或对象而不暴露其内部实现细节。

   ```c++
   #指针
   int* handle = new int(5); // handle 是指向动态分配的 int 的句柄
   std::cout << *handle << std::endl; // 使用句柄访问数据
   delete handle; // 删除资源
   
   #智能指针
   std::shared_ptr<int> handle = std::make_shared<int>(5); // handle 是 shared_ptr 句柄
   std::cout << *handle << std::endl; // 使用句柄访问数据
   ```

   **案例**：当组件的某个功能需要使用到其他组件的功能，该组件接收功能总类的句柄，通过该总类执行其他组件的功能。而总类中又有该组件的一些功能，在使用时需要先把this(当前对象)传递给该组件作为句柄

   **问题**：注意循环引用的问题导致双方对象无法释放。

   ```c++
   #include <iostream>
   #include <memory>
   
   class Manager; // 前向声明 Manager
   
   class Worker {
   public:
       // 设置 Manager 的句柄（std::weak_ptr 以避免循环引用）
       void setManagerHandle(std::shared_ptr<Manager> manager) {
           this->managerHandle = manager;
       }
       // 执行任务并尝试访问 Manager 的功能
       void performTask() {
           if (auto mgr = managerHandle.lock()) { // 使用 lock() 将 weak_ptr 转换为 shared_ptr
               std::cout << "Worker is performing a task and accessing Manager's function." << std::endl;
               mgr->managerFunction();
           } else {
               std::cout << "Manager is no longer available." << std::endl;
           }
       }
   private:
       std::weak_ptr<Manager> managerHandle; // 使用 weak_ptr 防止循环引用
   };
   
   class Manager : public std::enable_shared_from_this<Manager> {
   public:
       Manager() {
           worker = std::make_shared<Worker>(); // 创建 Worker 实例
           worker->setManagerHandle(shared_from_this()); // 将自身句柄传递给 Worker
       }
       void managerFunction() {
           std::cout << "Manager's function is called by Worker." << std::endl;
       }
       void startWork() {
           worker->performTask(); // 通过 Worker 执行任务
       }
   private:
       std::shared_ptr<Worker> worker; // 持有 Worker 的 shared_ptr
   };
   
   int main() {
       auto manager = std::make_shared<Manager>();
       manager->startWork(); // Manager 调用 Worker 的函数，Worker 访问 Manager 的方法
       return 0;
   }
   ```

10. 向前声明和使用头文件的区别
    1. **减少依赖**：使用向前声明，你不需要直接包含其他头文件。这可以减少类之间的直接依赖，有助于减少编译时间和避免循环依赖。例如，在 `Worker.h` 中向前声明 `Manager`，这样你可以避免包含 `Manager.h`，从而减少编译开销。
    2. **完整定义**：如果你需要使用类的具体实现（如访问成员、调用函数、创建实例等），则必须包含该类的头文件，以便编译器了解该类的具体结构。**向前声明只让编译器知道类型的名称，但不提供任何方法或成员的信息**。
    3. **避免循环依赖**：在两个类相互引用的情况下，使用向前声明可以打破循环依赖。例如，`Manager` 和 `Worker` 互相依赖时，使用向前声明可以避免直接包含彼此的头文件。

11. 为什么头文件有的需要加上文件夹前缀

12. 为什么只能传递IGeneralOperator，而不能传递CGeneralOperatorAdapter？

13. 代码整体缩进或者右移：选中代码->`ctrl+[`  / `ctrl+]`

14. 循环依赖问题(当class a 导入class b的头文件，classb 也导入class a的头文件时)，需要使用前置声明。不然编译器会不知道先编译哪个

    ```c++
    // A.h
    class B;  // 前向声明B
    class A {
        B* b;  // 这里我们不需要完整的B类，只需要声明指针
    public:
        void doSomething();
    };
    
    // B.h
    class A;  // 前向声明A
    class B {
        A* a;  // 同样，使用A的指针，不需要完整的A类
    public:
        void doSomethingElse();
    };
    
    ```

14. 智能指针循环依赖问题

    ```c++
    class A;
    class B;
    
    class A {
    public:
        std::shared_ptr<B> b;
    };
    
    class B {
    public:
        std::shared_ptr<A> a;
    };
    
    void createCycle() {
        auto a = std::make_shared<A>();
        auto b = std::make_shared<B>();
        a->b = b;
        b->a = a;
        // 循环引用导致内存泄漏
    }
    ```

15. 句柄传递`this`的智能指针问题

    1. 如果在母类中直接传递`this`裸指针，一旦母类析构，则在使用该`this`指针的类就会造成**指针失效**

    2. 如果使用`this`创建`make_shared`后传递，会将 `this` 指针当作一个新对象的起点进行管理，并且会在 `shared_ptr` 的引用计数归零时自动删除该对象。这对一个已经在其他地方管理生命周期的对象（例如，栈上或由另一个指针管理的对象）是危险的，因为它会导致对象的**双重销毁**。

       >  但是我使用的是make_shared，重写了构造函数。相当于把主类的成员共享对象指针++。当我析构主类的使用句柄的对象时，一样会使指针--，在类里面的句柄所以没问题。

    3. 让母类继承 `std::enable_shared_from_this`，并在对象本身被 `shared_ptr` 管理时使用 `shared_from_this()` 来安全地生成 `shared_ptr`。

       ```c++
       #只适用于下面这种，已经被shared_ptr管理的对象中使用，否则会抛出 std::bad_weak_ptr 异常
       std::shared_ptr<MyClass> p = std::make_shared<MyClass>();
       p->show();  // 在成员函数中使用 shared_from_this
       ```

16. `shared_ptr`作为函数参数传递：
    1. **`std::shared_ptr<T>`**：只能传递 `shared_ptr` 类型对象，除非使用 `std::make_shared` 生成 `shared_ptr`或者`shared_from_this()` 将类本身被创建时的`shared`指针传递，否则不能传递临时对象(指针)。
    2. **`std::shared_ptr<T>`**：传递 `shared_ptr` 意味着函数将共享该对象的所有权。每次传递 `shared_ptr` 参数，引用计数会增加，直到离开作用域或显式释放才会减少
