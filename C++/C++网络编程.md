# Socket网络编程

> 主要通过**套接字(ip+端口号)**实现两个主机(服务端和客户端)之间的通信

## 1. 基础概念

### 1.1 什么是网络编程

网络编程其实和我们计算机上的**文件读取**操作很类似，通俗地讲，网络编程就是编写程序使两台联网的计算机相互交换数据。那么，数据具体怎么传输呢？其实操作系统会提供名为“套接字”的部件，**套接字就是网络数据传输用的软件设备而已**。因此，网络编程常常又称为套接字编程。

### 1.2 什么是套接字

Socket是套接字，不是协议，是为了方便使用传输协议（TCP\UDP）而封装出来的一组接口API，它位于应用层和传输层之间

### 1.3 网络编程分类

socket编程分为TCP和UDP两个模块，其中TCP是可靠的、安全的，常用于发送文件等，而UDP是不可靠的、不安全的，常用作视频通话等

<img src="source/images/C++%E7%BD%91%E7%BB%9C%E7%BC%96%E7%A8%8B/61f5e06e5d3740c0b378963a3fe09c37.png" alt="在这里插入图片描述" style="zoom: 80%;" />

### 1.4 不同操作系统上的网络编程

1. 基于linux操作系统：Linux上，socket操作与文件操作没有区别，在Linux上，socket也被认为是文件的一种。
2. 基于windows操作系统：Windows套接字大部分是参考BSD系列UNIX套接字设计的，所以很多地方都跟Linux套接字类似。也有不同，只需要更改Linux环境下编好的一部分网络程序内容，就能再Windows平台下运行



## 2. windows系统上的网络编程

### 2.1 基于TCP的Socket编程

<img src="source/images/C++%E7%BD%91%E7%BB%9C%E7%BC%96%E7%A8%8B/fe912cdd9efe4e77a1a3e8c2b8d16496.png" alt="在这里插入图片描述" style="zoom: 50%;" />

#### 1. 服务端

第一步：初始化套接字库，声明套接字版本

第二步：调用socket()函数创建套接字。

第三步：调用bind()函数分配IP地址和端口号。

第四步：调用listen()函数转为可接收请求状态。

第五步：调用accept()函数受理连接请求。

第六步：读取数据和传输数据

第七步：关闭连接

##### 1.  套接字库初始化

```c++
int WSAStartup(WORD wVersionRequested, LPWSADATA lpWSAData);
```

> 第一个参数：Winsock中存在多个版本，应准备WORD类型的(WORD是typedef声明的unsigned short)套接字版本信息。若版本为1.2，则其中1是主版本号，2是副版本号，应传递0x0201。高8位为副版本号，低8位为主版本号。我们还可以直接使用宏，MAKEWORD(2,2); //主版本号为2，副版本为2，返回0x0202。
>
> 第二个参数：就是传入WSADATA型结构体变量地址，存储初始化信息。

```c++
//初始化socket环境的函数
bool init_socket() {
	WSADATA wsadata;
	if (WSAStartup(MAKEWORD(2,2), &wsadata) != 0) {
		std::cout << "socket启动失败，错误码：" << WSAGetLastError() << std::endl;
		return false;
	}
	else {
		return true;
	}
}
```

##### 2. 创建TCP套接字

```c++
SOCKET socket(int domain, int type, int protocol);
```

> 第一个参数：地址族，AF_INEF：ipv4   AF_STREAM：ipv6
>
> 第二个参数：指明socket类型，SOCK_STREAM：tcp   SOCK_DGRAM：udp
>
> 第三个参数：指定与地址族对应的通信协议，这个参数直接写0，代表系统自己选择合适的协议
>
> 返回值：返回创建好的服务端socket对象

```c++
SOCKET soc = socket(AF_INET, SOCK_STREAM, 0);
if (soc == INVALID_SOCKET) {
	std::cout << "服务端socket对象创建失败，错误码：" << WSAGetLastError() << std::endl;
	return soc;
}
```

##### 3. 绑定ip和端口号

```c++
int bind(SOCKET s, struct sockaddr *myaddr, socklen_t addrlen);
```

> 第一个参数：要绑定的服务端socket对象
>
> 第二个参数：绑定的信息结构体sockaddr
>
> 第三个参数：结构体的大小

