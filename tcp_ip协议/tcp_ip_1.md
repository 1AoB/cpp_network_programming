参考《TCP IP网络编程》.((韩)尹圣雨).[PDF]@jb51.net.pdf



```cpp
套接字socket
使用socket创建电话机
#include<sys/socket.h>
int socket(int domain,int type,int protocol);
domain:协议族:IPV4还是IPV6
type:套接字数据传输类型信息:SOCK_STREAM表示流式传输协议,是TCP;SOCK_DGRAM是报式传输协议,是UDP
eg:socket(PF_INET,SOCK_STREAM,0);

socket函数生成套接字:
成功时返回文件描述符,失败时返回-1  
```

```cpp
调用bind函数(分配电话号码)时进行的对话
问:您的电话号码是?
答:我的电话号码是10086-123456

IP地址和端口号构成了我们通信的"电话号码"
#include<sys/socket.h>
int bind(int sockfd,struct sockaddr *myaddr,socklen_t addrlen);
成功时返回0,失败时返回-1
```

```cpp
调用listen函数(连接电话线)时进行的对话
问:已架设电话机后是否需要连接电话线?
答:对,只需要连接就能接听电话

#include<sys/socket.h>
int listen(int sockfd,int backlog);
成功返回0,失败返回-1
连接好电话线后,如果有人拨打电话就会响铃,拿起话筒才能接听电话
```

```cpp
调用accept函数(拿起话筒)时进行的对话
问:电话铃响了,我该怎么办?
答:难道您真的不知道?接听呀!(虽然有点搞笑,但是如果说对于第一次使用电话的萌新来说,情形确实如此,哈哈哈哈!!!)

(阶段性感受:TCP IP通信的步骤就像"将大象放进冰箱一样步骤分明":打开冰箱门;把大象放进去;把冰箱门关上)

#include<sys/socket.h>
int accept(int sockfd,struct sockaddr *addr,socklen_t *addrlen);
成功时返回接听到的文件描述符,失败返回-1


对上面几步进行总结:
第一步:调用socket函数创建套接字
第二步:调用bind函数分配IP地址和端口号
第三步:调用listen函数转为可接收请求的状态
第四步:调用accept函数受理连接请求
```

# 服务端代码示例

```cpp
接下来进行实战:编写"Hello world"服务端
//hello_server.c
#include<stdio.h>
#include<stdlib.h>
#include<string.h>
#include<unistd.h>
#include<arpa/inet.h>
#include<sys/socket.h>

void error_handing(char *message);

int main(int argc,char *argv[])//argc为参数个数,其中argv[0]表示这个程序的名字,argv[1]输入的是端口号
{
	int serv_sock;
	int clnt_sock;
	
	struct sockaddr_in serv_addr;
	struct sockaddr_in clnt_addr;
	socklen_t clnt_addr_size;
	
	char message[]="Hello world!";
	if(argc != 2)
	{
		printf("Usage : %s <port>\n",argv[0]);
		exit(1);
	}
	
	ser_sock = socket(PF_INET,SOCK_STREAM,0);//使用socket函数创建套接字
	if(serv_sock == -1)
		error_handing("socket() error");
	
	memset(&serv_addr,0,sizeof(serv_addr));
	serv_addr.sin_family = AF_INET;
	serv_addr.sin_addr.s_addr=htonl(INADDR_ANY);
	serv_addr.sin_port = htons(atio(argv[1]));//atio用于将字符串转换成整数
	
    //调用bind函数分配IP地址和端口号
	if( bind(serv_sock,(struct sockaddr*)&serv_addr,sizeof(serv_addr)) ==-1)
		error_handling("listen() error");
	
    //listen函数将套接字转为可接收连接状态
	if( listen(serv_sock,5)==-1 )
		error_handling("listen() error");
	
    //调用accept函数将套接字受理连接请求.如果在没有连接的情况下调用该函数,则不会返回,直到有连接请求为止
	clnt_addr_size = sizeof(clnt_addr);
	clnt_sock=accpet(serv_sock,(struct sockaddr*)&clnt_addr,&clnt_addr_size);
	if(clnt_sock == -1)
		error_handling("accept() error");
	//write函数用于传输数据
	write(clnt_sock,message,sizeof(message));
	close(clnt_sock);
	close(serv_sock);
	
	retrun 0;
}
void error_handling(char *mesage)
{
	fputs(message,stderr);
	fputc('\n',stderr);
	exit(1);
}
```

# 客户端代码示例

