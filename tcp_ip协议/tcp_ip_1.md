参考《TCP IP网络编程》.((韩)尹圣雨).[PDF]@jb51.net.pdf

注:如果想要将这个md文件完整上传到github,图片是无法识别的,需要你花点时间将图片再截取一遍,然后再copy到github上

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
	
	serv_sock= socket(PF_INET,SOCK_STREAM,0);
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

void error_handling(char * message);
int main(int argc,char *argv[])
{
    int sock;
    char message[BUF_SIZE];
    int str_len,recv_len,recv_cnt;
    struct sockaddr_in serv_adr;
    if(argc != 3)
    {
        printf("Usage : <ip> <port>\n",argv[0]);
        exit(1);
    }
    
    sock = socket(PF_INET,SOCK_STREAM,0);
    if(sock == -1)
        error_handling("socket() error");
    
    memset(&serv_adr,0,sizeof(serv_adr));
    serv_adr.sin_family=AF_INET;
    serv_adr.sin_addr.s_addr=inet(argv[1]);
    serv_adr.sin_port=htons(atoi(argv[2]));
    
    if(connect(sock,(struct sockaddr*)&serv_adr,sizeof(serv_adr)) == -1)
        error_handling("connect() error");
    else
        puts("Connected ...");
    
    while(1)
    {
        fputs("Input message (Q to quit): ",stdout);
        fgets(message,BUF_SIZE,stdin);
        if(!strcmp(message,"q\n") || !strcmp(message,"Q\n"))
			break;
        
        str_len = write(sock,message,strlen(message));
        
        recv_len = 0;
        while(recv_len < str_len)
        {
            recv_cnt = read(sock,&message[recv_len,BUF_SIZE-1]);//read的第三个参数表示这次可读取的最大字节数
            if(recv_cnt == -1)
                error_handling("read() error");
            recv_len += recv_cnt;
		}
        message[recv_len] = 0;
        printf("Message from server : %s\n",message);
    }
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
如果问题不在于回声客户端:定义应用层协议

"回声客户端可以提前知道接收的数据长度,但是我们意识到:大多数情况不是这样.我们无法填预知接收的数据长度"
然如此,若无法预知接收数据长度时应如何收发数据?此时需要的就是应用层协议的定义。之前的回声服务器端/客户端中定义了如下协议。
"收到O就立即终止连接。"
同样,收发数据的过程也需要定好规则(协议)以表示数据的边界，或提前告知收发数据的大小。("这就涉及到分包粘包的解决办法:封装数据的头部信息,让对方知道自己此次要发给你多少数据")
服务器端/客户端实现过程中逐步定义的这些规则集合就是应用层协议。由此可见，应用层协议并不是高深莫测的存在，只不过是为特定程序的实现而制定的规则。

下面编写程序以体验应用层协议的定义过程。该程序中，服务器端从客户端获得多个数字和运算符信息。服务器端收到数字后对其进行加减乘运算，然后把结果传回客户端。例如，向服务器端传递3、5、9的同时请求加法运算，则客户端收到3+5+9的运算结果;若请求做乘法运算，则客户端收到3x5x9的运算结果。而如果向服务器端传递4、3、2的同时要求做减法，则客户端将收到4-3-2的运算结果,即第一个参数成为被减数。
//op_client.c
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
图5-5表示通过SEQ为1301的数据包向主机B传递100字节的数据.但中间发生了错误,主机B未收到.经过一段时间后,主机A依旧未收到对于SEQ为1301的ACK确认,因此试着重传该数据包.为了完成数据包重传,TCP套接字启动计时器以等待ACK应答.若响应计时器发生超时则重传.
```

```cpp
TCP的内部工作原理3:断开与套接字的连接

TCP套接字的结束过程也非常优雅.如果对方还有数据需要传输时直接断开连接会出问题,所以断开连接时需要双方协商.断开连接时双方对话如下:
套接字A:“我希望断开连接。”套接字B:“哦，是吗?请稍候。”
套接字B:“我也准备就绪，可以断开连接。”套接字A:“好的,谢谢合作。”

先由套接字A向套接字B传递断开连接的消息，套接字B发出确认收到的消息，然后向套接字A传递可以断开连接的消息,套接字A同样发出确认消息，如图5-6所示。
```

![1691690203718](C:\Users\wxn\AppData\Roaming\Typora\typora-user-images\1691690203718.png)


```cpp
图5-6数据包内的FIN表示断开连接.也就是说,双方各发送1次FIN消息后断开连接,此过程经历4个阶段,因此又称"四次握手"

SEQ和ACK的含义之前写过了,故省略,下面的填空你自己填.
SEQ的含义:("此次的数据包序号,如果ok,请通知我发给你SEQ+1的数据包")
ACK的含义:("刚才接收的数据包无误,下次请给我发送SEQ为ACK的数据包")

容一致，故省略。图5-6中向主机A传递了两次ACK 5001，也许这会让各位感到困惑。其实，第二次FIN数据包中的ACK 5001只是因为接收ACK消息后未接收数据而重传的。(实际上,在主机B向主机A第一次发送了ACK 5001后,主机A要和主机B断开的目的已经达到了,主机A就没必要再回主机B了[但是主机B和主机A还没断开连接,主机B还可以给主机A发生数据信息])
```







```cpp
第6章 基于UDP的服务端/客户端(P101)
    
在4层TCP/IP模型中,上数第二层传输层分为TCP和UDP这2种.(复习一下,TCP/IP四层模型分别是:应用层->传输层->网络层->网络接口层)数据交换过程可以分为通过TCP套接字完成的TCP方式和通过UDP套接字完成的UDP方式.
    
UDP套接字特点:
下面用一个寄信的例子来说明:
(它与UDP特性完全相符)
寄信前应该先在信封上填好寄信人和收信人的地址,之后贴上邮票放进邮桶即可.
(当然,信件的特点使我们无法确认对方是否收到.另外,邮寄过程中也可能发生信件丢失的情况.也就是说,信件是一种不可靠的传输方式.与之类似,UDP提供的同样是不可靠的数据传输服务)
   
    
TCP与UDP的对比:   
如果只考虑可靠性,TCP的确比UDP好.
但UDP在结构上比TCP更简洁.UDP不会发送类似ACK的应答消息,也不会像SEQ那样给数据包分配序号.因此,UDP的性能有时比TCP高很多.而且,在编程中实现UDP也比TCP要简单.另外,UDP的可靠性虽然不如TCP,但也不会像想象中那么频繁地发生数据损毁.因此,在更重视性能而非可靠性的情况下,UDP是一种很好的选择.如QQ聊天,视频直播等.
    
    
既然如此,UDP的作用到底是什么?为了提供可靠的数据传输服务,TCP在不可靠的IP层进行流控制,而UDP就缺少这种流控制机制.   
    
"提问:UDP和TCP的差异只在于流控制机制吗?"
answer:是的,流控制是区分UDP和TCP最重要的标志.但若从TCP中除去流控制,所剩内容也屈指可数了.也就是说,TCP的生命在于流控制.(之前讲过的:"与对方套接字连接及断开连接的过程"也属于流控制的一部分.)  
    
    
    
需要额外声明一点:我们把TCP比喻成电话,把UDP比喻成信件.但这只是形容协议工作方式,并没有包含数据交换速率.不要误认为"电话的速度比信件快,那么TCP的数据收发速率也比UDP快".事实上刚好相反,TCP的速率无法超过UDP,但在收发某些类型的数据时有可能接近UDP.(eg:每次交换的数据量越大,TCP的传输速率就越接近UDP的传输速率)
```

```cpp
UDP的内部工作原理:
与TCP不同的是,UDP不会进行流控制.
在UDP中,IP的作用就是让离开主机B的UDP数据包准确传递到主机A.但把UDP最终交给主机A的某一UDP套接字的过程则是由UDP完成的.UDP最重要的作用就是根据端口号将传到主机的数据包交付给最终的UDP套接字.    
    
UDP的高效使用:
TCP比UDP慢的原因通常有以下两点:
- 收发数据前后进行的连接设置及清除过程.
- 收发数据过程中为保证可靠性而添加的流控制
如果收发的数据量小但需要频繁连接时,UDP比TCP更高效.
```

```cpp
实现基于UDP的服务端/客户端

要点一:UDP中服务器端和客户端没有连接
要点二:UDP服务器端和客户端均只需1个套接字
TCP中,套接字之间应该是一对一的关系.若要向10个客户端提供服务,则除了守门的服务器套接字(也称为"监听套接字",该套接字负责监听客户端发起连接,并接受这些请求.一旦有客户端发起连接,守门的服务器套接字就会接受请求并创建一个新的服务器套接字,用于与该客户端套接字进行通信.这样,每个客户端都会有一个独立的服务器端套接字与服务器进行通信)外,还需要10个服务器端套接字.
但在UDP中,不管是服务器端还是客户端都只需要1个套接字.
    
要点三:基于UDP的数据I/O函数

#include<sys/socket.h>
ssize_t sendto(int sock,void *buff,size_t nbytes,int flags,struct sockaddr* to,socklen_t addrlen);
->成功时返回传输的字节数,失败时返回-1
    - sock : 用于传输数据UDP套接字文件描述符
    - buff : 保存待传输数据的缓冲地址值
    -nbytes: 待传输的数据长度,以字节为单位               (发)
    - flags: 可选项参数,若没有则传递0
    - to   : 存有目标地址信息的sockaddr结构体变量的地址值
    -addrlen:传递给参数to的地址值结构体变量长度
上述函数与之前的TCP输出函数最大的区别在于,此函数需要向它传递目标地址信息.
    
    
接下来介绍接受UDP数据的函数.UDP数据的发送端不固定,因此该函数定义为可接收发送端信息的形式,也就是将同时返回UDP数据包中的发送端信息.
#include<sys/socket.h>
ssize_ recvfrom(int sock,void* buff,size_t nbytes,int flags,struct sockaddr *from,socklen_t *addrlen);
->成功时返回接收的字节数,失败时返回-1
    - sock : 用于传输数据UDP套接字文件描述符
    - buff : 保存待传输数据的缓冲地址值
    -nbytes: 可接收的最大字节数,故无法超过参数buff所指的缓冲大小      (收)
    - flags: 可选项参数,若没有则传递0
    - to   : 存有目标地址信息的sockaddr结构体变量的地址值
    -addrlen:传递给参数to的地址值结构体变量长度

编写UDP程序时最核心的部分就在于上述两个函数!
要点四: 基于UDP的回声服务器端/客户端

//uecho_server.c
#include<stdio.h>
#include<stdio.h>
#include<string.h>
#include<unistd.h>
#incldue<arpa/inet.h>
#include<sys/socket.h>

#define BUF_SIZE 30
void error_handling(char *message);
int main(int argc,char *argv[])
{
    intserv_sock;
    char message[BUF_SIZE];
    int srr_len;
    socklen_t clnt_adr_sz;
    struct sockaddr_in serv_adr,clnt_adr;
    if(argc != 2)
    {
        printf("Usage : %s <port>\n",argv[0]);
        exit(1);
    }
    serv_sock = socket(PF_INET,SOCK_DGRAM,0);//为了创建UDP套接字,向socket函数第二个参数传递SOCK_DGRAM
    if(serv_sock == -1)
        error_handling("UDP socket creation error");
    
    memset(&serv_adr,0,sizeof(serv_adr));
    serv_adr.sin_family = AF_INET;
    serv_adr.sin_addr.s_addr=htonl(INADDR_ANY);
    serv_adr.sin_port=htons(atoi(argv[1]));
    
    if( bind(serv_sock,(struct sockaddr*)&serv_adr,sizeof(serv_adr)) ==-1)
        error("bind() error");
    
    while(1)
    {
        clnt_adr = sizeof(clnt_adr);
        
        //不限制数据传输对象
        str_len = recvfrom(serv_sock,message,BUF_SIZE,0,
                          (struct sockaddr*)&clnt_adr,&clnt_adr_sz);//我记得accept函数最后也要传引用
        sendto(serv_sock,message,str_len,0,
              (struct sockaddr*)&clnt_adr,clnt_adr_sz);
    }
    close(serv_sock);//因为while循环中并未加入break语句,所以这个close没有意义
    return 0;
}
void error_handling(char *message)
{
    fputs(message,stderr);
    fputc('\n',stderr);
    exit(1);
}



//uecho_client.c
#include<stdio.h>
#include<stdio.h>
#include<string.h>
#include<unistd.h>
#incldue<arpa/inet.h>
#include<sys/socket.h>

#define BUF_SIZE 30
void error_handling(char *message);
int main(int argc,char *argv[])
{
    int sock;
    char message[BUF_SIZE];
    int str_len;
    socklen_t adr_sz;
    
    struct sockaddr_in serv_adr,from_adr;
    if(argc != 3)
    {
        printf("Usage : %s <IP> <PORT>\n",argv[0]);
        exit(1);
    }
    
    sock = socket(PF_INET,SOCK_DGRAM,0);//创建UDP套接字要用SOCK_DGRAM
    if(sock == -1)
        error_handling("socket() error");
    
    memset(&serv_adr,0,sizeof(serv_adr));
    serv_adr.sin_family=AF_INET;
    serv_adr.sin_addr.s_addr=inet_addr(argv[1]);
    serv_adr.sin_port=htos(atoi(argv[2]));
    
    while(1)
    {
        fputs("Insert message(q to quit): " , stdout);
        fgets(message,sizeof(message),stdin);
        if(!strcmp(message,"q\n") || !strcmp(message,"Q\n"))
            break;
        sendto(sock,message,strlen(message,0),
              (struct sockaddr*)&serv_adr,sizeof(serv_adr));//向服务器端传输数据
        adr_sz = sizeof(from_adr);
        str_len=recvfrom(sock,message,BUF_SIZE,0,
                         (struct sockaddr*)&from_adr,&adr_sz);//接收数据
        message[str_len] = 0;
        printf("Message from server : %s",message);
    }
    close(sock);
    
	return 0;
}
void error_handling(char *message)
{
    fputs(message,stderr);
    fputc('\n',stderr);
    exit(1);
}

要点五:UDP客户端套接字的地址分配
仔细观察UDP客户端的小朋友会发现,它缺少把IP和端口分配给套接字的过程.TCP调用connect函数自动化完成这个过程,而UDP中连能承担相同功能的函数调用语句都没有.那么,UDP究竟在何时分配IP和端口号呢?
	在UDP程序中,调用sendto函数传输数据前应完成对套接字的地址分配工作,因此调用bind函数.当然, bind函数并不区分TCP和UDP,也就是说,在UDP程序中同样可以调用.另外,如果调用sendto函数时发现尚未分配地址信息,则在首次调用sendto函数时给相应套接字自动分配IP和端口.而且此时分配的地址一直保留到程序结束为止,因此也可用来与其他UDP套接字进行数据交换.当然,IP用主机IP,端口号选尚未使用的任意端口号.
    综上所述，调用sendto函数时自动分配IP和端口号，因此，UDP客户端中通常无需额外的地址分配过程。所以之前示例中省略了该过程，这也是普遍的实现方式。    
```



```cpp
UDP的数据传输特性和调用connect函数(p109)

TCP传输的数据不存在数据边界,而UDP数据传输中存在数据边界.
    
存在数据边界的UDP套接字:
"因为TCP数据传输中不存在边界,数据传输过程中调用IO函数的次数不具有任何意义".相反,UDP是具有数据边界的协议,传输中调用IO函数的次数非常重要.因此,输入函数的调用次数应和输出函数的调用次数完全一致,这样才能保证接收全部已发送数据.例如,调用3次输出函数发送的数据必须通过3次输入函数才能接收完.
下面通过简单示例进行验证:
//bound_host1.c
#include<stdio.h>
#include<stdio.h>
#include<string.h>
#include<unistd.h>
#incldue<arpa/inet.h>
#include<sys/socket.h>

#define BUF_SIZE 30
void error_handling(char *message);
int main(int argc,char *argv[])
{
    int sock;
    char message[BUF_SIZE];
    struct sockaddr_in my_adr,your_adr;
    socklen_t adr_sz;
    int str_len,i;
    if(argc!=2)
    {
        printf("Usage : %s <port>\n",arghv[0]);
        exit(1);
    }
    sock = socket(PF_INET,SOCK_DGRAM,0);
    if(sock==-1)
        error_handling("socket() error");
    
    memset(&my_adr,0,sizeof(my_adr));
    my_adr.sin_family = AF_INET;
    my_adr.sin_addr.s_addr=htonl(INADDR_ANY);
    my_adr.sin_port=htons(atoi(argv[1]);
                          
    if( bind(sock,(struct sockaddr*)&my_adr,sizeof(my_adr) )==-1)
          error_handling("bind() error");
                        
     for(i = 0 ; i < 3 ; i ++)
     {
         sleep(5);//停顿5s,使程序停顿时间等于传递来的时间参数;也就是说,for循环每隔5s调用1次recvfrom函数
         adr_sz = sizeof(your_adr);
         str_len = recvfrom(sock,message,BUF_SIZE,0,
         	(struct sockaddr*)&your_adr,&adr_sz);
         printf("Message %d:%s \n",i+1,message);
     }
     close(sock);
     return 0;
}
void error_handling(char *message)
{
    fputs(message,stderr);
    fputc('\n',stderr);
    exit(1);
}
                          

下面的示例向之前的bound_host1.c传输数据,该示例共调用3次以传输字符串数据  
//bound_host2.c                          
#include<stdio.h>
#include<stdio.h>
#include<string.h>
#include<unistd.h>
#incldue<arpa/inet.h>
#include<sys/socket.h>

#define BUF_SIZE 30
void error_handling(char *message);
int main(int argc,char *argv[])
{
	int sock;
	char msg1[] = "Hi";
	char msg2[] = "I'm another UDP host!";
	char msg3[] = "Nice to meet you";
	
	struct sockaddr_in your_adr;
	socklen_t your_adr_sz;
	if(argc!=3)
	{
		printf("Usge : % s <IP> <port>\n",argv[0]);
		exit(1);
	}
	sock = socket(PF_INET,SOCK_DGRAM,0);
	if(sock == -1)
		error_handling("socket() error");
		
	memset(&your_adr,0,sizeof(your_adr));
	your_adr.sin_family = AF_INET;
	your_adr.sin_addr.s_addr = inet(argv[1]);
	your_adr.sin_port=htons(atoi(argv[2]));
	
	//3次调用sendto函数以传输数据
	sendto(sock,msg1,sizeof(msg1),0,
		(struct sockaddr*)&your_adr,sizeof(your_adr));
	sendto(sock,msg1,sizeof(msg1),0,
		(struct sockaddr*)&your_adr,sizeof(your_adr));
	sendto(sock,msg1,sizeof(msg1),0,
		(struct sockaddr*)&your_adr,sizeof(your_adr));
	
	close(sock);
	return 0;
}
void error_handling(char *message)
{
    fputs(message,stderr);
    fputc('\n',stderr);
    exit(1);
}



<额外说明一点:UDP数据报(Datagram)>
UDP套接字传输的数据包又称数据报,实际上数据报也属于数据包的一种.
只是与TCP包不同煤气本身可以成为1个完整数据.
这与UDP的数据传输特性有关,UDO中存在数据边界,1个数据包即可成为1个完整数据,因此称为数据报.
```



```cpp
已连接UDP套接字与未连接UDP套接字(和"和如何提高UDP的传输性能有关")

TCP套接字中需注册待传输数据的目标IP和端口号,而UDP中无序注册.因此,通过sendto函数传输数据的过程大致可分为以下3个阶段.
第1阶段:向UDP套接字注册目标IP和端口号
第2阶段:传输数据
第3阶段:删除UDP套接字中注册的目标地址信息.

每次调用sendto函数时重复上述过程,每次都变更目标地址,因此可以重复利用同一UDP套接字向不同目标传输数据.这种未注册目标地址信息的套接字称为未连接套接字,反之,注册了目标地址的套接字称为连接套接字."显然,UDP套接字默认属于未连接套接字".但UDP套接字在下述情况下就显得不太合理:

"IP为211.210.147.82的主机82号端口共准备了3个数据,调用3次sendto函数进行传输"
此时需要重复3次上述阶段,因此,要与同一主机进行长时间通信时将UDP套接字变成已连接套接字会提高效率.上述三个阶段中,第一阶段(注册)和第三阶段(删除或者叫解绑)
占整个通信过程近1/3的时间,缩短这部分时间将大大提高整体性能.
```

```cpp
创建已连接的UDP套接字

创建已连接UDP套接字的过程格外简单,只需针对UDP套接字调用connect函数.

sock = socket(PF_INET,SOCK_DGRAM,0);//初始化struct sockaddr_in后,再调用connect函数
memset(&adr,0,sizeof(adr));
adr.sin_family = AF_INET;
adr.sin_addr.s_addr = ...
adr.sin_port = ...
connect(sock,(struct sockaddr*)&adr,sizeof(adr));

上述代码看似与TCP套接字创建过程一致,但socket函数的第二参数分明是SOCK_DGRAM.也就是说,这个确实是UDP套接字.Sure,针对UDP套接字调用connect函数并不意味着雨对方UDP套接字连接,这只是向UDP套接字注册目标IP和端口信息.
之后就与TCP套接字一样,每次调用sendto函数时只需传输数据.因为已经指定了收发对象,所以不仅可以使用sendto,recvfrom函数,还可以使用write,read函数进行通信.


下列示例将之前的uecho_client.c程序改成基于已连接UDP套接字的程序
#include<stdio.h>
#include<stdio.h>
#include<string.h>
#include<unistd.h>
#incldue<arpa/inet.h>
#include<sys/socket.h>

#define BUF_SIZE 30
void error_handling(char *message);
int main(int argc,char *argv[])
{
    int sock;
    char message[BUF_SIZE];
    int str_len;
    socklen_t adr_sz;//多余变量!
    
    struct sockaddr_in serv_adr,from_adr;//不再需要from_adr!
    if(argc != 3)
    {
        printf("Usage : %s <IP> <PORT>\n",argv[0]);
        exit(1);
    }
    
    sock = socket(PF_INET,SOCK_DGRAM,0);//创建UDP套接字要用SOCK_DGRAM
    if(sock == -1)
        error_handling("socket() error");
    
    memset(&serv_adr,0,sizeof(serv_adr));
    serv_adr.sin_family=AF_INET;
    serv_adr.sin_addr.s_addr=inet_addr(argv[1]);
    serv_adr.sin_port=htos(atoi(argv[2]));
    
    //多了下面这句话!!!
    connect(sock,(struct sockaddr*)&serv_adr,sizeof(serv_adr));
    
    while(1)
    {
        fputs("Insert message(q to quit): " , stdout);
        fgets(message,sizeof(message),stdin);
        if(!strcmp(message,"q\n") || !strcmp(message,"Q\n"))
            break;
        /*
        sendto(sock,message,strlen(message,0),
              (struct sockaddr*)&serv_adr,sizeof(serv_adr));//向服务器端传输数据
        */
        write(sock,message,strlen(message));
        /*
        adr_sz = sizeof(from_adr);
        str_len=recvfrom(sock,message,BUF_SIZE,0,
                         (struct sockaddr*)&from_adr,&adr_sz);//接收数据
        */
        str_len = read(sock,message,sizeof(message)-1);
        message[str_len] = 0;
        printf("Message from server : %s",message);
    }
    close(sock);
    
	return 0;
}
void error_handling(char *message)
{
    fputs(message,stderr);
    fputc('\n',stderr);
    exit(1);
}
关于上面的代码,做出一丢丢解释:加了connect函数,使用write函数,read函数代替了sendto,recvfrom函数
```


# 第7章 优雅地断开套接字连接 (P118)
```cpp
基于TCP的半关闭

TCP中断开连接过程比建立连接过程更重要,因为连接过程中一般不会出现大的变数,但断开过程有可能发送预想不到的情况,因此应准确掌控.
"只有掌握了下面要讲解的半关闭(Half_close),才能明确断开过程"

下面有五个要点:
```

```cpp
要点一:单方面断开连接带来的问题

Linux的close函数意味着:完全断开连接。完全断开不仅指无法传输数据，而且也不能接收数据。
因此，在某些情况下，通信一方调用close或closesocket函数断开连接就显得不太优雅。

例如:2台主机正在进行双向通信。主机A发送完最后的数据后，调用close函数断开了连接，之后主机A无法再接收主机B传输的数据。实际上，是完全无法调用与接收数据相关的函数。最终，由主机B传输的、主机A必须接收的数据也销毁了。
为了解决这类问题，“只关闭一部分数据交换中使用的流”(Half-close )的方法应运而生。断开一部分连接是指，可以传输数据但无法接收，或可以接收数据但无法传输。顾名思义就是只关闭流的一半。
```

```cpp
要点二:套接字和流（Stream)
  两台主机通过套接字建立连接后进入可交换数据的状态，又称“流形成的状态”。也就是把建立套接字后可交换数据的状态看作一种流。
此处的流可以比作水流。水朝着一个方向流动，同样，在套接字的流中，数据也只能向一个方向移动。因此，为了进行双向通信,应该创建2个指向对方的IO流。
  一旦两台主机间建立了套接字连接，每个主机就会拥有单独的输人流和输出流。当然，其中一个主机的输入流与另一主机的输出流相连，而输出流则与另一主机的输人流相连。
  另外，本章讨论的“优雅地断开连接方式”只断开其中1个流,而非同时断开两个流。Linux的close将同时断开这两个流，因此与“优雅”二字还有一段距离。

```
```cpp
要点三:针对优雅断开的shutdown函数
接下来介绍用于半关闭的函数。下面这个shutdown函数就用来关闭其中1个流。

#include<sys/socket.h>
int shutdown(int sock,int howto);
-> 成功时返回0,失败时返回-1
	sock:需要断开的套接字文件描述符
	howo:传递断开方式信息
	
调用上述函数时，第二个参数决定断开连接的方式，其可能值如下所示。	- SHUT_RD:断开输入流。
- SHUT_WR:断开输出流。
- SHUT_RDWR:同时断开IO流。
若向shutdown的第二个参数传递SHUT_RD，则断开输入流，套接字无法接收数据。即使输入缓冲收到数据也会抹去，而且无法调用输入相关函数。如果向shutdown函数的第二个参数传递SHUT_WR，则中断输出流，也就无法传输数据。但如果输出缓冲还留有未传输的数据，则将传递至目标主机。最后，若传入SHUT_RDWR，则同时中断I/O流。这相当于分2次调用shutdown,其中一次以SHUT_RD为参数,另一次以SHUT_WR为参数。

```
```cpp
要点四:为何需要半关闭
我从小到大就有一个疑惑:
	"究竟为什么需要半关闭?是否只要留出足够长的连接时间，保证完成数据交换即可?只要不急于断开连接，好像也没必要使用半关闭。(而且我在读这本书之前,从来不知道shutdown函数,我还做过共享单车这样企业级的项目,我在全文搜索这个函数也没有收到的,这说明这个函数在企业级项目中我尚未接触到)"
这句话也不完全是错的。如果保持足够的时间间隔，完成数据交换后再断开连接，这时就没必要使用半关闭。但要考虑如下情况:
	"一旦客户端连接到服务器端，服务器端将约定的文件传给客户端，客户端收到后发送字符串‘Thank you'给服务器端。"
此处字符串“Thank you”的传递实际是多余的，这只是用来模拟客户端断开连接前还有数据需要传递的情况。此时程序实现的难度并不小，因为传输文件的服务器端只需连续传输文件数据即可，而客户端则无法知道需要接收数据到何时。客户端也没办法无休止地调用输人函数，因为这有可能导致程序阻塞（调用的函数未返回)。
	"是否可以让服务器端和客户端约定一个代表文件尾的字符?"
