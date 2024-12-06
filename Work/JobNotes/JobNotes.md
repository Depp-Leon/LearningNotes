

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

8. `BetterComments`注释插件

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

9. vscode导入头文件爆红的解决方式

   > 按atrl+shirft+p打开cppconfigure，手动添加头文件路径

11. VScode快捷键：

    ```
    ctrl+p			#查找文件快捷键
    ctrl + 左键	   #进入函数
    ctrl + alt + -	#默认返回函数，这个-不能用小键盘上的
    alt + leftarrow(左箭头)  #修改后的返回函数快捷键 
    ctrl + ` (esc下面那个) #打开终端
    选中代码->`ctrl+[`  / `ctrl+]`  #代码整体缩进或者右移：
    ctrl+G			#vs里面查找某一行快捷键
    ```

12. ubuntu清屏快捷键

    ```
    ctrl + L		#等同于clear
    ```

13. 远程文件拷贝

    ```
    scp file.txt user@remote_host:/path/to/destination/
    scp JYNSAFED libJYVirusScan.so leslie@192.168.3.96:~/
    # file.txt 是本地文件。
    # user@remote_host 是远程主机用户名和地址。
    # /path/to/destination/ 是远程主机上的目标路径。
    ```

14. ubuntu给非root用户添加sudo权限

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

15. ubuntu下查看本地的包

    ```
    dpkg -l | grep <包名>
    ```

16. `dpkg -b` 命令用于 **构建** 一个 Debian 包（`.deb` 文件），它将指定的目录或源文件打包为一个 `.deb` 文件。

    ```
    dpkg -b <目录> [<输出文件>]
    ```

17. ubuntu下卸载使用包管理模式安装的包

    ```
    sudo apt remove --purge <包名>
    sudo apt autoremove
    	#remove：卸载软件包。
    	#--purge：同时删除与软件包相关的配置文件。
    	#autoremove：清理未使用的依赖项。
    ```

18. linux授权指令

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
       ```

## 二、项目框架

### 2.1 客户端层次

1. 客户端整体框架层次分布

   ```
   1. UI界面层和Model层合称为人机交互层(RJJH)
      		UI界面层:界面和弹窗
     	    UI moudle层:通过IPC与助手通信，并使用信号槽与UI层建立通信
   3. 组件层与插件层合称为助手层(safed)
      		组件层(component): 主要复用的功能，包括与UI和中控之间信息交换、升级、任务队列等功能
      		插件层(plugin)：不同独立的功能封装为一个插件，使用组件的功能与UI和中控进行信息交换(用的其实		   				  还是组件的功能，需要走组件)。
   4. ZDFY和GJCZ层：助手调用这两个执行文件的功能
   		ZDFY主要负责系统防护和数据防护；
   		GJCZ是实现扫描功能(插件scan只是调度GJCZ)；
   		两个可执行文件开启系统后就在后台启动。
   ```

2. 组件(`modules/component`)主要是一些共用的功能模块，插件(`plugins`)是单独的可以增加、去掉的模块。比如不同的插件都可以使用组件里的认证、防护日志、任务管理、定时任务、更新等

3. `include`文件夹里面主要包含的是定义的头文件，把复杂的头文件/接口摘出来

4. `common`文件夹里面主要包含的是整个项目封装好的工具类，比如`nlohman`(`json`文件)、`thread`(线程)、`utils`(工具包，各种时间、类型转换)、`epoll`(`io`相关)

5. 组件实现方式与插件类似，都是接口`impl`和实现分离。

6. `src2.0/oem`文件夹下面是实现不同品牌的贴牌图片和贴牌脚本。当打包时就会根据版本类型替换对应的skin图标文件



### 2.2 RJJH层

#### 2.2.1 UI层

> 该层主要负责各种界面和弹窗的界面文件，并使用信号槽与model层之间建立通信。

1. UI的FramelessWindow：构造函数内部除了建立信号槽以外，末尾部分调用了各model的请求函数，向safed/中控发出请求，并在内部有接收函数，进而刷新后台数据展示到界面

   ```
   m_pTerminalInfoModel->requestTerminalInformation();
   m_pScanModel->requestScanProcessResult();
   m_pTerminalAuthModel->requestTerminalAuthorizeInfo();
   m_pUIConfigModel->requestTerminalConfigInfo();
   m_pProtectLogModel->slotRequestProtectLogInfo(10);
   m_pUpgradeModel->terminalVersionInfoRequest();
   ```



#### 2.2.2 UI Model层

> 该层主要负责UI层与助手层之间的交互，接收到助手层消息后发送信号到UI界面



### 2.3 插件层/助手

#### 2.3.1 通述

1. 组件(`modules/component`)主要是一些共用的功能模块，插件(`plugins`)是单独的可以增加、去掉的模块。比如不同的插件都可以使用组件里的认证、防护日志、任务管理、定时任务、更新等

2. mingfei代码属于是让功能类继承CGeneralOperator,CGeneralOperator需要有参(IGeneralOperator)构造，然后在接口处初始化时直接让该功能类使用传来的IGeneralOperator进行构造。由于功能类继承于CGeneralOperator，所以可以使用其定义的函数进行与前端交互。

   > 接口类继承于CPluginHelper，该类继承于CPlugin和CGeneralOperator，所以接口类也可以使用插件类的函数和前端交互的函数

   kongbin代码属于是类的聚合，定义一个总类，里面包含功能类和CGeneralOperator(与前端交互类)实现功能互用

4. 插件对象，子类可以当作父类传入实现多态。

6. 组件/插件开启时使用线程

   ```
   JYThread::autoRun(std::bind(&CTaskManagerComponentImpl::taskQueueDistribution, this));
   ```

5. 组件之间通信都是靠`message_center`的发布-订阅模式进行传递消息并执行相应操作。给组件传递消息是通过`plugin_manager`的注册消息订阅以及对回调时对不同插件调用`onNewNotify`函数

6. core中插件管理是通过使用动态库来使用不同的插件类的。(`dlopen、dlclose`)

#### 2.3.2 运转核心/core

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

##### 2. 组件管理 `component_manager`

```
1. 通过createComponent创建对应组件接口实例，并将他们插入到std::vector<std::shared_ptr<IComponentInterface>>中
2. 统一进行开启，释放
```

##### 3. 插件管理`plugin_manager`

```
loadPlugin：根据plugin.conf加载插件到map中(通过dlsym读取动态库的方式、获取到插件接口的CreatePlugin函数创建插件接口实例)
enablePlugin：遍历map，依次启动插件的init和start，并注册消息订阅，触发消息时执行publishPlugin
unloadPlugin：根据插件名称卸载插件
disablePlugin: 停止所有插件，启动插件的stop和release
publishPlugin：找到对应插件，执行对应插件的onNewNotify操作
```

##### 4. 消息队列`message_center`

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



#### 2.3.3 组件/component

##### 1. 组成部分

1. 组件接口/抽象：

   ```
   1. CGeneralOperatorImp: 初始化创建不同组件的功能对象(非组件接口impl)；所有组件的接口使用组件功能类时都通过该类获取到功能对象
   
   2. CComponentInterface: 
   	组件的总接口CComponentInterface，继承于IComponentInterface(接口类，定义虚函数初始化、开启、停止、释放等);所有的组件impl都继承于这个类。
   	初始化时需要CGeneralOperatorImpl和IMessageCenter
       自己定义模板返回创建组件接口impl对象、sendToUi向UI层发送消息这两个函数
   
   3. CGeneralOperatorAdapter: 向插件层提供组件的功能，在core下面的插件管理，将该类传递作为插件初始化参数。
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
   ctrl_center组件，实现向中控发送信息(asyncReport)
   local_service组件：实现接收UI层和中控的消息(插入消息队列，依次调用回调);向UI层发送信息
   protect_logger组件：实现UI层面的日志上报和详细日志上报(recordProtectionLogs)
   authorize组件：实现各种认证、版本信息、病毒库信息、引擎信息、授权信息等
   timed_tasks组件：实现注册定时任务(registerTask)
   task_manager组件：实现任务队列(添加、删除任务到任务队列)
   upgrade组件：实现软件和病毒库更新。没有在general_Operator_impl中定义(插件用不了)。
   terminal_details组件：资料组件；实现填写用户信息、更新用户信息、授权、密码文件等功能。
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



#### 2.3.4 插件/plugin

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
8. gjcz_scan: 负责与gjcz交互，gjcz实现查杀功能；包含：
	1. app_scan: 扫描服务，根据扫描文件路径进行扫描
    2. ScanPostman: 负责与gjcz之间通信(包括暂停、继续、清理等)
9. post: 上报服务，(上报client_action到中控)
# 4-9全部作为flow的成员对象进行任务处理调度
```



### 2.4  数据传输

数据传输方式有`protobuffer`和`Json`格式，作为项目中的数据传输类型

##### 2.4.1 protobuffer

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

3. 传递时候的序列化(转字符串传输)

   ```c++
   ReportProtect report;				#ReportProtect是protbuf生成的class
   report.SerializeAsString()
   ```

4. 获取非重复非嵌套字段

   ```
   直接.加类型名()
   ```

5. 获取重复字段

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