```cpp
创建打电话的套接字

上面没有提到connect函数,这是因为其调用是客户端套接字
#include<sys/socket.h>
int connect(int sockfd,struct sockaddr *serv_addr , socklen_t addrlen);
成功时返回0,失败时返回-1

客户端程序只有两个步骤:"调用socket函数创建套接字";"调用connect函数向服务器端发送连接请求"
如下客户端:
第一,调用socket函数和connect函数;
第二,与服务端共同运行已发收字符串数据

//hello_client.c
#include<stdio.h>
#include<stdlib.h>
#incluce<string.h>
#include<unistd.h>
#include<arpa/inet.h>
#include<sys/socket.h>

void error_handling(char *message);
int main(int argc,char* argv[])
{
	int sock;
	stuct sockaddr_in serv_addr;
	char message[30];
	int str_len;
	
	if(argc != 3)//程序名,ip,port
	{
		printf("Usage : % s <IP> <port>\n",argv[0]);
		exit(1);
	}
	
	//创建套接字,但是此时套接字并不马上分为服务端和客户端.如果紧接着调用bind和listen,将成为服务端的套接字;如果调用connect函数,将成为客户端套接字
	sock = socket(PF_INET,SOCK_STREAM,0);
	if(sock == -1)
		error_handling("socket() error");
	
	memset(&serv_addr,0,sizeof(serv_addr));
	serv_addr.sin_family=AF_INET;
	serv_addr.sin_addr.s_addr=inet_addr(argv[1]);//ip
	serv_addr.sin_port=htos(atoi(argv[2]));//port
	
	//调用connect函数向服务端发送连接请求.
	if( connect(sock,(struct sockaddr*)&serv_addr,sizeof(serv_addr)) == -1)
		error_handling("connect() error!");
	
	//read 函数最多只会读取到 sizeof(message)-1 个字节的数据，保证了缓冲区不会溢出。最后一个字符用于存储字符串的结尾标记 \0
	str_len = read(sock,message,sizeof(message)-1);
	if(str_len == -1)
		error_handling("read() error!");
		
	printf("Message from server :%s \n",message);
	close(sock);
	return 0;
}

void error_handling(char *message)
{
	fputs(message,stderr);
	fputc('\n',stderr);
	exit(1);
}
```
对于fd : 在Window中,称为"句柄";在Linux中,则称为"描述符"

```cpp
打开文件
#include <unistd.h>
int open(const char *path,int flag);
成功时返回文件描述符,失败返回-1
```
关于参数flag:

![1691080548788](C:\Users\wxn\AppData\Roaming\Typora\typora-user-images\1691080548788.png)




```cpp
关闭文件
#include<unistd.h>
int close(int fd);
成功时返回0,失败时返回-1
    
注意:该函数不仅可以关闭文件,还可以关闭套接字.这再次证明了"Linux操作系统不区分文件与套接字"的特点
```

```cpp
将数据写入文件
#include<unistd.h>
ssize_t write(int fd,const void * buf,size_t nbytes);
成功时返回写入的字节数,失败时返回-1
```

```cpp
知识补充:以_t为后缀的数据类型
size_t是通过typedef声明的unsigned int 类型.
对于ssize_t来说,size_t前面多加的s代表signed,即ssizet_t是通过typedef声明的signed int类型

size_t,ssize_t这些都是元数据类型,在sys/types.h头文件中一般由typedef声明定义
(int在16位系统中占两字节;在32位或64位系统中占4字节)
```

```cpp
写一个例子:打开一个txt文件,并写入文件
//low_open.c
#include<stdio.h>
#include<stdlib.h>
#include<fcntl.h>
#include<unistd.h>
void error_handling(char *message);
int main(void)
{
    int fd;
    char buf[]="Let's go!\n";
    
    fd=open("data.txt",O_CREAT|O_WRONLY|O_TRUNC);
    if(fd==-1)
        error_handling("open() error!");
    printf("file descriptor : %d \n",fd);//打开的文件描述符
    
    if(write(fd,buf,sizeof(buf))==-1)
        error_handling("write() error!");
    close(fd);
    return 0;
}
void error_handling(char *message)
{
    fputs(message,stderr);
    fputc('\n',stderr);
    exit(1);
}
```

```cpp
读取文件中的数据
read函数用来输入(接收)数据
#include<unistd.h>
ssize_t read(int fd,void *buf,size_t nbytes);
成功时返回接收的字节数(但遇到文件结尾则返回0),失败时返回-1

//low_read.c
#include<stdio.h>
#include<stdlib.h>
#include<fcntl.h>
#include<unistd.h>

#define BUF_SIZE 100
void error_handling(char *message);
int main(void)
{
	int fd;
	char buf[BUF_SIZE];
	
	fd = open("data.txt",O_RDONLY);//打开读取专用文件data.txt
	if(fd==-1)
		error_handling("open() error!");
	printf("file descriptor : %d \n",fd);
	
	if( read(fd,buf,sizeof(buf))==-1 )//使用read函数向第11行中声明的数组buf保存读入的数据
		error_handling("read() error!");
	printf("file data: %s",buf);
	close(fd);
	return 0;
}
void error_handling(char *message)
{
	fputs(message,stderr);
	fputc('\n',stderr);
	exit(1);
}
```






```cpp
文件描述符从3开始从小到大的顺序编号,因为0,1,2是分配给标准I/O的描述符
#include<stdio.h>
#include<fcntl.h>
#include<unistd.h>
#include<sys/socket.h>
int main(void)
{
	int fd1,fd2,fd3;
	fd1 = socket(PF_INET,SOCK_STREAM,0);
	fd2 = open("test.dat",O_CREAT|O_TRUNC);
	fd3 = socket(PF_INET,SOCK_DGRAM,0);
	
	printf("file descriptor 1:%d\n",fd1);
	printf("file descriptor 2:%d\n",fd2);
	printf("file descriptor 3:%d\n",fd3);
	
	close(fd1);
	close(fd2);
	close(fd3);
	
	return 0;
}
```