```c++
//准备绑定信息
SOCKADDR_IN addr;
addr.sin_family = AF_INET; //设置地址家族，此处跟上面创建socket对象的时候必须保持一致，即ipv4
addr.sin_port = htons(8888); //设置端口号，要使用htons完成本地到网络字节顺序的转换
addr.sin_addr.S_un.S_addr = inet_addr("192.168.43.20"); //设置ip，inet_addr函数也是完成网络字节序从默认的大端转到小端

if (bind(soc,(sockaddr*)&addr,sizeof(addr)) == SOCKET_ERROR) {
	std::cout << "服务端绑定失败，错误码：" << WSAGetLastError << std::endl;
}
```

##### 4. 监听端口

```c++
int listen(SOCKET s, int backlog);
```

> 第一个参数：需要要监听服务的服务端socket对象
>
> 第二个参数：监听队列的最大允许数

```c++
if (listen(soc, 10) == SOCKET_ERROR) {
	std::cout << "服务端监听失败，错误码：" << WSAGetLastError() << std::endl;
}
```

##### 5. 受理连接请求

```c++
SOCKET accept(SOCKET s, struct sockaddr *addr, socklen_t *addrlen);
```

> 第一个参数：受理的服务端socket对象
>
> 第二个参数：接收连接的客户端socket对象的信息sockaddr结构体地址
>
> 第三个参数：接收连接的客户端socket对象信息的长度
>
> 第二个和第三个参数也可以设置为NULL，等函数调用完之后就会自动设置
>
> 返回值：返回成功建立连接的客户端对象，后续的数据接收和发送都需要这个对象

```c++
//返回值，成功会返回一个客户端的socket对象，用于继续后续处理；失败了会返回失败的对象
SOCKET accept_soc = accept(serversocket, NULL, NULL);
if (accept_soc == INVALID_SOCKET) {
    std::cout << "服务端accept失败，错误码：" << WSAGetLastError() << std::endl;
}
```

##### 6. 数据接收

```c++
int recv(SOCKET s, char *buf, int len, int flags);
```

> 第一个参数：连接后的客户端对象
>
> 第二个参数：接收消息的字符数组地址
>
> 第三个参数：读取的消息的最大字节数
>
> 第四个参数：是个标志位，用不到，设置成0
>
> 返回值：返回的是接收到实际消息字节数量

```c++
if (recv(accept_soc,buf,512,0)>0)//如果接收成功，就输出展示，否则就打印错误码
{
    std::cout << "客户端说：" << buf << std::endl;
}
else {
    std::cout << "接收失败，错误码：" << WSAGetLastError()<< std::endl;
    break;
}
```

##### 7. 数据发送

```c++
int send(SOCKET s, const char *buf, int len, int flags);
```

> 同接收函数的参数一样

```c++
//如果发送和接收使用同一个字符数组，需要先清洗数组
for (int i = 0; i < strlen(buf); i++) {
    buf[i] = '\0';
}
//使用输入流回消息
std::cin.getline(buf,512);
//发送消息
if (send(accept_soc,buf,512,0)>0)//如果接收成功，就输出展示，否则就打印错误码
{
    std::cout << "服务端说：" << buf << std::endl;
    std::cout << buf << std::endl;
}
else {
    std::cout << "发送失败,错误码：" << WSAGetLastError() << std::endl;
    break;
}
```

##### 8. 释放套接字库

1. 先关闭socket对象，断开连接

```c++
int closesocket(SOCKET S);
```

2. 对Winsock库资源进行释放

```c++
int WSACleanup(void);   //返回0成功，失败返回SOCKET_ERROR
```

```c++
//使用closesocket函数完成socket对象的回收
closesocket(serversocket);
closesocket(accept_soc);

if (WSACleanup()!=0) {
	std::cout << "socket关闭失败，错误码：" << WSAGetLastError() << std::endl;
	return false;
}
else {
	return true;
}
```



##### 9. 所有代码实现

1. 头文件

```c++
#pragma once

//windows下的socket网络编程，需要引入windows的socket库
#include<WinSock2.h>
//接下来还需要将一个静态库初始化
#pragma comment(lib,"ws2_32.lib")

//接下来声明所需要的函数
//初始化socket环境的函数
bool init_socket();
//关闭socket资源的函数
bool close_socket();
//创建服务端的socket对象，用来实现具体的通信过程
SOCKET serverSocket();
```

2. 头文件实现

