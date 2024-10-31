# 技术笔记

## GDB

1. 前提

   **GDB** 是由 GUN 软件系统社区提供的**调试工具**，同 GCC 配套组成了一套完整的开发环境，GDB 是 Linux 和许多 类Unix系统的标准开发环境。

2. 启动

   ```
   gdb	<可执行文件>				#调试Debug版本的执行文件
   gdb	<lib/so库名>			  #调试库
   ```

   > 需要使用**Debug**调试版本的构建程序进行调试，而**Release**发布版本是给测试人员提供的，同客户使用版本

3. 行号显示

   ```
   l/list	<行号>/<函数名>		#显示对应的code，每次10行
   ```

   > 一次只展示10行，若要继续，只需要按**Enter**就可以重复执行上次执行的命令

4. 断点设置

   ```
   b/breakpoint	<行号>		 			 #在某行打断点
   b				<源文件:函数名>			   #在该函数的第一行打上断点
   b				<源文件:行号>				#在该源文件的某行打上断点
   ```

5. 查看断点信息

   ```
   info	b/breakpoint
   ```

6. 删除断点

   ```
   d/delete	<断点编号>				#先查看断点信息获取断点编号再删除
   d			breakpoints			   #删除所有断点		  
   ```

   > 若删除了其他断点，再新加断点时就可以发现其编号为【4】，而并不是从1开始，这是因为我们没有退出过gdb，所以会持续上一次的编号继续往下

7. 运行/调试

   ```
   r/run				#F5【无断点直接运行、有断点从第一个断点处开始运行】
   r/run	arg1 arg2   #若程序接受参数，可以加在run后面
   ```

   > 若有断点则在第一个断点处停下来

8. 逐过程/逐语句

   ```
   n/next				#逐过程【相当于F10，执行当前行代码，遇到函数调用时跳过函数内部执行】
   s/step				#逐语句【相当于F11，一次走一条代码，可进入函数体内执行】
   c/continue			#逐断点【继续执行程序，直到遇到下一个断点】
   ```

9. 强制执行完函数

   ```
   finish				#在一个函数内部时，会直接执行完此函数
   ```

10. 查看函数调用

    ```
    bt/backtrace		#查看当前函数调用栈的过程
    ```

11. 跳转执行

    ```
    jump	<行号/文件名:行号>		#跳转到指定行号执行，不会执行当前到目标行之间的任何代码
    until	<行号/文件名:行号>		#程序继续运行直到指定行，会执行当前行到目标行之间的代码，并且在到达目标行时暂停执行
    ```

12. 变量

    ```
    p/print		<变量名>		#打印变量值
    display		<变量名>		#跟踪查看一个变量，每次停下来都显示它的值【变量/结构体…】
    undisplay   <变量名编号>	   #取消对先前设置的那些变量的跟踪
    set	var		<变量名=新值>	#修改变量的值
    ```

13. 多线程

    ```
    info threads					#查看所有线程
    thread	<线程编号> 		 		 #切换编号对应线程	
    break <行号> thread <线程编号>	#设定断点时只指定某个线程
    ```

14. 退出

    ```
    quit
    ```

15. 重新编译后继续调试

    ```
    file <新可执行文件>			#如果重新编译了程序，使用 file 加载新的可执行文件
    ```

16. 补充

    ```
    attach	<进程号>			  #调试目的进程
    detach						#断开该进程调试
    ```

17. 实例：调试libJYNetprotection.so生成库文件。JYNSAFED内部已使用了dlopen进行动态库的加载，所以让执行文件执行到相应位置就会执行加载动态库

    1. CMakelist是否写对，add_liberary和include_directories是否添加对
    2. 生成包进行迁移
    3. 注意使用"sudo"切换root用户进行试执行
    4. 使用"sudo gdb JYNSAFED"
    5. 打断点：在目的函数名CJYNetProtectionPluginImpl::onNewNotify
    6. 执行：r
    7. 逐过程：n        /不能逐步骤，因为逐步骤会进入调用的所有函数
    8. p 变量名：打印变量名的值，确认是否正确收到并解析

    9. quit 退出



## Proto_buffer

protocol buffers 是一种语言无关、平台无关、可扩展的序列化结构数据的方法，它可用于（数据）通信协议、数据存储等。