```cpp
关于协议(Protocol),即对话中使用的通信规则.
这不得不让我回忆起protobuf->由goole开发的一种与语言无关的数据序列化格式
```

```cpp
//tcp_client.c
#include<stdio.h>
#include<stdlib.h>
#incluce<string.h>
#include<unistd.h>
#include<arpa/inet.h>
#include<sys/socket.h>

void error_handling(char *message);
int main(int argc,char* argv[])
{
	int sock;
	stuct sockaddr_in serv_addr;
	char message[30];
	int str_len;
	int idx =0,read_len=0;
	
	if(argc != 3)//程序名,ip,port
	{
		printf("Usage : % s <IP> <port>\n",argv[0]);
		exit(1);
	}
	
	//创建TCP套接字,但是此时套接字并不马上分为服务端和客户端.如果紧接着调用bind和listen,将成为服务端的套接字;如果调用connect函数,将成为客户端套接字
	sock = socket(PF_INET,SOCK_STREAM,0);
	if(sock == -1)
		error_handling("socket() error");
	
	memset(&serv_addr,0,sizeof(serv_addr));
	serv_addr.sin_family=AF_INET;
	serv_addr.sin_addr.s_addr=inet_addr(argv[1]);
	serv_addr.sin_port=htos(atoi(argv[2]));
	
	//调用connect函数向服务端发送连接请求.
	if( connect(sock,(struct sockaddr*)&serv_addr,sizeof(serv_addr)) == -1)
		error_handling("connect() error!");
	
	//while循环中反复调用read函数,每次读取一个字节.如果read返回0,则循环条件为假,跳出while循环
	while(read_len = read(sock,&message[idx++],1))
	{
        if(str_len == -1)
            error_handling("read() error!");
        str_len += read_len;//变量read_len的值始终是1,因为第29行每次读取一个字节.跳出while循环后,str_len中存有读取的总字节数
	}	
	
	printf("Message from server :%s \n",message);
	printf("Function read call count :%d\n",str_len);
	close(sock);
	return 0;
}

void error_handling(char *message)
{
	fputs(message,stderr);
	fputc('\n',stderr);
	exit(1);
}
```

```cpp
知识补充:路由器和交换机

若想要构建网络,需要一种物理设备完成外网与本网主机之间的数据交换,这中设备便是路由器或交换机.
他们实际上也是一种计算机,只不过是为特殊目的而设计运行的,因此就有了别名.
所以,如果在我们使用的计算机上安装适当的软件,也可以将其用作交换机.

另外,交换机比路由器功能要简单一点,而实际用途差别不大.
```

![1690996182042](C:\Users\wxn\AppData\Roaming\Typora\typora-user-images\1690996182042.png)

![1690996266467](C:\Users\wxn\AppData\Roaming\Typora\typora-user-images\1690996266467.png)

对于sockaddr_in的成员分析:

成员sin_family:用宏定义来表示用到的枚举值AF_INET,AF_INET6,AF_LOCAL

成员sin_port:16位端口号,它应该以网络字节序保存

成员sin_addr:该成员保存32位地址信息,且**也以网络字节序保存.**为理解好该成员,应该同时观察结构体in_addr.但结构体in_addr声明为uint32_t,因此只需当做32位整型即可.

成员sin_zero:无特殊含义.代码中不用给它赋值.它只是为使结构体sockaddr_in的大小与sockaddr结构体保持一致而插入的成员.且必须填充为0,否则无法得到想要的结果(这也是为什么要memset(&serv_addr,0,sizeof(serv_addr));)

最后将封装好的sockaddr_in强转为sockaddr型的结构体变量,再传递给bind函数即可

```cpp
字节序与网络字节序
大端字节序:高位字节存放低位地址
小端字节序:高位字节存放高位地址

#include<stdio.h>
#include<arpa/inet.h>

int main(int argc,char *argv[])
{
    unsigned short host_port=0X1234;
    unsigned short net_port;
    unsigned short host_addr = 0x12345678;
    unsigned short net_addr;
    
    net_port = htos(host_port);
    net_addr = htol(host_addr);
    
    printf("Host ordered port:%#x \n",host_port);//0x1234
    printf("Network ordered port:%#x \n",net_port);
    printf("Host ordered address:%#x \n",host_addr);
    printf("Network ordered address:%#x \n",net_addr);
    
    return 0;
}
```

```cpp
将"字符串信息"转换为网络字节序的整数型

对于IP地址来说,他是点分十进制,而非整数,
这就无法使用htol()函数,幸运的是,系统提供了inet_addr,
该函数会帮我们将字符串形式的IP地址转换为32位整型数据,
且,此函数在转换类型的同时进行网络字节序转换.

#include<arpa/inet.h>
in_addr_t inet_addr(const char* string);
成功时返回32位大端序整数型值,失败时返回INADDR_NONE
eg:如果向该函数传递类似"211.214.107.99"的点分十进制的字符串,他就会将其转换为32位整型数据并返回.

//inet_addr.c
#include<stdio.h>
#include<arpa/inet.h>

int main(int argc,char *argv[])
{
    char *addr1="1.2.3.4";
    char *addr2="1.2.3.256";//1个字节能表示的最大整数为255,也就是说错误的ip地址.利用该错误地址验证inet_addr函数的错误检测能力.
  
    unsigned long conv_addr=inet_addr(addr1);//调用正常
    if(conv_addr == INADDR_NONE)
        printf("Error occured! \n");
    else
        printf("Network ordered integer addr: %1x \n",conv_addr);
    
    conv_addr=inet_addr(addr2);//调用异常
    if(conv_addr == INADDR_NONE)
        printf("Error occured! \n");
    else
        printf("Network ordered integer addr: %1x \n\n",conv_addr);
    
    return 0;
}
```



