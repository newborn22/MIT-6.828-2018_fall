## MIT 6.828 第三课

#### GDB

##### 1.GDB启动

可以对可执行程序（需在gcc编译时增加 -g选项 生成调试信息， gdb进入后list查看代码并b添加断点 ，r执行程序。 可看help）、core文件（核心文件，记录段错误。ulimit -c可以查看是否允许生成核心文件）和正在运行的服务程序进行调试。

##### 2.GDB调试命令

https://pdos.csail.mit.edu/6.828/2018/lec/gdb_slides.pdf，看每一个分类命令下写满的ppt就行。

断点也可以增加条件，可以方便进行循环/递归上的调试。

watch命令可以让程序在某个内存内容或表达式的值改变时停止，watch需要在程序有运行上下文的情况下才能设置，且wa的变量如果脱离了其作用域wa就自动删除。

set命令可以在运行期间给变量赋值。

#### HW

##### 1.6.828shell代码基本结构：

获取指令，判断是否改变工作目录。如果没有，fork子程序解析执行命令(runcmd，本质上是调用了exec这个系统调用去执行命令，即相应的功能函数，当功能函数结束后，exit退出系统调用会返回shell,shell从wait中退出)，父进程等待。命令分为三种基本类型，可执行命令（" "），管道命令("|")，重定向（">"和"<"）

```
int
main(void)
{
  static char buf[100];
  int fd, r;

  // Read and run input commands.
  while(getcmd(buf, sizeof(buf)) >= 0){
    if(buf[0] == 'c' && buf[1] == 'd' && buf[2] == ' '){
      // Clumsy but will have to do for now.
      // Chdir has no effect on the parent if run in the child.
      buf[strlen(buf)-1] = 0;  // chop \n
      if(chdir(buf+3) < 0)
        fprintf(stderr, "cannot cd %s\n", buf+3);
      continue;
    }
    if(fork1() == 0)
      runcmd(parsecmd(buf));
    wait(&r);
  }
  exit(0);
}
```

##### 2.植入代码：

###### 1.执行类命令无法执行

因为runcmd中尚未植入execv库函数。

> exec有一系列函数，包括execl,execlp,execle,execv,execvp等

> man的分区
>
> 1. 一般命令
>
> 2. 系统调用
>
> 3. 库函数，涵盖了c标准函数库
>
> 4. 特殊文件（通常是/dev中的设备）和驱动程序
>
> 5. 文件格式和约定
>
> 6. 游戏和屏保
>
> 7. 杂项
>
> 8. 系统管理命令和守护进程

> int execv(const char *progname, char *const argv[]))
>
> argv第一个元素是命令本身，最后一个元素是NULL(parser解析出来的命令也是这样的， 于是在调用execv时就可以简单地使用execv(ecmd->argv[0], ecmd->argv)，例如下：
>
>   char *argv[] = {"ls","-l",NULL};
>   ret = execvp("ls",argv);
>
> execv调用执行的命令如果正常执行是不会返回的，会在调用内部以exit的方式返回给shell的wait，如果返回-1则说明调用出错。以下实验证明，argv第一个参数是什么不重要，第一个参数应该只是留给编程者的接口：
>
> ![image-20210926124842067](C:\Users\95395\AppData\Roaming\Typora\typora-user-images\image-20210926124842067.png)

###### 2.重定向命令，

同样需要补充缺失代码，用open和close。

重定向的核心原理是文件描述符。通过关闭进程内相应的0或1描述符，并open一个文件，由于文件描述符的序号是顺序增加并且补缺的，而printf、scanf等库函数都是操作1、0，1、0现在指向了新open的文件，以此来实现输入ian输出重定向。

文件相关的表：

![image-20210926154540761](C:\Users\95395\AppData\Roaming\Typora\typora-user-images\image-20210926154540761.png)

> <fcntl.h>
>
> open 函数原型：int open(const char *pathname, int flags, mode_t mode（可以不填）);
>
> flags must include one of the following access modes: O_RDONLY, O_WRONLY, or O_RDWR.
> mode: 创建文件时设定的其他用户权限: S_IRUSR | S_IWUSR | S_IRGRP | S_IROTH;
> 返回值：调用成功时返回一个文件描述符fd；调用失败时返回-1，并修改errno
>
> S_IRUSR：用户读权限
>
> S_IWUSR：用户写权限
>
> S_IRGRP：用户组读权限
>
> S_IWGRP：用户组写权限
>
> S_IROTH：其他组都权限
>
> S_IWOTH：其他组写权限

![image-20210926161657376](C:\Users\95395\AppData\Roaming\Typora\typora-user-images\image-20210926161657376.png)

之所以关闭掉1或0不会影响到接下来的命令，是因为每个指令都是fork后交给一个新子进程去处理的。文件描述符表是进程级别的。

###### 3.植入pipes

需用pipe、fork、close、dup补充代码。

> dup(int oldfd)，在进程文件打开表中再增加一个文件描述符，指向oldfd的文件。

> \#include<unistd.h>
>
> 管道，参考资料：https://blog.csdn.net/qq_35433716/article/details/86171936
>
> 在使用层面，管道的本质就是两个文件描述符，是一个有两个元素的数组，f[0]用来读（配合read），f[1]用来写(配合write)。pipe(两个大小的数组进行初始化); write(f[0],缓冲区数据,大小)；read(f[1],缓冲区数据，大小);
>
> read()在管道里没有数据时进程会阻塞，可以用fcntl来设置文件的非阻塞属性。
>
> write在管道满时进程会阻塞。
>
> 管道的缓冲区有大小限制。为64kB，可以实验得到。（char buf[1024];然后进程循环往管道中写buf，并计数。会发现在64的时候停下）
>
> 管道用完记得关闭。如果要用管道替换原来的stdin或者stdout，close、dup后就可以应该close掉两个管道。如果连续pipe两次相同的数组，那么原来的那个数组的fd值就丢失了。（可以用新的-2）。

> 僵尸进程：子进程先结束，父进程没有wait其销毁。 若父进程先退出，子进程会被init(pid 1)进程回收。

> \#include <unistd.h>
>
> fork()后子程序从哪里运行？从fork完的语句开始运行。
>
> ![image-20210926195604358](C:\Users\95395\AppData\Roaming\Typora\typora-user-images\image-20210926195604358.png)之所以会这样是由于printf的缓冲机制，它并不是立即打印屏幕上，而是会先缓存着，如果遇到\n就会立即送到stdout中。由于缓存，fork后的子进程也继承了缓存的内容。所以如果上面的"fork！"增加\n就只会有一个fork了。
>
> 下面的代码中，这样的方法来创建两个子进程实际上会一共有四个进程。
>
> ![image-20210926200336842](C:\Users\95395\AppData\Roaming\Typora\typora-user-images\image-20210926200336842.png)
>
> ![image-20210926200459270](C:\Users\95395\AppData\Roaming\Typora\typora-user-images\image-20210926200459270.png)



错误代码： 出现一直卡住不动的情况。经过各种实验后发现是管道右边的命令没有得到输入。经过排查发现是由于管道没有及时关闭的原因，没有关闭可能导致缓冲区的内容没有及时写入导致没法读。把管道符close掉后问题解决。