> 就是方便数据传输，数据序列化传输。与它相同的还有Json格式传输。

### 1. 语法

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

> 1. 在 **Protobuf 3** 中，**`optional`** 是**默认规则**，所以你可以直接定义字段而不加任何前缀。适用于：当字段的值在某些情况下是可忽略的，例如用户可能没有填写昵称或邮箱。而且会有默认值
>
>    > 为什么会有默认值？：**未赋值的 optional 字段**不会占用额外空间。消息接收方会将**缺失的字段**视为该字段的**默认值**。**省略机制**不仅减少了网络传输数据的大小，还确保了系统在不同版本之间的**兼容性**。
>    >
>    > **`int32`、`int64`** 等数值类型：`0`
>    >
>    > **`bool`**：`false`
>    >
>    > **`string`**：空字符串 `""`
>
> 2. `required`适用于**关键数据**，缺少该字段时无法继续处理，例如唯一标识符（如 `id`）。但是在**Protobuf3**中已被废弃，因为它会导致不兼容问题。
>
> 3. `repeated`用于表示列表、数组或集合类型的数据。
>
>    ```
>    #示例
>    syntax = "proto3";
>    
>    message User {
>      int32 id = 1;
>      repeated string phone_numbers = 2;  // 多个电话号码
>    }
>    
>    #使用
>    User user;
>    user.add_phone_numbers("123-456-7890");
>    user.add_phone_numbers("098-765-4321");
>    
>    for (const auto& number : user.phone_numbers()) {
>        std::cout << "Phone: " << number << std::endl;
>    }
>    ```
>
> 4. 在 **Protobuf** 中，每个字段后面紧跟的**数字**被称为 **字段编号（field number）**。这些编号是 **Protobuf 消息的内部标识符**，在序列化和反序列化时用于唯一标识字段。了解字段编号非常重要，因为它直接关系到消息的解析、效率和兼容性
>
>    > 1. 为什么使用编号：
>    >
>    >    **唯一标识字段**：
>    >    在序列化后的二进制数据中，字段是通过**编号**（而不是名称）进行识别的。
>    >
>    >    **保持版本兼容性**：
>    >    当你更新消息结构时（如增加字段），编号不变可以保证旧版本仍能正确解析。
>    >
>    >    **高效传输**：
>    >    编号比字段名短，节省了带宽，提高了序列化和反序列化的效率。
>    >
>    > 2. 序列化后的二进制格式
>    >
>    >    - 每个字段会存为 `(字段编号 << 3) | 类型`，然后是对应的数据。例如：
>    >
>    >      > `1 << 3` 将 `1` 的二进制表示 `0001` 向左移动 3 位，变为 `1000`，即 `8`。
>    >      >
>    >      > 2 << 3 为16，然后 16|2 （按位或运算）,为18，十六进制表示为12
>    >
>    >      - `id = 101` -> 序列化为：`0x08 0x65`
>    >        （编号 1，类型 int32）
>    >      - `name = "Alice"` -> 序列化为：`0x12 0x05 0x41 0x6c 0x69 0x63 0x65`
>    >        （编号 2，类型 string）
>
> 

### 2. 编译

```
// $SRC_DIR: .proto 所在的源目录
// --cpp_out: 生成 c++ 代码
// $DST_DIR: 生成代码的目标目录
// xxx.proto: 要针对哪个 proto 文件生成接口代码
 
protoc -I=$SRC_DIR --cpp_out=$DST_DIR $SRC_DIR/xxx.proto
```

### 3. 使用

1. getter

   ```
   直接.加类型名
   ```

2. setter

   ```
   set_XX(XXX)
   ```

3. 嵌套时的getter和setter

   ```
   add_address()：向 repeated 字段中 新增一个地址。
   mutable_address(index)：获取并修改指定索引处的地址。
   address(index)：只读访问指定索引处的地址。
   ```

4. 传递时候的序列化

   ```
   isoMsg.SerializeAsString()
   ```

5. 接受时解序列化

   ```
   scanCtrl.ParseFromString(param)
   ```

   > 都是转为string类型传输






## Cmake / Cmakelist

CMake 是一个跨平台的自动化构建系统，用于管理软件构建过程。CMake 使用一种简单的文本格式（CMakeLists.txt）来描述项目的构建过程。