6. 获取非重复嵌套字段

   ```c++
   #点就完事
   scanNotify = commonC.notification().scan().status()     
   ```

7. 设置非重复非嵌套字段

   ```c++
   #set_字段名
   Person person;
   person.set_name("Alice");  // 设置名字
   person.set_age(30);        // 设置年龄	 	
   ```

8. 设置重复嵌套字段

   ```c++
   #先定义嵌套字段类型，再给该类型set数据
   ClientIsolationAreaInfo isoMsg{};
   ClientIsolationAreaInfo::InfectInfo* isoInfo = isoMsg.add_infects();
   isoInfo->set_infect_name(info.param.virusname);
   isoInfo->set_file_name(info.param.path);
   isoInfo->set_isolate_time(info.m_time);
   isoInfo->set_md5(info.param.md5);
   ```

9. mutable_ 

   ```c++
   // 获取指向嵌套 Address 消息的指针，并设置其字段。
   Address* addr = person.mutable_address();  
   addr->set_city("New York");
   addr->set_zip("10001");
   ```

10. add_和mutable\_的区别

    ```c++
    add_address()：向 repeated 字段中 新增一个地址。
    mutable_address(index)：获取并修改指定索引处的地址。
    address(index)：只读访问指定索引处的地址。
    ```

11. 枚举类型直接通过下标_获取

    ```c++
    HmiToScan::VirusScan_ScanType_THREAT_CLEAN
    ```

12. protobuf的msg使用二进制类型byte和一个type，就可以根据不同的type的message进行解析

    > 详见jy_virus_scan_impl.cpp   158行

13. 关于`protobuffer`初始化

    ```
    #protobuffer中定义的结构体
    message Person {
        string name = 1;
        int32 id = 2;
    }
    #proto自动生成的类
    class Person {
    public:
        Person() {
            name = "";
            id = 0;
        }
        std::string name;
        int id;
    };
    
    Person person = { "John", 123 }; // 使用默认构造函数初始化，并通过列表初始化指定字段值
    Person person;  // 使用默认构造函数，字段为默认值
    Person person{}；/ 值初始化 的方式，这意味着该对象会被默认初始化为其类型的默认值。等同于上句
    ```

    1. 对于 **Protocol Buffers** 生成的类（如 `Person`），protobuf 会为每个字段提供默认值
    2. protobuf 生成的 C++ 类支持列表初始化和通过构造函数进行初始化的功能，原因是 C++ 标准库支持了 **聚合初始化** 和 **初始化列表** 语法。
    3. 如果你使用 `{}` 来初始化 `Person` 类型的对象，它会通过默认构造函数初始化对象，字段会被赋予其 **默认值**（例如，`name` 为 `""`，`id` 为 `0`）。对于普通 C++ 类，构造函数的默认行为可能不会自动为每个字段赋值，你必须手动初始化，除非你自己为其提供默认构造函数。

    4. `Person person{};` 是一种使用 **统一初始化** 语法（uniform initialization）来初始化对象的方式，这种语法是 C++11 引入的。`Person person{};` 等价于 `Person person;`，但更明确地表示 **值初始化**

    ```
    #C++11值初始化
    int d(20); 	
    int age{20};
    #C++11初始化列表
    int c = {1001};
    ```

14. protobuffer中`HmiToHelper::TerminalAuthorizeSession_Response` 和 `HmiToHelper::TerminalAuthorizeSession::Response` 是一个效果，都是嵌套的结构。`_`是`::`的别名，两种proto的生成风格，一般都会生成两套，所以都可使用。

15. protobuf使用枚举字段、使用（取值、赋值）

    ```
    HmiToScan::VirusScan_ScanType_THREAT_CLEAN
    ```

16. 关于protobuffer的枚举类型指针，反向获取枚举类型名

    ```c++
    #获取枚举类型描述符
    const google::protobuf::EnumDescriptor *descriptor = HmiToScan::VirusScan_ScanType_descriptor();
    #通过枚举数值反向找该数值对应描述符
    const google::protobuf::EnumValueDescriptor *enumValueDesc = descriptor->FindValueByNumber(static_cast<int>(scanMessage.scan_type()));、
    #通过name()函数得到string类型的类型
    enumValueDesc->name()
    ```

17. protobuffer打印出来为空

    这可能是由于 `SerializeAsString()` 方法默认序列化到二进制格式，直接打印到日志时会因为它是不可读的二进制数据而显示为空，实际进行protobuffer转换是可以输出的。

##### 2.4.2 Json

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

##### 2.4.3 protobuffer和json的区别



### 2.5 日志

1. 导入qlog库，使用LOG日志记录

   ```
   #include "glog/logging.h"
   ```