这种方式也有问题，因为这意味着文件中不能有与约定字符相同的内容。为解决该问题，服务器端应最后向客户端传递EOF表示文件传输结束。客户端通过函数返回值接收EOF，这样可以避免与文件内容冲突。剩下最后一个问题:服务器如何传递EOF?
	"断开输出流时向对方主机传输EOF。"(<记住这句话,很很重要>)
当然，调用close函数的同时关闭IO流，这样也会向对方发送EOF。但此时无法再接收对方传输的数据。换言之,若调用close函数关闭流,就无法接收客户端最后发送的字符串“Thank you”。这时需要调用shutdown函数，只关闭服务器的输出流（半关闭)。这样既可以发送EOF，同时又保留了输入流，可以接收对方数据。
下面结合已学内容实现收发文件的服务器端/客户端:
```
```cpp
要点五:基于半关闭的文件传输程序
//file_server.c
#include<stdio.h>
#include<sdlib.h>
#include<sring.h>
#include<unisd.h>
#include<arpa/inet.h>
#include<sys/socket.h>

#define BUF_SIZE 30
void error_handling(char *message);
int main(int argc,char *argv[])
{
	int serv_sd,clnt_sd;
	FILE *fp;
	char buf[BUF_SIZE];
	int read_cnt;
	
	struct sockaddr_in serv_adr,clnt_adr;
	socklen_t clnt_adr_sz;
	if(argc != 2)
	{
		printf("Usge : %s <port>\n",argv[0]);
		exit(1);
	}
	
	fp = fopen("file_server.c","rb");//打开文件以客户端传输服务器端源文件file_server.c(只读方式打开)
	serv_sd = socket(PF_INET,SOCK_STREAM,0);//TCP
	
	memset(&serv_adr,0,sizeof(serv_adr));
	serv_adr.sin_family = AF_INET;
	serv_adr.sin_addr.s_addr = htonl(INADDR_ANY);
	serv_adr.sin_port = htons(atoi(argv[1]));
	
	bind(serv_sd,(struct sockaddr*)&serv_adr,sizeof(serv_adr));
	listen(serv_sd,5);
	
	clnt_adr_sz = sizeof(clnt_adr);
	clnt_sd = accept(serv_sd,(struct sockaddr*)&clnt_adr,&clnt_adr_sz);
	
	//为客户端传输文件数据的循环语句,此客户端是从accept函数调用中连接的.
	while(1)
	{
		read_cnt = fread((void*)buf,1,BUF_SIZE,fp);
		if(read_cnt < BUF_SIZE)
		{
			write(clnt_sd,buf,read_cnt);
			break;
		}
		write(clnt_sd,buf,BUF_SIZE);
	}
	
	shutdown(clnt_sd,SHU_WR);//发送文件后对输出流进行半关闭.这样就向客户端传输了EOF,而客户端也知道文件传输已完成
	read(clnt_sd,buf,BUF_SIZE);//值关闭了输出流,依然可以通过输入流接收数据.
	printf("Message from client : % s \n",buf);
	
	fclose(fp);//关闭文件描述符
	close(clnt_sd);//关闭客户端
	close(serv_sd);//关闭服务器
	return 0;
}
void error_handling(char *message)
{
	fputs(message,stderr);
	fputc('\n',stderr);
	exit(1);
}



//file_client.c
#include<stdio.h>
#include<sdlib.h>
#include<sring.h>
#include<unisd.h>
#include<arpa/inet.h>
#include<sys/socket.h>

#define BUF_SIZE 30
void error_handling(char *message);
int main(int argc,char *argv[])
{
	int sd;
	FILE *fp;
	 
	char buf[BUF_SIZE];
	int read_cnt;
	struct sockaddr_in serv_adr;
	if(argc != 3)
	{
		printf("Usage : %s <IP> <port> \n",argv[0]);
		exit(1);
	}
	
	fp = fopen("receive.dat","wb")//(只写方式打开)
	sd = socket(PF_INET,SOCK_STREAM,0);//TCP
	
	memset(&serv_adr,0,sizeof(serv_adr));
	serv_adr.sin_family = AF_INET;
	serv_adr.sin_addr.s_addr = inet_addr(argv[1]);
	serv_adr.sin_port = htons(atoi(argv[2]));
	
	connect(sd,(struct sockaddr*)&serv_adr,sizeof(serv_adr));
	
	//接收数据并保存到第18行创建的文件,直到接收EOF
	while( (read_cnt = read(sd,buf,BUF_SIZE))!= 0)
		fwrite((void*)buf,1,read_cnt,fp);
		
	puts("Receive file data");
	//向服务器端发送表示感谢地消息.若服务器端未关闭输入流,则可接收此消息.
	write(sd,"Thank you",10);//9个字符+一个文件结束符
	
	fclose(fp);
	close(sd);
	
	return 0;
}

void error_handling(char *message)
{
	fputs(message,stderr);
	fputc('\n',stderr);
	exit(1);
}
```



# 第8章 域名及网络地址 P(128)

## 8.1 域名系统
DNS是对IP地址和域名进行互相转换的系统,其核心是DNS服务器

```cpp
什么是域名

提供网络服务的服务器端也是通过IP地址区分的，但几乎不可能以非常难记的IP地址形式交换服务器端地址信息。因此,将容易记、易表述的域名分配并取代IP地址。
```

```cpp
DNS服务器

在浏览器地址栏中输入Naver网站的IP地址222.122.195.5即可浏览Naver网站主页。但我们通常输入Naver网站的域名www.naver.com 也可以访问网站。二者之间究竟有何区别?

从进入Naver网站主页这一结果上看，没有区别，但接入过程不同。域名是赋予服务器端的虚拟地址,而非实际地址。因此,需要将虚拟地址转化为实际地址。那如何将域名变为IP地址呢?DNS服务器担此重任,可以向DNS服务器请求转换地址。
“请问DNS服务器，www.naver.com的IP地址是多少?”
所有计算机中都记录着默认DNS服务器地址,就是通过这个默认DNS服务器得到相应域名的IP地址信息。在浏览器地址栏中输入域名后，浏览器通过默认DNS服务器获取该域名对应的IP地址信息，之后才真正接入该网站。
```

```cpp
<提示>
1.使用
nslookup
命令,可以知道自己计算机中注册的默认DNS服务器地址

2.计算机内置的默认DNS服务器并不知道网络上所有域名的IP地址信息.若该DNS服务器无法解析,则会询问其他DNS服务器,并提供给用户
```



## 8.2 IP地址和域名之间的转换

```cpp
程序中有必要使用域名吗?