```cpp
inet_aton函数与inet_addr函数在功能上完全相同,
也是将字符串的ip地址转换为32位网络字节序整数并返回.
只不过该函数利用了in_addr结构体,且其使用频率更高.
    
    
#include<arpa/inet.h>
int inet_aton(const char *string,struct in_addr *addr);
成功时返回1(true),失败时返回0(false)
    
    
//inet_aton.c
#include<stdio.h>
#include<stdlib.h>
#include<arpa/inet.h>
    
    
void error_handling(char *message);
int main(int argc,char argv[])
{
    char *addr = "127.232.124.79";
    struct sockaddr_in addr_inet;
    
    if(!inet_aton(addr,&addr_inet.sin_addr))//const char *string,struct in_addr *addr
        error_handling("Conversion error");
    else
        printf("Network ordered integer addr : %#x \n",
              addr_inet.sin_addr.s_addr);
    return 0;
}
void error_handling(char *message)
{
    fputs(message,stderr);
    fputc('\n',stderr);
    exit(1);
}
```

```cpp
与inet_aton函数刚好相反的函数:inet_ntoa

inet_ntoa的作用:此函数可以把网络字节序整数型IP地址转换为我们熟悉的字符串形式并返回
#include<arpa/inet.h>
char * inet_ntoa(struct in_addr adr);
成功时返回转换的字符串地址值,失败返回-1
 
//inet_ntoa.c
#include<stdio.h>
#include<string.h>
#include<arpa/inet.h>
int main(int argc,char *argv[])
{
    struct sockaddr_in addr1,addr2;
    char *str_ptr;
    char str_arr[20];
    
    addr1.sin_addr.s_addr = htonl(0x1020304);//变成网络字节序
    addr2.sin_addr.s_addr = htonl(0x1010101);//变成网络字节序
    
    str_ptr=inet_ntoa(addr1.sin_addr);//变成主机字节序
    strcpy(str_arr,str_ptr);
    printf("Dotted-Decimal notation1:%s \n",str_ptr);//1.2.3.4
    
    inet_ntoa(addr2.sin_addr);
    printf("Dotted-Decimal notation2:%s \n",str_ptr);//1.1.1.1
    printf("Dotted-Decimal notation3:%s \n",str_arr);//1.2.3.4
    return 0;
}
```
```cpp
网络地址初始化的方法

客户端和服务端的写法只有在初始化ip时才不一样:

struct sockaddr_in addr;
//声明IP地址字符串
char * serv_ip = "211.217.168.13";
//声明端口号字符串
char * serv_port = "9190";
//结构体变量addr的所有成员初始化为0
memset(&addr,0,sizeof(addr));
//指定地址族
addr.sin_family = AF_INET;

//基于字符串的ip地址初始化
addr.sin_addr.s_addr = inet_addr(serv_ip);//如果是服务端是这样写:serv_addr.sin_addr.s_addr=htonl(INADDR_ANY);

//基于字符串的端口号初始化
addr.sin_port = htons(atoi(serv_port));
```
```cpp
关于INADDR_ANY

每次创建服务端套接字都要输入IP地址有些繁琐,此时可以如下初始化地址信息
struct sockaddr_in addr;
char * serv_port = "9190";
memset(&addr,0,sizeof(addr));
addr.sin_family=AF_INET;
addr.sin_addr.s_addr=htonl(INADDR_ANY);//可自动获取运行服务端的计算机IP地址,不必亲自输入;服务端优先考虑这种方式
addr.sin_port=htos(atoi(serv_port));
```
```cpp
127.0.0.1是回送地址,指的是计算机自身IP地址

./hclient 127.0.0.1 9190
```
```cpp
向套接字分配网络地址

//bind是服务端用的
#include<sys/socket.h>
int bind(int sockfd,struct sockaddr * myaddr,socklen_t addrlen);
成功返回0,失败返回-1
    
eg:
int serv_sock;
struct sockaddr_in serv_addr;
char * serv_port = "9190";

/*创建服务器端套接字(监听套接字)*/
serv_sock = socket(PF_INET,SOCK_STREAM,0);

/*地址信息初始化*/
memset(&serv_addr,0,sizeof(serv_addr));
serv_addr.sin_family = AF_INET;
serv_addr.sin_addr.s_addr = htonl(INADDR_ANY);
serv_addr.sin_port = htos(atoi(serv_port));

/*分配地址信息*/
bind(serv_sock,(struct sockaddr *)&serv_addr,sizeof(serv_addr));
```
# 实现基于TCP的服务端/客户端