2. 开启日志打印

   ```
   vim /opt/apps/chenxinSd/etc/LogConf.ini
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

   1. **`LOG(INFO)`**：输出信息级别日志。**场景**：用于记录**正常的程序执行信息**。它表示程序按预期工作，没有异常情况。常用于打印一些程序流程、状态信息、统计数据等。
   2. **`LOG(WARNING)`**：输出警告信息。**场景**：用于记录**非致命的异常情况**或潜在问题。这些问题不会导致程序崩溃，但可能影响性能、用户体验，或者暗示更严重的问题可能即将发生。
   3. **`LOG(ERROR)`**：输出错误信息。**场景**：用于记录**严重的错误**，这些错误会导致程序某些功能失败，甚至崩溃。通常表示程序中无法忽略的错误。
   4. **`LOG(FATAL)`**：输出致命错误并终止程序。`LOG(FATAL)` 表示**不可恢复的错误**，程序在记录该日志后会**立即终止执行**。它用于捕捉那些使程序无法继续运行的严重错误，通常表示逻辑问题或异常情况无法修复。

   > 使用 **`LOG`** 宏时，不需要手动添加换行符。它会**自动在每条日志末尾添加换行符**，确保每一条日志信息在新行中打印。



### 2.6 数据库

##### 2.6.1 SQlite

1. sqlite的常用命令

   ```
   sqlite3 <数据库.db>
   .tables   #查看所有表
   SELECT * FROM TABLE;    #类似于sql语句,注意需要加分号
   .quit 
   .exit	#退出
   
   ```

2. 项目中sqlite的路径

   ```
   /opt/apps/chenxinsd/cache/xxx.db
   ```

3. Sqlite3查看表的结构

   ```
   PRAGMA table_info(table_name);			#在Sqlite下
   ```

   ```
   #表的结构如下
   cid   name        type       notnull   dflt_value   pk
   0     id          INTEGER    1         NULL         1
   1     name        TEXT       0         NULL         0
   2     age         INTEGER    0         NULL         0
   cid: 字段的序号（从 0 开始）。
   name: 字段的名称。
   type: 字段的数据类型。
   notnull: 是否允许 NULL 值（1 表示不允许）。
   dflt_value: 默认值。
   pk: 是否为主键（1 表示是主键）
   ```

4. C++提取Sqlite3数据库的两种方式

   ```c++
   #方式一：使用sqlite3_get_table函数
   
   /**
    * @brief: 读取sqlite表中数据
    * @param db: 打开的数据对象
    * @param zSql: 命令
    * @param pazResult: 读到的数据
    * @param pnRow: 行数
    * @param pnColumn: 列数
    * @retval: sqlite3_get_table的返回值
    */
   int jy_sqlite_get_table(sqlite3 *db, const char *zSql, char ***pazResult, int *pnRow, int *pnColumn)
   {
       int res;
       char *err_msg;
       res = sqlite3_get_table(db, zSql, pazResult, pnRow, pnColumn, &err_msg);
       if (res != SQLITE_OK)
           LOG(ERROR) << "CDBTimedTasksOper Select! err:[" << err_msg << "]";
       if (err_msg)
           sqlite3_free(err_msg);
       return res;
   }
   ```

   ```c++
   #方式二：使用sqlite3_exec函数
   
   bool CTerminalDetailsCache::setLastScanInfo(const std::string &result, const std::string &time)
   {
       if (!m_sqlite) {
           return false;
       }
       char *errmsg = nullptr;
       char sql[1024] = {0};
       snprintf(sql, sizeof(sql), "UPDATE terminal_details SET last_scan_result = \"%s\", last_scan_time = \"%s\" WHERE id = 1;", 
                result.c_str(), time.c_str());
       int rc = sqlite3_exec(m_sqlite, sql, nullptr, nullptr, &errmsg);
       if (rc != SQLITE_OK) {
           sqlite3_free(errmsg);
           return false;
       }
       return true;
   }
   ```

   


### 2.7 编译

1. 详见**CMakelist**，打包so文件，即动态库

2. 将编译好的文件移动到程序目录下

   ```
   cd Output/bin2.0/JingyunSd_linux_2/bin/X86_64/
   
   sudo cp cur_user JYNGJCZ2 JYNRJJH2 JYNRJJH2-UTRAY1 JYNSAFED JYNZDFY2 JYUpdateUI /opt/apps/chenxinsd/bin/
   
   sudo cp libglog.so.0.3.5 libkvcache.so libnetplugin.so libPostDataReport2.0.so libSysMonManage.so libZyAuthPlug.so libZyAVCache.so libJYVirusScanEnginePlugin.so libZyUploadFile.so /opt/apps/chenxinsd/lib/modules/
   
   sudo cp libJYFileShred.so libJYNetProtection.so libJYSystemController.so libJYUDiskProtection.so libJYVirusScan.so libJYZDFY.so /opt/apps/chenxinsd/lib/plugins/system/
   ```

3. 什么时候需要重新执行cmake和make

   > 当更改了cmakelist中的库依赖、文件路径等需要重新对项目进行cmake
   >
   > 若只修改了代码并没有改变项目架构，那么只需要执行make编译就可

4. Cmakelost添加头文件路径时，要指定到最终的文件夹下，不然还是找不到头文件。(它只会在你指定的那个而文件夹下面找头文件)

5. `.cmake`文件：后缀是 `.cmake` 的文件是 **CMake** 使用的脚本文件，通常用于定义构建系统的配置、设置变量、导入/导出目标以及包含其他模块

   ```
   include(MyCustomFile.cmake)				
   #在cmakelist中使用cmake脚本
   #在 .cmake 文件中定义的变量和函数会被导入当前的 CMake 环境
   ```

6. cmake脚本和cmakelist的区别:

   1. `CMakeLists.txt`：用于描述项目的构建规则，是项目的入口文件
   2. `.cmake` 文件：通常是辅助文件，提供模块、工具或特定功能的实现，`.cmake` 文件通过 `include()` 或其他机制被调用

7. cmakelist中的list类型：

   ```
   list(APPEND ...) 命令用于将新的路径添加到现有的列表中
   ```

   ```
   set(CURL_LIB_PATHS)    
   if(EXISTS "${CX_THIRD_PARTY_DIR}")
   	list(APPEND CURL_LIB_PATHS "${CX_THIRD_PARTY_DIR}/curl-7.81.0_prefix/lib")
   	list(APPEND CURL_LIB_PATHS "${CX_THIRD_PARTY_DIR}/curl-7.81.0_prefix/lib64")
   else ()
   	list(APPEND CURL_LIB_PATHS "${CX_ARCHIVE_DIR}/curl-7.81.0_prefix/lib")
   	list(APPEND CURL_LIB_PATHS "${CX_ARCHIVE_DIR}/curl-7.81.0_prefix/lib64") 
   endif()
   ```

8. cmakelist中使用`find_library` 命令查找指定的库文件并将其路径存储到变量中

   ```
   find_library(<VAR> NAMES <name> PATHS <path1> <path2> ... NO_DEFAULT_PATH)
   <VAR>：存储找到的库路径的变量名。例如，这里是 LIB_CURL 和 LIB_CPR。
   NAMES <name>：要查找的库的名称。例如：
   	curl 表示要查找的 curl 库。
   	cpr 表示要查找的 cpr 库。
   PATHS <path1> <path2> ...：指定库的搜索路径列表。
   NO_DEFAULT_PATH:限制只能从指定的路径列表下搜，如果不设置的话若找不到会从默认的路径下寻找。
   
   #案例： find_library(LIB_CURL NAMES curl PATHS ${CURL_LIB_PATHS})
   这里用的是 ${CURL_LIB_PATHS}，它们之前通过 list(APPEND ...) 定义了库的多个可能路径。
   ```

   

### 2.8 调试

1. 使用"sudo gdb JYNSAFED"

2. 打断点：在目的函数名 b CJYNetProtectionPluginImpl::onNewNotify

3. 执行：r

4. 逐过程：n       

5. 逐步骤：s      /会进入到函数内部

6. 完成当前函数：finish    /使用s进入到内部后，可以使用finish退出这个函数

7. p 变量名：打印变量名的值，确认是否正确收到并解析

8. bt：查看栈调用情况，如果程序崩溃可以通过栈查看那个地方有问题

9. quit ：退出

10. c/C：可以跳过当前断点要操作的内容，继续执行

11. 使用GDB调试：遇到某个函数执行，但是又不知道是那部分在调用这个函数时，使用`gdb`在这个函数打上断点，执行到这个函数时，执行`bt`查看栈，就可以看到调用函数一层一层的顺序了。

12. 当打印数据类型是原子类型时，比如`atmoc_bool`

    ```
    (gdb) p m_reportLog
    $11 = {_M_base = {static _S_alignment = 1, _M_i = true}}
    ```

    1. `_M_base` 和 `_M_i` 是 `std::atomic` 或其他标准库类型的内部表示。它们通常用于实现原子操作和存储状态。
    2. `_S_alignment` 是一个静态成员，表示这个类型在内存中的对齐要求。在这里，对齐为 `1` 字节，意味着该类型的对象在内存中必须以 1 字节对齐，这通常适用于 `bool` 类型。

13. gdb调试报错：

    1. 是否编译的依赖的so没有导过去
    2. **哪部分需要调试，就将哪部分执行文件单独拿出来使用gdb执行，其他部分正常启动执行**

    > 补充：后台使用systemctl start jyn*启动的是三个服务，CZ控制，ZDFY主动防御、safed安全三个。一个界面只展示两个，没有展示三个

11. 使用`systemctl`和`sudo`来启动某些服务的区别

    1. 有些可执行文件需要通过 **`systemctl`** 启动是因为它们作为 **服务（Service）** 或 **守护进程（Daemon）** 来运行，这种方式适用于在后台长期运行且需要稳定管理的程序
    2. **`sudo`**是用来以超级用户权限执行任意命令的工具，涉及到文件读取等


12. 在40上面编写的包在本地安装后。当使用gdb调试，打断点发现打的路径是40上面的路径。所以需要在本地打对应的包移动并替换，并且修改完JYNSAFED不仅需要把JYNSAFED转出，还需要把lib生成的动态库也转出。

13. gdb调试打断点，停在目标函数时，上方有传来的参数内存信息，可以查看是否传来的是否为空
14. gdb调试模式下程序崩溃，使用bt查看崩溃在那个函数。重新在该函数打断点，一步步复现，查看崩在那个位置。

### 2.9 打包

1. 插件的打包配置(如果有)

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

3. 系统打包环境：`192.168.0.40` 、ssh进入服务器（系统服务器将会执行打包脚本适配不同系统的环境包）在系统打包环境下 pull仓库，同步代码后进行编译。编译后执行脚本开始打包

   ```
   #1. SSH连接
   ssh work@192.168.0.40		
   zyinfo						#密码
   #2. 更新代码
   cd /home/work/workspace2/708/normal_develop/
   git pull
   #3. 编译代码,在build目录下
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

7. 测试远程ip、账号、密码

   ```
   192.168.2.14
   root
   Vsecure@2016
   
   192.168.2.2
   root
   1QAZ2wsx
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

         1. 前端界面：`/opt/apps/chenxin/bin/JYNRRJJH2`   
      2. 后端中控+防护+主防：`systremctl start jyn*`     #JYNSAFED防护模块 、JYNGJCZ2中控模块、JYNZDFY主防模块

   ```
     #主动启动服务，适用于调试，看日志
     cd /opt/apps/chenxinsd/bin
     ./JYNRJJH2
     sudo ./JYNSAFED
     sudo ./JYNGJCZ show 
   ```

6. cmake显示缺失LIB_CPR：拷贝lib到normal_development文件夹下



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
   (origin/master, origin/HEAD) 说明：
   	远程仓库的 HEAD 当前指向 master 分支。
   	origin/master 是远程仓库 origin 上的 master 分支的最新提交。
   (HEAD -> master) 表示：
   	当前 HEAD 指向本地的 master 分支，意味着你正在 master 分支上工作。
   	当你提交新的更改时，master 分支的指针会随之移动到新的提交，而 HEAD 也会继续跟随指向 master 分支。
   ```

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
    ```

15. 查看某次提交修改的详细内容

    ```
    git show <commit-hash>
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

    4. git restore --staged的作用：取消暂存，同3

    ```
    git restore --staged <文件>
    ```

17. `git diff`: 可以查看和分析改动，包括查看两个文件之间、文件修改前后、暂存区中的文件和最新提交之间的差异

    ```
    git diff		 	#比较工作目录中的文件和暂存区的文件之间的差异
    git diff --cached	#比较暂存区中的文件和最新提交之间的差异
    git diff commit1 commit2 #比较两个特定提交的差异
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
    git log -- <file-path>		# -- 后面加上想要查找的文件的历史记录
    git checkout <commit-hash> -- <file-path>	# 单独给目标文件恢复到那次提交前的状态(已经add暂存过的)
    ```

24. 关于`git checkout` 和 `git rese`t 的版本回溯

    ```
    git checkout <commit-hash> -- <file-path>#只是把本地恢复成提交记录的状态，但是不会修改版本历史
    git checkout ./	#修改到最新的状态
    
    git reset <commit-hash>#重置到某次提交的状态，会影响版本历史、暂存区和工作目录。之前的记录都被清掉
    --soft：只重置 HEAD，不修改工作目录和暂存区。适用于将提交回退，但保留修改，以便重新提交。
    --mixed（默认）：重置 HEAD 和暂存区，但不修改工作目录。适用于将提交回退，同时将文件移出暂存区。
    --hard：重置 HEAD、暂存区和工作目录。所有未提交的更改都会丢失，非常危险，但适用于完全回退到某个提交的状态。
    ```

    

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

   **思路**：将每次下发或者保存的配置保存下来，在界面打开`settingDialog`的时候统一`emit`一遍信号更改设置界面

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

