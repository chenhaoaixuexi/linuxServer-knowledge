# fork_wait [linux]

### fork

##### 注意

- 程序调用 fork内核程序, fork在用户空间中的**操作**
  1. 分配新的内存块和内核数据结构
  2. 复制父进程到新的进程
  3. 向运行进程集添加新的进程
  4. 将控制权由内核转向父子进程
- fork 如何区分父子进程
  - 父进程的fork返回 子进程的 pid
  - 子进程的fork返回 0
- fork 出的子进程 从fork语句开始的地方开始执行, 并不从main开始执行

##### 代码展示

```C
#include <stdlib.h>
#include <stdio.h>
#include <sys/types.h>
#include <unistd.h>
int main(){
	printf("the parent id is :%d\n",getpid());
	printf("calling fork\n");
	sleep(1);
	//call fork
	int fork_id = fork();// the child process begin 
	printf("my id is %d , the fork return value is %d\n",getpid(),fork_id);
	return 0;
}

```



##### 结果展示

- 我的系统默认是父进程先执行(?)

![1538116195113](C:\Users\26306\AppData\Roaming\Typora\typora-user-images\1538116195113.png)

### wait

##### 注意

- 函数原型`   pid_t wait(int *wstatus);`

- wait 返回的是子进程的 pid

- wstatus 此项若不为NULL , 则将子进程exit 的状态存入其中, 存储的值是16位的数字

  - 高八位: exit value 
    - 就是我们经常 return 0 或者 exit (0) 那个'0'
    - 当然可以不为0 , 我的代码返回了255
  - 中间一位: core dump flag 
    - 指明发生错误并产生内核映像(core dump)
  - 低7位: 记录信号序号

- 工作原理图

  ![1538127412858](assets/1538127412858.png)

##### 代码展示

```C
#include <stdlib.h>
#include <signal.h>
#include <stdio.h>
#include <sys/types.h>
#include <unistd.h>
#include <sys/wait.h>
#include <string.h>

void printBinary(unsigned num);
// binary output
void printBinary(unsigned num){
	int remainder[20] = {0};
	int index= 0;
    while(0 != num){
		remainder[index]=num % 2 ;
		num = num / 2;
		index ++;
	}
	int count = 0;
	for(int i=--index;i>=0;i--){
		if(0 == count%4){
			printf(",");
		}
		count ++;	
		printf("%d",remainder[i]);
	}
	printf("\n");
}
int main(){
		
	signal(SIGTERM,SIG_IGN);
	printf("the parent process id : %d\n",getpid());
	int fork_id = fork();// mark is not prarent
	if(-1 == fork_id){
		perror("fork fail\n");// when the memorry is full
	}else{
		int state;
		if(0 != fork_id){//parent process is runing
			int wait_id =wait(&state);
			printf("i am parent process , my id is %d,the wait function return value is %d\n ",getpid(),wait_id);
			printf("the exit num :");
			printBinary(state);
		}

		if(0 == fork_id){// child process is runing
			printf("i am child process , my id is %d \n",getpid());
			printf("child process about to sleep 10 second\n");
			sleep(10);
			exit(17);
		}
	}

}

```



##### 结果展示

![1538126285974](assets/1538126285974.png)