1. 详见**CMakelist**，打包so文件，即动态库

   > 1. `cmake` 只是**配置工具**；执行cmake会根据cmakelist(项目的配置)生成对应的makefile文件，供make使用
   > 2. `make` 负责实际的**构建**工作（**编译、链接**）。它根据 `Makefile` 文件中的规则和依赖关系，检查哪些部分需要重新构建，生成最终的可执行文件或库。

2. 将编译好的文件移动到程序目录下

   ```
   cd Output/bin2.0/JingyunSd_linux_2/bin/X86_64/
   
   sudo cp cur_user JYNGJCZ2 JYNRJJH2 JYNRJJH2-UTRAY1 JYNSAFED JYNZDFY2 JYUpdateUI /opt/apps/chenxinsd/bin/
   
   sudo cp libglog.so.0.3.5 libkvcache.so libnetplugin.so libPostDataReport2.0.so libSysMonManage.so libZyAuthPlug.so libZyAVCache.so libJYVirusScanEnginePlugin.so libZyUploadFile.so  /opt/apps/chenxinsd/lib/modules/
   
   sudo cp libJYFileShred.so libJYNetProtection.so libJYSystemController.so libJYUDiskProtection.so libJYVirusScan.so libJYZDFY.so libJYVirusLibraryManage.so libJYClientUpgradeManage.so /opt/apps/chenxinsd/lib/plugins/system/
   ```

3. 什么时候需要重新执行cmake和make

   > 当更改了cmakelist中的库依赖、文件路径等需要重新对项目进行cmake
   >
   > 若只修改了代码并没有改变项目架构，那么只需要执行make编译就可

4. Cmakelist添加头文件路径时，要指定到最终的文件夹下，不然还是找不到头文件。(它只会在你指定的那个而文件夹下面找头文件)

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

9. CMakeList常用变量：

   ```
   CMAKE_SOURCE_DIR：表示 CMake 项目主目录的路径，也就是 CMakeLists.txt 文件首次被执行时所在的目录
   				  值不会发生变化，即项目cmake时的根cmakelist文件夹地址
   				  
   CMAKE_CURRENT_SOURCE_DIR：表示当前正在处理的 CMakeLists.txt 文件所在的目录。
   						  值会动态变化，即当前cmakelist所在文件夹地址。
   ```

10. CMakelist，项目使用的cpp放入`add_library`中，项目使用的头文件要把头文件的所在文件夹放入`include_directories`中。打包动态库的时候需要。原理？

11. `add_subdirectory(zdfy)`是什么意思

    1. 进入子目录构建:`add_subdirectory(zdfy)` 会告诉 CMake 进入 `zdfy` 子目录，查找并执行该目录中的 `CMakeLists.txt` 文件。
    2. 管理依赖关系:主项目会与 `zdfy` 子目录的目标共享构建环境，例如头文件路径、链接库等。

    > JYNSAFED是组件的可执行文件`add_executable`，插件每个都是JYNSAFED的一个动态库`add_library`

12. `make clean` ：删除在编译过程中生成的所有中间文件（如目标文件 `.o`、可执行文件、自动生成的依赖文件等），目的是清理项目目录，为重新编译做准备。

13. `make  -j8`  编译多线程

14. 问题：make的时候报`moc_xx.cpp`文件`had not been declared`

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
    