5. 软件卸载上报bug：在中控下发卸载指令后，执行卸载操作并增加执行卸载上报。中控下发的卸载 由强制卸载 和 非强制两种  强制是中控数据库直接把这个端的信息删了  非强制是端在收到卸载指令后  上报卸载 然后 中控再清除这个端的信息  

   **问题**：为什么不在组件terminal_details中，而在upgrade插件中。

   **原因**：detail是各种资料，不适合放在这个组件，放在upgrade插件下，一样通过ipc接收消息到插件。

6. 启动safed出现下面这样的错误，需要把timedTask的数据库删掉

   > 同11

   ```
   0x00005555556eff27 in CDBTimedTasksOper::disposal (this=0x55555625cce0 <sdbTimedTasksOper>,
   ```

7. 暂停、继续界面文案bug：

   **笔记**：

   1. 中控下发暂停，ui界面文案显示bug：接收到中控下发的暂停扫描时，ui界面文案修改

   2. 中控暂停逻辑：当中控下发暂停扫描->scan插件收到pause任务->scan插件给gjcz发送任务，同时给UI发送状态信息
   3. 界面暂停逻辑：UI界面下发暂停->scan_moudle向scan插件发送信息->scan插件接收到任务->给gjcz发送
   4. 界面恢复逻辑：scan_moudle接收scan插件收到信息，通过信号槽响应到UI界面

   > UI moudle层执行的各种操作都是通过插件(safed)层

   **问题**：

   1. 中控下发只给gjcz给暂停了，并没有给ui发送状态信息，导致gjcz暂停而字段没有更改。
   2. 如果说在插件部分分情况处理，没有办法判断哪一次是中控下发，哪一次是UI界面
   3. 想法：把UI界面的响应和中控的响应放一块，把界面的修改放到moudle发送的信号响应槽中。

   **解决**：

   1. UI点击信号槽：如果是暂停，那就给助手发送暂停，如果是继续，就给助手发送继续
   2. 助手收到消息向gjcz发送命令，同时向moudle发送响应
   3. moudle接收消息与UI发送信号槽，从而改变UI文案

8. 使用授权弹窗bug：当授权信息达到3次就会在下一次启动RJJH时弹窗

   **思路**：

   1. RJJH启动时都会向`safed`获取授权信息。`safed`在向RJJH发送数据时判断下是否超过三次，并加入一个判断字段。`RJJH`检测该字段来弹窗
   2. 弹窗复用`Warnning`弹窗，在`DLG_STATUS`里面加个授权弹窗字段，在`setStatus`中加入该`case`选项。
   3. 在`FramelessWindow`中，加入弹窗点击"更换中控"的信号槽，跳转到授权管理界面

   **笔记**:

   1. qt下的`moudle`工作原理：先向中控发送请求消息(通过助手),中控将对应的信息传递过来(通过助手的`subscribe`和`publish`)

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

   **解决**：

   1. 重写了病毒库升级，并使用`callbcak`，让病毒库升级可以用到其他组件的功能。
   2. 在强力查杀前先检查病毒库升级，待升级结束后再进行查杀任务。
   3. 修改了读重启文件，修复了RJJH那块的逻辑

10. 关于病毒库升级：需要在查杀前进行病毒库升级，所以目的把病毒库单独拿出来

    **思路**：

    1. 病毒库升级只留内部的`terminalUpgradeVirusLibraryCenterMode`处理信息
    2. 上报、通信UI在外层，在查杀那块调用
    3. 内层需要获取版本号等其他组件功能，使用`callback`回调函数，在`general_operator_impl`中传递给它

    **解决**：

    1. 在scan插件下单独封装一个病毒库升级类。scan_flow_controller所有需要使用的升级函数都在这里面
    2. taskExecutionBegins、notifyUIVirusLibraryUpgrade、detectingUpgradeClientConnectionStatus、terminalVersionInfoSync、taskExecutionCompleted、reportAction。这些是在插件直接拿出来用的。
    3. callback：getAuthKeySNCode、virusLibraryVersion、checkDetectionAuthorization、sendMsgToUI、asyncReport这些是要作为callback参数传递的

11. 定时任务和定时查杀导致safed崩溃bug

    **思路**：dispose函数崩溃？加载不出来。读取SQlite数据库的问题

    **解决**：Sqlite表中有8列数据，而实际需要提取的只有7列。所以循环末尾加`index++`跳过最后一个

    > 参考terminal_details_cache.cpp  129实现sqlite数据库读取方式

12. 主防编译时问题：

    **问题**：未找到目标静态库时使用系统自带库的bug导致主防异常

    **思路**：

    1. `find_library`：查看其用法，尝试只编译静态库，如果找不到静态库返回异常
    2. 把那两个库使用set的用法编译

    **解决**：第一种方式，加上`NO_DEFAULT_PATH`字段即可限制只在目标路径下寻找
    
13. 强力查杀之病毒库

    **问题**：

    1. 连续下发两次强力查杀，病毒库依旧升级，且中控取消不掉
    2. 病毒库升级策略：先向中控申请、再向外网、最后如果都升级不了就用本地的
    3. 中控取消查杀后不要后续的关机操作
    4. 查杀结束后的弹窗，改为提示弹窗，倒计时为60秒

    **解决**：

    1. 在病毒库升级加个判断逻辑，如果队列中有强力查杀就不执行本次病毒库升级。第二次的查杀也会在任务队列中等待清除

    2. 关于病毒库版本上报：修改了病毒库升级逻辑；由于会重启，所以等待重启后的版本号上报即可。

    3. `scanUiChange`的几个状态：

       - `SCAN_FINAL_RES`没有病毒的情况下
       - `SCAN_VIR_SHOW_AND_CLEAN`查到有病毒且设置了自动清理
       - `SCAN_VIR_SHOW`查到有病毒的情况下，展示病毒

       在scanmodel中获取取消操作数据，在scanUIChange事件中设置isScanCancel。最后在事件处理器中判断是否为手动取消，如果取消那么就不执行关机或者重启。

    4. 在手动取消任务执行前，先将强力查杀标记文件删除，这样下次就不会执行全盘查杀操作。
    
14. 查杀查出病毒后的日志上报

    **思路**：

    1. `ScanCount::recordScanDetails`  617行的 627有问题。使用bt 查看哪里改动了clean_result字段

    2. 扫描停止就记录日志？还是说处理后再记录？

    3. 67行`virusResult`是扫描结束后自动触发sacnstop

       285行`scanAction`是手动取消后触发scanstop

       `scanstop`中有各种上报和记录日志，其中就有`recordScanLog`

       `recordScanLog`中有记录日志和记录详细日志。但是这是没有处理的，所以处理数量为0，且详细日志中显示未处理

    4. `recordProtectionLogs`它的参数param的 `isUpdate`，用于刷新上一次的日志

    **解决**：

    1. 上报实际上上报了两次，stopScan上报一次，等到界面下发结束标志之后又上报一次。标记参数iosupdate为true，只是刷新上一次保存在sqlite中的记录。
    2. 问题在于处理威胁的过程中，传递的action是错误的。
    
15. UKey授权bug：

    **问题**：点击激活UKey，RJJH崩溃

    **解决**：发现点击激活也走的是刷新信号槽里面的信号槽。问题在于safed传递请求时判断错误，导致所有的回复都走的是默认第一个(枚举默认为0)。执行了两次eventLoop.quit()导致崩溃。
    
16. 注册界面文案修改

    **问题**：点击激活，若直接收到safed返回的信息，则提示激活成功。但是如果中控开启填报信息，则需要等待提交完信息后，待safed回复了信息才会提示激活成功。

    **思路**：当填报信息界面开启时，发送信号，在authManage中将计时器停止

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

9. `ifstream` 和 `ofstream` 对象在它们的作用域结束时，都会自动释放资源，包括关闭文件流。

   > 这是因为 C++ 中的文件流类（如 `ifstream` 和 `ofstream`）遵循 RAII（Resource Acquisition Is Initialization）的原则。RAII 确保当对象超出其作用域时，自动调用其析构函数来释放资源，包括关闭文件流。

10. 使用map容器进行迭代器遍历时，`first`和`second`是成员变量而不是函数，不需要加括号(因为map使用`pair`对组作为成员，只有map使用`first`和`second`)

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

19. `try catch`：当代码中涉及文件的操作必须加上`trycatch`，避免因为文件相关的问题导致整个程序崩溃

    ```
    throw std::runtime_error("Unable to open files:");  #抛出运行时错误
    ```

20. 使用JSON传递string数据，不能直接解JSON数据的时候`toInt()`,必须先`toString()`再`toInt()`;

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

40. `enum class`是C++11的强枚举类型

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

41. 对于原子类型，线程安全的方式读取该值使用`load()`函数;

    `load()` 函数的作用是读取原子变量的当前值并返回。在多线程环境中，使用 `load()` 可以确保读取的值是线程安全的，即便其他线程可能同时在修改该变量。

