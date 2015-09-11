title: winsock编程（1）——基于Tcp的Echo
date: 2014-01-25 17:00:17
tags: [Web,Code]
categories:	Socket
---
最近整理一下之前写过的一些东西，就从这里开始吧~
![image](http://d.pr/i/kkVo.jpg)
## 服务端结构：
### 1.使用socket函数获取套接字

```c
echo_soc = socket(AF_INET,SOCK_STREAM,0);
//从socket的库里随机获取，两次调用相同程序可能会得到相同的套接字
```
echo_soc的值可以看做服务器端打开的一个**通道**。

### 2.使用bind函数将一本地地址与套接口捆绑

```c
result = bind(echo_soc,( struct sockaddr*)&serv_addr,sizeof (serv_addr));
//当result = SOCKET_ERROR时，绑定出现错误，退出程序
```

### 3.listen监听端口

```c
listen(echo_soc, SOMAXCONN);
//这里SOMAXCONN为最大合理值（不太清楚啥意思）
```
**附：**
>- msdn原文第二个参数:
>- Maximum length of the queue of pending connections. If set to SOMAXCONN, the underlying service provider responsible for socket s will set the backlog to a maximum reasonable value. There is no standard provision to obtain the actual backlog value.

### 4.监听开始后服务端启动
通过accept函数接受套接字用recv和send函数收发数据
```c
while(1)
{
    acpt_soc = accept(echo_soc,( struct sockaddr*)&clnt_addr,&addr_len);
     //ip在clnt_addr中存储？
     if(acpt_soc == INVALID_SOCKET)
    {
        printf( "[Echo Sever] accept error: %d\n" ,WSAGetLastError());
         break;
    }
 
    result = recv(acpt_soc, recv_buf, ECHO_BUF_SIZE,0);
     if(result > 0)
    {
        recv_buf[result] = 0;
        printf( "[Echo Sever] receives: \"%s\",from %s\r\n" ,
            recv_buf, inet_ntoa(clnt_addr.sin_addr));
 
        result = send(acpt_soc, recv_buf, result,0);
    }
    closesocket(acpt_soc);
}
//while（1）死循环持续接受客户端的套接字,当接收成功时打印收到信息与客户端ip地址，然后用send发送收到的信息。
```
关于服务端先到这里= =

下面是服务端完整源码：
```c
#include <stdio.h>
#include <winsock2.h>
 
#pragma comment(lib,"ws2_32")
 
#define ECHO_DEF_PORT 7
#define ECHO_BUF_SIZE 256

int main(int argc,char **argv)
{
	WSADATA wsa_data;
	SOCKET echo_soc = 0,
		    acpt_soc = 0;
	struct sockaddr_in serv_addr,
			clnt_addr;
	unsigned short port = ECHO_DEF_PORT;
	int result = 0;
	int addr_len = sizeof(struct sockaddr_in);
	char recv_buf[ECHO_BUF_SIZE];
 
	if(argc == 2)
		port = atoi(argv[1]);
 
	WSAStartup(MAKEWORD(2,2),&wsa_data);
	echo_soc = socket(AF_INET,SOCK_STREAM,0);

	serv_addr.sin_family = AF_INET;
	serv_addr.sin_port = htons(port);
	serv_addr.sin_addr.s_addr = INADDR_ANY;

	result = bind(echo_soc,(struct sockaddr*)&serv_addr,sizeof(serv_addr));
	if(result == SOCKET_ERROR)
	{
		printf("[Echo Sever] bind error: %d\n", WSAGetLastError());
		closesocket(echo_soc);
		return -1;
	}
 
	listen(echo_soc, SOMAXCONN);
 
	printf("[Echo Sever] is running ……\n");
	while(1)
	{
		acpt_soc = accept(echo_soc,(struct sockaddr*)&clnt_addr,&addr_len);
		if(acpt_soc == INVALID_SOCKET)
		{
			printf("[Echo Sever] accept error: %d\n",WSAGetLastError());
			break;
		}
 
		result = recv(acpt_soc, recv_buf, ECHO_BUF_SIZE,0);
		if(result > 0)
		{
			recv_buf[result] = 0;
			printf("[Echo Sever] receives: \"%s\",from %s\r\n",
				recv_buf, inet_ntoa(clnt_addr.sin_addr));

			result = send(acpt_soc, recv_buf, result,0);
		}
		closesocket(acpt_soc);
	}
	closesocket(acpt_soc);
	WSACleanup();
 
	return 0;
}
```


## 客户端结构：
### 1.使用socket函数获取套接字
```c
echo_soc = socket(AF_INET,SOCK_STREAM,0);
```

### 2.使用connect函数与服务器进行连接
```c
result = connect(echo_soc,( struct sockaddr *)&serv_addr,sizeof (serv_addr));
//result为0时连接成功
```

**附：**
>- 第二个参数为一结构体指针其中包含服务器的ip地址端口号等信息，在本程序中端口号在宏定义中确定，ip地址从命令行获得。

### 3.使用send与recv函数进行数据收发
```c
result = send(echo_soc,test_data,send_len,0);
result = recv(echo_soc,recv_buf,ECHO_BUF_SIZE,0);
```

**附：**
```c
if(result > 0){
	recv_buf[result] = 0;
	printf( "[ECHO Client] receives: \"%s\"\r\n" ,recv_buf);
}   
else
	printf( "[ECHO Client] error: %d.\"\r\n" ,WSAGetLastError());
//result在后续函数中作为判断条件，大于0说明成功收发send、recv函数成功执行，则打印数据。
```

* 最后关闭套接字，结束winsock资源既结束。
	
下面是客户端完整源码：
```c
#include <stdio.h>
#include <stdlib.h>
#include <WINSOCK2.H>
 
#pragma comment(lib,"ws2_32")//winsock使用的函数库
 
#define ECHO_DEF_PORT   7//连接的默认端口
#define ECHO_BUF_SIZE 256//缓冲区大小
 
int main(int argc, char **argv){
 
    WSADATA wsa_data;
    SOCKET echo_soc = 0;//socket句柄
    struct sockaddr_in serv_addr;//服务器地址
    unsigned short port = ECHO_DEF_PORT;
    int result = 0, send_len = 0;
    char *test_data = "Hello World!", recv_buf[ECHO_BUF_SIZE];
   
	if(argc < 2){
        printf("input %s server_address [port]\n",argv[0]);
        return -1;
    } 
      
    if(argc >2)
        port = atoi(argv[2]);
        
    WSAStartup(MAKEWORD(2,2),&wsa_data);//初始化winsock资源
    send_len = strlen(test_data);
    
	//服务器地址
    serv_addr.sin_family = AF_INET;
    serv_addr.sin_port = htons(port);
    serv_addr.sin_addr.s_addr = inet_addr(argv[1]);
     
    if(serv_addr.sin_addr.s_addr == INADDR_NONE){
        printf("[ECHO] invalid address\n");
        return -1;
    }    
    
    echo_soc = socket(AF_INET,SOCK_STREAM,0);
    result = connect(echo_soc,(struct sockaddr *)&serv_addr,sizeof(serv_addr));
    
    if(result == 0){//连接成功
        result = send(echo_soc,test_data,send_len,0);
        result = recv(echo_soc,recv_buf,ECHO_BUF_SIZE,0);
    }
     
    if(result > 0){
        recv_buf[result] = 0;
        printf("[ECHO Client] receives: \"%s\"\r\n",recv_buf);
    }    
    else
        printf("[ECHO Client] error: %d.\"\r\n",WSAGetLastError());
        
    closesocket(echo_soc);
    WSACleanup();
    
    return 0;
}    
```

**附录：**
关于socket（套接字）
```c
typedef struct in_addr {
        union {
                struct { UCHAR s_b1,s_b2,s_b3,s_b4; } S_un_b;
                struct { USHORT s_w1,s_w2; } S_un_w;
                ULONG S_addr;
        } S_un;
#define s_addr  S_un.S_addr /* can be used for most tcp & ip code */
#define s_host  S_un.S_un_b.s_b2    // host on imp
#define s_net   S_un.S_un_b.s_b1    // network
#define s_imp   S_un.S_un_w.s_w2    // imp
#define s_impno S_un.S_un_b.s_b4    // imp #
#define s_lh    S_un.S_un_b.s_b3    // logical host
} IN_ADDR, *PIN_ADDR, FAR *LPIN_ADDR;

//第一个结构体，既
//struct { UCHAR s_b1,s_b2,s_b3,s_b4; } S_un_b;
//用于存储ip地址，其中s_b1到s_b4存储的为ip地址的四节
```

就这么多了=。=