```c++
#include"tcpsocket.h"
#include<iostream>

#pragma warning(disable:4996)

//初始化socket环境的函数
bool init_socket() {
	//先准备一个结构体WSADATA，用于存放socket的初始化信息，包括版本号等信息，版本号使用2.2，其他使用默认值
	WSADATA wsadata;
	//使用WSAStartup函数启动socket环境，有两个参数
	//第一个参数：协议版本
	//第二个参数：初始化信息存放的内存地址
	//返回值：成功返回0，失败的话，打印错误码，错误码可以通过WSAGetLastError()获取
	if (WSAStartup(MAKEWORD(2,2), &wsadata) != 0) {
		std::cout << "socket启动失败，错误码：" << WSAGetLastError() << std::endl;
		return false;
	}
	else {
		return true;
	}

}

//关闭socket资源的函数
bool close_socket() {
	//调用WSACleanup函数完成socket资源的释放
	//此函数无参数，返回0代表成功
	if (WSACleanup()!=0) {
		std::cout << "socket关闭失败，错误码：" << WSAGetLastError() << std::endl;
		return false;
	}
	else {
		return true;
	}
}

//创建服务端的socket对象，用来实现具体的通信过程
SOCKET serverSocket() {
	//1. 创建socket对象
	//直接使用socket方法就可以创建，有三个参数
	//no.1: 指定地址族，使用的是AF_INET,表示使用IPV4的host::port的方式解析网络地址
	//no.2: 指定socket的类型，使用流式套接字SOCK_STREAM(用于TCP),还有数据报套接字SOCK_DGRM(用于UDP)
	//no.3: 指定与地址族对应的通信协议，这个参数直接写0，代表系统自己选择合适的协议
	//返回值： 成功会返回一个socket对象
	SOCKET soc = socket(AF_INET, SOCK_STREAM, 0);
	if (soc == INVALID_SOCKET) {
		std::cout << "服务端socket对象创建失败，错误码：" << WSAGetLastError() << std::endl;
		return soc;
	}

	//2. 如果创建成功了，接下来要给SOCKET对象绑定一些信息，用于通信
	//使用bind函数完成绑定，绑定需要使用结构体类型：sockaddr
	//还需要使用另外一个结构体sockaddr_in来完成赋值，绑定的时候需要将sockaddr_in转换成sockaddr类型
	SOCKADDR_IN addr;
	addr.sin_family = AF_INET; //设置地址家族，此处跟上面创建socket对象的时候必须保持一致
	addr.sin_port = htons(8888); //设置端口号，要使用htons完成本地到网络字节顺序的转换
	//TCP/IP协议规定了，网络传输使用的字节顺序和本地存储的字节顺序是相反的。网络使用的顺序叫做大端字节序，本地使用的叫做小端字节序。
	//大端字节序使用低位的内存地址保存高位的数据，小端字节序使用低位的内存地址保存低位的数据
	addr.sin_addr.S_un.S_addr = inet_addr("192.168.43.20"); //inet_addr函数也是完成网络字节序的转换，这个函数可能会有一个安全警告，建议直接屏蔽掉

	//开始绑定，bind函数有三个函数；
	//第一个参数，需要绑定的socket对象，即soc
	//第二个参数，是要绑定的结构体，它是sockaddr*类型，需要将sockaddr_in类型转换
	//第三个参数，是结构体的大小，字节单位
	//返回值，失败会返回SOCKET_ERROR常量
	if (bind(soc,(sockaddr*)&addr,sizeof(addr)) == SOCKET_ERROR) {
		std::cout << "服务端绑定失败，错误码：" << WSAGetLastError << std::endl;
	}

	//3. 启动对8888端口的监听，等待客户端的消息
	//用listen函数完成监听，有两个参数：
	//第一个参数，是socket对象，即soc
	//第二个参数，是监听队列的最大允许数。
	//返回值：如果失败会返回
	if (listen(soc, 10) == SOCKET_ERROR) {
		std::cout << "服务端监听失败，错误码：" << WSAGetLastError() << std::endl;
	}

	return soc;
}
```

3. 服务端主函数