42. C++标准库的**承诺-未来机制**`std::promise`和 `std::future`，用于在线程之间传递值或状态，尤其是在异步编程中协助传递结果。实现在两个线程间同步任务完成状态

    > 有点像Qt中的eventLoop，事件循环机制

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

43. 关于`std::nothrow` 

    ```
    int* p = new(std::nothrow) int[10000000000]; // 请求分配大量内存
    ```

    `std::nothrow` 是 C++ 标准库中的一个常量，用于控制内存分配失败时的行为。通常在使用 `new` 操作符进行内存分配时，分配失败会抛出 `std::bad_alloc` 异常。而使用 `std::nothrow`，可以使 `new` 操作符在分配失败时返回 `nullptr`，而不是抛出异常。

44. 关于回调函数：回调函数（Callback Function）是一种通过函数指针、函数对象或 lambda 表达式传递到另一函数中的 **可调用对象**。回调函数可以理解为一个“回去调用”的函数，即某个函数 `A` 把函数 `B` 的地址传递给另一函数 `C`，然后 `C` 在适当的时机调用 `B`。

    > 个人理解就是把函数指针/lambda/函数对象作为函数参数，在该函数内部可以执行传来的作为参数的函数

45. 关于**发布-订阅模式**：使用了回调函数机制，订阅方使用`subscribe`将执行函数注册，插入到订阅-发布类的map中，当调用`publish`时，就会将执行函数发布/执行

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

46. **闭包**：闭包是 **函数和它的捕获上下文的组合**，这个上下文允许闭包函数在被调用时访问定义它时所在作用域中的变量，即便该作用域已超出生命周期。

    举例：在 C++ 中，lambda 表达式可以捕获当前作用域中的变量，因此 lambda 可以作为闭包的一个具体实现。

    1. 按值捕获 (`[=]`)：复制变量值到闭包中。
    2. 按引用捕获 (`[&]`)：捕获变量的引用，使得闭包可以访问和修改原变量。
    3. 捕获 `this` 指针 (`[this]`)：捕获 lambda 所在对象的 `this` 指针，以便在 lambda 中访问对象的成员。

47. 句柄：**句柄（handle）** 是一种特殊的指针或引用，通常指一个**可以访问或操作其他对象的“引用”或“指针”**,用来标识和管理资源的访问。句柄通常不直接提供对资源的底层访问，句柄广泛用于资源管理，比如文件、内存、线程、网络连接等。句柄本质上是一种间接引用，指向一个资源或对象而不暴露其内部实现细节。

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

48. 回调函数和句柄的区别

    1. 句柄就是一个对象指针。通过传递对象指针可以使用该对象的成员函数
    2. 回调函数就是一个函数指针。通过该函数指针可以随时使用该函数。

49. 向前声明和使用头文件的区别

    1. **减少依赖**：使用向前声明，你不需要直接包含其他头文件。这可以减少类之间的直接依赖，有助于减少编译时间和避免循环依赖。例如，在 `Worker.h` 中向前声明 `Manager`，这样你可以避免包含 `Manager.h`，从而减少编译开销。
    2. **完整定义**：如果你需要使用类的具体实现（如访问成员、调用函数、创建实例等），则必须包含该类的头文件，以便编译器了解该类的具体结构。**向前声明只让编译器知道类型的名称，但不提供任何方法或成员的信息**。
    3. **避免循环依赖**：在两个类相互引用的情况下，使用向前声明可以打破循环依赖。例如，`Manager` 和 `Worker` 互相依赖时，使用向前声明可以避免直接包含彼此的头文件。

50. 循环依赖问题(当class a 导入class b的头文件，classb 也导入class a的头文件时)，需要使用前置声明。不然编译器会不知道先编译哪个

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

51. 智能指针循环依赖问题

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

52. 句柄传递`this`的智能指针问题

    1. 如果在母类中直接传递`this`裸指针，一旦母类析构，则在使用该`this`指针的类就会造成**指针失效**

    2. 如果使用`this`创建`make_shared`后传递，会将 `this` 指针当作一个新对象的起点进行管理，并且会在 `shared_ptr` 的引用计数归零时自动删除该对象。这对一个已经在其他地方管理生命周期的对象（例如，栈上或由另一个指针管理的对象）是危险的，因为它会导致对象的**双重销毁**。

       >  但是我使用的是make_shared，重写了构造函数。相当于把主类的成员共享对象指针++。当我析构主类的使用句柄的对象时，一样会使指针--，在类里面的句柄所以没问题。

    3. 让母类继承 `std::enable_shared_from_this`，并在对象本身被 `shared_ptr` 管理时使用 `shared_from_this()` 来安全地生成 `shared_ptr`。

       ```c++
       #只适用于下面这种，已经被shared_ptr管理的对象中使用，否则会抛出 std::bad_weak_ptr 异常
       std::shared_ptr<MyClass> p = std::make_shared<MyClass>();
       p->show();  // 在成员函数中使用 shared_from_this
       ```

53. `shared_ptr`作为函数参数传递：

    1. **`std::shared_ptr<T>`**：只能传递 `shared_ptr` 类型对象，除非使用 `std::make_shared` 生成 `shared_ptr`或者`shared_from_this()` 将类本身被创建时的`shared`指针传递，否则不能传递临时对象(指针)。
    2. **`std::shared_ptr<T>`**：传递 `shared_ptr` 意味着函数将共享该对象的所有权。每次传递 `shared_ptr` 参数，引用计数会增加，直到离开作用域或显式释放才会减少

54. 关于Qt界面先`init()`后再`show()/exec()`的问题：，在创建 `QDialog` 对象并调用 `init()` 后，`init()` 函数中的代码会立即执行，因为这部分代码并不依赖事件循环。只有那些需要事件循环来触发的操作（如计时器、动画、信号槽的触发等）才不会运行

55. 回调函数`bind`第二个参数是对象类型或者指针类型的区别

    ```
    auto boundFunc = std::bind(&CupdateVirusLibHelper::UsendMsgToUI, &m_updateVirusLib, std::placeholders::_1);
    当你通过 boundFunc 调用时，std::bind 会自动将 m_updateVirusLib（即指向对象的指针）作为第一个参数传递给 UsendMsgToUI，而其他参数则通过 std::placeholders::_1 来传递。
    ```

    1. 当你传递 **对象的引用** 给 `std::bind` 时，回调函数会直接通过该对象引用访问对象的成员函数，这里不需要空指针检查，因为引用总是指向一个有效对象
    2. 当你传递 **对象的指针** 给 `std::bind` 时，回调函数会通过该指针来访问对象的成员函数。需要特别注意的是，指针可能为 `nullptr`，因此你需要在回调函数中检查指针是否有效
    3. **`std::bind`** 中的 **`std::placeholders`** 需要根据你要绑定的成员函数的参数数量来指定对应的占位符。每个占位符（如 `std::placeholders::_1`, `std::placeholders::_2`, ...）都会对应于你绑定函数的一个参数。

56. 普通函数与成员函数取地址的区别

    1. 对于普通函数，函数名本身就可以作为一个指向函数的指针

    2. **成员函数** 是与特定的对象实例绑定的，因此它的行为与普通函数不同。成员函数指针不仅仅是函数的地址，还包含了**对象的上下文**，即它需要一个对象实例来调用

    3. 在成员函数指针中，`&MyClass::processMessage` 表示获取 `processMessage` 成员函数的指针。这里的 `&` 是必要的，它用于表示 **成员函数的指针**。这是因为成员函数指针不仅仅是函数的地址，还需要知道哪个对象来调用该函数。

       > **成员函数指针** 是一种特定类型的指针，它不仅指向函数本身，还需要知道哪个对象来调用该函数。
       >
       > **普通函数指针** 直接指向函数的实现，可以直接用函数名表示。

57. 类的成员对象，在本类的构造时对其使用初始化列表

    1. **成员对象的构造必须在初始化列表中进行**。
    2. 构造函数体 **不会重新初始化成员对象**，它只会在初始化列表中调用成员对象的构造函数

58. 问题：当你通过 `include_directories` 添加了某些头文件路径，这些头文件对应的 `.cpp` 文件是否需要在 `add_library` 中显式地列出，或者是否可以仅通过头文件路径来引用这些源文件

    答：

    1. 如果 `.cpp` 文件是你自己定义的，并且它们需要被编译成库的一部分，你必须将这些 `.cpp` 文件列出在 `add_library` 中。如果你仅通过 `include_directories` 引用了其他目录的头文件，而没有将对应的 `.cpp` 文件包含到 `add_library` 中，编译器将找不到这些源文件并报错
    2. 如果你的代码依赖于外部库（比如系统库或第三方库），你可以通过 `target_link_libraries` 来指定这些库