## TCP服务器端的默认函数调用顺序

![1691080760316](C:\Users\wxn\AppData\Roaming\Typora\typora-user-images\1691080760316.png)



```cpp
进入等待连接请求状态

#include<sys/socket.h>
int listen(int sock,int backlog);
成功时返回,失败时返回-1
```

```cpp
受理客户端连接请求

下面这函数将自动创建套接字,并连接到发起请求的客户端

#include<sysy/socket.h>
int accept(int sock,struct sockaddr * addr,socklen_t addrlen);
成功时返回创建的套接字文件描述符,失败时返回-1
```
## TCP客户端的默认函数调用顺序

![1691081098149](C:\Users\wxn\AppData\Roaming\Typora\typora-user-images\1691081098149.png)

```cpp
关于connect

#include<sys/socket.h>
int connect(int sock,struct sockaddr * servaddr,socklen_t addrlen);
成功时返回0,失败时返回-1
    
    
客户端调用connect后,发送以下情况之一会返回(完成函数调用).
    服务端接收连接请求
    发生断网等异常情况而中断连接请求
    
(需要注意的是,所谓的"接收连接"并不意味着服务器端调用accept函数,其实是服务器端把连接请求信息记录到等待队列.
因此connect函数返回后并不立即进行数据交换)
```
```cpp
知识补充:
客户端套接字地址信息在哪?
想搞懂这问题,不妨问如下三问:
客户端套接字何时,何地,如何分配地址呢?
    何时?调用connect函数时.
    何地?操作系统,更准确地说是在内核中.
    如何?IP用计算机(主机)的IP,端口随机.
    
    
    
(客户端的IP地址和端口在调用connect函数时自动分配,无需调用绑定的bind函数进行分配)
```





## 基于TCP的服务器端/客户端函数调用关系

![1691152358602](C:\Users\wxn\AppData\Roaming\Typora\typora-user-images\1691152358602.png)

# 实现迭代服务器端/客户端