```c++
#include <iostream>
#include"../myclient/tcpsocket.h"

int main()
{
    //1. 初始化socket环境
    init_socket();

    //2. 创建socket对象，做配置，
    //3. 如果监听到客户端的连接请求，需要通过accept函数来收到这个请求，它有三个参数
    //第一个：socket对象启动监听端口
    SOCKET serversocket = serverSocket();
    std::cout << "服务端正在等待连接。。。" << std::endl;

    //第二个：第三个都填NULL，都是两个地址，等函数调用完之后就会自动设置
    //返回值，成功会返回一个客户端的socket对象，用于继续后续处理；失败了会返回失败的对象
    SOCKET accept_soc = accept(serversocket, NULL, NULL);
    if (accept_soc == INVALID_SOCKET) {
        std::cout << "服务端accept失败，错误码：" << WSAGetLastError() << std::endl;
    }

    //4. 至此通信通道就建立好了，可以进行数据传输了，recv()函数用来接收数据，send()函数用来发送
    char buf[512] = {'\0'}; //用来存储消息

    //循环让聊天一直进行下去，直到关闭程序
    while (true) {
        //循环中，每次收发消息使用的都是同一个数组，所以需要清空数组
        for (int i = 0; i < strlen(buf); i++) {
            buf[i] = { '\0' };
        }
        //4.1 接收，使用recv()函数，四个参数
        //第一个参数：已经建立连接的那个客户端socket对象
        //第二个参数：是客户端数据的缓存位置，就是buf
        //第三个参数，是读取的消息的字节数，共有512个字节大小，就读512
        //第四个参数，是个标志位，用不到，设置成0
        //返回值是读出来的字节数量，如果失败返回socketerror
        if (recv(accept_soc,buf,512,0)>0)//如果接收成功，就输出展示，否则就打印错误码
        {
            std::cout << "客户端说：" << buf << std::endl;
        }
        else {
            std::cout << "接收失败，错误码：" << WSAGetLastError()<< std::endl;
            break;
        }


        //4.2 发送，使用send()函数，它的参数和revc是一样的
        ////返回值是发送的字节数量，如果失败返回socketerror
        std::cout << "服务端发送给客户端的消息：" << std::endl;
        //首先需要清空这个数组
        for (int i = 0; i < strlen(buf); i++) {
            buf[i] = '\0';
        }
        //使用输入流回消息
        std::cin.getline(buf,512);
        //发送消息
        if (send(accept_soc,buf,512,0)>0)//如果接收成功，就输出展示，否则就打印错误码
        {
            std::cout << "服务端说：" << buf << std::endl;
            std::cout << buf << std::endl;
        }
        else {
            std::cout << "发送失败,错误码：" << WSAGetLastError() << std::endl;
            break;
        }
    }

    //5. 循环结束时，聊天结束，可以做socket资源释放
    //首先先清理对应的socket对象，第二部关闭socket系统资源
    //使用closesocket函数完成socket对象的回收
    closesocket(serversocket);
    closesocket(accept_soc);

    //关闭socket系统资源
    close_socket();
}
```





#### 2. 客户端

第一步：套接字库初始化

第二步：创建客户端tcp套接字

第三步：调用connect函数向服务端发送连接请求

第四步：数据发送和接收

第五步：断开连接

##### 1. 套接字初始化

同服务端一样

```c++
int WSAStartup(WORD wVersionRequested, LPWSADATA lpWSAData);
```

##### 2. 创建套接字

同服务端一样

```
SOCKET socket(int domain, int type, int protocol);
```

##### 3. 连接客户端

```c++
int connect(int sockfd, struct sockaddr *serv_addr, socklen_t addrlen);
```

> 参数同bind()函数的参数一样
>
> accept()函数将会接收到该socket对象的信息

```c++
if(connect(soc,(sockaddr*)&addr, sizeof(addr)) == SOCKET_ERROR) {
	std::cout << "客户端绑定失败，错误码：" << WSAGetLastError << std::endl;
}
```

##### 4. 数据发送

```c++
int send(SOCKET s, const char *buf, int len, int flags);
```

> 参数和服务端一样，只不过这个对象是客户端自己本身的，而服务端是accept之后接收的客户端对象

##### 5. 数据接收

```c++
int recv(SOCKET s, char *buf, int len, int flags);
```

##### 6. 释放套接字

1. 先关闭socket对象，断开连接

```c++
int closesocket(SOCKET S);
```

2. 对Winsock库资源进行释放

```
int WSACleanup(void);   //返回0成功，失败返回SOCKET_ERROR
```



##### 7. 所有代码实现

1. 头文件

```c++
#pragma once

//windows下的socket网络编程，需要引入windows的socket库
#include<WinSock2.h>
//接下来还需要将一个静态库初始化
#pragma comment(lib,"ws2_32.lib")

//接下来声明所需要的函数
//初始化socket环境的函数
bool init_socket();
//关闭socket资源的函数
bool close_socket();
//创建客户端的socket对象，用来实现具体的通信过程
SOCKET clientSocket(const char* ip); //ip代表服务端的ip地址
```

2. 头文件实现