51. 通过逗号分割提取字符串

    ```c++
    std::istringstream stream(ipList);
    std::string ip;
    while (std::getline(stream, ip, ',')) {
           // 去掉可能存在的空格
           ip.erase(std::remove(ip.begin(), ip.end(), ' '), ip.end());
           if (!ip.empty()) {
               ips.push_back(ip);  // 将非空 IP 添加到结果中
           }
    }
    ```

    1. `std::istringstream` 是 C++ 中的一个输入流类，它允许你将一个字符串作为流进行处理。通过它，你可以像读取其他输入流（如 `std::cin`）一样读取字符串内容。

    2. `std::getline()`的几种用法：

       1. 从输入流读取一行数据

          ```
          std::getline(std::cin, line); // 默认以换行符分隔
          ```

       2. 从输入流读取一行数据并指定分隔符，默认为换行符（`\n`）。此处使用`istringstream`模拟流数据

          ```c++
          std::string input = "apple,banana,orange,grape";
          std::istringstream stream(input);
          std::string token;
          
          // 使用逗号作为分隔符
          while (std::getline(stream, token, ',')) {
              std::cout << token << std::endl;
          }
          ```

       3. 从文件中逐行读取

          ```c++
          std::ifstream file("example.txt");
          std::string line;
          
          // 逐行读取文件内容
          while (std::getline(file, line)) {
              std::cout << line << std::endl;
          }
          ```

52. 关于槽函数slot的访问权限(public protected private)

    1. signal没有访问权限限制。
    2. **访问权限只对外部代码的直接调用有效**，对信号-槽机制无影响(只要是槽函数就可以进行信号-槽机制的连接)。
    3. 比如：private slot只能进行信号槽机制的连接，不可以在外面直接像调用成员函数一样调用该槽函数

    | 特性                 | `private slots`              | `protected slots`            | `public slots`             |
    | -------------------- | ---------------------------- | ---------------------------- | -------------------------- |
    | **访问权限**         | 类本身和友元可以访问         | 类本身、子类和友元可以访问   | 外部代码可以访问           |
    | **典型用途**         | 内部逻辑实现细节，不对外开放 | 需要子类继承、扩展的内部逻辑 | 提供对外接口，同时响应信号 |
    | **子类访问**         | 不可以                       | 可以                         | 可以                       |
    | **外部代码直接调用** | 不可以                       | 不可以                       | 可以                       |

53. Qt的Warnging弹窗，执行exec()模态框时，会阻塞当前界面，直到用户点击确定(返回`QDialog::Accepted`)或者关闭/取消(返回`QDialog::Rejected`)

    ```
    if (wDlg.exec() == QDialog::Rejected){
        QApplication::exit();
    } else {
        emit slotStartAuthManager();
    }
    ```

54. `dlopen`、`dlsym` 和 `dlclose` 是 Linux 动态链接库相关的函数，用于**在运行时加载和使用共享库**（动态库）。它们定义在 `<dlfcn.h>` 头文件中，属于 POSIX 标准的一部分

    1. `dlopen`：用于加载共享库，并返回一个句柄以供后续操作

       ```
       void *dlopen(const char *filename, int flag);
       
       #参数：
       filename：动态库的路径。如果是绝对路径，则直接加载。如果是相对路径，则相对于当前工作目录查找。
       			如果传递 NULL，则返回默认程序的全局符号表句柄。
       flag：控制库的加载行为，可使用以下标志：
       	RTLD_LAZY：延迟解析符号，只有在真正调用时才会加载。
       	RTLD_NOW：立即解析所有未定义的符号。
       	RTLD_GLOBAL：将库的符号放入全局符号表，使其对后续加载的库可见。
       	RTLD_LOCAL：符号仅对当前加载的库可见（默认行为）
       	
       #案例
       void *handle = dlopen("libexample.so", RTLD_LAZY);
       if (!handle) {
       	#可通过 dlerror() 获取详细错误信息
           fprintf(stderr, "Error: %s\n", dlerror());
       }
       ```

    2. `dlsym`：从动态库中获取指定符号的地址，通常用于获取函数指针或变量地址

       > 在插件接口cpp中：`PLUGIN_EXPORT std::unique_ptr<IPlugin> createPlugin()`

       ```
       void *dlsym(void *handle, const char *symbol);
       
       #参数：
       handle：dlopen 返回的句柄。
       symbol：需要查找的符号名称（通常是函数名或全局变量名）。
       
       #案例：
       using CreatePluginFunc = std::unique_ptr<IPlugin>();
       CreatePluginFunc *createPlugin = reinterpret_cast<CreatePluginFunc *>(dlsym(handle, "createPlugin"));
       if (!createPlugin) {
             std::cerr << "Error getting createPlugin function: " << dlerror() << std::endl;
              dlclose(handle);
              return false;
       }
       ```

    3. `dlclose`：关闭动态库，并释放相关资源

       ```
       int dlclose(void *handle);
       
       #返回值：
       成功：返回 0。
       失败：返回非零值
       ```

    4. `dlerror`：返回最近一次 `dlopen`、`dlsym` 或 `dlclose` 调用的错误信息。

       ```
       const char *dlerror(void);
       
       #效果：
       成功调用 dlopen、dlsym 或 dlclose 后，会清除错误状态。
       返回值是一个描述错误的字符串，或 NULL 表示没有错误。
       ```

55. **事件**是如何定义、触发、使用的

    1. 事件拦截：通过事件过滤器，某个对象可以捕获并处理其他对象的事件；

       ```
       installEventFilter(QObject *filterObj)
          installEventFilter() 是目标对象的方法，用于为目标对象安装一个事件过滤器（filterObj）。
       	一旦安装了事件过滤器，filterObj 的 eventFilter() 方法会接收目标对象的所有事件。
       
       #作用范围：
       1. 目标对象本身的事件：包括鼠标事件、键盘事件等。
       2. 目标对象的子对象的事件（如果 filterObj 安装在父对象上）。
       ```

       ```
       eventFilter(QObject *watched, QEvent *event)
       	eventFilter() 是过滤器对象（filterObj）的方法，用于处理目标对象传递的事件。当目标对象接收到事件时，Qt 会先调用 filterObj 的 eventFilter() 方法。
       	
       #参数
       1. watched：目标对象（即安装了当前过滤器的对象）。
       2. event：发生的具体事件（如鼠标事件、键盘事件等）。
       #返回值
       1. true：表示事件已被过滤器处理，目标对象不会再处理该事件。
       2. false：事件未被处理，会继续传递到目标对象。
       ```

       ```
       #步骤：
       1. 调用 installEventFilter() 在对象上安装过滤器，设置哪个对象的事件需要被过滤器对象监控。
          
       2. 重载过滤器对象的 eventFilter() 方法处理事件，eventFilter() 是事件过滤器的具体实现，用于拦截和处理事件
       ```

       ```c++
       #include <QApplication>
       #include <QWidget>
       #include <QEvent>
       #include <QDebug>
       
       # 示例：当widget的鼠标被按下时，将会进入eventFilter处理中
       class MyFilter : public QObject {
       protected:
           bool eventFilter(QObject *watched, QEvent *event) override {
               if (event->type() == QEvent::MouseButtonPress) {
                   qDebug() << "Mouse button press detected on" << watched;
                   return true; // 阻止事件继续传递
               }
               return QObject::eventFilter(watched, event);
           }
       };
       
       int main(int argc, char *argv[]) {
           QApplication app(argc, argv);
       
           QWidget widget;
           MyFilter filter;
       
           widget.installEventFilter(&filter);
           widget.show();
       
           return app.exec();
       }
       ```

    2. 自定义事件

       ```
       #步骤
       1. 创建自定义事件类，继承自 QEvent。
       2. 使用 QCoreApplication::postEvent 或 QApplication::sendEvent 发送事件。
       3. 重载 event() 函数处理事件。
       ```

       ```c++
       #include <QEvent>
       #include <QApplication>
       #include <QWidget>
       #include <QDebug>
       
       // 自定义事件类型
       class MyCustomEvent : public QEvent {
       public:
           static const QEvent::Type EventType;
           MyCustomEvent() : QEvent(EventType) {}
       };
       
       const QEvent::Type MyCustomEvent::EventType = static_cast<QEvent::Type>(QEvent::User + 1);
       
       class MyWidget : public QWidget {
       protected:
           bool event(QEvent *event) override {
               if (event->type() == MyCustomEvent::EventType) {
                   qDebug() << "Custom event received!";
                   return true;
               }
               return QWidget::event(event);
           }
       };
       
       int main(int argc, char *argv[]) {
           QApplication app(argc, argv);
       
           MyWidget widget;
           widget.show();
       
           // 发送自定义事件
           QCoreApplication::postEvent(&widget, new MyCustomEvent());
       
           return app.exec();
       }
       ```

    3. 重载特定事件处理函数：每个控件都有一组专门的事件处理函数，你可以重载这些函数以实现自定义行为

       ```
       #常用事件处理函数
       鼠标事件：mousePressEvent、mouseReleaseEvent、mouseMoveEvent。
       键盘事件：keyPressEvent、keyReleaseEvent。
       窗口事件：resizeEvent、paintEvent。
       焦点事件：focusInEvent、focusOutEvent。
       ```

       ```c++
       #include <QWidget>
       #include <QMouseEvent>
       #include <QDebug>
       
       #示例：鼠标事件处理
       class MyWidget : public QWidget {
       protected:
           void mousePressEvent(QMouseEvent *event) override {
               if (event->button() == Qt::LeftButton) {
                   qDebug() << "Left mouse button clicked at:" << event->pos();
               }
           }
       };
       ```

    4. 重载 `event()` 函数：`event()` 是所有事件的**入口点**。如果需要统一处理某些事件类型，可以重载 `event()`。

       ```c++
       #include <QEvent>
       #include <QDebug>
       
       #示例：拦截特定事件
       class MyWidget : public QWidget {
       protected:
           bool event(QEvent *event) override {
               if (event->type() == QEvent::KeyPress) {
                   qDebug() << "Key press detected!";
                   return true; // 阻止进一步处理
               }
               return QWidget::event(event); // 调用基类处理
           }
       };
       
       ```

    5. 优先级顺序

       ```
       1. eventFilter() 处理最优先。
       2. 如果过滤器不处理，则进入 event()。
       3. 如果 event() 不处理，则进入特定事件处理函数（如 mousePressEvent）。
       ```

    6. 总结

       ```
       简单事件处理：重载特定事件处理函数。
       复杂事件处理：重载 event()。
       自定义事件：扩展 QEvent，并通过 postEvent 或 sendEvent 发送。
       全局监听：使用事件过滤器
       ```