```cpp
本节任务:编写一个回声服务器端/客户端,顾名思义,就是服务器端将客户端传输的字符串数据原封不动地传回客户端,就像回声一样.
    
    
    
    
首先要实现迭代服务器端:
之前的Hello world服务器端处理完一个客户端连接请求即退出,连接请求等待队列实际上没有太大意义.但这并非我们想象的服务器端.设置好等待队列的大小后,应向所有客户端提供服务.如果想要继续受理后序的客户端连接请求,应该进行相关的代码扩展.
    那么如何扩展呢?最简单的办法就是:插入循环语句反复调用accept函数.
    就目前而言,我们假设我们只会单线程,那么同一时刻就只能服务一个客户端.
    
服务端/客户端的基本运行方式:
	服务器在同一时刻值与一个客户端相连,并提供回声服务
	服务器一次向我国客户端提供服务并退出
	客户端接收用户输入的字符串并发送给服务器端
	服务器将接受的字符串数据传回客户端,即"回声"
    服务端与客户端之间的字符串回声一直执行到客户端输入Q为止

//echo_server.c
#include<stdio.h>
#include<stdlib.h>
#include<string.h>
#include<unistd.h>
#include<arpa/inet.h>
#include<sys/socket.h>

#define BUF_SIZE 1024
void error_handling(char * message);
int  main(int argc,cha *argv[])
{
	int serv_sock , clnt_sock;
	char message[BUFF_SIZE];
	int str_len,i;
	
	struct sockaddr_in serv_adr,clnt_adr;
	socklen_t clnt_adr_sz;
	
	if(argc != 2)
	{
		printf("Usage :%s <port>\n",argv[0]);
		exit(1);
	}
	
	serv_sock= sock(PF_INET,SOCK_STREAM,0);
	if(serv_sock == -1)
	{
		error_handling("socket() error");
	}
	
	memset(&serv_adr,0,sizeof(serv_adr));
	serv_adr.sin_family = AF_INET;
	serv_adr.sin_addr.s_addr=htonl(INADDR_ANY);
	serv_adr.sin_port = htos(atoi(argv[1]));
	
	if(bind(serv_sock,(struct sockaddr*)&serv_adr,sizeof(serv_adr)) ==-1)
	error_handling("bind() error");
	
	if(listen(serv_sock,5) == -1)
		error_handling("listen() error");
		
	clnt_adr_sz=sizeof(clnt_adr);
	
	for(i = 0 ; i < 5 ; i ++)
	{
		clnt_sock = accept(serv_sock,(struct sockaddr*)&clnt_adr,&clnt_adr_sz);
		if(clnt_sock == -1)
			error_handling("accept() error");
		else
			printf("Connected clinet %d \n",i+1);
			
		while( (str_len = read(clnt_sock,message,BUFF_SIZE)) != 0)
        {
            write(clnt_sock,message,str_len);
        }	
       	/*
       	而在服务器端，read() 函数读取客户端发送来的消息，并立即写回到客户端，没有对读取到的数据进行字符串处理（如打印或者其他字符串操作），因此不需要留出空间给字符串结束符 '\0'。
       	所以,如果要读取字符串的话,就要做如下判断
       	while( (str_len = read(clnt_sock,message,BUFF_SIZE)) != 0)
        {
        	if(str_len<BUFF_SIZE)
        		message[str_len] = '\0'; // 手动添加字符串结束标志
            else if(str_len==BUFF_SIZE)
            	message[str_len-1] = '\0'; // 手动添加字符串结束标志
            
            printf("Message from server :%s\n",message);
            
            write(clnt_sock,message,str_len);
        }
       	*/
        close(clnt_sock);
	}
    close(serv_sock);
	return 0;
}	
void error_handling(char * message)
{
	fputs(message,stderr);
	fputc('\n',stderr);
	exit(1);
}




//echo_client.c
#include<stdio.h>
#include<stdlib.h>
#include<string.h>
#include<unistd.h>
#include<arpa/inet.h>
#include<sys/socket.h>

#define BUF_SIZE 1024
void error_handling(char * message);
int  main(int argc,cha *argv[])
{
	int sock;
    char message[BUF_SIZE];
    int str_len;
    struct sockaddr_in serv_adr;
    
    if(argc != 3)
    {
        printf("Usage ; % <IP> <Port> \n",argv[0]);
        exit(1);
    }
    
    sock = socket(PF_INET,SOCK_STREAM,0);
    if(sock == -1)
    {
        error_handling("socket() error");
    }
    
    memset(&serv_adr , 0 ,sizeof(serv_adr));
    serv_adr.sin_family = AF_INET;
    serv_adr.sin_addr.s_addr = inet_addr(argv[1]);
    serv_adr.sin_port = htos(atoi(argv[2]));
    
    if(connet(sock,(struct sockaddr*)&serv_adr,sizeof(serv_adr)) ==-1)
        error_handling("connect() error");
    else
    {
        printf("Connect...\n");
    }
    
    while(1)
    {
        fputs("Inputs message(Q to quit):",stdout);
        fgets(messge,BUF_SIZE,stdin);
        
        if(!strcmp(message,"q\n") !! !strcmp(message,"Q\n"))
            break;
        
        write(sock,message,strlen(message));
        str_len = read(sock,message,BUF_SIZE-1);
        /*
        在客户端代码中，read() 函数读取服务器返回的消息，并把它存储在 message 数组中。BUF_SIZE-1 是为了给字符串的结束符 '\0' 留出空间，这样 message 就可以被当作一个字符串来处理（使用 printf() 函数打印）。
        */
        message[str_len] = 0;
        printf("Message from server :%s\n",message);
        
    }
    close(sock);
	return 0;
}	
void error_handling(char * message)
{
	fputs(message,stderr);
	fputc('\n',stderr);
	exit(1);
}




上面的回声客户端存在的问题(P76):
	假设服务端有如下情况:
		"字符串太长,需要分两个数据包发送!"
     服务端希望通过调用一次write函数传输数据,但是如果数据太大,操作系统就有可能把数据分成多个数据包发送给客户端.另外,在此过程中,客户端可能在尚未收到全部数据包时就用read函数.(经过"专家"考查,将这个问题命名为"分包问题")
      产生这个问题的原因是:"TCP不存在数据边界"
          
      关于分包粘包这个问题,可以使用封装数据头来进行解决(以前学的),而这本书的第五章对分包问题的解决办法提出了一种更简单的办法("但是那种解决办法好像无法解决粘包问题")
```

