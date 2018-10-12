# 编写 web服务器 和 线程的使用

### 线程的基本使用(入门)

##### 线程的定义

线程于函数相当于 进程于程序, 都给前者提供了运行环境

- 线程三大性质

  1. 互斥
  2. 有限等待
  3. 前进

  ​	

- 几个编译时常量

![1539002054334](assets/1539002054334.png)

##### 线程的创建和等待
```C
int pthread_create(pthread_t *thread, const pthread_attr_t *attr,                 void *(*start_routine) (void *), void *arg);

int pthread_join(pthread_t thread, void **retval);
```

##### 线程的分工合作

- 互斥变量 (互斥)

  ​	

```C
// 加锁
int pthread_mutex_lock(pthread_mutex_t *mutex);
//解锁
int pthread_mutex_unlock(pthread_mutex_t *mutex);
```

- 条件变量(有限等待 + 前进性)

```C
// 挂起本线程,自动解锁, 等待信号  (调用此函数前,要先上锁), 激活后自动加锁
int pthread_cond_wait(pthread_cond_t *cond, pthread_mutex_t *mutex);
// 发信号 (前进性)
int pthread_cond_signal(pthread_cond_t *cond);
```

##### 代码

- 头文件

```C
#include <netinet/in.h>
#include <netinet/ip.h>
#include <sys/types.h>
#include <sys/socket.h>
#include <stdio.h>
#include <stdlib.h>
#include <unistd.h>
#include <string.h>
#include <fcntl.h>
#include <pthread.h>
```



```C
// public variable
int count = 100;

// mutex lock
pthread_mutex_t mutex = PTHREAD_MUTEX_INITIALIZER;

// operator the public variable
void * show(void * a){
	while(1){
		pthread_mutex_lock(&mutex);
		if (count > 0 ) {

			printf("%s",(char *)a);
			printf("%d : %d\n",(int)pthread_self(),count --);
		}
		pthread_mutex_unlock(&mutex);
	}
}


int main(int argc, char *argv[])
{
	pthread_t t1;
	pthread_t t2;

	pthread_create(&t1,NULL,show,(void *)"hello");
	pthread_create(&t2,NULL,show,(void *)"world");
	pthread_join(t1,NULL);
	pthread_join(t2,NULL);
	return 0;
}
```



​	

##### 线程进程的相关性

![1538915791321](assets/1538915791321.png)

### 服务器设计

##### 创建socket , bind绑定 监听listening

- 函数原型们

```C
int socket(int domain, int type, int protocol);
int bind(int socket, const struct sockaddr *address,socklen_t address_len);
int listen(int socket, int backlog);
int accept(int sockfd, struct sockaddr *addr, socklen_t *addrlen);
pid_t waitpid(pid_t pid, int *stat_loc, int options);
int close(int fildes);
int fscanf(FILE *stream, const char *format, ...);
```

- 头文件们

```C
// base fork
#include <netinet/in.h>
#include <netinet/ip.h>
#include <sys/types.h>   
#include <sys/socket.h>
#include <signal.h>
#include <stdio.h>
#include <stdlib.h>
#include <unistd.h>
#include <sys/wait.h>
#include <string.h>
#include <fcntl.h>
// base thread

```

- 服务器 创建socket 并绑定监听伪代码

```C
int make_server_connect (int port){
	int state_socket ;
	int state_bind;
	int state_listen;

	struct sockaddr_in addr ;
	int  fd;

	if (-1 != (state_socket = socket(AF_INET,  SOCK_STREAM,0))){
		addr.sin_family=AF_INET;
		addr.sin_port=htons(port);
		addr.sin_addr.s_addr=INADDR_ANY;

		if(bind(state_socket, (const struct sockaddr *)&addr, sizeof(addr))==-1){
			perror("cannot bind");
			exit(1);
		}else {
			state_listen = listen(state_socket, 1);
			if (-1 == state_listen) {
				perror("cannot listen ")	;
				exit(3);
			}
			return state_socket;
		}
	}else {
		perror("cannot get socket")	;
		exit(2);
	}
}
```

##### 服务器端接受和处理客户端请求(结合 fork )

- 服务器端接受 伪代码

```C
int main(int argc, char *argv[])
{
	char  request[1024];
	int interrupt = 1;	
	int fd;
	signal (SIGCHLD,MyWait);
	int socket = make_server_connect(9999);
	if (-1 == socket) {
		perror("make server connect fail")	;
		exit(4);
	}
	while (1){
		fd = accept(socket,NULL,NULL);	
		read(fd,request,sizeof(request));
		if (interrupt == 1 || -1 != fd  ) {
			interrupt = process(fd,request);			
			close(fd);
		}else{
			break;
		}
	}
	return 0;
}
```

- SIGCHLD信号处理 

```C
void MyWait(int signum){
	while(waitpid(-1,NULL,WNOHANG)>0);//  用足够的wait 把所有的zombie进程去除(考虑多个信号同时到达的情况)
}
```

- 处理客户端请求

```C
int process (int fd,char * request){
	int fork_id = fork();
	char  cmd[1024];
	char  file_name[1024];
	char temp[1024];
	int ffd;
	int n;
	char buf[1024];
	switch (fork_id){
		case 0:
			sscanf(request,"%s%s%s",cmd,file_name,temp);

			if(strcmp(cmd,"GET")== 0){
				if((ffd=open(file_name+1, O_RDONLY))==-1){
					//handle_404(fd);
					printf("404");
					write(fd, "HTTP/1.1 404 Not Found\r\nContent-Type: text/html; charset=iso-8859-1\r\n\r\n", strlen("HTTP/1.1 404 Not Found\r\nContent-Type: text/html; charset=iso-8859-1\r\n\r\n"));
					return 0;
				}
				printf("200");
				write(fd, "HTTP/1.0 200 OK\r\n\r\n", strlen("HTTP/1.0 200 OK\r\n\r\n"));
				while((n=read(ffd, buf, sizeof(buf)))>0){
						write(fd, buf, n);
					}
				close(ffd);
			}else{
				printf("400");
				write(fd, "HTTP/1.1 400 Operator Not Found\r\nContent-Type: text/html; charset=iso-8859-1\r\n\r\n", strlen("HTTP/1.1 400 Operaor Not Found\r\nContent-Type: text/html; charset=iso-8859-1\r\n\r\n"));
			}
			break;
		default:
			break;
	}
	return 1;
}
```

### web服务器协议

##### http 1.1 和 http 1.0的区别

https://blog.csdn.net/ForgotAboutGirl/article/details/6936982

### 参考链接

API概览介绍: https://zh.wikipedia.org/wiki/POSIX%E7%BA%BF%E7%A8%8B

参考博客: https://blog.csdn.net/ithomer/article/details/5921003cd