1. 执行顺序

   ```
   mkdir build			// 在build目录下执行操作
   cd build
   cmake ..			// cmake从上级目录寻找cmakelist
   make				// 构建build
   ```

   > 1. 配置阶段：当运行 `cmake ..` 命令时，CMake 会读取项目的 `CMakeLists.txt` 文件，根据其中的指令和项目结构来生成适合当前平台和环境的构建文件（例如：`Makefile`）。
   > 2. 生成阶段：在配置阶段之后，CMake 会基于配置生成构建工具的输入文件`Makefile`，生成的 `Makefile` 或其他构建文件将定义如何编译、链接、打包项目中的代码。
   > 3. 构建阶段：当你运行 `make` 命令时，`make` 程序会读取 `Makefile`，按照其中定义的规则逐步编译源代码、生成目标文件（如 `.o` 文件），然后进行链接操作生成最终的可执行文件或动态库。在此阶段，CMake 不再参与，而是由 `make` 或其他构建工具实际执行构建操作

2. 限制最低版本要求

   ```
   cmake_minimum_required (VERSION 2.8)
   ```

3. 定义项目名称和版本

   ```
   project(MyProject VERSION 1.0)
   ```

4. 设置C++标准

   ```
   set(CMAKE_CXX_STANDARD 11)               # 设置 C++11 标准
   set(CMAKE_CXX_STANDARD_REQUIRED True)    # 强制使用 C++11 标准
   ```

5. 设置变量

   ```
   set(BIN_NAME JYVirusScan)				// 后续可以直接使用${BIN_NAME} 获取后面的值
   ```

6. 设置/添加源文件 | 生成可执行文件

   ```
   // 方式一
   add_executable(MyExectuable main.cpp utils.cpp)  # 第一个参数是生成的可执行文件名，后面是源文件
   
   // 方式二
   set(SOURCES main.cpp utils.cpp)          # (设置源文件) 定义源文件列表，第一个参数是自己定义的变量名
   add_executable(MyExecutable ${SOURCES})  
   
   // 方式三
   aux_source_directory(. SRC_LIST)		 # 第一个参数是源文件，第二个是自己定义的变量名
   add_executable(MyExecutable ${SRC_LIST}) 	
   ```

7. 添加头文件目录(头文件在别的文件夹下)

   ```
   include_directories(./inc_dir ../common/inc_dir)			#参数是文件夹(文件夹里面是本项目引用过的头文件)，用空格隔开
   ```

   ```
   target_include_directories(					# 如果是打包成库，则指定库名
           ${BIN_NAME}
           PRIVATE
           ${CMAKE_CURRENT_SOURCE_DIR}
           ${CMAKE_CURRENT_SOURCE_DIR}/../common
           ${CMAKE_CURRENT_SOURCE_DIR}/../../common
           ${CMAKE_CURRENT_SOURCE_DIR}/../../common/framework
   )
   ```

   > **`PRIVATE`**: 这是一个可见性标志，指示这些包含目录仅对当前目标可见。其他链接到该目标的目标不能看到这些目录。它的可选值还有 `PUBLIC` 和 `INTERFACE`，具体定义了目录的可见性。
   >
   > **`CMAKE_CURRENT_SOURCE_DIR`**: 指向当前处理的 CMakeLists.txt 文件的目录。通常，你会在这里查找当前项目或模块的头文件。
   >
   > **PROJECT_BINARY_DIR**是cmake系统变量，意思是执行cmake命令的目录

   > 可见性控制：
   >
   > 1. `PUBLIC`: 包含目录对目标本身及其所有依赖目标可见。
   > 2. `PRIVATE`: 包含目录仅对目标本身可见，不对依赖目标可见。
   > 3. `INTERFACE`: 包含目录对目标的依赖可见，但不对目标本身可见

   > **`include_directories`**和**`target_include_directories`**的区别：
   >
   > 1. **`include_directories`**:
   >    - 这是一个全局设置，它对所有后续的目标（target）有效，直到下一个 `include_directories` 调用或结束作用域。
   >    - 适合在没有特定目标的情况下使用，但可能会导致意外的依赖关系或头文件冲突。
   >
   > 2. **`target_include_directories`**:
   >    - 这是一个目标特定的设置，作用范围仅限于指定的目标（target）。
   >    - 你可以为不同的目标指定不同的包含目录，使得项目的结构更清晰，更易于管理和维护。

