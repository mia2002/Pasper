---
layout: post
date: 2017-12-19
title: POSIX Socket
tags: Android
---

## 创建socket

```c
int socket(int domain,int type,int protocol);
```

- domain: 使用的协议族。
	- PF_LOCAL: 主机内部通信协议族，该协议族使得物理上运行在同一台设备上的应用程序可以彼此通信。
	- PF_INET: Internet第四版协议，该协议使应用程序可以与网络上其他运行的应用程序进行通信。
- type: 指定通信的语义。
	- SOCK_STREAM: 提供使用TCP协议的、面向连接的通信stream socket类型。
	- SOCK_DGRAM: 提供使用UDP协议的、无连接的通信数据包类型。
- protocol: 指定将会用到的协议。对于大多数协议族和协议类型来说，只能使用一个协议。若使用默认协议，该参数可设为0。

示例代码：

```c
int tcpSocket = socket(PF_INEF,SOCKET_STREAM,0);

//检查socket构造是否正确
if(-1 == tcpSocket){
	//抛出带错误号的异常
	ThrowErrnoException(env,"java/io/IOException",errno);
}

```

## 绑定地址

当用socket函数创建一个socket后，该socket存在一个socket族控件中，且没为该socket分配协议地址。为了使客户能够定位到这个socket并与之相连，它需要先与一个地址绑定，可以用bind函数将socket与地址绑定。

```c
int bind(int socketDescriptor,const struct sockaddr* address,socklen_t addressLength);
```

- socketDescriptor: 指定将绑定到指定地址的socket实例
- address: 指定socket被绑定的协议地址。
- address length: 指定传递给函数的协议地址结构的大小。

不同协议族使用不同的协议地址。PF_INET协议族使用sockaddr_in结构体指定协议地址。

```c
struct sockaddr_in {
	sa_family_t sin_family;
	unsigned short int sin_port;
	struct in_addr sin_addr;
}

```

如果绑定成功，bind函数将返回0，否则返回-1且errno全局变量被设置为相应的错误值。

示例代码：

```c
struct socketaddr_in address;

//绑定socket的地址
memset(&address,0,sizeof(address));
address.sin_family = PF_INET;

//绑定到所有地址
address.sin_addr.s_addr = htonl(INADDR_ANY);

//将端口转换为网络字节顺序
address.sin_port = htons(port);

//绑定socket
if(-1 == bind(sd,(struct sockaddr*) &address,sizeof(address))){
	//抛出带错误号的异常
	ThrowErrnoException(env,"java/io/IOException",errno);
}
```

如果在地址结构中将端口号设置为0，bind函数会将第一个可用端口号分配给socket。可以用getsocketname函数在socket中检索到这个端口号。

```c
unsigned short port = 0;
struct socketaddr_in address;
socklen_t addressLength = sizeof(address);

// 获取socket地址
if(-1 == getsocketname(sd,(struct sockaddr*) &address,&addressLength)){
	ThrowErrnoException(env,"java/io/IOException",errno);
}else{
	//将端口转换为主机字节顺序
	port = ntohs(address.sin_port);
}
```

## 网络字节排序

在硬件层上，不同的机器体系结构使用不同的数据排序和表示规则，这被称为机器字节排序或字节序。不同的CPU有不同的字节序类型，这些字节序是指整数在内中保存的顺序，这个叫主机序。最常见的有两种：

- Big-endian 字节顺序首先储存最重要的字节。
- Little-endian 字节顺序首先储存最不重要的字节。

字节排序规则不同的机器不能直接交换数据。为了使字节排序规则不同的机器能在网络上通信，IP将big-endian字节排序声明为官方的数据传输网络字节排序规则。

由于Java虚拟机本身使用的就是big-endian字节排序，所以在网络通信时，无需做数据转换，但原生组件并不是运行在java虚拟机上的，因此它们使用的是机器字节排序。

- ARM和x86机器结构使用的是little-endian字节排序。
- MIPS机器结构使用big-endian字节排序。

在网络通信时，原生代码需要在机器字节排序和网络字节排序间做必要的转换。

socket api已经提供了一组可以方便地处理字节排序转换的函数，这些函数在sys/endian.h头文件中声明。