```cpp
第五章:

标题:回声服务器的完美实现

就上面的回声服务端/客户端代码而言:
	服务端100%将自己接收到的数据传回,只不过客户端在接收时有些问题:
回声客户端传输的是字符串,而且是通过调用write函数一次性发送的.之后还调用了一次read函数,期待接收自己传输的字符串,这就是问题所在.
    "既然回声客户端会收到所有字符串数据,是否只需多等一会儿?过一段时间后再调用read函数是否可以一次性读取所有字符串数据?"
    过一会再接收这当然可以,但是需要等多长时间呢?要10分钟吗?这不符合常理,理想的客户端应该在收到字符串数据时立即读取并返回
    
    
    
回声客户端问题的解决办法:
其实很容易解决,因为可以提前确定接收数据的大小.若之前传输了20字节长的字符串,则在接收时循环调用read函数读取20个字节即可.
代码如下:
//echo_client2.c
#include<stdio.h>
#include<stdlib.h>
#include<string.h>
#include<unistd.h>
#include<arpa/inet.h>
#include<sys/socket.h>
#define BUF_SIZE 1024
#define RLT_SIZE 4
#define OPSZ 4
void error_handling(char * message);
int main(int argc,char *argv[])
{
    int sock;
    char message[BUF_SIZE];
    int str_len,recv_len,recv_cnt;
    struct sockaddr_in serv_adr;
    
    if(argc != 3)
    {
        printf("Usage : %s <IP> <port>\n",argv[0]);
        exit(1);
    }
    
    sock = socket(PF_INET,SOCK_STREAM,0);
    if(sock == -1)
        error_handling("socket() error");
    
    memset(&serv_adr,0,sizeof(serv_adr));
    serv_adr.sin_family = AF_INET;
    serv_adr.sin_addr.s_addr=inet_addr(argv[1]);
    serv_adr.sin_port = htos(atoi(argv[2]));
    
    if(connect(sock , (struct sockaddr*)&serv_adr, sizeof(serv_adr)) == -1)
    {
       error_handling("connect() error"); 
    }else
    {
        puts("Connected ...");
    }
    
    fputs("Operand count: ",stdout);
    scanf("%d",&opnd_cnt);
    opmsg[0]=(char)opnd_cnt;
    
    for(i=0;i<opnd_cnt;i++)
    {
        printf("Operand %d: ",i+1);
        scanf("%d",(int*)&opmsg[i*OPSZ+i]);
    }
    fgetc(stdin);
    fputs("Operator: ",stdout);
    scanf("%c",&opmsg[opnd_cntt*OPSZ+1]);
    write(sock,opmsg,opnd_cntt*OPSZ+2);
    read(sock,&result,RL_SIZE);
    
    printf("Operation result:%d \n",result);
    close(sock);
    
	return 0;
}

void error_handling(char *message)
{
    fputs(message,stderr);
    fputc('\n',stderr);
    exit(1);
}

//op_server.c
#include<stdio.h>
#include<stdlib.h>
#include<string.h>
#include<unistd.h>
#include<arpa/inet.h>
#include<sys/socket.h>
#define BUF_SIZE 1024
#define OPSZ 4
void error_handling(char *message);
int calculate(int opnum,int opnds[],char oprator);
int main(int argc,char *argv[])
{
	int serv_sock,clnt_sock;
    char opinfo[BUF_SIZE];
    int result,opnd_cnt,i;
    int recv_cnt,recv_len;
    struct sockaddr_in serv_adr,clnt_adr;
    socklen_t clnt_adr_sz;
    if(argv != 2)
    {
        printf("Usage : %s <port>\n",argv[0]);
        exit(1);
    }
    
    serv_sock=socket(PF_INET,SOCK_STREAM,0);
    if(serv_sock == -1)
        error_handling("socket() error");
    memset(&serv_adr,0,sizeof(serv_adr));
    serv_adr.sin_family = AF_INET;
    serv_adr.sin_addr.s_addr=htonl(INADDR_ANY);
    serv_adr.sin_port=htos(atoi(argv[1]));
    
    if(bind(serv_sock,(struct sockaddr*)&serv_adr,sizeof(serv_adr))==-1)
        error_handling("bind() error");
    
    if(listen(serv_sock,5)==-1)
        error_handling("listen() error");
    
    clnt_adr_sz=sizeof(clnt_adr);
    
    for(i = 0; i < 5 ; i++)
    {
        opnd_cnt = 0;
        clnt_sock=accept(serv_sock,(struct sockaddr*)&clnt_adr,&clnt_adr_sz);
        read(clnt_sock,&opnd_cnt,1);
        
        recv_len = 0;
        while( (opnd_cnt*OPSZ+1)>recv_len )
        {
            recv_cnt = read(clnt_sock,&opinfo[recv_len],BUF_SIZE-1);
            recv_len += recv_cnt;
        }
        result=calculate(opnd_cnt,(int*)opinfo,opinfo[recv_len-1]);
        write(clnt_sock,(char*)&result,sizeof(result));
        close(serv_sock);
    }
    close(serv_sock);
    return 0;
}
int calculate(int opnum,int opnds[],char oprator)
{
    int result = opnds[0],i;
    switch(op)
    {
        case '+':
            for(i=1;i<opnum;i++)result+=opnds[i];
            break;
        case '-':
            for(i=1;i<opnum;i++)result+=opnds[i];
            break;
        case '*':
            for(i=1;i<opnum;i++)result+=opnds[i];
            break;
    }
    return result;
}
void error_handling(char *message)
{
    fputs(message,stderr);
    fputc('\n',stderr);
    exit(1);
}
```

```cpp
TCP套接字中的I/O缓冲

TCP套接字的数据收发无边界.服务器端即使调用1次write函数传输40字节的数据,客户端也有可能通过4次read函数调用每次读取10字节.
此处也有一些疑问,服务器端一次性传输了40字节,而客户端居然可以缓慢地分批接收接收.那么,客户端接收10字节后,剩下的30字节在何处等候呢?是不是想飞机为等待着陆而在空中盘旋一样,剩下30字节也在网络中徘徊并等待接收呢?
    实际上,write函数调用后并非立即传输数据,read函数调用也并非马上接收数据.更准确的是,write函数调用瞬间,数据将移至输出缓存;read函数调用瞬间,从输入缓冲读取数据.
    也就是说,调用write函数时,数据将移到 输出缓存 ,在适当的时候(不管是分批传送还是一次传送)传向对方的输入缓冲.这时对方调用read函数从输入缓冲读取数据.
        
        
    那么,下面这种情况会引发什么事情?理解I/O缓冲后,各位应该可以猜出其流程:
"客户端输入缓冲为50字节,而服务器端传输了100字节".
   而实际上,我们直接给出结论:"不会发生超过输入缓冲大小的数据传输".也就是说,根本不会发生这类问题,因为TCP会控制数据流.TCP中有滑动窗口协议,用对话方式呈现如下:
套接字A:“你好，最多可以向我传递50字节。”
套接字B:“OK!”
套接字A:“我腾出了20字节的空间，最多可以收70字节。”
套接字B:“OK!”
数据收发也是如此,因此TCP中不会因为缓冲溢出而丢失数据.(再强调一遍:因为有滑动窗口协议来控制数据流)
```


