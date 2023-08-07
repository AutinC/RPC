# RPC
Based on the lightweight Muduo network library and the efficient Protobuf serialization protocol, the RPC communication framework is implemented, which is used to implement TCP-based remote procedure calls between distributed systems, and has the characteristics of high performance and high reliability.

![img](https://github.com/AutinC/RPC/blob/main/images/%E9%A1%B9%E7%9B%AE%E4%BB%A3%E7%A0%81%E4%BA%A4%E4%BA%92%E5%9B%BE-%E7%94%A8%E7%94%BB%E5%9B%BE%E6%9D%BF%E6%89%93%E5%BC%80.png?raw=true)

# 一、集群和分布式的理论讲解

## 单机聊天服务器存在什么缺点

- 单机聊天服务器所能承受的用户并发量受限于硬件资源的限制，如Linux端口数只有65536（2^16）个；
- 任意模块的修改，都会导致整个项目代码的重新编译、部署；
- 系统中有些模块是属于CPU密集型，有些属于IO密集型，所以各个模块对于硬件资源的需求不同，造成内存、CPU、带宽都有可能成为瓶颈

## 集群服务器的优缺点

- 优点：架构简单；每台机器是独立的系统，硬件资源提升，用户并发量提升
- 缺点：代码做修改需要整体重新编译并多次部署；某些模块（如管理员模块）不需要高并发，多处部署的意义不大

## 分布式服务器的优缺点

- 优点：
  - **可靠性**：由于分布式系统是由多个节点组成的，因此任何一个节点发生故障不会影响整个系统的运行。
  - **可扩展性：**当需要增加处理能力时，可以通过增加节点的方式来扩展系统的性能，而无需对整个系统进行重新设计。
  - **高性能：**由于分布式系统可以利用多个节点的处理能力，因此可以更快地完成处理任务。
  - **资源共享：**分布式系统中的节点可以共享资源，如存储空间、处理能力、网络带宽等，从而更加高效地利用系统资源。
  - **灵活性：**分布式系统中的节点可以动态地加入或退出系统，因此可以根据需要随时调整系统的规模。
- 缺点：
  - **复杂性：**分布式系统由多个节点组成，节点之间需要进行通信和协调，这增加了系统的复杂性，需要更多的开发和维护工作。
  - **一致性：**在分布式系统中，数据存储在不同的节点上，节点之间需要保持数据一致性，这增加了数据管理的难度。
  - **安全性：**由于分布式系统中的节点数量较多，因此需要更高的安全性保护，以避免恶意攻击或数据泄露。
  - 性能瓶颈：由于节点之间需要进行通信和协调，因此在某些情况下可能会出现性能瓶颈，影响系统的性能。
  - 成本：建立和维护分布式系统需要更多的硬件、软件和人力资源，因此成本较高。



# 二、RPC通信原理及项目技术选型

![img](https://github.com/AutinC/RPC/blob/main/images/1682240550705.png?raw=true)

**黄色部分**：设计rpc方法参数的打包和解析，也就是数据的**序列化和反序列化**，使用Protobuf。 

> Protobuf，即 Protocol Buffers，是一种轻量级的数据序列化协议，由Google公司开发，可用于将结构化数据序列化为可在网络上传输的格式。它支持多种编程语言，包括C++、Java、Python等，并且具有跨平台和语言互操作性的特性。
>
> 使用Protobuf，可以将数据结构定义为.proto文件，然后使用特定的编译器将其编译为目标语言的类。这些类可以用于在不同的系统和编程语言之间传输数据，并且具有更小的体积、更高的效率和更好的可读性。
>
> 与json比较：
>
> 1.Protobuf是二进制存储的，xml和json都是文本存储的；
>
> 2.Protobuf不需要存储额外信息，json存储键值对
>
> 因此，Protobuf序列化后的数据更小、更快、更灵活，尤其适合在高并发和网络通信中使用。

绿色部分：网络部分，包括寻找rpc服务主机，发起rpc调用请求和响应rpc调用结果，使用**muduo网络库和zookeeper服务配置中心**（专门做服务发现）。 

# 三、环境配置与使用

## 项目代码工程目录

bin：可执行文件
build：项目编译文件
lib：项目库文件
src：源文件
test：测试代码
example：框架代码使用范例
CMakeLists.txt：顶层的cmake文件
README.md：项目自述文件
autobuild.sh：一键编译脚本 

## Windows + VScode配置远程Linux开发环境

参考博客：https://blog.csdn.net/qq756684177/article/details/94236990 

1. linux系统运行sshd服务 
2. 在vscode上安装Remote Development插件，其依赖插件会自动安装 
3. 配置远程linux主机的信息 
4. 在vscode上开发远程连接linux 

## muduo源码编译安装与使用样例

### 编译安装

参考博客：https://blog.csdn.net/QIANGWEIYUAN/article/details/89023980

### 基于muduo的客户端服务器编程 

muduo网络库的编程很容易，要实现基于muduo网络库的服务器和客户端程序，只需要简单的组合TcpServer和TcpClient就可以，主要分为五个步骤：

1. 组合TcpServer对象
2. 创建EventLoop事件循环对象指针
3. 明确TcpServer构造函数的参数，输出ChatServer的构造函数
4. 在当前服务器类的构造函数中，注册处理连接的回调函数和处理读写事件的回调函数
5. 设置合适的服务端线程数量，muduo库会自己分配IO线程和worker线程

- 优点：灵活；根据节点的并发要求，对一个节点可以再做节点模块集群部署 
- 缺点：服务器共同构成一个系统，耦合度高

### 网络服务器编程常用模型

**【方案1】 ： accept + read/write**
不是并发服务器

**【方案2】 ： accept + fork - process-pre-connection**
适合并发连接数不大，计算任务工作量大于fork的开销

**【方案3】 ：accept + thread-pre-connection**
比方案2的开销小了一点，但是并发造成线程堆积过多

**【方案4】： muduo的网络设计：reactors in threads - one loop per thread**
方案的特点是one loop per thread，有一个main reactor负载accept连接，然后把连接分发到某个subreactor（采用round-robin的方式来选择sub reactor），该连接的所用操作都在那个sub reactor所处的线程中完成。多个连接可能被分派到多个线程中，以充分利用CPU。
Reactor poll的大小是固定的，根据CPU的数目确定 。

一个Base IO thread负责accept新的连接，接收到新的连接以后，使用轮询的方式在reactor pool中找到合适的sub reactor将这个连接挂载上去，这个连接上的所有任务都在这个sub reactor上完成。
如果有过多的耗费CPU I/O的计算任务，可以提交到创建的ThreadPool线程池中专门处理耗时的计算任务。 

**【方案5】 ： reactors in process - one loop pre process**
Nginx服务器的网络模块设计，基于进程设计，采用多个Reactors充当I/O进程和工作进程，通过一把accept锁，完美解决多个Reactors的“惊群现象” 。

### muduo中的Reactor模型

1. 事件驱动（event handling）
2. 可以处理一个或多个输入源（one or more inputs）
3. 通过Service Handler同步的将输入事件（Event）采用多路复用分发给相应的Request Handler（多个）处理 

![img](https://github.com/AutinC/RPC/blob/main/images/1683098044886.png?raw=true)

## Protobuf安装配置与使用

Protobuf（protocol buffer）是google 的一种数据交换的格式，它独立于平台语言。
Google 提供了protobuf多种语言的实现：java、c#、c++、go 和 python，每一种实现都包含了相应语言的编译器以及库文件。
由于它是一种二进制的格式，比使用 xml（20倍） 、json（10倍）进行数据交换快许多。可以把它用于分布式应用之间的数据通信或者异构环境下的数据交换。作为一种效率和兼容性都很优秀的二进制数据传输格式，可以用于诸如网络传输、配置文件、数据存储等诸多领域 。

### ubuntu protobuf环境搭建 

**github源代码下载地址：https://github.com/google/protobuf**
安装过程如下：
1、解压压缩包：unzip protobuf-master.zip
2、进入解压后的文件夹：cd protobuf-master
3、安装所需工具：sudo apt-get install autoconf automake libtool curl make g++ unzip
4、自动生成configure配置文件：./autogen.sh
5、配置环境：./configure
6、编译源代码：make 

7、安装：sudo make install
8、刷新动态库：sudo ldconfig 

### Protobuf示例

```cpp
syntax = "proto3"; // 声明了protobuf的版本

package fixbug; // 声明了代码所在的包（对于C++来说是namespace）

// 定义下面的选项，表示生成service服务类和rpc方法描述，默认不生成
option cc_generic_services = true;


message ResultCode
{
    int32 errcode = 1;
    bytes errmsg = 2;
}

// 数据   列表   映射表
// 定义登录请求消息类型  name   pwd
message LoginRequest
{
    bytes name = 1; //等于1表示这是消息体里的第一个字段
    bytes pwd = 2;
}

// 定义登录响应消息类型
message LoginResponse
{
    ResultCode result = 1;
    bool success = 2;
}

//定义获取好友列表请求消息类型
message GetFriendListsRequest
{
    uint32 userid = 1;
}


// 定义用户信息类型
message User
{
    bytes name = 1;
    uint32 age = 2;
    enum Sex
    {
        MAN = 0;
        WOMAN = 1;
    }
    Sex sex = 3;
}

// 定义获取好友列表响应消息类型
message GetFriendListsResponse
{
    ResultCode result = 1;
    repeated User friend_list = 2;  // 定义了一个列表类型
}

// 在protobuf里面怎么定义描述rpc方法的类型 - service
service UserServiceRpc
{
    rpc Login(LoginRequest) returns(LoginResponse);
    rpc GetFriendLists(GetFriendListsRequest) returns(GetFriendListsResponse);
}
```

![img](https://github.com/AutinC/RPC/blob/main/images/1683104270953.png?raw=true)

message对象LoginRequest、LoginResponse继承于Message类

![img](https://github.com/AutinC/RPC/blob/main/images/1683104510918.png?raw=true)

![img](https://github.com/AutinC/RPC/blob/main/images/1683166312879.png?raw=true)

UserServiceRpc类自动继承于Service类，是rpc服务的提供者；

UserServiceRpc_Stub类自动继承于UserServiceRpc类，是rpc服务的消费者，其包含一个RpcChannel类型的成员变量channel，所有成员方法的调用本质都是通过channel调用其CallMethod方法。

RpcChannel类只有一个纯虚函数CallMethod，需要用户自己重写CallMethond方法（MyRpcChannel）并继承于RpcChannel，在构造UserServiceRpc_Stub时传入MyRpcChannel

# 四、mprpc框架设计

## 框架的使用

example目录下的 `callee/UserService.cpp` 里面：

```C++
    RpcApplication::init(argc, argv);

    //框架服务提供provider
    RpcProvider provide;
    provide.notify_service(new UserService());
    provide.run();
```

主要做了三个事情：

- 首先 RPC 框架肯定是部署到一台服务器上的，所以我们需要对这个服务器的 ip 和 port 进行初始化
- 然后创建一个 porvider（也就是server）对象，将当前 UserService 这个对象传递给他，也就是其实这个 RPC 框架和我们执行具体业务的节点是在同一个服务器上的。RPC框架负责解析其他服务器传递过来的请求，然后将这些参数传递给本地的方法。并将返回值返回给其他服务器
- 最后是去让这个 provider 去 run 起来

```C++
//启动rpc服务节点，开始提供rpc远程网络调用服务
void RpcProvider::run()
{
    //获取ip和port
    string ip = RpcApplication::get_instance().get_configure().find_load("rpcserver_ip");
    uint16_t port = atoi(RpcApplication::get_instance().get_configure().find_load("rpcserver_port").c_str());
    //cout << ip << ":" << port << endl;
    InetAddress address(ip, port);

    //创建tcpserver对象
    TcpServer server(&eventloop_, address, "RpcProvider");
    //绑定链接回调和消息读写回调方法
    server.setConnectionCallback(bind(&RpcProvider::on_connection, this, _1));
    server.setMessageCallback(bind(&RpcProvider::on_message, this, _1, _2, _3));

    //设置muduo库的线程数量
    server.setThreadNum(4);

    //把当前rpc节点上要发布的服务全部注册到zk上面，让rpc client可以从zk上发现服务
    ZookeeperClient zk_client;
    zk_client.start();

    //在配置中心中创建节点
    for (auto &sp : service_map_)
    {
        string service_path = "/" + sp.first;
        zk_client.create(service_path.c_str(), nullptr, 0);
        for (auto &mp : sp.second.method_map_)
        {
            string method_path = service_path + "/" + mp.first;
            char method_path_data[128] = {0};
            sprintf(method_path_data, "%s:%d", ip.c_str(), port);
            //ZOO_EPHEMERAL 表示znode时候临时性节点
            zk_client.create(method_path.c_str(), method_path_data, strlen(method_path_data), ZOO_EPHEMERAL);
        }
    }

    RPC_LOG_INFO("server RpcProvider [ip: %s][port: %d]", ip.c_str(), port);
    //启动网络服务
    server.start();
    eventloop_.loop();
}
```

run函数的实现：

- 因为底层调用的是muduo网络库，所以这里会获取ip地址和端口号，然后初始化网络层
- 然后去设置一个连接回调以及发生读写事件时候的回调函数（稍后介绍）
- 然后设置整个 muduo 网络库工作的线程数量
- 然后创建zookeeper配置中心，将这些方法的信息以及本机的IP地址注册到zookeeper
- 然后开启本机服务器的事件循环，等待其他服务器的连接