8. 生成动态库和静态库

   ```
   #`add_library()`指定从某些源文件创建共享库
   add_library(MyLibrary_static STATIC mylib.cpp)  # 创建静态库
   add_library(MyLibrary_shared SHARED mylib.cpp)   # 创建动态库
   
   add_library(lib_name STATIC/SHARED src) 
   # 函数作用：生成库。
   # 参数lib_name：是要生成的库名称，
   # 参数STATIC/SHARED：指定生成静态库或动态库，
   # 参数src：指明库的生成所需要的源文件
   ```

   > 动态库生成的是so文件

9. 重新定义目标属性

   ```
   set_target_properties(MyLibrary_shared PROPERTIES OUTPUT_NAME "myfunc")
   ```

   > 重新定义了库的输出名称，如果不使用set_target_properties也可以，那么库的名称就是add_library里定义的名称

10. 链接库

    ```
    target_link_libraries(target lib_name)
    # 函数作用：把库lib_name链接到target可执行文件中
    ```

    ```
    // 例子一：
    target_link_libraries(
            ${BIN_NAME}			# 第一个参数是目标文件，如果是动态/静态库名，那么就连接到该库。如果是生成的可执行文件，那么就连接到可执行文件
            PRIVATE
            pthread				# 程序中使用了多线程，需要连接pthread       
    )
    ```

    ```
    // 例子二：
    add_executable(hello ${SOURCE})						# 生成可执行文件
    target_link_libraries(hello MyLibrary_shared)		# 把库连接到可执行文件中
    ```

11. 查找包/库

    ```
    find_library(var lib_name lib_path1 lib_path2)
    # 函数作用：查找库，并把库的绝对路径和名称存储到第一个参数里
    # 参数var：用于存储查找到的库(变量)
    # 参数lib_name：想要查找的库的名称，默认是查找动态库，想要指定查找动态库或静态库
    #       可以加后缀，例如 funcname.so 或 funcname.a 
    # 参数lib_path：想要从哪个路径下查找库，可以指定多个路径
    ```

12. 仅生成库但不使用库

    > 即使用add_library，但是可以不使用add_executable





## Unbuntu

1. 移动光标快捷键

   ```
   Ctrl + A: 移动光标到行首。
   Ctrl + E: 移动光标到行尾。
   ```
   
2. 切换用户

   ```
   sudo su	    		//切换root
   su - username 		//切换usernamm，该模式下执行命令前需要加上sudo
   ```


2. 设置root密码

   ```
   sudo passwd root
   ```

3. ubuntu下载命令

   ```
   apt-get install package  //包名
   ```

4. 主机防火墙

   ```
   apt-get install ufw
   ufw enable				 //开启
   ufw allow 22		  	 //开放端口
   ufw deny from 192.168.10.1 to any port 22 	//禁止ip访问
   ufw delete deny from 192.168.10.1 to any port 22	//恢复ip访问
   ufw status 				 //查看防火墙状态
   ```

   > 查看系统的端口号，必须切换到root用户

5. 远程ssh连接

   ```
   apt-get install openssh-server
   ```

6. 安装包

   ```
   sudo dpkg -i <package.deb>    #安装包
   sudo dpkg -r <package.deb>　  #删除包
   sudo dpkg -p <package.deb>　  #彻底删除包(包括配置文件)
   dpkg -l                       #列出当前已安装的包
   ```

7. 查看系统日志

   ```
   cat /var/log/auth.log
   ```

8. 查看ssh日志

   ```
   cat /etc/hosts.deny
   cat /etc/hosts.allow
   ```



## kaliLinux

1. 解压命令

   ```
   gunzip XXX.gz
   ```

2. hydra

   示例：使用hydra暴力破解ssh

   ```
   hydra -l root -P /usr/share/wordlists/rockyou.txt ssh://192.168.3.96 -t 4 -vV 
   
   hydra -l username -P /path/to/passwords.txt ssh://192.168.1.1 -t 4 -vV
   -l：指定用户名
   -L：指定用户名字典文件路径
   -P：指定密码字典文件路径
   -t：指定线程数
   -vV：显示详细输出信息
   -o：将结果输出到文件
   ```

3. kali自带字典

   ```
   cd /usr/share/wordlists/
   gunzip rockyou.txt.gz
   ```

   