```cpp
TCP内部工作原理1:与对方套接字的连接
TCP套接字从创建到消失所经过程分为如下3步。
1.与对方套接字建立连接。
2.与对方套接字进行数据交换。
3.断开与对方套接字的连接。
首先讲解与对方套接字建立连接的过程。连接过程中套接字之间的对话如下。
[Shake 1]套接字A:“你好，套接字B。我这儿有数据要传给你，建立连接吧。”
[Shake 2]套接字B:“好的，我这边已就绪。”
[Shake 3]套接字A:“谢谢你受理我的请求。”

实际上,TCP在通信过程中也会经过3次对方过程,因此,该过程又称"三次握手".
```

![1691684423612](C:\Users\wxn\AppData\Roaming\Typora\typora-user-images\1691684423612.png)

```cpp
套接字是以全双工方式工作的,也就是说可以双向传输数据.因此,收发数据前需要做一些准备.首先,请求连接的主机A向主机B传递如下信息:

[SYN]SEQ:1000,ACK:-(用大白话说就是:如果你这条消息[这条消息的数据包序号是1000,就和身份证id一样]接收无误,请  通知我 可以给你发1001号数据包;)[注意看: 是通知我发给你 !]

该消息中SEQ为1000,ACK为空,而SEQ为1000的含义如下:

"现传递的数据包序号为1000,如果接收无误,请  通知 我  向您  传递1001号数据包".

这是首次请求连接时使用的消息,又称SYN.( SYN表示收发数据前传输的同步消息 ).接下来主机B向A传递如下消息:



[SYN+ACK] SEQ:2000,ACK:1001 (用大白话说就是:如果你这条消息[这条消息的数据包序号是2000,就和身份证id一样]接收无误,请  通知我 可以给你发2001号数据包;你刚才给我发的1000号数据包我收到了,你现在可以给我发1001号数据包了)

此时SEQ为2000,ACK为1001,
而SEQ为2000的含义如下:
"现传输的数据包序号为2000,如果接收无误,请通知我向您传递2001号数据包".
而ACK 1001的含义如下:
"刚才传输的SEQ为1000的数据包接收无误,现在请传输SEQ为1001的数据包".

对主机A首次传输的数据包的确认消息(ACK 1001)和为主机B传输数据做准备的同步消息(SEQ 2000)绑定发送,因此,此种类型的消息又称SYN+ACK.

收发数据前向数据包分配序号,并向对方通报此序号,这都是为防止数据丢失所做的准备.通过向数据包分配序号并确认,可以在数据丢失时马上查看并重传丢失的数据包.因此,TCP可以保证可靠的数据传输.最后观察主机A向主机B传输的消息:




[ACK] SEQ: 1001,ACK:2001(用大白话说就是:[这条消息的数据包序号是1001,就和身份证id一样,与之前不同的是:这个SEQ没有其他特别的含义];你刚才给我发的2000号数据包我收到了,你现在可以给我发2001号数据包了)

之前就说过,TCP连接过程中发送数据包时需分配序号.在之前的序号1000的基础上+1,也就是给SEQ分配1001,此时该数据包的ACK传递如下消息:
"已正确收到传输的SEQ为2000的数据包,现在请传输SEQ为2001的数据包
 <这样就传输了添加ACK为2001的ACK消息.至此,主机A和主机B确认了彼此均就绪.>
```

```cpp
TCP内部工作原理2:与对方主机的数据交换
通过第一步三次握手过程完成了"数据交换准备",下面就正式开始收发数据
```

![1691688372331](C:\Users\wxn\AppData\Roaming\Typora\typora-user-images\1691688372331.png)

```cpp
图5-4给出了主机A分2次(分2个数据包)向主机B传递200字节的过程.首先,主机A通过1个数据包发送100个字节的数据,数据包的SEQ为1200.主机B为了确认这一点,向主机A发送ACK1301消息.
    此时的ACK号位1301而非1201,原因在于ACK号的增量为传输的数据字节数.假设每次ACK号不加传输的字节数,这样虽然可以确认数据包的传输,但无法明确100字节全都正确传输还是丢失了一部分,比如只传输了80字节.因此按如下公式传递ACK消息:
ACK号->SEQ号+传递的字节数+1
    
    与三次握手协议相同,最后+1是为了告知对方下次要传递的SEQ号(大白话就是说,我收到了一些数据了,下次发给我的数据包的序号SEQ请从ACK开始传输[这次的ACK号就是"对方下次要给我发的SEQ号"]).
    下面分析传输过程中数据包消失的情况,如下图:
```

![1691689056173](C:\Users\wxn\AppData\Roaming\Typora\typora-user-images\1691689056173.png)


```cpp
图5-5表示通过SEQ为1301的数据包向主机B传递100字节的数据.但中间发生了错误,主机B未收到.经过一段时间后,主机A依旧未收到
```

```cpp

```


```cpp

```

```cpp

```

```cpp

```

```cpp

```