假设各位是运营 www.SuperOrange.com 域名的公司系统工程师，需要开发客户端使用公司提供的服务。该客户端需要接入如下服务器地址:
	ip 211.102.204.12,port 2012
工程师应向程序用户提供便利的运行方法，因此，程序不能像运行示例程序那样要求用户输入IP和端口信息。
那该如何将上述信息传递到程序内部?
IP地址比域名发生变更的概率要高，所以利用P地址编写程序并非上策。
还有什么办法呢?一旦注册域名可能永久不变，因此利用域名编写程序会好一些。这样，每次运行程序时根据域名获取IP地址，再接入服务器，这样程序就不会依赖于服务器的IP地址了。所以说，程序中也需要IP地址和域名之间的转换函数。
```

```cpp
利用域名获取IP地址

使用以下函数可以通过传递字符串的域名获取IP地址:
#include<netdb.h>
struct hostent *gethostbyname(const char * hostname);
->成功时返回hostent结构体地址,失败时返回NULL指针

上面这个函数只要传递域名字符串,就会返回域名对应的IP地址.只是返回时,地址信息装入hostent结构体.此结构体定义如下:
struct hostent
{
	char * h_name;//official name 官方域名
	char ** h_aliases;//alias list 可以通过多个域名访问同一主页
	int h_addrtype;//host address type 地址族信息,ipv4或ipv6;若是ipv4则是AF_INET
	int h_length;//address length 保存IP地址长度(ipv4是4字节,所以报存4;ipv6是16字节,所以报存16)
	char ** h_addr_list;//address list 通过此变量以整数形式保存域名对应的IP地址 
}
```

```cpp
利用IP地址获取域名

//gethostbyname.c
#include<stdio.h>
#include<stdlib.h>
#incldue<unistd.h>
#include<arpa/inet.h>
#incldue<netdb.h>
void error_handling(char *message);
int main(int argc,char *argv[])
{
	int i;
	struct hostent *host;
	if(argc != 2)
	{
		printf("Usage :%s <addr>\n",argv[0]);
		exit(1);
	}
	
	host = gethostbyname(argv[1]);//将通过main函数传递的字符串用作参数调用gethostbyname
	if(!host)
		error_handling("gethost... error");
		
	printf("Official name :%s \n",host->h_name);//输出官方域名
	
	for(i = 0 ; host->h_aliases[i] ; i++)//输出除官方域名以外的域名
		printf("Aliases %d :%s \n",i+1,host->h_aliases[i]);
	
	printf("Address type :%s\n",
		(host->h_addrtype==AF_INET)?"AF_INET":"AF_INET6");
	
	for(i = 0 ; host->h_addr_list[i] ;i++)//输出IP地址信息
		printf("IP addr %d:%s \n",i+1,
			inet_ntoa(*(struct in_addr*)host->h_addr_list[i]));
			
	return 0;
}
void error_handling(char *message)
{
	fputs(message,stderr);
	fputc('\n',stderr);
	exit(1);
}
```

![1691939855053](C:\Users\wxn\AppData\Roaming\Typora\typora-user-images\1691939855053.png)


```cpp
利用IP地址获取域名

gethostbyaddr函数利用IP地址获取域相关信息
#include<netdbb.h>
struct hostent *gethostbyaddr(const char *addr,socklen_t len,int family);
->成功时返回hostent结构体变量地址值,失败时返回NULL指针
	addr:含有ip地址信息的in_addr结构体指针.为了同时传递ipv4地址之外的其他信息,该变量的类型声明为char指针
	len:向第一个参数传递的地址信息的字节数,ipv4时为4,ipv6时为16
	family:传递地址族信息,ipv4时为AF_INET,ipv6时为AF_INET6
	
	
//gethostbyaddr.c
#include<stdio.h>
#include<stdlib.h>
#include<string.h>
#include<unistd.h>
#include<arpa/inet.h>
#include<netdb.h>

void error_handling(char *message);
int main(int argc,char *argv[])
{
    int i;
    struct hostent *host;
    struct sockaddr_in addr;
    if(argc != 2)
    {
        printf("Usage : %s <ip> \n",argv[0]);
        exit(1);
    }
    
    memset(&addr,0,sizeof(addr));
    addr.sin_addr.s_addr = inet_addr(argv[1]);
    host=gethostbyaddr((char*)&addr.sin_addr,4,AF_INET);//除该行的gethostbyaddr函数调用过程外，与gethostbyname.c并无区别，因为函数调用的结果是通过hostent结构体变量地址值传递的。

    if(!host)
        error_handling("gethost... error");
    
    printf("Official name %s\n",host->h_name);
    for(i = 0 ; host->h_aliases[i];i++)
        printf("Aliases %d :%s\n",i+1,host->h_aliases[i]);
    
    printf("Address type :%s\n",
          (host->h_addrtype==AF_INET)?"AF_IENT":"AF_IENT6");
    
    for(i = 0 ; host->h_addr_list[i];i++)]
        printf("IP addr %d : %s\n",i+1,
              inet_ntoa(*(struct in_addr*)host->h_addr_list[i]));
    return 0;
}
void error_handling(char *message)
{
    fputs(message,stderr);
    fputc('\n',stderr);
    exit(1);
}
```

# 第9章 套接字的多种可选项 p(140)

## 9.1 套接字可选性和IO缓冲大小

```cpp
套接字多种可选性

可以上网查
```

```cpp
getsockopt & setsockopt

我们可以针对 套接字 的所有可选项进行读取（Get）和设置（Set)(当然，有些可选项只能进行一种操作)。可选项的读取和设置通过如下2个函数完成。


读取套接字可选性
#include<sys/socket.h>
int getsockopt(int sock,int level,int optname,void *optval,socklen_t *optlen);
->成功时返回0,失败时返回-1
    sock : 用于查看选型套接字文件描述符
    level:要查看的可选项的协议层
    optname:要查看的可选项名
    optval:保存查看结果的缓冲地址值
    optlen:向第四个参数optval传递的缓冲大小.调用函数后,该变量中保存通过第四个参数返回的可选项信息的字节数
    

更改套接字可选性
#include<sys/socket.h>
int setsockopt(int sock,int level,int optname,const void *optval,socklen_t optlen);
->成功时返回0,失败时返回-1
    sock : 用于更改可选项的套接字文件描述符
    level:要更改的可选项协议层
    optname:要更改的可选项ming
    optval:保存要更改的选项信息的缓冲地址值
    optlen:向第四个参数optval传递的可选性信息的字节数
    
    
接下来介绍这些函数的调用方法。关于setsockopt函数的调用方法在其他示例中给出，先介绍getsockopt函数的调用方法。下列示例用协议层为SOL_SOCKET、名为SO_TYPE的可选项查看套接字类型(TCP或UDP)。
//sock_type.c
#include<stdio.h>
#incldue<stdlib.h>
#include<unistd.h>
#include<sys/socket.h>
void error_handling(char *message);
int main(int argc,char *argv[])
{
    int tcp_sock,udp_sock;
    int sock_type;
    socklen_t optlen;
    int state;
    
    optlen = sizeof(sock_type);
    tcp_sock = socket(PF_INET,SOCK_STREAM,0);//生成TCP套接字
    udp_sock = socket(PF_INET,SOCK_DGRAM,0);//生成UDP套接字
    printf("SOCK_STREAM : %d \n",SOCK_STREAM);//1
    printf("SOCK_DGRAM : %d \n",SOCK_DGRAM);//2
    
    state = getsockopt(tcp_sock,SOL_SOCKET,SO_TYPE,(void*)&sock_type,&optlen);
    if(state)
    {
        error_handling("getsockopt() error");
    }
    printf("Socket type one : %d \n",sock_type);//1
    
    state = getsockopt(udp_sock,SOL_SOCKET,SO_TYPE,(void*)&sock_type,&optlen);
    
    if(state)
    {
        error_handling("getsockopt() error");
    }
    printf("Socket type two : %d \n",sock_type);//2
	return 0;
}
void erroe_handling(char *message)
{
    fputs(message,stderr);
    fputc('\n',stderr);
	exit(1);
}

"套接字类型只能在创建时决定，以后不能再更改。"
```

```cpp
SO_SNDBUF & SO_RCVBUF

前面介绍过，创建套接字将同时生成IO缓冲。
so_RCVBUF是输入缓冲大小相关可选项，SO_SNDBUF是输出缓冲大小相关可选项。用这2个可选项既可以读取当前IO缓冲大小，也可以进行更改。通过下列示例读取创建套接字时默认的I/O缓冲大小。

//get_buf.c
#include<stdio.h>
#include<stdlib.h>
#include<unistd.h>
#include<sys/socket.h>
void error_handling(char *message);
int main(int argc,char *argv[])
{
	int sock;
	int snd_buf,rcv_buf,state;
	socklen_t len;
	
	sock = socket(PF_INET,SOCK_STREAM,0);
	len = sizeof(snd_buf);
	state = getsockopt(sock,SOL_SOCKET,SO_SNDBUF,(void*)&snd_buf,&len);
	
	if(state)
		error_handling("getsockopt() error");
		
	len = sizeof(rcv_buf);
	state = getsockopt(sock,SOL_SOCKET,SO_RCVBUF,(void*)&rcv_buf,&len);
	
	if(state)
		error_handling("getsockopt() error");
		
	printf("Input buffer size : %d \n",rcv_buf);//87380
	printf("Output buffer size : %d \n",snd_buf);//16384
	return 0;
}
void error_handling(char *message)
{
	fputs(message,stderr);
	fputc('\n',stderr);
	exit(1);
}



//set_buf.c
#include<stdio.h>
#include<stdlib.h>
#include<unistd.h>
#include<sys/socket.h>
void error_handling(char *message);
int main(int argc,char *argv[])
{
	int sock;
	int snd_buf = 1024*3,rcv_buf=1024*3;
    int state;
    socklen_t len;
    
    sock = socket(PF_INET,SOCK_STREAM,0);
    state = setsockopt(sock,SOL_SOCKET,SO_RCVBUF,(void*)&rcv_buf,sizeof(rcv_buf));//更改IO缓冲大小为3M
    if(state)
    	error_handling("setsockopt() error");
    	
    state = setsockopt(sock,SOL_SOCKET,SO_SNDBUF,(void*)&snd_buf,sizeof(snd_buf));//更改IO缓冲大小为3M
    if(state)
    	error_handling("setsockopt() error");
    
    len = sizeof(snd_buf);
    state = getsockopt(sock,SOL_SOCKET,SO_SNDBUF,(void*)&snd_buf,&len);
    if(state)
    	error_handling("getsockopt() error");
    	
    len = sizeof(rcv_buf);
    state = getsockopt(sock,SOL_SOCKET,SO_RCVBUF,(void*)&rcv_buf,&len);
    if(state)
    	error_handling("getsockopt() error");
    
    printf("Input buffer size:%d\n",rcv_buf);
    printf("Output buffer size:%d\n",snd_buf);
	return 0;
}
void error_handling(char *message)
{
	fputs(message,stderr);
	fputc('\n',stderr);
	exit(1);
}
```


## 9.2 SO_REUSEADDR

本节的可选项 SO_REUSEADDR 及其相关的 Time-wait 状态很重要，希望大家务必理解并掌握。

```cpp
发生地址分配错误(Binding Error)

学习SO_REUSEADDR可选项之前,应理解好Time-wait状态。

//reuseadr_eserver.c
#include<stdio.h>
#include<stdlib.h>
#include<string.h>
#include<unistd.h>
#include<arpa/inet.h>
#include<sys/socket.h>

#define TRUE 1
#defien FALSE 0
void error_handling(char *message);

int main(int argc,char *argv[])
{
	int serv_sock ,clnt_sock;
    char message[30];
    int option , str_len;
    socklen_t optlen , clnt_adr_sz;
    struct sockaddr_in serv_adr,clnt_adr;
    if(argc != 2)
    {
    	printf("Usage : %s <port> \n",argv[0]);
    	exit(1);
    }
    
    serv_sock = socket(PF_INET,SOCK_STREAM,0);
    if(serv_sock == -1)
    	error_handling("socket() error");
    
    /*
    optlen = sizeof(option);
    option = TRUE;
    setsockopt(serv_sock,SOL_SOCKET,SO_REUSEADDR,(void*)&option,optlen);
    */
    
    memset(&serv_adr,0,sizeof(serv_adr));
    serv_adr.sin_family = AF_INET;
    serv_adr.sin_addr.s_addr = htonl(ANADDR_ANY);
    serv_adr.sin_port=htons(atoi(argv[1]));
    
    if(bind(serv_sock,(struct sockaddr*)&serv_adr,sizeof(serv_adr)))
    	error_handling("bind() error");
    	
    if(listen(serv_sock,5) == -1)
    	error_handling("listen error");
    	
    clnt_adr_sz = sizeof(clnt_adr);
    clnt_sock = accept(serv_sock,(struct sockaddr*)&clnt_adr,&clnt_adr_sz);
    
    while( ((str_len=read(clnt_sock,message,sizeof(message))))!=0 )
    {
    	write(clnt_sock,message,str_len);
    	write(1,message,strlen);
    }
    close(clnt_sock);
    close(serv_sock);
	return 0;
}


void error_handling(char *message)
{
	fputs(message,stderr);
	fputc('\n',stderr);
	exit(1);
}


如果通过ctrl+c结束程序,那服务端要重新运行时就会输出"bind() error"消息,且无法再次运行,但在这种情况下,大约再过3分钟即可重新运行服务器端
```


```cpp
Time-wait状态

问题是，套接字经过四次握手过程后并非立即消除，而是要经过一段时间的Time-wait状态。当然，只有先断开连接的(先发送FIN消息的）主机才经过Time-wait状态。因此，若服务器端先断开连接，则无法立即重新运行。套接字处在Time-wait过程时，相应端口是正在使用的状态。因此，就像之前验证过的，bind函数调用过程中当然会发生错误。



<补充:客户端套接字不会经过Time-wait过程吗?>
有些人会误以为 Time-wait过程只存在于服务器端。但实际上，不管是服务器端还是客户端，套接字都会有Time-wait过程。"先断开连接的套接字必然会经过Time-wait过程。"
"但无需考虑客户端Time-wait状态。"因为客户端套接字的端口号是任意指定的。与服务器端不同，客户端每次运行程序时都会动态分配端口号，因此无需过多关注Time-wait状态。

```

![1691989424419](C:\Users\wxn\AppData\Roaming\Typora\typora-user-images\1691989424419.png)

```cpp
<补充:到底为什么会有Time-wait状态?>

假设主机A向主机B传输ACK消息后立即消除套接字。
但最后这条ACK消息在传递途中丢失，未能传给主机B。
这时会发生什么?
主机B会认为之前自己发送的FIN消息未能抵达主机A，继而试图重传。但此时主机A已是完全终止的状态，因此主机B永远无法收到从主机A最后传来的ACK消息。

相反，若主机A的套接字处在Time-wait状态，则会向主机B重传最后的ACK消息，主机B也可以正常终止。基于这些考虑，"先传输FIN消息的主机应经过Time-wait过程。"
```

```cpp
地址再分配

Time-wait看似重要，但并不一定讨人喜欢。考虑一下系统发生故障从而紧急停止的情况。这时需要尽快重启服务器端以提供服务，但因处于Time-wait状态而必须等待几分钟。因此，Time-wait并非只有优点，而且有些情况下可能引发更大问题。

解决方案就是在套接字的可选项中更改SO_REUSEADDR的状态。适当调整该参数，可将Time-wait状态下的套接字端口号重新分配给新的套接字。SO_REUSEADDR的默认值为0（假),这就意味着无法分配Time-wait状态下的套接字端口号。因此需要将这个值改成1(真)。具体做法已在示例reuseadr_eserver.c中给出，只需去掉下述代码的注释即可。

optlen = sizeof(option);
option = TRUE;
setsockopt(serv_sock,SOL_SOCKET,SO_REUSEADDR,(void*)&option,optlen);
```
## 9.3 TCP_NODELAY

在洛阳理工学Java网络编程时，老师经常提到以下问题:
“什么是Nagle算法?使用该算法能够获得哪些数据通信特性?”
JAVA老师提到这个问题就非常的兴奋，因为他就是因为能够讲出这个问题最终被公司录取,因为开发人员容易忽视的一个问题就是Nagle算法，下面进行详细讲解。

![1691990114391](C:\Users\wxn\AppData\Roaming\Typora\typora-user-images\1691990114391.png)

ON:开

OFF:关

```cpp
Nagle算法

为防止因数据包过多而发生网络过载，Nagle算法在1984年诞生了。它应用于TCP层，非常简单。