56. `event->type() > QEvent::User` 的含义：`QEvent::User` 是一个事件类型常量，用于标识用户自定义事件的起始范围。它的值通常定义为一个整数（如 1000），大于这个值的事件类型都属于用户定义事件。

57. `QApplication::instance()`获取当前正在运行的 `QApplication` 对象

    ```
    QApplication::instance()->installEventFilter(this);
    ```

58. `QEventLoop`:主要用于执行事件循环，以便处理不同类型的事件（如用户输入、定时器事件、信号和槽的调用等）

    1. **事件循环**是 Qt 应用程序中不可或缺的一部分，它是一个等待并处理事件的机制。Qt 中的事件循环通过 `QEventLoop` 类来实现，基本的事件循环由 `QCoreApplication` 或 `QApplication` 管理。每个 Qt 程序都需要一个事件循环来处理用户输入（如鼠标点击、键盘输入），系统事件（如定时器超时），以及其他组件之间的信号和槽的调用。

       ```C++
       #`QApplication` 的事件循环
       int main(int argc, char *argv[])
       {
           QApplication app(argc, argv);  // QApplication 启动事件循环
           MainWindow w;
           w.show();
           return app.exec();  // 进入事件循环，等待处理事件
       }
       ```

       ```c++
       #`QEventLoop` 的事件循环
       void MyClass::someFunction()
       {
           QEventLoop loop;
           connect(someObject, &SomeObject::someSignal, &loop, &QEventLoop::quit);
           
           // 执行一些操作...
           
           loop.exec();  // 等待直到 someSignal 被发射，事件循环会结束
       }
       ```

    2. `QEventLoop` 会在事件循环中运行，等待并处理事件。它的主要工作方式是**阻塞等待**，直到**事件队列**中有事件可以处理为止。或者直到 `quit()` 被调用

       - 用户输入事件（鼠标、键盘等）
       - 系统消息（如文件系统的更改）
       - 定时器事件
       - 信号与槽的触发
       - 自定义的事件（通过 `QCoreApplication::postEvent` 发送）

    3. 信号槽与事件循环：

       1. 信号触发时，信号会被放入事件队列。
       2. 事件循环会在处理完当前的事件后继续处理队列中的信号。
       3. 如果信号与槽的连接是 **队列连接**，槽函数会在事件循环的下一次迭代中执行，而不会阻塞当前的槽函数。
       4. 如果信号与槽的连接是 **直接连接**，槽函数会同步执行（通常是在信号发射的地方

59. 如果在使用lambda的情况下使用信号槽，需要解绑时，需要保存lambda函数的实例。

    ```
    // 定义一个 lambda 函数
        auto lambdaSlot = [=]() {
            label->setText("Button clicked!");
        };
    ```

60. 可以发射成员对象里的信号

    ```
    emit m_authManager->sigRegistInfoTable();
    ```

61. 信号和槽函数都使用 `void` 类型的原因：

    1. 设计如此，**异步事件处理**不需要关心返回值
    2. 信号槽机制，信号只管发射，可以有多个槽函数应答；同理，多个信号可以只有一个槽函数应答。

62. 操作系统、架构、指令集、内核之间的关系

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

63. md5:MD5（Message-Digest Algorithm 5）是一种广泛使用的加密哈希函数，能够将任意长度的输入数据（通常称为消息）转换为固定长度的输出，具体来说是 128 位（16 字节）长的哈希值。

64. 计算md5的值

    ```
    md5sum <库名>
    ```



# 未完成笔记

### 1. 项目部分

2. qrc文件和rcc文件    gcc？

3. titlebar？如何将titlebar单独的放到别的界面中

4. RJJH中的event文件夹？cur_user干什么的？归纳下qy_ui（UI层）的具体构造

5. 看下Frameless中的720行事件过滤器

6. 针对不同版本(开发环境)的右键库：

   ```
   #add_subdirectory(libsource/nautilus_scan)
   #add_subdirectory(libsource/caja_scan)
   #add_subdirectory(libsource/peony_scan)
   ```

7. 如何修改ubuntu下的权限，省的每次都得用sudo

8. 查看`.clang-format`如何设置，规格化工具

9. 使用VScode插件

   ```
   Clang-Format  代码格式化插件
   koroFileHeader 注释文件和函数的快捷方式
   Better Comments 注释分类，不同注释不同颜色
   gitlens 展示每行代码的提交人、提交时间
   git history 展示git历史提交记录
   Tabnine 人工智能
   remote development 远程开发插件
   Color Highlight 显示代码中关于颜色的代码直接显示颜色
   ```

10. qt的moudle层如何与safed通信的？safed如何与中控通信的？

   > RJJH下面的ipc文件夹下的IIPCBaseModelInterface，负责send和receive助手之间的数据

11. 执行文件(SAFED ZDFY GZCZ)和包(.so)的分布情况

12. 动态库之间如何相互调用，动态库是如何使用的

13. plugin.conf的message的Key，在哪个地方初始化。如果key没有卸载config文件里面会如何

   ```
   #下面两句如何实现的?
   std::string msgType = BundleHelper::getBundleAString(pIn, JYMessageBundleKey::JYMessageType, "");
   std::string msgData = BundleHelper::getBundleBinary4String(pIn, JYMessageBundleKey::JYMessageBinValue, "");
   ```

11. 线程类(`ThreadWrapper`)是如何实现的？如何通过集成该类就可以实现线程的功能？线程池如何运转

12. `core`与组件和插件之间的关系，画图梳理->类图建立

13. 关于RJJH里面的自定义事件处理：

    1. 在**event**文件夹中创建自定义事件

    2. 在scan**Model**中接收信息、创建事件并通过发送信号`signal`将事件传递到主界面**UI**的Framless中

    3. **UI**通过槽函数，`postEvent`在本对象Framless中发送事件

    4. Framless重写了`eventFilter`事件处理器后，初始化构造中通过`installEvent`绑定自己本身

       ```
       QApplication::instance()->installEventFilter(this)
       ```

### 2. 工作部分

#### 2.1 正在进行

1. 后续暴力破解需要合并到主防，且前端有可能要重构。后续可能要我参加前端的操作，下月主要解bug，脚本和界面优化：颜色、`tableview`展示设计整体框架。

2. 12月任务

   |                           任务名称                           | 优先级 | 时间  | 截至  |
   | :----------------------------------------------------------: | :----: | :---: | ----- |
   | 查杀后，临近查杀结束时点击暂停，待任务自动结束后，再次查杀，偶现弹窗提示有任务 |   高   | 0.5天 | 12.13 |
   |                界面颜色，图标等适配为最新版本                |   高   |  2天  | 12.13 |
   |                          网络白名单                          |   高   |  3天  |       |
   |                      所属用户及权限修改                      |   中   |  1天  |       |
   |                  卸载后删除残留与旧版本处理                  |   中   |  1天  |       |

   1. - [x] 任务一：

      1. 原因：在stop停止信号未发送到界面时，点击暂停，之后界面收到stop数据切换界面。但是暂停任务继续执行，又向界面发送暂停信息。导致后续再执行查杀，显示已有查杀正在进行。
      2. 查看是否是scan_flow中，暂停和停止的信号冲突了，考虑加一个判断

   2. - [x] 任务二：

      1. 界面颜色：根据给的切图，oem/jyn文件夹下是景云版本的要替换的图片。

         > 复制blue不行，是否是因为代码中写的固定为grey了

      2. 图标：在gen脚本中替换jy产品的log为log64-blue即可

      3. 默认的版本样式为RJJH/skins文件夹下，如果是新添加的图片需要在widget.qrc文件中加入。

      4. 使用贴牌脚本oem/gen_rcc.sh生成对应的rcc文件，复制到对应安装目录下即可改变样式
      
         > 注意使用脚本时，先修改本rcc路径为自己本地的
      
      5. 在rjjh_main函数中OEMChooser::Load() --> qproduct_rcc_file --> get_oem，通过etc/version.ini中的版本号来执行不同的贴牌rcc

3. - [x] 界面重新配置


   1. 图标为logo_64_grey.png。界面左侧颜色：上main_left.png 下main_left_bottom.png
   2. 托盘图标位置在RJJH/main_window中对SystemTray的设置上
   3. 桌面图标和文件管理中心的图标在 RJJH.desktop中
   4. 升级界面图标更改
   5. 产品名称更换：在desktop里面更改(和关于我们里面的一样)，实际应该在打包脚本中修改