- htons函数：将unsigned short从主机字节排序转换到网络字节排序。
- ntohs函数：和htons相反，将unsigned short从网络字节排序转换到主机字节排序。
- htonl函数：将unsigned integer从主机字节排序转换到网络字节排序。
- ntohl函数：和htonl函数相反，将unsignedinteger从网络字节排序转换到主机字节排序。

## 监听进入的连接

监听socket是通过listen函数完成的。

```c
int listen(int socketDescriptor,int backlog);
```

- socketDescriptor：指定应用程序想要监听的输入链接socket实例。
- backlog：指定保存挂起的输入连接的队列大小。如果应用程序正在忙于为客户端服务，其他输入连接就要排队，队列中挂起的连接数的最大值由backlog指定。当输入连接数达到backlog所限定的值时，其他的输入连接将被拒绝。

若函数成功，则返回0，否则返回-1，且errno全局变量被设置为相应的错误码。

示例代码：

```c
if(-1 == listen(sd,backlog)){
	//异常处理
}
```

## 接收传入连接

accept函数用来显式地将输入连接从监听队列里取出并接受它。

```c
int accept(int socketDescriptor, struct sockaddr* address, socklen_t* addressLength);
```

accept函数是一个阻塞函数。如果在监听队列中没有即将带来的输入连接请求，它会使调用进程进入挂起状态，直到有新的输入连接到达。函数参数如下：

- socket desctiptor：指定应用程序想要从其上接收输入连接的socket实例。
- address指针：提供了一个地址结构，在该结构中填入被连接的客户协议地址。如果应用程序不需要该信息，可以设置为NULL。

如果accept请求成功，该函数返回连接的socket实例，否则返回-1且全局变量errno被设置为对应的错误码。

示例代码：

```c
struct sockaddr_in address;
socklen_t addressLength = sizeof(address);

//阻塞和等待进来的客户端连接
int clientSocket = accept(sd,(struct socketaddr*) &address,&addressLength);

如果客户socket无效
if(-1 == clientSocket){
	//抛出异常
}
else{
	//处理请求
}
```

## 接收数据

用recv函数实现从socket接收数据。

```c
ssize_t recv(int socketDescriptor,void* buffer,size_t bufferLength,int flags);
```

recv函数也是一个阻塞函数。如果没有从给定的socket接收到数据，它会使调用进程进入挂起状态，直到接收到可用数据。函数参数说明如下：

- socket descriptor：接收数据的socket实例。
- buffer指针：指向内存地址的指针，该内存用来保存从socket接收的数据。
- buffer长度：指定缓存区的大小，recv函数只会向缓存区中写入该函数指定大小的内容然后返回。
- flags：指定接收所需要的额外标志。

如果recv函数成功，它将会返回从socket那里接收到的字节数；否则返回-1，且全局变量errno将被设置为相应的错误码。如果该函数返回0，表示socket连接失败。

示例代码：

```c
//阻塞并接收来自socket的数据，并放到缓冲区
ssize_t recvSize = recv(sd, buffer,bufferSize - 1,0);

//如果接收失败
if(-1 == recvSize){
	//错误处理
}
else{
	//以NULL结尾缓存区形成一个字符串
	buffer[recvSize] = NULL;
	//如果数据接收成功
	if(recvSize > 0){
		
	}else{
	
	}
}
```

## 向socket发送数据

向socket发送数据是使用send函数来完成的。

```c
ssize_t send(int socketDescriptor,void* buffer,size_t bufferLength,int flags);
```

与recv函数一样，send函数也是一个阻塞函数。如果socket在忙着发送数据，它会使调用进程进入挂起状态直到socket可以传输数据。函数参数描述如下：

- socket descriptor：指定应用程序想要向其发送数据的socket实例。
- buffer指针：指向内存地址的buffer指针，该内存是给定的socket发送数据的目的地。
- buffer length：指定缓存区的大小。send函数只会向缓冲区传输该参数所指定大小的数据然后返回。
- flags：指定发送所需要的额外标志。

如果发送操作成功，send函数会返回传送的字节数；否则返回-1，且全局变量errno将被设置为相应的错误。与recv函数一样，如果send函数返回0，表示socket连接失败。

示例代码：

```c
ssize_t sentSize = send(sd,buffer,bufferSize,0);

//如果发送失败
if(-1 == sentSize){
	//异常处理
}else{
	if(sentSize > 0){
		//发送成功
	}
	else{
		//连接失败
	}
}
```