图9-3展示了通过Nagle算法发送字符串“Nagle”和未使用Nagle算法的差别。可以得到如下结论:
"只有收到前一数据的ACK消息时，Nagle算法才发送下一数据。"
TCP套接字默认使用Nagle算法交换数据，因此最大限度地进行缓冲，直到收到ACK。图9-3左侧正是这种情况。为了发送字符串“Nagle”，将其传递到输出缓冲。这时头字符“N”之前没有其他数据（没有需接收的ACK)，因此立即传输。之后开始等待字符“N”的ACK消息，等待过程中，剩下的“agle”填入输出缓冲。接下来，收到字符“N”的ACK消息后，将输出缓冲的‘agle”装人一个数据包发送。也就是说，共需传递4个数据包以传输1个字符串。
接下来分析未使用Nagle算法时发送字符串“Nagle”的过程。假设字符“N”到“e”依序传到输出缓冲。此时的发送过程与ACK接收与否无关,因此数据到达输出缓冲后将立即被发送出去。从图9-3右侧可以看到，发送字符串“Nagle”时共需10个数据包。由此可知，不使用Nagle算法将对网络流量(Traffic:指网络负载或混杂程度）产生负面影响。即使只传输1个字节的数据，其头信息都有可能是几十个字节。因此，为了提高网络传输效率，必须使用Nagle算法。

但Nagle算法并不是什么时候都适用。根据传输数据的特性，网络流量未受太大影响时，不使用Nagle算法要比使用它时传输速度快。最典型的是"传输大文件数据"。将文件数据传入输出缓冲不会花太多时间，因此，即便不使用Nagle算法，也会在装满输出缓冲时传输数据包。这不仅不会增加数据包的数量，反而会在无需等待ACK的前提下连续传输，因此可以大大提高传输速度。
一般情况下，不使用Nagle算法可以提高传输速度。但如果无条件放弃使用Nagle算法，就会增加过多的网络流量，反而会影响传输。因此，未准确判断数据特性时不应禁用Nagle算法。
```

```cpp
禁用Nagle算法

刚才说过的"大文件数据"应禁用Nagle算法。换言之，如果有必要，就应禁用Nagle算法。
	"Nagle算法使用与否在网络流量上差别不大，使用Nagle算法的传输速度更慢”
禁用方法非常简单。从下列代码也可看出，只需将套接字可选项TCP_NODELAY改为1(真)即可。

int opt_val = 1;
setsockopt(sock,IPPROTO_TCP,TCP_NODELAY,(void*)&opt_val,sizeof(opt_val));

可以通过TCP_NODELAY的值查看Nagle算法的设置状态。
int opt_val;
socklen_t opt_len;
opt_len = sizeof(opt_val);
getsockopt( sock，IPPROTO_TCP，TCP_NODELAY，(void*) &opt_val，&opt_len);

如果正在使用Nagle算法，opt_val变量中会保存0;如果已禁用Nagle算法，则保存1。
```



# 第10章 多进程服务器端(P155)

## 10.1 进程概念及应用
如果一个服务器要服务100个客户端,每个客户端平均服务0,5s,则第100个客户端会对服务器端产生相当大的不满.

```cpp
两种类型的服务器端

"所有连接请求的受理时间不超过1秒，但平均服务时间为2~3秒。"
```
```cpp
并发服务器端的实现方法


- 多进程服务器:通过创建多个进程提供服务。
- 多路复用服务器:通过捆绑并统一管理1/O对象提供服务。
- 多线程服务器:通过生成与客户端等量的线程提供服务。

```
```cpp
理解进程(Process)
"占用内存空间的正在运行的程序"


<补充:CPU核的个数与进程数>
拥有2个运算设备的CPU称作双核CPU，拥有4个运算器的CPU称作4核CPU。也就是说,1个CPU中可能包含多个运算设备(核)o核的个数与可同时运行的进程数相同。相反，若进程数超过核数，进程将分时使用CPU资源。但因为 CPU运转速度极快，我们会感到所有进程同时运行。当然,核数越多，这种感觉越明显。

```

```cpp
进程ID(PID)

无论进程是如何创建的，所有进程都会从操作系统分配到ID。
此ID称为“进程ID”，其值为大于2的整数。1要分配给操作系统启动后的首个进程,用于协助操作系统，因此用户进程无法得到ID值1.

在Linux中,可以使用ps命令语句观察Linux中正在运行的进程.
```

```cpp
通过调用fork函数创建进程-用于创建多进程服务器端的fork函数

#include<unistd.h>

pid_t fork(void);
->成功时返回进程ID,失败时返回-1
	- 父进程:fork函数返回子进程ID。
	- 子进程:fork函数返回0。

//fork.c
#include<stdio.h>
#include<unistd.h>

int gval = 0;
int main(int argc,char *argv[])
{
	pid_t pid;
	int lval = 20;
	gval++,lval+=5;
	
	pid=fork();//创建子进程.父进程的pid中存有子进程的ID,子进程的pid是0
	if(pid==0)//if Child Process   子进程
		gval+=2,lval+=2;
	else //父进程
		gval-=2,lval-=2;
		
	if(pid==0)//子进程
		printf("Child Proc : [%d,%d] \n",gval,lval);//13,27
	else//父进程
		printf("Parent Proc : [%d,%d] \n",gval,lval);//9,23
	return 0;
}
```
## 10.2 进程和僵尸进程

```cpp
僵尸进程

进程在完成工作后(执行完main函数中的程序后)应该被销毁,但有时这些进程将变成僵尸进程,占用系统中的重要资源.这种状态下的进程称作"僵尸进程",也是给系统带来负担的原因之一.
```

```cpp
产生僵尸进程的原因

为了防止僵尸进程的产生，先解释产生僵尸进程的原因。利用如下两个示例展示调用fork函数产生子进程的终止方式。
- 传递参数并调用exit函数。
- main函数中执行return语句并返回值。

//zombie.c
#include<stdio.h>
#include<unistd.h>

int main(int argc,char *agrv[])
{
	pid_t pid=fork();
	
	if(pid == 0) //if Child Process
	{
		puts("Hi ,I am a Child Process");
	}else
	{
		printf("Child Process ID : %d \n",pid);
		sleep(30);//睡眠30s
	}
	if(pid==0)
	{
		puts("End Child Process");
	}
	else{
		puts("End parent Process");
	}
	return 0;
}
子退出,父未退出,那么子进程就无人回收,就成了僵尸进程.

我们可以在Linux终端输入:ps au,可以看到Z+就代表僵尸进程
```

```cpp
销毁僵尸进程1:利用wait函数

#include<sys/wait.h>
pid_t wait(int *statloc);
->成功时返回终止的子进程ID,失效时返回-1

调用此函数时如果已有子进程终止，那么子进程终止时传递的返回值(exit函数的参数值、main函数的return返回值)将保存到该函数的参数所指内存空间。但函数参数指向的单元中还包含其他信息,因此需要通过下列宏进行分离。
WIFEXITED子进程正常终止时返回“真”。
WEXITSTATUS返回子进程的返回值。
也就是说，向wait函数传递变量status的地址时，调用wait函数后应编写如下代码。
if(WIFEXITED(status)) //是正常终止的吗?
{
	puts ( "Normal termination! ");
	printf("child pass num: %d"，WEXITSTATUS(status));//如果正常退出,那么返回值是多少?
}
根据上述内容编写示例,此示例中不会再让子进程变成僵尸进程。

//wait.c
#include <stdio.h>
#include <stdlib.h>
#include <unistd.h>
#include <sys/wait.h>
int main(int argc,char *argv[])
{
    int status;
    pid_t pid=fork();// 创建子进程
    
    if(pid==0)// 子进程代码
    {
        return 3;// 子进程返回值为3
    }
    else{// 父进程代码
        printf(“Chi1d PID: %d ln", pid);
        pid=fork();// 创建第二个子进程
        if(pid==0)// 第二个子进程代码
        {
        	exit(7);// 第二个子进程以值7退出
    	}
    }
    else
    {
        printf( "Child PID: %d in"， pid);
        wait(&status);// 等待第一个子进程终止
        
        if(WIFEXITED(status))// 打印第一个子进程的退出状态
        	printf( "Child send one: %d \n"，WEXITSTATUS(status));
        
        wait(&status);// 等待第二个子进程终止
        
        if(wIFEXITED(status))// 打印第二个子进程的退出状态
        	printf("child send two: %d \n", WEXITSTATUS(status));
        
        sleep( 30);// sleep 30 sec.
    }
    return 0;
}

调用wait函数时，如果没有已终止的子进程,那么程序将阻塞（Blocking）直到有子进程终止。

- WIFEXITED(status)是一个宏，用于检查进程是否正常终止。如果进程正常终止，则该宏返回一个非零值，否则返回0。
- WEXITSTATUS(status)是一个宏，用于提取进程的退出状态。如果进程正常终止，该宏将返回进程的退出状态值。
```

```cpp
销毁僵尸进程2:利用waitpid函数

wait函数会引起程序阻塞，还可以考虑调用waitpid函数。这是防止僵尸进程的第二种方法，也是防止阻塞的方法。

#include<sys/wait.h>
pid_t waitpid(pid_t pid,int *statloc,int options);
->成功时返回终止的子进程ID(或0),失败时返回-1
	pid:等待终止的目标子进程的ID,若传递-1,则与wait函数相同,可以等待任意子进程终止.
	statloc:与wait函数的statloc参数具有相同含义.
	options:传递头文件sys/wait.h中声明的常量WNOHANG,即使没有终止的子进程也不会进入阻塞状态,而是返回0并退出函数.
	
	
//下面介绍调用上述函数的示例.调用waitpid函数,程序不会阻塞.
//waitpid.c
#include<stdio.h>
#include<unistd.h>
#include<sys/wait.h>

int main(int argc,char *argv[])
{
	int status;
	pid_t pid = fork();
	
	if(pid == 0)
	{
		sleep(15);
		return 24;
	}
	else{
		while(!waitpid(-1,&status,WNOHANG))//若之前没有终止的子进程将返回0,所以waitpid函数并未阻塞
		{
			sleep(3);
			puts("sleep 3sec.")
		}
		if(WIFEXITED(status))
			printf("Child send %d \n",WEXITSTATUS(status));
	}
	return 0;
}
```
## 10.3 信号处理
我们已经知道了进程创建及销毁方法，但还有一个问题没解决。
"子进程究竟何时终止?调用waitpid函数后要无休止地等待吗?"
父进程往往与子进程一样繁忙，因此不能只调用waitpid函数以等待子进程终止。接下来讨论解决方案。

```cpp
向操作系统求助

子进程终止的识别主体是操作系统,因此若操作系统能把如下信息告诉正忙于工作的父进程,将有助于构建高效的程序.
"嘿,父进程!你创建的子进程终止!了"
此时父进程将暂时放下工作,处理子进程终止相关事宜.这是不是合理又很酷的想法呢?
为了实现该想法,我们引入信号处理机制.此时的"信号"是在特定时间发生时由操作系统向进程发送的消息.另外,为了响应该消息,执行于消息相关的自定义操作的过程称为"处理"或"信号处理".
```

```cpp
信号与signal函数

下面的对话有助于帮我们理解:
进程:"嘿,操作系统!如果我之前创建的子进程终止,就帮我调用zombie_handler函数"
操作系统:"好的!如果你的子进程终止,我会帮你调用zombie_handler函数,你先把该函数要执行的语句编好!"

上述对话中进程所讲的相当于"注册信号"过程,即进程发现自己的子进程结束时,请求操作系统调用特定函数.该请求通过如下函数调用完成(因此称此函数为信号注册函数)
#include<signal.h>
void (*signal(int signo,void (*func)(int) ) )(int);
->为了在产生信号时调用,返回之前注册的函数指针.
上述函数的声明整理如下:
函数名:signal
参数:int signo,void (* func)(int)
返回类型:参数为int类型,返回void型函数指针

signal数中注册的部分特殊情况和对应的常数:
- SIGALRM:已到通过调用alarm函数注册的时间。
- SIGINT:输入CTRL+C。
- sIGCHLD:子进程终止。


接下来编写调用signal函数的语句完成如下请求:
"子进程终止则调用mychild函数。"
此时"mychild函数"的参数应为int，返回值类型应为void。只有这样才能成为signal函数的第二个参数。
另外，常数SIGCHLD定义了子进程终止的情况，应成为signal函数的第一个参数。也就是说，,signal函数调用语句如下。
signal(SIGCHLD,mychild);


关于alarm函数:
#include<unistd.h>
unsigned int alarm(unisigned int seconds);
->返回0或以秒为单位的距SIGALRM信号发生所剩时间
如果调用该函数的同时向它传递一个正整型参数，相应时间后（以秒为单位）将产生SIGALRM信号。若向该函数传递0，则之前对SIGALRM信号的预约将取消。如果通过该函数预约信号后未指定该信号对应的处理函数，则（通过调用signal函数）终止进程，不做任何处理。
接下来给出信号处理相关示例:
//signal.c
#include<stdio.h>
#include<unistd.h>
#include<signal.h>

/*
timeout()函数是一个信号处理函数，当接收到SIGALRM信号时被调用。在这个函数中，它简单地打印出"Time out!"的消息，并通过调用alarm(2)函数再次预约2秒后发生SIGALRM信号，从而创建一个定时器循环。
*/
void timeout(int sig)//这是一个信号:定义信号处理函数.这种类型的函数称为信号处理器(Handler)
{
	if(sig == SIGALRM)
		puts("Time out!");
	alarm(2);
}

/*
keycontrol()函数是另一个信号处理函数，当接收到SIGINT信号（即按下CTRL+C）时被调用。在这个函数中，它简单地打印出"CTRL+C pressed"的消息。
*/
void keycontrol(int sig)//这是一个信号:定义信号处理函数.这种类型的函数称为信号处理器(Handler)
{
	if(sig == SIGINT)
		puts("CTRL+C pressed");
}
int main(int argc ,char *argv[])
{
	int i;
	signal(SIGALRM,timeout);//注册SIGALRM和SIGINT信号及相应处理器
	signal(SIGINT,keycontrol);
	/*
	- SIGALRM:已到通过调用alarm函数注册的时间。
	- SIGINT:输入CTRL+C。
	- sIGCHLD:子进程终止。
	*/
	alarm(2);//预约2秒后发生SIGALRM信号.
	for(i = 0 ; i < 3; i++)
	{
		puts("wait...");
		sleep(100);//为了查看信号产生和信号处理器的执行并提供每次100s,共3次的等待时间,在循环中调用sleep函数.也就是说,再过300s,约5分钟后终止程序,这是相当长的一段时间,但实际执行时只需要不到10s.
	}
	return 0;
}

"发生信号时将唤醒由于调用sleep函数而进入阻塞状态的进程。"
"进程一旦被唤醒,就不会再进入睡眠状态。" 
在for循环中,程序进入等待状态并打印出"wait..."的消息，然后通过调用sleep(100)函数等待100秒。这样循环执行3次，总共等待300秒。在这个过程中，如果计时器到达2秒（即先前预约的时间），就会触发SIGALRM信号，从而调用timeout()函数。另外，如果按下CTRL+C键，则会触发SIGINT信号，从而调用keycontrol()函数。

总结起来，这段代码会每隔2秒打印一次"Time out!"的消息，并且在接收到CTRL+C时打印"CTRL+C pressed"的消息。整个程序会执行3次循环，每次循环等待100秒(但是因为alarm函数,实际执行时只需不到10s)。
```

```cpp
利用sigaction函数进行信号处理

接下来介绍sigaction函数,他类似于signal函数,而且完全代替后者,也更稳定.
为什么稳定?是因为如下原因:
"signal函数在UNIX系列的不同操作系统中可能存在区别,但在sigaction函数完全相同."
而实际上现在很少使用signal函数编写程序,它只是为了保持对旧程序的兼容.下面介绍sigaction函数,但是只说可替换signal函数的功能.

#include<signal.h>
int sigaction(int signo,const struct sigaction *act,struct sigaction *oldact);
->成功时返回0,失败时返回-1
- signo:与signal函数相同，传递信号信息。
act:对应于第一个参数的信号处理函数(信号处理器）信息。
oldact:通过此参数获取之前注册的信号处理函数指针，若不需要则传递0。

声明并初始化sigaction结构体变量以调用上述函数，该结构体定义如下:
struct sigaction
{
	void (*sa_handler)(int);//保存信号处理函数的指针值(地址值)
	sigset_t sa_mask;//初始化为0即可
	int sa_flags;//初始化为0即可
}

下面的例子,使用sigaction函数防止产生僵尸进程
//sigaction.c
#include<stdio.h>
#include<unistd.h>
#include<signal.h>

void timeout(int sig)
{
	if(sig == SIGALRM)
	{
		puts("Time out!");
	}
	alarm(2);
}
int main(int argc ,char *argv[])
{
	int i;
	struct sigaction act;
	act.sa_handler = timeout;//为了注册信号处理函数，声明sigaction结构体变量并在sa_handler成员中保存函数指针值。

	sigemptyset(&act.sa_mask);//调用sigemptyset函数将sa_mask成员的所有位初始化为O。

	act.sa_flags = 0;//sa_flags成员同样初始化为0即可
	
	//注册SIGALRM信号的处理器。调用alarm函数预约2秒后发生SIGALRM信号。
	sigaction(SIGALRM,&act,0);//SIGALRM:已到通过调用alarm函数注册的时间。
	alarm(2);
	for(i=0;i<3;i++)
	{
		puts("wait...");
		sleep(100);
	}
	return 0;
}
```

```cpp
利用信号处理技术消灭僵尸进程
//sIGCHLD:子进程终止。

子进程终止时将产生SIGCHLD信号，知道这一点就很容易完成。接下来利用sigaction函数编写示例。

//remove_zombie.c
#include<stdio.h>
#include<sdlib.h>
#include<unistd.h>
#include<signal.h>
#include<sys/wait.h>

void read_childproc(int sig)
{
	int status;
	pid_t id = waitpid(-1,&status,WNOHANG);//若传递-1,则与wait函数相同,可以等待任意子进程终止.//传递头文件sys/wait.h中声明的常量WNOHANG,即使没有终止的子进程也不会进入阻塞状态,而是返回0并退出函数.
	if(WIFEXITED(status))
	{
		printf("Remove proc id : %d \n",id);
		printf("Child send :%d \n",WEXITSTATUS(status));
	}
}

int main(int argc , char *argv[])
{
	pid_t pid;
	
	/*
	注册SIGCHLD信号对应的处理器。若子进程终止，则调用read_childproc函数。处理函数中调用了waitpid函数，所以子进程将正常终止，不会成为僵尸进程。
	*/
	struct sigaction act;
	act.sa_handler = read_childproc;
	sigemptyset(&act.sa_mask);
	act.sa_flags=0;
	sigaction(SIGCHLD,&act,0);//子进程终止,就调用act的内置函数
	
	pid = fork();
	if(pid==0)//子进程执行区域
	{
		puts("Hi I'm child process");
		sleep(10);
		reurn 12;
	}else//父进程执行区域
	{
		printf("Child proc id : %id \n",pid);
		pid=fork();
		if(pid == 0)//另一子进程执行区域
		{
			puts("Hi I'm child process");
			sleep(10);
			exit(24);
		}
		else
		{
            int i;
            printf("Child proc id : %d \n",pid);
            
            /*
            为了等待发生SIGCHLD信号，使父进程共暂停5次，每次间隔5秒。发生信号时，父进程将被唤醒,因此实际暂停时间不到25秒。
            */
            for(i = 0;i<5;i++)
            {
                puts("wait...");
                sleep(5);
            }
		}
	}  
	return 0;
}
```

## 10.4 基于多任务的并发服务器

```cpp
基于进程的并发服务器模型
```

```cpp
实现并发服务器
//echo_mpserv.c
#include<stdio.h>
#include<stdlib.h>
#include<string.h>
#include<unistd.h>
#include<signal.h>
#incldue<sys/wait.h>
#include<arpa/inet.h>
#include<syssocket.h>

#define BUF_SIZE 30
void error handling(char *message);
void read_childproc(int sig);

int main(int argc,char *aggv[])
{
    int serv_sock,clnt_sock;
    struct sockaddr_in serv_adr,clnt_adr;
    
    pid_t pid;
    struct sigaction act;
    socklen_t adr_sz;
    int str_len,state;
    char buf[BUF_SIZE];
    if(argc != 2)
    {
    	printf("Usage : %s <port> \n",argv[0]);
    	exit(1);
    }
    
    act.sa_handler=read_childproc;
    sigemptyset(&act.sa_mask);
    act.sa_flags=0;
    state=sigaction(SIGCHLD,&act,0);
    serv_sock=socket(PF_INET,SOCK_STREAM,0);
    memset(&serv_adr,0,sizeof(serv_adr));
    serv_adr.sin_family=AF_INET;
    serv_adr.sin_addr.s_addr=htonl(INADDR_ANY);
    serv_adr.sin_port=htons(atoi(argv[1]));
    
    if(bin(serv_sock,(struct sockaddr*)&serv_adr,sizeof(serv_adr)) == -1)
    {
    	error_handling("bind() error");
	}
	if(listen(serv_sock,5)==-1)
	{
		error_handling("listen() error");
	}
    
    while(1)
    {
    	adr_sz=sizeof(clnt_adr);
    	clnt_sock=accept(serv_sock,(struct sockaddr*)&clnt_adr,&adr_sz);
    	if(clnt_sock == -1)
    		contine;
    	else
    		puts("new client connected...");
    	pid = fork();
    	if(pid == -1)
    	{
    		close(clnt_sock);
    		conntinue;
    	}
    	if(pid == 0)
    	{
    		close(serv_sock);
    		while( (str_len = read(clnt_sock,buf,BUF_SIZE)) !=0 )
    		{
    			write(clnt_sock,buf,str_len);
    		}
    		close(clnt_sock);
    		puts("client disconnected...");
    		return 0;
    	}
    	else
    		close(clnt_sock);
    }
    close(serv_sock);
    return 0;
}

void read_childproc(int sig)
{
	pid_t pid;
	int status;
	pid = waitpid(-1,&status,WNOHANG);
	printf("remove proc id : %d \n",pid);
}
void error_handling(char *message)
{
	fputs(message,stderr);
	fputc('\n',stderr);
	exit(1);
}

```
```cpp
通过fork函数负责文件描述符


回声客户端的i/o程序分割
//echo_mpclient.c
#include<stdio.h>
#include<stdlib.h>
#include<string.h>
#include<unistd.h>
#include<arpa/inet.h>
#include<sys/socket.h>

#define BUF_SIZE 30
void error_handling(char *message);
void read_routine(int sock,char *buf);
void write_routine(int sock,char *buf);

int main(int argc,char *argv[])
{
	int sock;
    pid_t pid;
    char buf[BUF_SIZE];
    struct sockaddr_in serv_adr;
    if(argc != 3)
    {
        printf("Usage: %s <IP> <port>\n", argv[0]);
        exit(1);
    }

    sock = socket(AF_INET, SOCK_STREAM,0);
    memset(&serv_adr,0,sizeof(serv_adr));
    serv_adr.sin_family=AF_INET;
    serv_adr.sin_addr.s_addr=inet_addr(argv[1]);
    serv_adr.sin_port=htons(atoi(argv[2]));

    if(connect(sock,(struct sockaddr*)&serv_adr,sizeof(serv_adr))==-1)
        error_handling("connect() error!");

    pid = fork();
    if(pid == 0)
        write_routine(sock,buf);
    else
        read_routine(sock,buf);
    
    close(sock);
	return 0;
}
void read_routine(int sock, char *buf)
{
    while(1){
        int str_len = read(sock,buf,BUF_SIZE);
        if(str_len == 0)
            return 0;
        buf[str_len] = '\0';
        printf("Message from server : %s", buf);
    }
}
void write_routine(int sock, char *buf)
{
    while (1)
    {
        fgets(buf, BUF_SIZE ,stdin);
        if(!strcmp(buf , "q\n") || !strcmp(buf,"Q\n"))
        {
            shutdown(sock,SHUT_WR);
            return;
        }
        write(sock,buf,strlen(buf));

    }
    
}

void error_handling(char *message)
{
    fputs(message,stderr);
    fputc('\n',stderr);
    exit(1);
}
```

## 10.5 分割TCP的IO程序


# 第11章 进程间通信(P183)


## 11.1进程间通信的基本概念
```cpp
通过管道实现进程间通信

#include<unistd.h>
int pipe(int filedes[2]);
->成功时返回0,失败时返回-1


//pipe1.c
#include <stdio.h>
#include <unistd.h>
#define BUF_SIZE 30

int main(int argc, char *argv[])
{
    int fds[2];
    char str[] = "who are you?";
    pid_t pid;

    pipe(fds);
    pid = fork();
    if(pid == 0)
    {
        write(fds[1], str, BUF_SIZE);//fd[1]是管道入口

    }
    else{
        read(fds[0], str, BUF_SIZE);//fd[0]是管道出口
        puts(buf);
    }
    return 0;
}
```

```cpp
通过管道进行进程间双向通信

//pipe2.c
//只用一个管道
#include <stdio.h>
#include <unistd.h>
#define BUF_SIZE 30

int main(int argc, char *argv[])
{
    int fds[2];
    char str1[] = "Who are you?";
    char str2[] = "Thank you for your message.";
    char buf[BUFSIZE];
    pid_t pid ;

    //一个管道用两次
    pipe(fds);
    if(pid == 0)
    {
        write(fds[1], str1, sizeof(str1));
        sleep(1);
        read(fds[0], buf, BUF_SIZE);
        printf("Child proc output : %s\n", buf);
    }
    else{
        read(fds[0], buf, BUF_SIZE);
        printf("Parent proc output : %s\n", buf);
        write(fds[1], str2, sizeof(str2));
        sleep(3);
    }
    return 0;
}
/*
Parent proc output: who are you?
Child proc output: Thank you for your message
*/




// pipe3.c
// 使用两个管道
#include <stdio.h>
#include <unistd.h>
#define BUF_SIZE 30

int main(int argc, char *argv[])
{
    int fds1[2], fds2[2];
    char str1[] = "Who are you?";
    char str2[] = "Thank you for your message";
    char buf[BUF_SIZE];
    pid_t pid;

    pipe(fds1), pipe(fds2);
    pid = fork();
    if (pid == 0)
    {
        write(fds1[1], str1, sizeof(str1));
        read(fds2[0], str2, BUF_SIZE); // read用BUF_SIZE
        printf("Child proc output : %s \n", buf);
    }
    else
    {
        read(fds1[0], str2, BUF_SIZE); // read用BUF_SIZE
        printf("Parent proc output : %s \n", buf);
        write(fds2[1], str2, sizeof(str2));
        sleep(3);
    }
    return 0;
}
/*
Parent proc output: who are you?
Child proc output: Thank you for your message
*/

```
## 11.2 运用进程间通信
```cpp
运用进程间通信

保存信息的回声服务器端:
下面扩展第10章的echo_mpserv.c，添加如下功能:
“将回声客户端传输的字符串按序保存到文件中。”
我希望将该任务委托给另外的进程。换言之，另行创建进程，从向客户端提供服务的进程取字符串信息。当然，该过程中需要创建用于接收数据的管道。
下面给出示例。
该示例可以与任意回声客户端配合运行，但我们将用第10章介绍过echo_mpclient.c。


//echo_storeserv.c
#include<stdio.h>
#include<stdlib.h>
#include<string.h>
#include<unistd.h>
#include<arpa/inet.h>
#include<sys/socket.h>
#define BUF_SIZE 100

void error_handling(char *message);
void read_childproc(int sig);

int main(int argc,char *argv[])
{
    int serv_sock,clnt_sock;
    struct sockaddr_in serv_adr,clnt_adr;
    int fds[2]; // 用于创建管道的文件描述符数组

    pid_t pid;
    struct sigaction act;
    socklen_t adr_sz;
    int str_len,state;
    char buf[BUF_SIZE];
    if(argc!=2)
    {
        printf("Usage : %s <port> \n",argv[0]);
        exit(1);
    }

    act.sa_handler=read_childproc;
    sigemptyset(&act.sa_mask);
    act.sa_flags=0;
    state=sigaction(SIGCHLD,&act,0);

    serv_sock = socket(PF_INET,SOCK_STREAM,0); // 创建服务器套接字
    memset(&serv_adr,0,sizeof(serv_adr));
    serv_adr.sin_family=AF_INET;
    serv_adr.sin_addr.s_addr = htonl(INADDR_ANY); // 服务器IP地址
    serv_adr.sin_port=htons(atoi(argv[1])); // 服务器端口号

    if(bind(serv_sock,(struct sockaddr*)&serv_adr,sizeof(serv_adr))==-1) // 绑定服务器套接字
        error_handling("bind() error");
    if(listen(serv_sock,5)==-1) // 监听连接请求
        error_handling("listen() error");

    pipe(fds); // 创建管道
    pid=fork(); // 创建子进程
    if(pid == 0) // 子进程代码
    {
        FILE * fp = fopen("echomsg.txt","wt"); // 打开文件以写入接收到的字符串
        char msgbuf[BUF_SIZE];
        int i,len;

        for(i = 0 ; i  < 10 ; i++) // 接收并保存10次字符串
        {
            len  = read(fds[0],msgbuf,BUF_SIZE); // 从管道中读取字符串
            fwrite((void*)msgbuf,1,len,fp); // 写入文件
        }
        fclose(fp); // 关闭文件
        return 0;
    }
    while(1){
        adr_sz=sizeof(clnt_adr);
        clnt_sock=accept(serv_sock,(struct sockaddr*)&clnt_adr,&adr_sz); // 接受客户端连接请求
        if(clnt_sock==-1)
            continue;
        else
            puts( "new client connected...");
    
        pid=fork(); // 创建子进程处理客户端请求
        if(pid==0){ // 子进程代码

            close(serv_sock); // 子进程关闭服务器套接字
            while((str_len=read(clnt_sock, buf,BUF_SIZE))!=0){ // 从客户端接收字符串
                write(clnt_sock, buf, str_len); // 发送字符串回客户端
                write(fds[1], buf, str_len); // 将字符串写入管道，传递给子进程
            }
        
            close(clnt_sock); // 关闭客户端套接字
            puts( "client disconnected. . .");
            return 0; // 子进程结束
        }
        else
            close(clnt_sock); // 父进程关闭客户端套接字
    }
    close(serv_sock); // 关闭服务器套接字
    return 0;
}

void read_childproc(int sig)
{
    pid_t pid;
    int status;
    pid = waitpid(-1,&status,WNOHANG); // 回收子进程
    printf("remove proc id : %d \n",pid);
}

void error_handling(char *message)
{
    fputs(message,stderr);
    fputc('\n',stderr);
    exit(1);
}
这段代码创建了一个服务器，可以接受客户端的连接请求，并将客户端传输的字符串保存到文件中。其中，使用管道实现了将字符串传递给另外的进程进行保存的功能。


```

# 第12章 IO多路复用(P194)

## 12.1 基于IO复用的服务器端



## 12.2 理解select函数并实现服务器端

```cpp
FD_ZERO(fd_set* fdset) : 
将fd_set变量的所有位初始化为0。
FD_SET(int fd, fd_set * fdset) :
在参数fd_set指向的变量中注册文件描述符fd的信息。
FD_CLR(int fd, fd_set * fdset) :
从参数fd_set指向的变量中清除文件描述符fd的信息。
FD_ISSET(int fd, fd_set *fdset):
若参数fd_set指向的变量中包含文件描述符fd的信息,则返回“真”。




设置检查(监视)范围及超时
#include<sys/select.h>
#include<sys/time.h>
int select(
    int maxfd,fd_set *readset,fd_set *writeset,fd_set *exceptset,const struct timeval *timeout
)
->成功时返回>0的值,失败时返回-1

- maxfd:监视对象文件描述符数量。
- readset:将所有关注“是否存在待读取数据”的文件描述符注册到fd_set型变量,并传递其地址值。
- writeset:将所有关注“是否可传输无阻塞数据”的文件描述符注册到fd_set型变量,并传递其地址值。//0
- exceptset:将所有关注“是否发生异常”的文件描述符注册到fd_set型变量,并传递其地址值。//0
- timeout:调用select函数后,为防止陷入无限阻塞的状态，传递超时(time-out)信息。
- 返回值:发生错误时返回-1,超时返回时返回0。因发生关注的事件返回时,返回大于0的值,该值是发生事件的文件描述符数。

如上所述,select函数用来验证3种监视项的变化情况。
根据监视项声明3个"fd_set"型变量,分别向其注册文件描述符信息,
并把变量的地址值传递到上述函数的第二到第四个参数。
但在此之前(调用select函数前)需要决定下面2件事。
"文件描述符的监视（检查）范围是?""如何设定select函数的超时时间?"

第一,文件描述符的监视范围与select函数的第一个参数有关(maxfd)。
需要得到注册在fd_set变量中的文件描述符数。但每次新建文件描述符时，其值都会增1，故只需将最大的文件描述符值加1再传递到select函数即可。
加1是因为文件描述符的值从0开始。
第二，select函数的超时时间与select函数的最后一个参数有关，其中timeval结构体定义如下。
struct timeval
{
    long v_sec;//秒
    long v_usec;//毫秒
};
如果不想设置超时,则传递NULL参数.




调用select函数后查看结果
select函数调用示例
//select.c  客户端代码
//它使用 select 函数来监听标准输入（键盘输入），并在指定的等待时间内读取并处理输入。
//在这个代码片段中，并没有涉及网络连接或服务器端的功能。它只是展示了如何使用 select 函数进行输入的非阻塞监听。
#include<sdio.h>
#include<unistd.h>
#include<sys/time.h>
#include<sys/select.h>
#define BUF_SIZE 30

int main(int argc,char *argv[])
{
    fd_set reads,temps;//fd_set 类型的 reads 和 temps
    int result,str_len;
    char buf[BUF_SIZE];
    struct timeval timeout;
    FD_ZERO(&reads);//使用 FD_ZERO 宏将 reads 初始化为空集。
    FD_SET(0,&reads);//使用 FD_SET 宏将标准输入文件描述符 0（对应于键盘输入）添加到 reads 集合中。

    /*
    timeout.tv_sec = 5;
    timeout.tv_usec = 5000;
    */

    while(1)
    {
        temps=reads;//首先将 reads 集合赋值给 temps，因为 select 函数会修改传入的集合，所以需要使用临时变量。
        timeout.tv_sec = 5;//设置 timeout 结构体的值，表示等待时间。在这个例子中，设置为 5 秒。
        timeout.tv_usec = 0;
        
        result=select(1,&temps,0,0,&timeout);
        if(result == -1)
        {
            puts("select() error!");
            break;
        }
        else if(result == 0)
        {
            puts("Time-out!");
        }
        else//>0
        {
            //使用 FD_ISSET 宏检查标准输入文件描述符 0 是否在 temps 集合中可读。
            if(FD_ISSET(0,&temps))
            {
                str_len = read(0,buf,BUF_SIZE);
                buf[str_len] = 0;
                printf("message from console: [%s]\n",buf);
            }
        }
    }
    return 0;
}
代码的主要目的是通过 select 函数监听标准输入文件描述符，如果在指定的等待时间内有输入发生，就读取并处理输入。这样可以实现非阻塞式的输入操作。
```

```cpp
实现I/O复用服务器端
//echo_selectserv.c
#include<stdio.h>
#include<stdlib.h>
#include<string.h>
#include<unistd.h>
#include<arpa/inet.h>
#include<sys/socket.h>
#include<sys/time.h>
#include<sys/select.h>

#define BUF_SIZE 100
void error_handling(char *buf);

int main(int argc, char *argv[]) 
{
    int serv_sock,clnt_sock;
    struct sockaddr_in serv_adr,clnt_adr;
    struct timeval timeout;
    fd_set reads,cpy_reads;

    socklen_t adr_sz;
    int fd_max,str_len,fd_num,i;
    char buf[BUF_SIZE];
    if(argc != 2)
    {
        printf("Usage: %s <port>\n", argv[0]);
        exit(1);
    }

    serv_sock = socket(PF_INET,SOCK_STREAM,0);
    memset(&serv_adr,0,sizeof(serv_adr));
    serv_adr.sin_family = AF_INET;
    serv_adr.sin_addr.s_addr=htonl(INADDR_ANT);
    serv_adr.sin_port=htons(atoi(argv[1]));

    if(bind(serv_sock,(struct sockaddr*)&serv_adr,sizeof(serv_adr))==-1)
        error_handling("bind() error");
    if(listen(serv_sock,5)==-1)
        error_handling("listen() error");

    FD_ZERO(&reads);
    FD_SET(serv_sock,&reads);
    fd_max=serv_sock;

    while (1)
    {
        cpy_read=reads;
        timeout.tv_sec=5;
        timeout.tv_usec=5000;

        if((fd_num=select(fd_max+1,&cpy_reads,0,0,&timeout))==-1)
            break;
        if(fd_num == 0)
            continue;

        for(i = 0 ; i < fd_max +1 ; i++)
        {
            if(FD_ISSET(i,&cpy_reads)
            {
                if(i == serv_sock)
                {
                    adr_sz = sizeof(clnt_adr);
                    clnt_sock=
                        accept(serv_sock,(struct sockaddr*)&clnt_adr,&adr_sz);
                    FD_SET(clnt_sock,&reads);
                    if(fd_max<clnt_sock)
                        fd_max=clnt_sock;
                    prinf("connected client : %d \n",clnt_sock);
                }
                else
                {
                    str_len=read(i,buf,BUF_SIZE);
                    if(str_len==0)
                    {
                        FD_CLR(i,&reads);
                        close(i);
                        printf("close client : %d \n",i);
                    }
                    else{
                        write(i,buf,BUF_SIZE);
                    }
                }
            }
        }
    }
    
    close(serv_sock);
    return 0;
}

void error_handling(char *buf)
{
    fputs(buf,stderr);
    fputc('\n',stderr);
    exit(1);
}
```



# 第13章 多种IO函数

## 13.1 send & recv函数

```cpp
Linux中的send & recv

#include<sys/socket.h>
ssize_t send(int sockfd,const void *buf,size_t nbytes,int flags);
->成功时返回发送的字节数,失败时返回-1
- sockfd:表示与数据传输对象的连接的套接字文件描述符。
- buf:保存待传输数据的缓冲地址值。
- nbytes:待传输的字节数。
- flags:传输数据时指定的可选项信息。


#include<sys/socket.h>
ssize_t recv(int sockfd,void *buf,size_t nbytes,int flags);
->成功时返回接收的字节数(收到EFO时返回0),失败时返回-1
- sockfd:表示数据接收对象的连接的套接字文件描述符。
- buf:保存接收数据的缓冲地址值。
- nbytes:可接收的最大字节数。
- flags:接收数据时指定的可选项信息。


"send函数和recv函数的最后一个参数是收发数据时的可选项。该可选项可利用位或( bitOR )运算（|运算符）同时传递多个信息。"

```

```cpp
MSG_OOB:发送紧急消息
"当然应该优先处理"
//oob_send.c
#include<stdio.h>
#include<unistd.h>
#include<stdlib.h>
#include<string.h>
#include<sys/socket.h>
#include<arpa/inet.h>

#define BUF_SIZE 30
void error_handling(char *message);

int main(int argc, char *argv[])
{
    int sock;
    struct sockaddr_in recv_adr;
    if(argc != 3)
    {
        printf("Usage: %s <IP> <port>\n",argv[0]);
        exit(1);
    }

    sock = socket(PF_INET,SOCK_STREAM,0);
    memset(&recv_adr,0,sizeof(recv_adr));
    recv_adr.sin_family = AF_INET;
    recv_adr.sin_addr.s_addr=inet_addr(argv[1]);
    recv_adr.sin_port=htos(atoi(argv[2]));

    if(connect(sock,(struct sockadrr*)&recv_adr,sizeof(recv_adr))==-1)
        error_handling("connect() error");
    
    //write 写 send 发
    write(sock,"123",strlen("123"));
    send(sock,"4",strlen("4"),MSG_OOB);
    write(sock,"567",strlen("567"));
    send(sock,"890",strlen("890"),MSG_OOB);
    close(sock);
    return 0;
}
void error_handling(char *message)
{
    fputs(message,stderr);
    fpurc('\n',stderr);
    exit(1);
}

//oob_recv.c
#include<sdio.h>
#include<unistd.h>
#include<stdlib.h>
#include<string.h>
#include<signal.h>
#include<sys/socke.h>
#include<neinet/in.h>
#include<fcntl.h>

#define BUF_SIZE 30
void error_handling(char *message);
void urg_handler(int signo);

int acpt_sock;
int recv_sock;

int main(int argc,char *argv[])
{
    struct sockaddr_in recv_adr,serv_adr;
    int str_len,state;
    socklen_t serv_adr_sz;
    struct sigaction act;
    char buf[BUF_SIZE];
    if(argc != 2)
    {
        printf("Usage : %s <port> \n",argv[0]);
        exit(1);
    }

    act.sa_handler=urg_handler;
    sigempyset(&act.sa_mask);
    act.sa_flags=0;

    acpt_sock=socket(PF_INET,SOCK_SREAM,0);
    memset(&recv_adr,0,sizeof(recv_adr));
    recv_adr.sin_family=AF_INET;
    recv_adr.sin_addr.s_addr=htonl(INADDR_ANY);
    recv_adr.sin_port=htons(atoi(argv[1]));

    if( bind(acpt_sock,(struct sockaddr*)&recv_adr,sizeof(recv_adr))==-1)
        error_handling("bind() error");
    listen(acpt_sock,5);
    serv_adr_sz=sizeof(serv_adr);
    recv_sock=accept(acpt_scok,(struct sockaddr*)&serv_adr,&serv_adr_sz);

    fcntl(recv_sock,F_SETOWN,getpid());
    state=sigaction(SIGURG,&act,0);

    while((str_len=recv(recv_sock,buf,sizeof(buf),0))!=0)
    {
        if(str_len == -1)
            continue;
        buf[str_len]=0;
        puts(buf);
    }
    close(recv_sock);
    close(acpt_sock);
    return 0;
}
void urg_handler(int signo)
{
    int str_len;
    char buf[BUF_SIZE];
    str_len=recv(recv_sock,buf,sizeof(buf)-1,MSG_OOB);
    buf[str_len]=0;
    printf("Urgent message:%s \n",buf);
}
void error_handling(char *message)
{
    fputs(message,stderr);
    fputc('\n',stderr);
    exit(1);
}
```

```cpp
下列示例给出了使用MSG_PEEK和MSG_DONTWAIT选项的结果。(P219)
//peek_recv.c
#include<stdio.h>
#include<unistd.h>
#include<stdlib.h>
#include<string.h>
#include<sys/socket.h>
#include<arpa/inet.h>

#define BUF_SIZE 30
void error_handling(char *message);

int main(int argc, char *argv[])
{
    int acpt_sock,recv_sock;
    struct sockaddr_in acpt_adr,recv_adr;
    int str_len,state;
    socklen_t recv_adr_sz;
    char buf[BUF_SIZE];
    if(argc != 2)
    {
        printf("Usage: %s <port> \n",argv[0]);
        exit(1);
    }

    acpt_sock=socket(PF_INET,SOCK_STREAM,0);
    memset(&acpt_adr,0,sizeof(acpt_adr));
    acpt_adr.sin_family=AF_INET;
    acpt_adr.sin_addr.s_addr=htonl(INADDR_ANY);
    acpt_adr.sin_port = htons(atoi(argv[1]));

    if(bind(acpt_sock,(struct sockaddr*)&acpt_adr,sizeof(acpt_adr))==-1)
        error_handling("bind() error");
    listen(acpt_sock,5);

    recv_adr_sz=sizeof(recv_adr);
    recv_sock=accept(acpt_sock,(struct sockaddr*)&recv_adr,&recv_adr_sz);

    while(1)
    {
        str_len=recv(recv_sock,buf,sizeof(buf)-1,MSG_PEEK|MSG_DONTWAIT);
        if(str_len>0)
            break;
    }

    buf[str_len]=0;
    printf("Buffering %d bytes : %s\n",str_len,buf);

    str_len=recv(recv_sock,buf,sizeof(buf)-1,0);
    buf[str_len]=0;
    printf("Read again : %s \n",buf);
    close(acpt_sock);
    close(recv_sock);
    
    return 0;
}

void error_handling(char *message)
{
    fputs(message,stderr);
    fputc('\n',stderr);
    exit(1);
}

```



## 13.2 readv & writev 函数

```cpp
使用read & writev函数的功能可以概括如下:
"对数据进行整合传输及发送的函数"
也就是说,通过writev函数可以将分散保存在多个缓冲中的数据一并发送,
通过readv函数可以由多个缓冲分别接收。
因此,适当使用这2个函数可以减少IO函数的调用次数。
下面先介绍writev函数。
#include<sys/uio.h>
ssize_t writev(int fileds,const struct iovec * iov,int iovcnt);
->成功时返回发送的字节数,失败时返回-1
- filedes:表示数据传输对象的套接字文件描述符。但该函数并不只限于套接字,
因此,可以像read函数一样向其传递文件或标准输出描述符。
- iov:iovec结构体数组的地址值,结构体iovec中包含待发送数据的位置和大小信息。
- iovcnt:向第二个参数传递的数组长度。

上述函数的第二个参数中出现的数组iovec结构体的声明如下:
struct iovec
{
    void * iov_base; //缓冲地址
    size_t iov_len; //缓冲大小
}


//writev.c
#include<stdio.h>
#include<sys/uio.h>

int mian(int argc,char *argv[])
{
    struct iovec vec[2];
    char buf1[]="ABCDEFG";
    char buf2[]="1234567";
    int str_len;

    vec[0].iov_base = buf1;
    vec[0].iov_len = 3;
    vec[1].iov_base = buf2;
    vec[1].iov_len = 4;

    str_len = writev(1,vec,2);
    puts("");
    printf("Write bytes: %d\n",str_len);
    return 0;
}

```

```cpp
下面介绍readv函数,它与writev函数正好相反.
#include<sys/uio.h>
ssize_t readv(int filedes,const struct iovec *iov,int iovcnt);
->成功时返回接收的字节数,失败时返回-1

- filedes:传递接收数据的文件（或套接字）描述符。
- iov:包含数据保存位置和大小信息的iovec结构体数组的地址值。
- iovcnt:第二个参数中数组的长度。

//readv.c
#include<stdio.h>
#include<sys/uio.h>
#define BUF_SIZE 100

int main(int argc, char *argv[])
{
    struct iovec vec[2];
    char buf1[BUF_SIZE]={0,};
    char buf2[BUF_SIZE]={0,};
    int str_len;

    vec[0].iov_base = buf1;
    vec[0].iov_len = 5;
    vec[1].iov_base = buf2;
    vec[1].iov_len = BUF_SIZE;

    str_len = readv(0,vec,2);
    printf("Read bytes : %d \n",str_len);
    printf("First message : %s \n",buf1);
    printf("Second message : %s \n",buf2);
    return 0;
}

```



# 第14章 多播与广播(P230)

广播（ Broadcast )在“一次性向多个主机发送数据”这一点上与多播类似，但传输数据的范围有区别。多播即使在跨越不同网络的情况下，只要加入多播组就能接收数据。相反，广播只能向同一网络中的主机传输数据。

## 14.1 多播

```cpp
1.多播( Multicast )方式的数据传输是基于UDP完成的。
因此,与UDP服务器端/客户端的实现方式非常接近。
区别在于:UDP数据传输以单一目标进行,而多播数据同时传递到加入（注册)特定组的大量主机。
换言之，采用多播方式时，可以同时向多个主机传递数据。


2.多播的数据传输方式及流量方面的优点
- 多播服务器端针对特定多播组，只发送1次数据。
- 即使只发送1次数据,但该组内的所有客户端都会接收数据。
- 多播组数可在P地址范围内任意增加。
- 加入特定组即可接收发往该多播组的数据。
多播组是D类IP地址（224.0.0.0~239.255.255.255 )，“加入多播组”可以理解为通过程序完成如下声明:
"在D类IP地址中，我希望接收发往目标239.234.218.234的多播数据。"

多播是基于UDP完成的,也就是说,多播数据包的格式与UDP数据包相同.
只是与一般的UDP数据包不同,向网络传递1个多播数据包时,路由器将复制数据包并传递到多个主机.
像这样,多播需要借助路由器完成.



3.路由(Routing)和TTL(Time to Live),以及加入组的方法

为了传递多播数据包,必需设置TTL.

接下来给出TTL设置方法.程序中的TTL设置是通过套接字可选性完成的.与设置TTL相关的协议层为IPPROTO_IP,
选项名为IP_MULTICAST_TTL.因此,可以用如下代码把TTL设置为64.
int send_sock;
int time_live=64;
. . . .
send_sock=socket(PF_INET,SOCK_DGRAM,0);
setsockopt(send_sock,IPPROTO_IP,IP_MULTICAST_TTL,(void*) &time_live,sizeof(time_live));
. . . .
另外,加入多播组也通过设置套接字选项完成。加人多播组相关的协议层为IPPROTO_IP,选项名为IP_ADD_MEMBERSHIP。
可通过如下代码加人多播组。
int recv_sock;
struct ip_mreq join_adr;
. . . .
recv_sock=socket(PF_INET,sOCK_DGRAM,8);
. . . .
join_adr.imr_multiaddr.s_addr="多播组地址信息";
join_adr.imr_interface.s_addr="加入多播组的主机地址信息";
setsockopt(recv_sock,IPPROTO_IP,IP_ADD_MENBERSHIP，(void*) & join_adr,sizeof(join_adr));
. . . .



4.实现多播Sender和Receiver
Sender:向AAA组广播( Broadcasting)文件中保存的新闻信息。
Receiver:接收传递到AAA组的新闻信息。

接下来给出Sender代码:
//news_sender.c
#include<stdio.h>
#include<stdlib.h>
#include<string.h>
#include<unistd.h>
#include<arpa/inet.h>
#include<sys/socket.h>

#define TTL 64
#define BUF_SIZE 30
void error_handling(char * message);

int main(int argc, char *argv[])
{
    int send_sock;
    struct sockaddr_in mul_adr;
    int time_live = TTL;
    FILE *fp;
    char buf[BUF_SIZE];
    if(argc != 3)
    {
        printf("Usage:%s <GroupIP> <PORT> \n", argv[0);
        exit(1);
    }

    send_sock=socket(PF_INET,SOCK_DGRAM,0);
    memset(&mul_adr,0,sizeof(mul_adr));
    mul_adr.sin_family = AF_INET;
    mul_adr.sin_addr.s_addr=inet_addr(argv[1]);
    mul_adr.sin_port=htons(atoi(argv[2]));

    setsockopt(send_sock,IPPROTO_IP,
        IP_MULTICAST_TTL,(void*)&time_live,sizeof(time_live));
    if( (fp=fopen("news.txt","r")) == NULL )
        error_handling("fopen() error");
    
    while(!feof(fp))
    {
        fgets(buf,BUF_SIZE,fp);
        sendto(send_sock,buf,strlen(buf),
            0,(struct sockaddr*)&mul_adr,sizeof(mul_adr));
        sleep(2);
    }
    fclose(fp);
    close(send_sock);

    return 0;
}

void error_handling(char *message)
{
    fputs(message,stderr);
    fputc('\n',stderr);
    exit(1);
}

从上述代码中可以看到,Sender与普通的UDP套接字程序相比差别不大.
但多播RECEiver则有些不同.为了接收传向任意多播地址的数据,需要经过加入躲避组的过程.
除此之外,Receiver通信与UDP套接字程序差不多.
接下来给出与上述示例结合使用的Receiver程序.
//news_receiver.c
#include<stdio.h>
#include<stdlib.h>
#include<string.h>
#include<unistd.h>
#include<arpa/inet.h>
#include<sys/socket.h>

#define BUF_SIZE 30
void error_handling(char *message);

int main(int argc, char *argv[])
{
    int recv_sock;
    int str_len;
    char buf[BUF_SIZE];
    struct sockaddr_in adr;
    struct ip_mreq join_adr;

    if(argc != 3)
    {
        printf("Usage : %s <GroupIP> <PORT>\n",argv[0]);
        exit(1);
    }

    recv_sock=socket(PF_INET,SOCK_DGRAM,0);
    memset(&adr,0,sizeof(adr));
    adr.sin_family = AF_INET;
    adr.sin_addr.s_addr=htonl(INADDR_ANY);
    adr.sin_port=htons(atoi(argv[2]));

    if(bind(recv_sock,(struct sockaddr*)&adr,sizeof(adr))==-1)
        error_handling("bind() error");

    join_adr.imr_multiaddr.s_addr=inet_addr(argv[1]);
    join_adr.imr_interface.s_addr=htonl(INADDR_ANY);

    setsockopt(recv_sock,IPPROTO_IP,
        IP_ADD_MEMBERSHIP,(void*)&join_adr,sizeof(join_adr));

    while(1){
        str_len = recvfrom(recv_sock,buf,BUF_SIZE-1,0,NULL,0);
        if(str_len<0)
            break;
        buf[str_len]=0;
        fputs(buf,stdout);
    }
    close(recv_sock);
    return 0;
}

void error_handling(char *message)
{
    fputs(message,stderr);
    fputc('\n',stderr);
    exit(1);
}

```

## 14.2 广播

```cpp
实现广播数据的Sender和Receiver
//news_sender_brd.c
#include<stdio.h>
#include<stdlib.h>
#include<string.h>
#include<unistd.h>
#include<arpa/inet.h>
#include<sys/socket.h>

#define BUF_SIZE 30
void error_handling(char* message)
{
    puts(message,stderr);
    putc('\n',stderr);
    exit(1);
}
int main(int argc,char *argv[])
{
    int send_sock;
    struct sockaddr_in broad_adr;
    FILE *fp;
    char buf[BUF_SIZE];
    int so_brd=1;
    if(argc!=3)
    {
        printf("Usage :%s <Boradcast IP> <port>\n",argc);
        exit(1);
	}
    
    send_sock=socket(PF_INET,SOCK_DGRAM,0);
    memset(&broad_adr,0,sizeof(broad_adr));
    broad_adr.sin_family=AF_INET;
    broad_adr.sin_addr.s_addr=inet(argv[1]);
    broad_adr.sin_port=htons(atoi(argv[2]));
    
    setsockopt(send_sock,SOL_SOCKET,
              SO_BROADCAST,(void*)&so_brd,sizeof(so_brd));
    if((fp=fopen("news.txt","r"))==NULL)
        error_handling("fopen() error");
   
    while(!feof(fp))
    {
        fgets(buf,BUF_SIZE,fp);
        sendto(send_sock,buf,strlen(buf),
              0,(struct sockaddr*)&broad_adr,sizeof(broad_adr));
        sleep(2);
    }
    close(send_sock);
    return 0;
}

//news_receiver_brd.c
#include<"和上面代码的头文件一样">
#define BUF_SIZE 30

void error_handling(char* message)
{
    fputs(message,stderr);
    fputc'\n',stderr);
    exit(1);
}
int main(int argc,char *argv[])
{
    int recv_sock;
    struct sockaddr_in adr;
    int str_len;
    char buf[BUF_SIZE];
    if(argc != 2)
    {
        printf("Usage : %s <port>\n",argv[0]);
        exit(1);
    }
    
    recv_sock = socket(PF_INET,SOCK_DGRAM,0);
    memset(&adr,0,sizeof(adr));
    adr.sin_family=AF_INET;
    adr.sin_addr.s_addr=htonl(INADDR_ANY);
    adr.sin_port=htons(atoi(argv[1]));
    
    if(bind(recv_sock,(struct sockaddr*)&adr,sizeof(adr))==-1)
        error_handling("bind() error");
    while(1)
    {
        str_len=recvfrom(recv_sock,buf,BUF_SIZE-1,0,NULL,0);
        if(str_len<0)
            break;
        buf[str_len]=0;
        fputs(buf,stdout);
    }
    close(recv_sock);
    return 0;
}
```



# 基于Linux的编程



# 第15章 套接字和标准I/O (p246)

## 15.1 标准IO函数的优点

```cpp
标准I/O函数的两个优点:

- 标准IO函数具有良好的移植性
- 标准IO函数可以利用缓冲提高性能

```
```cpp
标准I/O函数和系统函数之间的性能对比

//syscpy.c  系统函数复制文件的示例
#include<stdio.h>
#include<fcntl.h>
#define BUF_SIZE 3//用最短数组长度组成

int main(int argc,char *argv[])
{
    int fd1,fd2;//保存在fd1和fd2中的是文件描述符
    int len;
    char buf[BUF_SIZE];
    
    fd1=open("news.txt",O_RDONLY);
    fd2=open("cpy.txt",O_WRONLY|O_CREAT|O_TRUNC);
    
    while((len=read(fd1,buf,sizeof(buf)))>0)
        write(fd2,buf,len);
    
    close(fd1);
    close(fd2);
    return 0;
}

//stdcpy.c  标准I/O函数复制文件的示例
#include<stdio.h>
#define BUF_SIZE 3//用最短数组长度组成

int main(int argc,char *argv[])
{
    FILE *fp1;//保存在fp1中的是FILE结构体指针
    FILE *fp2;//保存在fp2中的是FILE结构体指针
    char buf[BUF_SIZE];
    
    fp1 = fopen("news.txt","r");
    fp2 = fopen("cpy.txt","w");
    
    while(fgets(buf,BUF_SIZE,fp1) != NULL)
        fputs(buf,fp2);
    
    fclose(fp1);
    fclose(fp2);
	return 0;
}
```

```cpp
标准IO函数的几个缺点

- 不容易进行双向通信
- 有时可能频繁调用fflush函数
- 需要以FILE结构体指针的形式返回文件描述符

打开文件时,若希望同时进行读写操作,则应以r,w模式打开.但是因为缓冲的缘故,每次切换读写工作状态时应调用fflush函数.菏泽也会影响缓冲的性能提高.而且,为了使用标准io函数,需要FILE结构体指针.而创建套接字是默认返回文件描述符,因此需要将文件描述翻译转化为FILE指针.
    
   
    
fflush函数:
fflush 是一个 C 语言标准库函数，用于刷新（清空）输出缓冲区。它的原型定义在 <stdio.h> 头文件中：
int fflush(FILE *stream);
fflush 函数用于强制将输出缓冲区中的数据立即写入到文件或者设备中，以确保数据及时被处理或者显示出来。它的参数 stream 是一个指向 FILE 结构的指针，表示要刷新的流。
```



## 15.2 使用标准IO函数

```cpp
利用fdopen函数转换为FILE结构体

通过fdopen函数:可以将创建套接字时返回的文件描述符转换为标准IO函数中使用的FILE结构体指针。

#include<stdio.h>
FILE * fdopen(int fildes,const char *mode);
->成功时返回转换的FILE结构体指针,失败时返回NULL.
    fildes:需要转换的文件描述符
    mode:将要创建的FILE结构体指针的模式(mode)信息
    
//desto.c
#include<stdio.h>
#include<fcnt.h>

int main(void)
{
	FILE *fp;
    int fd=open("data.dat",O_WRONLY|O_CREAT|O_TRUNC);
	if(fd == -1)
	{
		fputs("file open error",stdout);
		return -1;
	}
     
    fp = fdopen(fd,"w");
    fputs("Network C programming \n",fp);
    fclose(fp);
    return 0;        
}
    
```

```cpp
利用fileno函数转换为文件描述符
#include<stdio.h>
int fileno(FILE *stream);
->成功时返回转换后的文件描述符,失败时返回-1.
    
//todes.c  fileno函数的调用示例
#include<stdio.h>
#include<fcntl.h>

int main(void)
{
	FILE *fp;
	int fd=open("data.dat",O_WRONLY|O_CREATE|O_TRUNC);
	if(fd == -1)
	{
		fputs("file open error",stdout);
		return -1;
	}
	
	printf("First file descriptor :%d \n",fd);
	fp = fdopen(fd,"w");
	fputs("TCP/IP SOCKET PROGRAMMING \n",fp);
	printf("Second file descriptor : %d \n",fileno(fp));
	fclose(fp);
	return 0;
}
```



## 15.3 基于套接字的标准I/O函数使用
```cpp
//echo_stdserv.c
#include<"与echo_server.c一致">
#define BUF_SIZE 1024
void error_handling(char *message)
{
	puts(message,stderr);
	putc('\n',stderr);
	exit(1);
}
int main(int argc,char *argv[])
{
	int serv_sock,clnt_sock;
	char message[BUF_SIZE];
	int str_len,i;
	
	struct sockaddr_in serv_adr;
	struct sockaddr_in clnt_adr;
	socklen_t clnt_adr_sz;
	FILE * readfp;
	FILE * writefp;
	if(argc != 2)
	{
		printf("Usage :%s <port>\n",argv[0]);
		exit(1);
	}
	
	serv_sock = socket(PF_INET,SOCK_STREAM,0);
	if(serv_sock == -1)
		error_handling("socket() error");
	
	memset(&serv_adr,0,sizeof(serv_adr));
	serv_adr.sin_family=AF_INET;
	serv_adr.sin_addr.s_addr=htonl(INADDR_ANY);
	serv_adr.sin_port=htons(atoi(argv[1]));
	
	if(bind(serv_sock,(struct sockaddr*)&serv_adr,sizeof(serv_adr))==-1)
		error_handling("bind() error");
		
	if(listen(serv_sock,5)==-1)
		error_handling("listen() error");
		
	clnt_adr_sz=sizeof(clnt_adr);
	
	for(i=0;i<5;i++)
	{
		clnt_sock = accept(serv_sock,(struct sockaddr*)&clnt_adr,&clnt_adr_sz);
		if(clnt_sock == -1)
			error_handling("accept() error");
		else
			printf("Connected client %d\n,i+1");
		
		readfp = fdopen(clnt_sock,"r");
		writefp = fdopen(clnt_sock,"w");
		while(!feof(readfp))
		{
			fgets(message,BUF_SIZE,readfp);
			fputs(message,writefp);
			fflush(writefp);
		}
		fclose(readfp);
		fclose(writefp);
	}
	close(serv_sock);
	return 0;
}
```

```cpp
//echo_client.c
#include<"头文件没什么好说的,还是那几个">
#define BUF_SIZE 1024
void error_handling(char *message)
{
    fputs(message,stderr);
    fputc('\n',stderr);
    exit(1);
}
int main(int argc,char *argv[])
{
	int sock;
    char message[BUF_SIZE];
    int str_len;
    struct sockaddr_in serv_adr;
    FILE * readfp;
    FILE * writefp;
    if(argc != 3)
    {
        printf("Usage: %s <IP> <port>\n", argv[0]);
        exit(1);
    }

    sock = socket(AF_INET, SOCK_STREAM, 0);
    if(sock == -1){
        error_handling("socket() error");
    }

    memset(&serv_adr,0,sizeof(serv_adr));
    serv_adr.sin_family = AF_INET;
    serv_adr.sin_addr.s_addr = inet_addr(argv[1]);
    serv_adr.sin_port = htons(atoi(argv[2]));

    if(connect(sock, (struct sockaddr *)&serv_adr, sizeof(serv_adr)) == -1)
        error_handling("connect() error");
    else
        puts("Connected ... ");
    
    readfp = fopen(sock, "r");
    writefp = fopen(sock, "w");

    while(1)
    {
        fputs("Input message(Q to quit): ",stdout);//输出
        fgets(message,BUF_SIZE, stdin);//输入

        if(!strcmp(message,"q\n") || !strcmp(message,"Q\n"))
            break;
        
        fputs(message,writefp);
        fflush(writefp);
        fgets(message,BUF_SIZE, readfp);
        printf("Message from server: %s\n",message);
    }
    fclose(writefp);
    fclose(readfp);
	return 0;
}
```
# 第16章 关于I/O流分离的其他内容
## 16.1 分离IO流

```cpp
//sep_serv.c
#include<stdio.h>
#include<stdlib.h>
#include<string.h>
#include<unistd.h>
#include<arpa/inet.h>
#include<sys/socket.h>
#define BUF_SIZE 1024

int main(int argc,char * argv[])
{
	int serv_sock,clnt_sock;
	FILE *readfp;
	FILE *writefp;
	
	struct sockaddr_in        serv_addr,clnt_adr;
	socklen_t                 clnt_adr_sz;
	char buf[BUF_SIZE]={0,};
	
	// Create a socket
	serv_sock=socket(PF_INET,SOCK_STREAM,0);
	
	// Initialize server address structure
	memset(&serv_adr,0,sizeof(serv_adr));
	serv_adr.sin_family=AF_INET;
	serv_adr.sin_addr.s_addr=htonl(INADDR_ANY);
	serv_adr.sin_port=htons(atoi(argv[1]));
	
	// Bind the socket to the specified address and port
	bind(serv_sock,(struct sockaddr*)&serv_adr,sizeof(serv_adr));
	
	// Listen for incoming connections
	listen(serv_sock,5);
	
	// Accept a client connection
	clnt_adr_sz=sizeof(clnt_adr);
	clnt_sock=accept(serv_sock,(strucr sockaddr*) &clnt_adr,&clnt_adr_sz);
	
	// Open file streams for reading and writing
	readfp=fdopen(clnt_sock,"r");
	writefp=fdopen(clnt_sock,"w");
	
	// Send messages to the client
	fputs ("FROM SERVER: Hi～client? \n", writefp);
	fputs("I love all of the world ln", writefp);
	fputs( "You are awesome! ln", writefp);
	fflush(writefp);
	fclose(writefp);//调用fclose函数终止套接字时，对方主机将收到EOF。

	// Receive a message from the client
	fgets(buf,sizeof(buf),readfp);//最后的字符串是客户端收到EOF后发送的.
	fputs(buf, stdout);
	fclose(readfp);

	return 0;
}
```

```cpp
//sep_clnt.c
#include"与上面一致"
#define BUF_SIZE 1024
int main(int argc, char *argv[]){
    int sock;
    char buf[ BUF_SIZE];
    struct sockaddr_in serv_addr;
    
    FILE*readfp;
    FILE * writefp;
    
    sock=socket(PF__INET,SOCK_STREAM,e);
    memset(&serv_addr, e, sizeof(serv_addr));
    serv_addr.sin_family=AF_INET;
    serv_addr.sin_addr.s_addr=inet_addr( argv[1]);
    serv_addr.sin_port=htons(atoi(argv[2]));
    connect(sock，(struct sockaddr*)&serv_addr，sizeof(serv_addr));
    
    readfp=fdopen(sock，“r");
    writefp=fdopen( sock,"w");
	while(1){
		//收到EOF时，fgets函数将返回NULL指针。因此，添加if语句使收到NULL时退出循环。
        if(fgets(buf, sizeof(buf), readfp)==NULL)
        	break;
        fputs(buf, stdout);
        ff1ush(stdout);
    }
    
    //通过该行语句向服务器端发送最后的字符串。当然，该字符串是在收到服务器端的EOF后发送的。
    fputs( "FROM CLIENT: Thank you! in", writefp);
    fflush(writefp);
        
    fclose(writefp); 
    fclose(readfp);
    return 0;
}

```
## 16.2 文件描述符的复制和半关闭
```cpp
下面给出文件描述符的复制方法,通过下列2个函数之一完成:
dup & dup2

#include<unistd.h>

int dup(int fildes);
int dup2(int fildes,int fildes2);
->成功时返回复制的文件描述符,失败时返回-1;

 fildes:需要复制的文件描述符
 fildes2:明确指定的文件描述符整数值
 (dup2函数明确指定复制的文件描述符整数值.向其传递大于0且小于进程能生成的最大文件描述符值时,该值将成为复制出的文件描述符值.)
 
 //dup.c
 #include <stdio.h>
 #include <unistd.h>
 
 int main(int argc,char *argv[])
 {
     int cfd1,cfd2;
     char str1[]="Hi~ \n";
     char str2[]="It's nice day~ \n";
     
     cfd1=dup(1);//复制文件描述符1
     cfd2=dup2(cfd1,7);//再次复制了文件描述符,并指定描述符整数值为7
     
     printf("fd1=%d,fd2=%d \n",cfd1,cfd2);
     write(cfd1,str1,sizeof(str1));
     write(cfd2,str2,sizeof(str2));
     
     //终止复制的文件描述符,但仍有1个描述符,因此可以进行输出.
     close(cfd1);
     close(cfd2);
     write(1,str1,sizeof(str1));
     
     //终止最后的文件描述符,所以下面的write函数无法完成
     close(1);
     write(1,str2,sizeof(str2));//无法完成
     
     return 0;
 }
  
```
```cpp
复制文件描述符后"流"的分离

//sep_serv2.c
#include<"懂的都懂">
#define BUF_SIZE 1024

int mian(int argc,char *argv[])
{
    int serv_sock,clnt_sock;
    FILE *readfp;
    FILE *writefp;
    
    struct sockaddr_in            serv_adr,clnt_adr;
    socklen_t clnt_adr_sz;
    char buf[BUF_SIZE]={0,};
    
    serv_sock = socket(PF_INET,SOCK_STREAM,0);
    memset(&serv_adr,0,sizeof(serv_adr));
    serv_adr.sin_family=AF_INET;
    serv_adr.sin_addr.s_addr=htonl(INADDR_ANY);
    serv_adr.sin_port=htons(atoi(argv[1]));
    
    bind(serv_sock,(struct sockaddr*)&serv_adr,sizeof(serv_adr));
    listen(serv_sock,5);
    clnt_adr_sz=sizeof(clnt_adr);
    clnt_sock=accept(serv_sock,(struct sockaddr*)&clnt_adr,&clnt_adr_sz);
    
    //调用fdopen函数生成FILE指针
    readfp=fdopen(clnt_sock,"r");
    writefp=fdopen(clnt_sock,"w");
    
    fputs("FROM SERVER: Hi~ client? \n",writefp);
    fputs("I love all of the world \n",writefp);
    fputs("You are awesome! \n",writefp);
    fflush(writefp);
    
    shutdown(fileno(writefp),SHUT_WR);//针对fileno函数返回的文件描述符调用shutdown函数.让服务器进入半关闭状态,服务器停止向客户端发送数据,但是仍然可以接收客户端的数据.
    fclose(writefp);//用来关闭文件,关闭文件指针writefp对应的文件.在关闭文件之前,他会将文件的缓冲区中的数据刷新到磁盘,并释放文件指针所占用的资源.此外,fclose函数还会向文件流发送EOF信号,表示文件的结束.
    
    /*
    通过调用 shutdown 函数关闭 fileno 函数返回的文件描述符（套接字描述符），服务器进入半关闭状态，并向客户端发送 EOF（End of File）信号。这样做的目的是告知客户端，服务器不会再向其发送数据。

在这种情况下，调用 shutdown 函数会关闭套接字描述符的发送通道，服务器不再发送数据，从客户端的角度看，就好像收到了一个表示文件结束的信号（EOF）。

这个解答的表述可能有些混淆，因为在网络编程中，并没有类似文件操作中的明确 EOF 信号的概念。在网络连接中，EOF 并不是一个特定的信号，而是通过连接的关闭来隐式表示。

因此，在这种情况下，调用 shutdown 函数关闭发送通道，向客户端发送一个特定的关闭信号，表示服务器不会再向其发送数据。这并不是一个明确的 EOF 信号。

请注意，在网络编程中，EOF 不表示文件的结束，而是表示连接的关闭。
    */
    
    fgets(buf,sizeof(buf),readfp);
    fputs(buf,stdout);
    fclose(readfp);
    
    return 0;
}
```



# 第17章 优于select的epoll

实现I/O复用的传统方法有select函数和poll函数。

在12章,我们介绍了select函数的使用方法，但各种原因导致这些方法无法得到令人满意的性能。(比如select:在while循环中进行IO复用操作时,需要一个临时变量来获取fd_set集合,因为<font color=red>select函数会修改传入的集合</font>.)

因此有了<font color=red>Linux 下的epoll、BSD 的 kqueue、Solaris 的/dev/poll和 Windows的IOCP</font>等复用技术。本章将讲解Linux 的epoll技术。

## 17.1 epoll理解及应用

```cpp
基于select的io复用技术速度慢的原因:
- 调用select函数后常见的针对所有文件描述符的循环语句.

- 每次调用select函数时都需要向该函数传递监视对象信息.
(对这句话的理解:每次调用select函数时都需要向操作系统传递监视对象信息.
提问:为什么需要把监视对象信息传递给操作系统?
答:select函数与文件描述符有关,更准确地说,是监视套接字变化的函数.而套接字是由操作系统管理的,所以select函数绝对需要借助于操作系统才能完成功能)
    

select函数也有优点

- 服务器端接人者少。
- 程序应具有兼容性。


实现epoll时必要的函数和结构体:
epoll_create:创建保存epoll文件描述符的空间。
epoll_ctl:向空间注册并注销文件描述符。
epoll_wait:与select的数类似，等待文件描述符发生变化。


select方式通过fd_set变量查看监视对象的状态变化(时间发送与否),而epoll方式中通过如下结构体epoll_event将发生变化(发生事件的)文件描述符单独集中到一起.

struct epoll_event
{
    __uint32_t events;
    epoll_data_t data;
}
typedef union epoll_data
{
    void *ptr;
    int fd;
    __uint32_t u32;
    __unit64_t u64;
}epoll_data_t;
声明足够大的epoll_event结构体数组后,传递给epoll_wait函数时,发送变化的文件描述符信息将被填入该数组.因此,无需像select函数那样针对所以文件描述符进行循环.
    
epoll_create函数:
#include<sys/epoll.h>
int epoll_create(int size);
->成功时返回epoll文件描述符,失败时返回-1
    (注:调用epoll_create函数时,创建的文件描述符保存空间称为"epoll例程",通过size参数的值来影响epoll例程的大小,但是!!!size并非用来决定epoll例程的大小,而是仅供操作系统参考.[size参数只是我们向操作系统提的一个建议而已]){有意思的一点是:在Linux2.6.8之后,内核将完全忽略epoll_create函数的size参数!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!}
    
   
epoll_ctl函数:
在调用epoll_create函数生成epoll例程后,应在其内部注册监视对象的文件描述符,此时应该使用epoll_ctl函数.
#include<sys/epoll.h>
int epoll_ctl(int epfd,int op,int fd,struct epoll_event *event);
->成功时返回0,失败时返回-1
    epfd:用于注册监视对象的epoll例程的文件描述符
	op:用于指定监视对象的添加,删除或更改等操作
	fd:需要注册的监视对象文件描述符
	event:监视对象的时间类型
下面举个例子:
epoll_ctl(A,EPOLL_CTL_ADD,B,C);
//第二个参数EPOLL_CTL_ADD意味着"添加",
//因此上述语句具有如下含义:
//"往epoll例程A中注册文件描述符B,主要目的是监视参数C中的事件"
下面再举个例子:
epoll_ctl(A,EPOLL_CTL_DEL,B,NULL);
上述语句中第二个参数EPOLL_CTL_DEL指'删除',因此该语句具有如下含义:"从epoll例程A中删除文件描述符B"
从上述语句得知:如果不需要监视类型(事件信息),则向第四个参数传递NULL


struct epoll_event event;
. . . . .
event.events=EPOLLIN;//发生需要读取数据的情况(事件)时
events.data.fd=sockfd;
epoll_ctl(epfd,EPOLL_CTL_ADD,sockfd,&event);
. . . . .
上述代码将sockfd注册到epoll例程epfd中,并在需要读取数据的情况下产生响应事件.
```

```cpp
epoll_wait函数:
#include<sys/epoll.h>
int epoll_wait(int epfd,struct epoll_event *events,int maxevents,int timeout);
->成功时返回发送事件的事件的文件描述符，失败时返回-1
    epfd:表示事件发生监视范围的epoll例程的文件描述符
    events:保存发送事件的文件描述符集合的结构体地址值(需要malloc动态分配)
    maxevents:第二个参数中可以保存的最大事件数
    timeout:以1/1000s为单位的等待时间,传递-1,一直等待直到发送事件
    
int event_cnt;
struct epoll_event * ep_events;
. . . . .
ep_events = malloc(sizeof(struct epoll_event)*EPOLL_SIZE);//EPOLL_SIZE是宏常量
. . . . .
event_cnt =epoll_wait(epfd,ep_events,EPOLL_SIZE,-1);
//调用函数后,返回发送事件的文件描述符数,同时在第二个参数指向的缓冲中保存发送事件的文件描述符集合.//因此,无需像select那样插入针对所有文件描述符的循环.
. . . . .
```

```cpp
基于epoll的回声服务器端
//echo_epollserv.c 服务端
#include<stdio.h>
#include<stdlib.h>
#include<string.h>
#include<unistd.h>
#include<arpa/inet.h>
#include<sys/socket.h>
#include<sys/epoll.h>

#define BUF_SIZE 100
#define EPOLL_SIZE 50
void error_handling(char *buf)
{
	fputs(message,stderr);
	fpuc('\n',stderr);
	exit(1);
}

int main(int argc,char *argv[])
{
	int serv_sock,clnt_sock;
	struct sockaddr_in serv_adr,clnt_adr;
	socklen_t adr_sz;
	int str_len,i;
	char buf[BUF_SIZE];
	
	struct epoll_event *ep_events;
	struct epoll_event event;
	int epfd,event_cnt;
	
	if(argc != 2)
	{
		printf("Usage : %s <port>\n",argv[0]);
		exit(1);
	}
	
	serv_sock = socket(PF_INET,SOCK_STREAM,0);
	memset(&serv_adr,0,sizeof(serv_adr));
	serv_adr.sin_family=AF_INET;
	serv_adr.sin_addr.s_addr=htonl(INADDR_ANY);
	serv_adr.sin_port=htons(atoi(argv[1]));
	
	if(bind(serv_sock,(struct sockaddr*)&serv_adr,sizeof(serv_adr))==-1)
		error_handling("bind() error");
	if(listen(serv_sock,5)==-1)
		error_handling("listen() error");
	
	epfd = epoll_create(EPOLL_SIZE);
	ep_events=malloc(sizeof(struct epoll_event)*EPOLL_SIZE);
	
	event.events=EPOLLIN;
	event.data.fd=serv_sock;
	epoll_ctl(epfd,EPOLL_CTL_ADD,serv_sock,&event);
	
	while(1)
	{
		event_cnt=epoll_wait(epfd,ep_events,EPOLL_SIZE,-1);
		if(event_cnt==-1)
		{
			puts("epoll_wait() error");
			break;
		}
		for(i=0;i<event_cnt;i++)
		{
			if(ep_events[i].data.fd==serv_sock)
			{
				adr_sz = sizeof(clnt_adr);
				clnt_sock=accept(serv_sock,(struct sockaddr*)&clnt_adr,&adr_sz);
				event.events=EPOLLIN;
				event.data.fd=clnt_sock;
				epoll_ctl(epfd,EPOLL_CTL_ADD,clnt_sock,&event);
				printf("connected client : %d \n",clnt_sock);
			}
			else
			{
				str_len=read(ep_events[i].data.fd,buf,BUF_SIZE);
				if(str_len==0)//close request!
				{
					epoll_ctl(epfd,EPOLL_CTL_DEL,ep_events[i].data.fd,NULL);
					close(ep_events[i].data.fd);
					printf("closed client : %d\n",ep_events[i].data.fd);
				}
				else
				{
					write(ep_events[i].data.fd,buf,str_len);//echo!
				}
			}
		}
	}
	close(serv_sock);
	close(epfd);
	return 0;
}
```


## 17.2 条件触发和边缘触发
```cpp
条件触发和边缘触发的区别在于发生事件的时间点

- 条件触发方式中,只要输入缓冲有数据就会一直通知该事件.

- 边缘触发中,输入缓冲收到数据时仅注册1次该事件.即输入缓冲中还留有数据,也不会再进行注册
```

```cpp
掌握条件触发的事件特征

epoll默认是以条件触发方式工作,因此可以通过该示例验证条件触发的特征
//echo_EPLTserv.c
#include<"懂的都懂">
#define BUF_SIZE 4/////////////////////将读缓存设置为4字节,这样,因为一次读不完,所以会多读几次,就可以验证epoll的条件触发特性
#define EPOLL_SIZE 50
void error_handling(char *buf)
{
	fputs(message,stderr);
	fputc('\n',stderr);
	exit(1);
}
int main(int argc,char *argv[])
{
	int serv_sock,clnt_sock;
	struct sockaddr_in serv_adr , clnt_adr;
	socklen_t adr_sz;
	int str_len,i;
	char buf[BUF_SIZE];
	
	struct epoll_event *ep_events;
	struct epoll_event event;
	int epfd,event_cnt;
	
	if(argc != 2)
	{
		printf("Usage : %s <port>\n",argv[0]);
		exit(1);
	}
	
	serv_sock=socket(PF_INET,SOCK_STREAM,0);
	memset(&serv_adr,0,sizeof(serv_adr));
	serv_adr.sin_family = AF_INET;
	serv_adr.sin_addr.s_addr = htonl(INADDR_ANY);
	serv_adr.sin_port=htons(atoi(argv[1]));
	
	if(bind(serv_sock,struct sockaddr*)&serv_adr,sizeof(serv_adr))==-1)
		error_handling("bind() error");
	
	epfd=epoll_create(EPOLL_SIZE);
	ep_events = malloc(sizeof(struct epoll_event)*EPOLL_SIZE);
	
	event.events=EPOLLIN;//现在是条件触发,如果要改成编译触发:event.events=EPOLLIN|EPOLLET;(再补充一下:EPOLLIN是可读,EPOLLOUT是可写)
	event.data.fd=serv_sock;
	epoll_ctl(epfd,EPOLL_CTL_ADD,serv_sock,&event);
	
	while(1)
	{
		event_cnt = epoll_wait(epfd,ep_events,EPOLL_SIZE,-1);
		if(even_cnt==-1)
		{
			puts("epoll_wait() error");
			break;
		}
		
		puts("return epoll_wait");////////////////
		for(i=0;i<event_cnt;i++)
		{
			if(ep_events[i].data.fd==serv_sock)
			{
				adr_sz=sizeof(clnt_adr);
				clnt_sock=accept(serv_sock,(struct sockaddr*)&clnt_sock,&event);
				printf("connected client:%d\n",clnt_sock);
			}
			else
			{
				str_len=read(ep_events[i].data.fd,buf,BUF_SIZE);
				if(str_len==0)//close request!
				{
					epoll_ctl(epfd,EPOLL_CTL_DEL,ep_events[i].data.fd,NULL);
					close(ep_events[i].data.fd);
					printf("close client : %d \n",ep_events[i].data.fd);
				}
				else
				{
					write(ep_events[i].data.fd,buf,str_len);//echo!
				}
			}
		}
	}
	close(serv_sock);
	close(epfd);
	return 0;
}
```

```cpp
提问:"select模型时条件触发还是边缘触发?"
```
# 第18章 多线程服务器端的实现

## 18.1 理解线程的概念

## 18.2 线程创建及运行

## 18.3 线程存在的问题和临界区

## 18.4 线程同步

## 18.5 线程的销毁和多线程并发服务器端的实现

## 18.6 习题

```cpp
(1)单CPU系统中如何同时执行多个进程?请解释该过程中发生的上下文切换。

(2)为何线程的上下文切换速度相对更?线程间数据交换为何不需要类似IPC的特别技术?

(3)请从执行流角度说明进程和线程的区别。

(4)下列关于临界区的说法错误的是?
a.临界区是多个线程同时访问时发生问题的区域。
b.线程安全的函数中不存在临界区，即便多个线程同时调用也不会发生问题。
c.1个临界区只能由1个代码块，而非多个代码块构成。换言之，线程A执行的代码块A和线程B执行的代码块B之间绝对不会构成临界区。
d.临界区由访问全局变量的代码构成。其他变量中不会发生问题。
(5)下列关于线程同步的描述错误的是?
a.线程同步就是限制访问临界区。
b.线程同步也具有控制线程执行顺序的含义。
c.互斥量和信号量是典型的同步技术。
d.线程同步是代替进程IPC的技术。
(6)请说明完全销毁Linux线程的2种方法。
(7)请利用多线程技术实现回声服务器端，但要让所有线程共享保存客户端消息的内存空间
(char数组)。这么做只是为了应用本章的同步技术，其实不符合常理。
(8)上一题要求所有线程共享保存回声消息的内存空间，如果采用这种方式，无论是否同步
都会产生问题。请说明每种情况各产生哪些问题。
```

# 第21章 异步通知IO模型(P344)

## 21.1 理解异步通知IO模型



## 21.2 理解和实现异步通知IO模型

# 第22章 重叠IO模型

## 22.1 理解重叠io模型

## 22.2 重叠io的io完成确认



# 第23章 IOCP

## 23.1 通过重叠IO 理解IOCP

## 23.2 分阶段实现IOCP程序

# 第24章 制作HTTP服务器端

## 24.1 HTTP概要

## 24.2 实现简单的Web服务器端

# 第25章 进阶内容

暂无