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
   report.SerializeAsString() 或 report.SerializePartialAsString()
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
   #使用add
   ClientIsolationAreaInfo::InfectInfo* isoInfo = isoMsg.add_infects();
   isoInfo->set_infect_name(info.param.virusname);
   isoInfo->set_file_name(info.param.path);
   isoInfo->set_isolate_time(info.m_time);
   isoInfo->set_md5(info.param.md5);
   ```

9. 设置非重复嵌套字段

   1. 声明嵌套字段指针，挨个给其赋值

      ```c++
      // 获取指向嵌套 Address 消息的指针，并设置其字段。
      Address* addr = person.mutable_address();  
      addr->set_city("New York");
      addr->set_zip("10001");
      ```

   2. 直接拷贝已有的嵌套字段，使用`CopyFrom()`函数

      ```
      scanEndInfo.mutable_taskstatus()->CopyFrom(m_scanTaskStatus);
      ```

10. add_和mutable\_的区别

    ```c++
    add_address()：向 repeated 字段中 新增一个地址。
    mutable_address(index)：获取并修改指定索引处的地址。
    address(index)：只读访问指定索引处的地址。
    ```

11. 枚举类型直接通过下标_获取

    ```c++
    #如果protoc文件定义了package必须用包名::后面的下标路径；
    #如果没有则不需要加包名，直接使用后面的下标路径
    HmiToScan::VirusScan_ScanType_THREAT_CLEAN
    ```

12. protobuf的msg使用二进制类型byte(可以根据不同类型放置不同`message`)和一个`type`(枚举类型)，就可以根据不同的type的message进行解析

    ```
    1. SerializeAsString() 是 Protocol Buffers 提供的一个方法，能将整个消息（包括嵌套的 message 和 bytes 字段）序列化为一个字符串（即字节流）。
    2. 序列化后的数据可以存储或传输，接收方可以使用 ParseFromString() 方法反序列化
    
    message MyMessage {
        bytes data = 1;  // 字节数组类型的字段
    }
    #示例：
    // 创建消息对象
    MyMessage message;
    // 创建字节数据（比如从某个文件读取）
    std::string byte_data = "Hello, Protocol Buffers!";
    // 将字节数据赋值给 `data` 字段
    message.set_data(byte_data);
    // 将消息序列化为字符串
    std::string serialized_data = message.SerializeAsString();
    
    ```

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

18. protobuffer中：定义了`package`就相当于使用了c++中的`namespace`，当使用其中的字段时都需要加上包名(命名空间)。

    1. 在 C++ 中：package 会生成对应的 namespace。
    2. 不定义 package，所有类型直接生成在顶层命名空间中，可以直接访问使用

19. `SerializePartialAsString()` 和 `SerializeAsString()` 是 Protocol Buffers（protobuf）库中用于将消息序列化为字符串的函数，但它们的行为在处理消息完整性方面有所不同：

    1.  `SerializeAsString()` :序列化整个消息到字符串：在序列化之前，会检查消息是否有效（完整性检查），即所有被标记为 **必需字段**（`required`）的字段是否都已设置。如果消息中缺少必需字段，则序列化会失败，并抛出异常（在某些实现中可能是返回错误状态）。使用较旧的 protobuf 版本（2.x 系列）时，因为 2.x 版本支持 `required` 字段
    2.  `SerializePartialAsString()`：序列化部分（可能不完整）的消息到字符串：不检查消息的完整性。即使消息中有必需字段未设置，也会继续序列化现有字段；在 protobuf 3.x 及更高版本中更为常用，因为这些版本中不再支持 `required` 字段，消息的完整性检查不再是主要关注点。

20. protobuffer成员名：当使用protobuffer转化时会默认将大写字母转为小写

21. protobuffer中关于enum和message，默认从0 给初值还是从1开始给

    1. 在 `enum` 中，字段的编号必须是正整数。默认情况下，`enum` 会**为第一个枚举值分配编号 `0`**，并且你可以显式地为其他枚举值指定编号，跟其他枚举类型一样。

       ```
       enum Status {
           NONE = 0;   // 默认值
           SUCCESS = 1;
           ERROR = 2;
       }
       ```

    2. `message` 类型的字段编号从 **1 开始**，而 **0 是保留的编号**，不可用于任何字段。字段编号是用来标识字段在二进制协议中的位置，因此它们必须是唯一的，并且按照升序分配。

22. 为什么要保留编号0？

    1. 在 Protocol Buffers 的编码和解码过程中，如果接收到的消息中没有包含某个字段（例如在序列化时没有该字段的值），它会使用该字段的默认值。如果使用 `0` 作为字段编号，可能会混淆 `0` 作为编号与 `0` 作为默认值的实际含义
    2. 关于默认值：如果不给 Protocol Buffers 中的字段赋值，那么该字段会使用其类型的默认值，整数类型默认值为`0`，布尔类型默认值为`false`。字符串默认值为空`""`，枚举默认为`0`，

23. protobuffer的变量种类、传输原理？

    `bytes`的作用：通常表示一个字节数组（如文件数据、图片等），直接将字节数据赋给该字段。跟传递`message`一样，存储字节流数据。

24. protobuffer中，如果成员是`message`，那么使用`mutable_data()`，不能使用`set_data()`，`set`的方式仅适用于普通类型。

    1. 因为 `message` 类型是一个复杂对象，而非简单数据类型，你需要通过指针来修改它。

       ```
       mutable_name()	// 单个message
       add_name()		// repeated类型的message
       
       Person person;
       Address* address = person.mutable_address();  // 获取一个指向 Address 的指针
       address->set_street("123 Main St");
       address->set_city("Springfield");
       ```

    2. **`set_`** 方法通常用于简单的字段类型（如 `int32`, `string`, `bool` 等），它会直接赋值并覆盖字段的内容。对于嵌套的消息，`set_` 方法不适用，因为 `set_` 只能用于基本数据类型的字段。

25. 在 Protocol Buffers (Protobuf) 中，如果一个字段的类型是 `bytes`，它表示该字段可以存储任意的二进制数据。以下为使用`bytes`存储`message`类型的数据

    `string` 类型在 Protobuf 中也是以 **UTF-8 编码的字节序列** 存储的，因此也可以视为某种二进制数据类型，尽管它通常用于存储文本数据。

    **核心在于：将message序列化和反序列化**

    ```c++
    // protobuffer
    message MyMessage {
        string name = 1;
        int32 age = 2;
    }
    
    message Wrapper {
        bytes data = 1;
    }
    
    // 存
    // 创建一个 MyMessage 实例
        MyMessage msg;
        msg.set_name("John");
        msg.set_age(30);
     // 序列化 MyMessage
        std::string serialized_msg;
        if (!msg.SerializeToString(&serialized_msg)) {
            std::cerr << "Failed to serialize MyMessage." << std::endl;
            return -1;
        }
    // 创建 Wrapper 实例，并将 serialized_msg 存放到 data 字段
        Wrapper wrapper;
        wrapper.set_data(serialized_msg);  // 将 MyMessage 的序列化数据存入 Wrapper 的 data 字段
        
    // 取
    // 从 Wrapper 中提取 data 字段（即 bytes 数据）
        std::string serialized_msg = wrapper.data();
    
    // 反序列化为 MyMessage
        MyMessage msg;
        if (!msg.ParseFromString(serialized_msg)) {
            std::cerr << "Failed to parse MyMessage from bytes." << std::endl;
            return -1;
        }
    ```

26. Protobuf 的所有数据类型，包括基本类型（如 `int32`, `float`, `string` 等）和复杂类型（如 `message` 和 `enum`），最终都会被转换为 **二进制数据**。这些二进制数据都可以通过 `bytes` 类型存储，并通过序列化和反序列化进行传输和存储。

    **实际上：`protobuffer`本质上就是使用二进制进行传输的，所以它可以用到的基本数据类型都可以转化为`bytes`。通过序列化和反序列化都可以将`message`转化为不同二进制格式进行存储或传输。比如(serialAsToString,serialAsToInt ....)**

    - **`bytes`**：专门用于存储**任意**二进制数据的类型。
    - **`string`**：虽然是文本类型，但实际上它是以 UTF-8 编码的字节序列存储的，因此也可以视为存储二进制数据。
    - **数字类型**（如 `int32`, `int64`, `fixed32`, `fixed64` 等）：这些类型虽然本质上是整数或浮点数，但在存储和传输时是以二进制格式表示的。
    - **enum和message**
    - **bool**

27. `Protobuf` 使用二进制格式而不是文本格式（如 JSON 或 XML）有几个优势：

    - **高效性**：二进制格式更紧凑，占用的空间更小，因此在传输和存储时更加高效。
    - **速度**：二进制格式的序列化和反序列化通常比文本格式要快，尤其是当数据量较大时。
    - **跨平台**：`Protobuf` 的二进制格式可以在不同的编程语言和平台之间高效地传输，保证了平台的独立性。

28. protobuffer，bytes类型的二进制转为string类型(同样也是二进制)

      ```c++
      if (virusInfo.action() == ProcessAction::ACTION_TYPE1) {
              // 将 bytes 转换为 string
              std::string str_data = std::string(virusInfo.data().begin(), virusInfo.data().end());
              std::cout << "Parsed string data: " << str_data << std::endl;
      }
      ```

29. XML、JSON、Protobuf都是用于数据交换和存储的格式。

      > 不管格式如何，他们最终传输时都会转为二进制格式(比如转为std::string)作为传递参数。

      **定义**：

      1. XML：XML（可扩展标记语言)，基于**标签语言**，数据以**树形结构存储**，每个元素都有开始标签、结束标签，和可选的属性；HTML语言就是基于XML，只是XML只用来存储和传输数据

         ```c++
         <person>
             <name>Alice</name>
             <age>30</age>
             <isActive>true</isActive>
         </person>
         ```

      2. JSON：基于**键值对**的数据表示。**key** 是字符串，**value** 可以是字符串、数字、布尔值、数组、对象或 `null`。

         ```c++
         {
             "name": "Alice",
             "age": 30,
             "isActive": true
         }
         ```

      3. Protobuffer：**二进制序列化** 格式。数据定义是基于 `.proto` 文件的，结构化的类型定义（包括消息、字段、数据类型）。

         ```c++
         message Person {
             string name = 1;
             int32 age = 2;
             bool isActive = 3;
         }
         ```

      **对比**：

    | 特性               | **XML**                                          | **JSON**                                  | **Protocol Buffers (protobuf)**                            |
    | ------------------ | ------------------------------------------------ | ----------------------------------------- | ---------------------------------------------------------- |
    | **格式类型**       | 文本格式                                         | 文本格式                                  | 二进制格式                                                 |
    | **可读性**         | 高，适合人类阅读，结构清晰                       | 高，简洁，易于人类理解                    | 低，不适合人类直接阅读（需使用生成的类来处理）             |
    | **文件大小**       | 较大，包含大量标签和冗余信息                     | 较小，比 XML 小                           | 极小，二进制格式优化了空间占用                             |
    | **解析速度**       | 较慢，解析文本格式比二进制格式慢                 | 较快，比 XML 快                           | 非常快，二进制解析速度极快                                 |
    | **类型支持**       | 支持复杂数据结构，类型多样（数字、布尔、文本等） | 支持基本数据类型（数字、字符串、布尔等）  | 完全自定义数据类型，通过 `.proto` 文件定义                 |
    | **数据表示**       | 通过嵌套标签和属性描述数据                       | 通过键值对表示数据                        | 使用自定义的结构和字段进行数据传输                         |
    | **支持的数据结构** | 树形结构（元素、属性、文本）                     | 对象（key-value pair）、数组              | 消息（message）、嵌套字段、数组、枚举等                    |
    | **灵活性**         | 高，可以自由定义标签和结构                       | 灵活，字段可以动态增加和删除              | 需要预定义 `.proto` 文件，较少灵活性                       |
    | **可扩展性**       | 非常高，可以在不破坏现有结构的情况下增加新元素   | 较高，可以添加新字段，但可能不向后兼容    | 高，支持字段添加、删除，但需要代码重生成                   |
    | **性能**           | 较差，文件冗长，解析性能较低                     | 一般，虽然比 XML 高效，但仍为文本格式     | 极高，二进制格式高效传输和解析                             |
    | **人类友好**       | 是，尤其适合复杂的文档结构和描述                 | 是，简洁易读                              | 否，二进制格式不可直接读懂                                 |
    | **主要用途**       | 配置文件、文档存储、SOAP等                       | Web API（特别是 RESTful API），前后端交互 | 高效数据存储和传输，微服务间通信，跨语言接口               |
    | **适用场景**       | 配置文件、文档处理、复杂结构的数据交换           | Web 应用、轻量级数据交换、前后端分离      | 数据量大或性能要求高的场景（例如微服务、日志传输）         |
    | **文件扩展名**     | `.xml`                                           | `.json`                                   | `.proto`（定义数据结构），编译后生成特定语言的文件         |
    | **兼容性**         | 广泛支持，几乎所有平台和语言都支持               | 广泛支持，尤其是现代 Web 开发             | 需要使用 `protobuf` 库，支持多语言（C++, Java, Python 等） |

30. protobuffer问题

    ```
    message TerminalStartStatusInfo {
        enum StatusType {
          CLIENT_STARTUP = 0;
          CLIENT_ATTEMPT_EXIT = 1;
          CLIENT_EXIT = 2;
        }
        StatusType type = 1;
    }
    ```

    使用上面的protobuffer来传输数据时，如果`type`为`CLIENT_STARTUP`时解析出来的数据为空，因为`proto`的空值的默认值为0，而这个`message`只有一个枚举类型的成员，当这个成员的值为0时，就会导致传递时当成空值来传递