15. 关于项目中使用第三方库

      **什么是三方库**：**三方库**（Third-party library）是指由第三方开发和维护的、用于在应用程序中提供特定功能的代码库。三方库在 C++ 项目中用于增强功能，减少重复工作，帮助开发者解决常见的编程任务（如网络通信、数据解析、图形渲染等），而不需要从头开始编写每一行代码。

      **下载三方库**：

      1. 通过包管理工具获取

      2. 通过源码编译：大多数开源三方库提供源代码，可以通过克隆仓库并手动编译得到库文件。

         > 一种是使用cmake进行编译；另一种是通过.sh脚本进行编译。一般获取到的都会给一种编译方式

      **三方库的构成**：未编译前，一般带有源码(.cpp .h)和cmakelist/.sh方便自己编译；若下载的是编译后的，那么包含：

      1. 头文件(include): 声明了库的接口，供程序引用，使用头文件，你可以在自己的代码中调用库的功能。

         > 在大多数情况下，编译时使用的头文件和使用库时的头文件是**同一个**。它们的作用是相同的：声明库中的接口（函数、类、结构等），使得编译器可以知道如何与库进行交互。
         >
         > 头文件在**编译阶段**告诉编译器如何构建代码，在**链接阶段**则通过提供接口让链接器知道如何连接库的实现

      2. 库文件(lib): 进行编译之后的产物，分为静态库或者动态库

         1. 静态库(.a)：静态库是编译后的文件，包含了库的代码，链接到程序时将代码嵌入到可执行文件中。
         2. 动态库(.so)：动态库在运行时由操作系统加载，不会嵌入到程序的可执行文件中。

         | 特性           | 静态链接                           | 动态链接                         |
         | -------------- | ---------------------------------- | -------------------------------- |
         | **库文件形式** | `.a` / `.lib`                      | `.so` / `.dll`                   |
         | **链接时间**   | 编译时链接                         | 运行时链接                       |
         | **文件大小**   | 较大，包含所有库的代码             | 较小，只有程序和库的入口地址     |
         | **共享性**     | 无法共享，不同程序有不同副本       | 可共享，多个程序可以共享同一个库 |
         | **部署难度**   | 简单，只需部署可执行文件           | 需要确保动态库文件存在并正确路径 |
         | **更新**       | 更新库时需要重新编译整个程序       | 更新库时不需要重新编译程序       |
         | **内存使用**   | 可能较高，多个程序各自有一份库副本 | 内存共享，同一库文件只加载一次   |
         | **启动速度**   | 较快，不需要加载外部库             | 较慢，程序需要加载动态库         |

      **三方库的使用(使用Cmake)**：

      1. 将编译好的三方库放入到项目的lib文件夹中(里面包含头文件和库文件)

      2. 编写代码的时候就可以引用三方库的头文件

         ```
         #include <boost/algorithm/string.hpp>  // Boost 库头文件
         ```

      3. 编译项目的时候在cmakelist中1. 要将头文件和项目引用的头文件放一起 2. 同时要指明链接外部库路径

         > 静态库和动态库的链接都可以通过 `target_link_libraries()` 来实现，无论是静态库还是动态库

         ```
         target_include_directories(
                 ${BIN_NAME}
                 PRIVATE
                 ${CMAKE_CURRENT_SOURCE_DIR}
                 ${CMAKE_CURRENT_SOURCE_DIR}/component
                 #libs
                 ${CURL_INCLUDE_DIR}
                 ${CPR_INCLUDE_DIR}
         )
         
         target_link_libraries(
                 ${BIN_NAME}
                 PRIVATE
                 ${LIB_UDEV}
                 ${LIB_CPR}
                 ${LIB_CURL}
         )
         ```

      4. 编译整个项目。

      **三方库的使用(单独使用动态库)**：使用动态库（如 `.so` 或 `.dll`），在运行程序时，需要告诉操作系统动态库的位置。对于 Linux/macOS，通常使用环境变量 `LD_LIBRARY_PATH`。

      > 将动态库放到可执行文件的同一目录下，或者将动态库添加到系统环境变量中

      ```
      export LD_LIBRARY_PATH=/path/to/boost/lib:$LD_LIBRARY_PATH
      ./my_program
      ```

      **三方库的使用(使用编译器和链接器)**：

      1. 如果库的头文件不在标准位置，你需要告诉编译器去哪里查找这些头文件。这可以通过编译选项 `-I` 来实现

         ```
         g++ -I/path/to/boost/include my_program.cpp -o my_program
         ```

      2. 需要告诉链接器去哪里找库文件（静态库 `.a` 或动态库 `.so`）。这可以通过 `-L` 参数指定。

         ```
         g++ my_program.cpp -L/path/to/boost/lib -lboost_system -o my_program
         ```

         > `-l` 后面是库的名称，不需要写前缀 `lib` 和后缀 `.a` 或 `.so`。

      **注意事项**：一般由于不同机子环境不同(gnu所需版本不同)，所以一般下三方源码之后，需要在自己的机子上编译。再移动include头文件和lib静态库到项目路径下。(移动其他机器编译后生成的库可能会出错)