4. - [x] bug修改


   1. 病毒库版本上报，看是什么问题

      **解决**：无需更新时没有上传旧的版本号导致传递的版本号为空

      **隐患**：

   2. 试用授权三次后，右键还可以查杀

      **解决**：在执行扫描的时候加上判断，是否试用授权超过三次，超过则不执行

5. - [ ] 隔离区的恢复和清除：在`scan_task`上增加删除和恢复的任务处理。

   
   思路：
   
   1. 看RJJH那块如何给safed处理的，直接调用那块的处理方式即可。
   2. 删除和恢复都新建scan任务，和get写到同一个/或者不写同一个。
   3. 在safed中看能否在safed中直接实现删除/恢复功能，然后同步/上传/上报数据到UI界面和中控
   4. 把需要彻底删除的文件统计到VirusIsolateRemoveInfoList中（scan_flow590行），然后传递给scan_flow中的virusScanProcessIsolateThoroughDelete进行删除。
   5. 同样，把要恢复的文件统计到VirusIsolateRecoverInfoList中（521行），然后传递给virusScanProcessIsolateRecover进行恢复
   6. 看一下界面到这块的逻辑，是否还需要更新界面等操作，是否需要跟界面交互

6. - [ ] 网络防护增加逻辑：

   1. 检测时间改为1秒内5次
   2. 界面禁用网络防护时，把配置文件清除掉(把之前封禁的ip去除、把hosts.deny清除掉)
   3. 日志记录里面二级分类为`-`，不是未知。
   4. bug: 
      1. 仅记录时，来一个攻击日志就记录一次。网络防护那块也是
      2. 弹窗只第一次有效，后续再有弹窗就无效了，不弹窗了
   5. 增加白名单：插件增加白名单功能，且界面增加白名单窗口。

   > 下一步先看隔离区的数据展示如何实现的，先解决5，为自己实现新窗口打铺垫



#### 2.2 暂时搁置

1. 查看麒麟系统下的右键扫描是否可ls

2. 根据不同的版本使用不同的右键库

   2. 在bvin.cmake中解注释对应的库

      ```
      #add_subdirectory(libsource/nautilus_scan)
      #add_subdirectory(libsource/caja_scan)
      #add_subdirectory(libsource/peony_scan)
      ```

   3. JYINSTALL脚本中的，copy_scan_menu()函数中，负责将那三个右键库复制到对应系统目录下
      
      1. 对文件右键打开程序，实际上就是执行 ./JYNRJJH2 --/home (文件名)

3. 打包脚本bug：

   **问题**：

   1. 产品图标和产品名称不统一
   2. 脚本打包三个版本，有一个版本失败，找出原因
   3. Python脚本和shell脚本。如何将shell脚本转Python脚本

   **思路**：

   1. 根据老脚本-贴牌配置脚本，使用python实现新脚本，注：单独放一个文件夹中

4. 防卸载bug:

   **问题**：

   1. terminal_detail_component中接收中控密码并传递给界面model
   2. terminal_info_model接收助手传来的密码信息，并通过信号槽传递给auth_model中保存密码
   3. 在uninstall脚本中，卸载前判断是否存在密码文件，若有，则解密提取密码，让其输入





### 3. 代码部分

1. 为什么头文件有的需要加上文件夹前缀

2. 为什么只能传递IGeneralOperator，而不能传递CGeneralOperatorAdapter？

3. 英语get、have、take常用口语的用法

   ```
   If you get something, then that means that the item was given to you. If you take something, then that means that you took it yourself and no one else was involved in you getting possession of the item. To get is to receive, to take is to retrieve.
   ```

4. in、for、of、to、等常用介词的使用方式

7. python脚本的实现顺序

   ```
   1. main.py
   2. handle_opts.py -> batchProcess函数中对IPKTS对应的包的参数，执行newpacket
   3. new_packet.py ->	NewPacket的__init__(self, **kwargs)初始化，并执行createPacket()
   				 -> savePacket()保存deb包，使用dpkg -b，并将打的包cp到包库中
   4. handle_opts.py -> Result()对象，在析构的时候执行打印结果
   ```

6. 麒麟系统上文件夹远程连接服务器：在文件夹索引上方输入`ftp://192.168.1.6`即可远程连接

   麒麟系统从远程文件夹拉取文件时，需要拉到左侧"桌面"文件夹下，不可以直接拉到桌面，否则报错

   麒麟系统实际上是linux的另一个发行版，与ubuntu的命令类似，安装时也使用dpkg

7. git 清除未跟踪的文件

   ```
   git clean -f    #清除未跟踪的文件
   git clean -fd	#清理未跟踪的文件和目录
   git clean -fdx	#只删除未跟踪的目录
   git clean -n	#模拟清理（预览将删除哪些文件）
   ```

8. 本地信号槽

   ```
   connect(this,&AuthManger::function,[](){})
   ```

9. `git pull --rebase`

10. `make  -j8`  编译多线程

11. `git restore --staged <文件>...`

12. git stash找到删除的记录

    ```
    git fsck --lost-found		#列出最近删除的stash commit记录
    
    git stash apply <commit-hash>	#恢复到对应stash记录下
    ```

13. UI层，main_window才是主界面，其在构造的时候初始化了FramelessWindow。

    rjjh_main是RJJH进程的入口，只有main才会执行app.exec事件循环
    
14. 关于Frameless中的事件过滤器`eventFilter`:（以scanModel为例）

    1. scanmodel在收到safed传来的扫描过程中的异常事件，将会通过信号槽`emit`将该`event`传递到Frameless

    2. Framless在槽函数中执行`postEvent`，发送事件，交给事件过滤器`eventfliter`处理

       ```
       QApplication::postEvent(this, pEvent);
       ```

    3. 事件处理器将会依次处理本轮事件队列，直到下一轮(app.exec)
    4. 事件event的总类(自定义事件)在event文件夹下

15. Qt图片路径：使用了 `:/` 前缀，表示这是一个 Qt 资源路径；资源文件必须在项目的资源文件（`.qrc` 文件）中声明。

    ```
    QIcon jy_icon(":/skins/icon/logo_64_blue.png");
    app.setWindowIcon(jy_icon);
    ```

16. `QAbstractListModel` 是 Qt 框架中的一个抽象基类，用于实现基于列表的数据模型。它继承自 `QAbstractItemModel`，并简化了仅需要一维数据结构（如列表或数组）的模型实现。

17. `QTableWidget`和`QTableView`的区别：

    1. `QTableWidget`基于项（Item-based）的表格控件，适合简单的表格数据展示，当数据量较大时，性能不如 `QTableView`

       ```c++
       #include <QApplication>
       #include <QTableWidget>
       #include <QTableWidgetItem>
       
       int main(int argc, char *argv[]) {
           QApplication app(argc, argv);
       
           QTableWidget table(3, 3); // 3行3列
           table.setWindowTitle("QTableWidget Example");
       
           // 添加数据
           for (int row = 0; row < 3; ++row) {
               for (int col = 0; col < 3; ++col) {
                   QTableWidgetItem *item = new QTableWidgetItem(QString("Item %1,%2").arg(row).arg(col));
                   table.setItem(row, col, item);
               }
           }
       
           table.show();
           return app.exec();
       }
       ```

    2. `QTableView`基于模型（Model-based）的表格控件，`QTableView` 是 `QAbstractItemView` 的子类，必须配合数据模型（`QAbstractTableModel` 或 `QStandardItemModel`）使用。符合MVC框架的支持。适合展示大量数据，性能较高，并可以自定义数据模型，支持动态数据更新。

       ```c++
       #include <QApplication>
       #include <QTableView>
       #include <QStandardItemModel>
       
       int main(int argc, char *argv[]) {
           QApplication app(argc, argv);
       
           QTableView tableView;
           tableView.setWindowTitle("QTableView Example");
       
           // 创建数据模型
           QStandardItemModel model(3, 3); // 3行3列
           for (int row = 0; row < 3; ++row) {
               for (int col = 0; col < 3; ++col) {
                   QStandardItem *item = new QStandardItem(QString("Item %1,%2").arg(row).arg(col));
                   model.setItem(row, col, item);
               }
           }
       
           tableView.setModel(&model);
           tableView.show();
       
           return app.exec();
       }
       ```

    18. 全选/取消全选操作：`CheckBoxHeaderView` 通常是指一个自定义的 **表格头部** 组件（例如，继承自 `QHeaderView`），它可以包含一个 **复选框（CheckBox）**，通常用于全选或取消全选的功能。

    19. 使用`static_cast`的时机：避免编译器执行隐式转换时可能会因为损失精度等提示异常。提前使用该类型告知，这是可以进行的，是已知的情况的。**它不会进行类型检查，所以需要用户提前确保这是正确的。**

        ```
        auto it = m_task.find(static_cast<CmdType>(item.operate()));
        
        #例子：此处的map，key为自定义enum类型。而获取到的是protobuffer数据格式的枚举，没有规定两者之间进行隐式转换的定义，会报错，此时使用static_cast就可以正常使用了
        ```

        

### 4. 末尾