```c++
#include"tcpsocket.h"
#include<iostream>

#pragma warning(disable:4996)

//初始化socket环境的函数
bool init_socket() {
	//先准备一个结构体WSADATA，用于存放socket的初始化信息，包括版本号等信息，版本号使用2.2，其他使用默认值
	WSADATA wsadata;
	//使用WSAStartup函数启动socket环境，有两个参数
	//第一个参数：协议版本
	//第二个参数：初始化信息存放的内存地址
	//返回值：成功返回0，失败的话，打印错误码，错误码可以通过WSAGetLastError()获取
	if (WSAStartup(MAKEWORD(2,2), &wsadata) != 0) {
		std::cout << "socket启动失败，错误码：" << WSAGetLastError() << std::endl;
		return false;
	}
	else {
		return true;
	}

}

//关闭socket资源的函数
bool close_socket() {
	//调用WSACleanup函数完成socket资源的释放
	//此函数无参数，返回0代表成功
	if (WSACleanup()!=0) {
		std::cout << "socket关闭失败，错误码：" << WSAGetLastError() << std::endl;
		return false;
	}
	else {
		return true;
	}
}

//创建客户端的socket对象，用来实现具体的通信过程
SOCKET clientSocket(const char* ip) {
	//1. 创建socket对象
	SOCKET soc = socket(AF_INET, SOCK_STREAM, 0);
	if (soc == INVALID_SOCKET) {
		std::cout << "客户端socket对象创建失败，错误码：" << WSAGetLastError() << std::endl;
		return soc;
	}
	//2. 给socket对象设置重要的信息，仍然要使用结构体类型sockaddr_in
	SOCKADDR_IN addr;
	addr.sin_family = AF_INET; //指明ipv4
	addr.sin_port = htons(8888); //被动模式，需要设置成服务端指定的端口
	addr.sin_addr.S_un.S_addr = inet_addr(ip); //设置要链接的服务端ip

	//3. 使用connect链接服务端
	//connect函数的参数跟bind是一样的用法
	if(connect(soc,(sockaddr*)&addr, sizeof(addr)) == SOCKET_ERROR) {
		std::cout << "客户端绑定失败，错误码：" << WSAGetLastError << std::endl;
	}
	return soc;
}
```

3. 客户端主函数

```c++
#include<iostream>
#include"tcpsocket.h"

int main() {
	//1. 初始化socket环境
	init_socket();

	//2. 创建客户端的socket对象,里面已经连接了服务端，后面只需要发送和接收数据就好了
	SOCKET clientsocket = clientSocket("127.0.0.1");

	//3. 发送和接收消息
    char buf[512] = { '\0' }; //用来存储消息
    
    //循环让聊天一直进行下去，直到关闭程序
    while (true) {
        //循环中，每次收发消息使用的都是同一个数组，所以需要清空数组
        for (int i = 0; i < strlen(buf); i++) {
            buf[i] = '\0';
        }

        //4.2 发送
        std::cout << "客户端发送给服务端的消息：" << std::endl;
        //首先需要清空这个数组
        for (int i = 0; i < strlen(buf); i++) {
            buf[i] = '\0';
        }
        //使用输入流回消息
        std::cin.getline(buf, 512);
        //发送消息
        if (send(clientsocket, buf, strlen(buf), 0) > 0)//如果接收成功，就输出展示，否则就打印错误码
        {
            std::cout << "客户端说：" << buf << std::endl;
            std::cout << buf << std::endl;
        }
        else {
            std::cout << "发送失败,错误码：" << WSAGetLastError() <<  std::endl;
            break;
        }

        //接收消息
        if (recv(clientsocket, buf, strlen(buf), 0) > 0)//如果接收成功，就输出展示，否则就打印错误码
        {
            std::cout << "服务端说：" << buf << std::endl;
        }
        else {
            std::cout << "接收失败,错误码：" << WSAGetLastError() <<  std::endl;
            break;
        }

        
    }
    //循环结束时，聊天结束，可以做socket资源释放
        //首先先清理对应的socket对象，第二部关闭socket系统资源
        //使用closesocket函数完成socket对象的回收
    closesocket(clientsocket);
    //关闭socket系统资源
    close_socket();
    return 0;

}
```





### 2.2 基于UDP的Socket编程

<img src="source/images/C++%E7%BD%91%E7%BB%9C%E7%BC%96%E7%A8%8B/92c761423633457b87a87104137f7580.png" alt="在这里插入图片描述" style="zoom: 50%;" />





## 3. linux系统上的网络编程