16. 在 CMake 中，生成 **可执行文件**、**静态库** 或 **动态库** 的过程中，执行 `make` 可能涉及 **编译** 和 **链接** 两个不同的阶段。

      1. 如果你的 `CMakeLists.txt` 中生成的是一个 **可执行文件**（例如使用 `add_executable()` 来指定生成可执行文件），执行 `make` 会经历 **编译** 和 **链接** 两个步骤。

         > 编译：每个源文件（`.cpp`）会编译为 `.o` 目标文件。
         >
         > 链接：**链接器**（`ld`）将这些目标文件和需要的库文件链接在一起，生成最终的 **可执行文件**

      2. 如果你要生成的是 **静态库**（使用 `add_library()` 且没有指定 `SHARED`）或 **动态库**（使用 `add_library()` 并指定 `SHARED`），则执行 `make` 过程中 **只会进行编译**，而不会进行链接到最终的可执行文件。

         > 编译：
         >
         > 1. 每个源文件（`.cpp`）会编译为 `.o` 目标文件。
         > 2. 如果是静态库，多个 `.o` 文件会被 **打包** 成一个 `.a` 文件。
         >
         > 3. 如果是动态库，多个 `.o` 文件会被 **链接** 成一个 `.so` 或 `.dll` 文件，但这个链接是与可执行文件的链接不同，它只是生成一个共享的库文件。
         >
         > 不进行最终可执行文件的链接：对于库文件来说，`make` 的目标是生成一个库，而不是一个最终的可执行文件。所以，**链接** 的部分是针对库内部的目标文件，而不是与其他可执行文件链接。

17. 查看可执行文件(二进制文件)链接的动态库：使用 `ldd` 命令查看可执行文件依赖(链接)的动态库

      ```
      ldd <executable_file>
      ldd /path/to/your/executable
      ```

18. 符号表是什么？

    **定义**：符号表（Symbol Table）是编译器在编译过程中生成的一张表格，记录了程序中各种符号（如函数名、变量名）的名称、地址和类型等信息。它主要用于链接阶段和调试阶段。

    **作用**：

    - **链接**：帮助链接器（linker）将代码中的符号引用（如函数调用）解析到对应的内存地址。
    - **调试**：为调试器（如 GDB）提供映射关系，让调试器能将内存地址关联到源代码中的函数名、变量名和行号。

19. 为什么centos出包使用gdb时没有符号，即使切换为debug也没有。**即使没有符号，但是可以看出执行的是哪个函数**

    1. 编译选项：

       编译时，可能默认启用了 -g（生成调试信息；检查你的构建脚本（Makefile、CMake 等），确认 CFLAGS 或 CXXFLAGS 是否一致包含 -g。

    2. 打包工具行为：

       **deb 包（Ubuntu）**：Debian 的打包工具（如 dpkg-buildpackage）**默认可能不会剥离调试符号**，或者将调试信息分离到独立的 `-dbg` 包中。

       **RPM 包（CentOS）**：RPM 构建工具（如 rpmbuild）**默认会剥离符号**，并将调试信息放入单独的 `debuginfo` 包中。如果你直接使用 RPM 包里的二进制文件，而没安装 `debuginfo`，就会缺少符号。

20. 关于目标架构机调试：目标机器安装的是release包，想要现场调试，需要编译(make)Dbug包，再将对应的执行文件拖到目标架构机的对应位置。

      > 注意带上函数返回值，某些架构因为函数没有返回值会卡住死机，某些架构只是会提醒。
    
21. cmakelist在导入头文件时：使用`target_include_directories`，定义头文件夹，只会在该文件夹下找`.h`或者`.hpp`文件，不会递归找其子目录的头文件