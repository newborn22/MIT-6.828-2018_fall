## 流程记录

### 1.LAB1

#### 安装必要的工具

git下载实验源码+gcc+QEMU

git：yum -y install git

gcc：centos自带

qemu：官网代码需要自己下载依赖库，centos需用yum源，参考https://www.cnblogs.com/chubuyu/p/14789834.html。其中报出的autoreconf、flex和bison错误用yum源下载即可解决。安装完成后用qemu-system-i386 测试是否安装成功。

#### lab1 p2 load the kenel ex4

![image-20210916202344562](C:\Users\95395\AppData\Roaming\Typora\typora-user-images\image-20210916202344562.png)



#### lab1 p3 formatted printting to the console 问题3调试

函数调用的汇编语言模板：先放参数，后call压栈eip，进入函数后 push ebp把上个函数的ebp保存入栈中, 把esp的值放入ebp（故此时有了一个新的栈，该新栈负责存储新函数的局部变量且可用于索引该函数的参数，**当前ebp指向的位置刚好就是压栈的上个函数的ebp值的位置（Pushing a value onto the stack involves decreasing the stack pointer and then writing the value to the place the stack pointer points to，也就是栈顶指针并不是指向空，而是指向栈顶元素，故空栈时栈顶指针指向-1（如果用数组来模拟的话））**，然后是返回地址，再是从左到右第一个参数（最后压入的）），函数返回时，要恢复上个函数的ebp和esp，把现在的ebp的值放回esp（或者esp一开始减了多少就加回多少，因一开始esp就等于ebp），并把原本存在栈中的ebp值弹出来放回ebp中。

![](C:\Users\95395\AppData\Roaming\Typora\typora-user-images\image-20210917223503894.png![image-20210917230659719](C:\Users\95395\AppData\Roaming\Typora\typora-user-images\image-20210917230659719.png)

![](C:\Users\95395\AppData\Roaming\Typora\typora-user-images\image-20210917231504497.png

![image-20210917231556597](C:\Users\95395\AppData\Roaming\Typora\typora-user-images\image-20210917231556597.png)

这里的图栈向下生长

![image-20210917233243869](C:\Users\95395\AppData\Roaming\Typora\typora-user-images\image-20210917233243869.png![image-20210920201051451](C:\Users\95395\AppData\Roaming\Typora\typora-user-images\image-20210920201051451.png)

#### lab1 练习11知识学习链条

	##### volatile

参考资料：https://blog.csdn.net/qq_29350001/article/details/54024070

what/why：volatile是C语言及C语言内联汇编语言的一个关键子，可以防止编译器对该关键词所指示的指令或者变量做一些性能优化上的修改，因为这些修改可能会造成业务逻辑上的偏差。用volatile指示的变量会发生一些意外的改变，这些改变是从代码逻辑上看不出来的。volatile和const都属于type qulifier（限定符，qualified有限定的意思），volatile显然和const是截然相反的。

How: volatie原理是 使用到变量的值时，将直接从内存单元中读取该值，而不会因为编译器的优化而将该值装入到寄存器中并直接从寄存器读取（对于每个程序（更底层就是对应的进程）来说，都有自己的上下文，寄存器是上下文的一部分，操作系统的进程切换机制保证了每个进程的寄存器由该进程内部的代码进行修改）。故只要内存单元被其他进程、线程、硬件、中断处理程序改变，该变量的值便会随之改变。

> 用法之一： #define 宏名 *((volatile unsigned int *)一个内存的地址)
>
> 例如：#define GPC1CON *((volatile unsigned int *)0xE0200080)
>
> 那么这个GPC1CON就相当于一个变量（变量的本质就相当于 * 内存地址，即访问内存地址，因为这样可以对内存地址内存进行修改 变量不是单纯的一个内存地址。）。
>
> 上面这个就是一个硬件编程的例子。硬件编程：外设的寄存器在内存地址空间有一个映射，面向硬件的变成本质上就是利用C语言对内存单元进行读写、操作。



> 编译器如何对代码进行优化
>
> gcc -02 .c文件名 -o 输出文件名 （其中的-02表示要求gcc编译器对其进行优化）
>
> 例子一：
>
> 1.int main{
>
> ​	int i=10; int a=i; int b=i;
>
> }
>
> 2.int main{
>
> ​	volatile int i=10; int a=i; int b=i;
>
> }
>
> 例子1中的汇编代码，gcc编译器会进行优化，它检测到b=i时i的值并没有发生改变，所以会直接让i的值赋给b，即将10直接赋给b。而2的代码中，由于施加了volatile关键子，所以每次涉及到i，都会从内存单元中将i的值读出，然后再赋给b。
>
> 所以当变量是寄存器变量（硬件中寄存器在内存中的映射，硬件和程序都可以对其进行操作）或者端口数据时volatile可以保证对这些值进行稳定的访问。
>
> 例子二：
>
> for(int i=0;i<100000;i++); 由于是空循环，会被编译器优化掉，不执行。
>
> for(volatile int i=0; i<100000;i++) 则编译器不会优化。



##### 内联汇编

参考资料1：https://blog.csdn.net/ml_1995/article/details/51044260

参考资料2：https://www.codeproject.com/Articles/15971/Using-Inline-Assembly-in-C-C

语法模板：

asm [volilate] (

​	"汇编语句1;"

​	"汇编语句2"

​	: /*output */

​	:/*input */

​	:/*clobbred register */

)

what：内联汇编是往C语言中嵌入汇编语言。

why：汇编语言可以对内存单元（当然C语言运用指针也可以）、寄存器进行直接操作，使用一些汇编命令等。在效率上相比于纯C肯定更高，可以用来做性能优化。在功能上肯定也有一些方便之处。

how：C中嵌入汇编，关注的内容肯定是C中的变量（存于内存函数栈帧的局部变量区）能与汇编进行互动。这就需要*magic number*（主要面向C的变量）和设置约束（面向汇编）。常用的约束有寄存器约束，内存约束和关联约束，详见参考资料1。在汇编中用%从0开始的序数 来占位，对应下面三个 : : : 中对应的C变量。 C变量前用约束。*"[=]<约束>"(<变量>)*  =针对output操作数，表示只写。

> lock 实现原子操作
>
> 汇编指令lock是指令前缀，如果使用了lock前缀的命令，则处理器会触发LOCK信号，并把总线锁起来，使得共享内存变成排它使用。（如果是单cpu内部的进程切换？那可能不一定能保证同步，单CPU内多进程的同步要用信号量、锁等机制保证同步）这样就使得对内存中数据的操作具有原子性。

> C语言的## : 用在宏定义中，将所写的部分和参数进行拼接

> inline
>
> inline函数以空间换时间。声明inline的函数，编译器会在函数调用处会将函数内部代码展开并替换调用（并不是所有情况下都这样），这样就无需进栈帧去执行函数。inline适用于较短函数。

> static
>
> 当static作用于函数前面或者全局变量前面时，意味着这个函数或者变量的作用范围只在该文件中，而不是全局作用 （在引入多个文件的项目时，就不会发生冲突）。如果static修饰头文件中的变量且头文件被多个地方引用时，那么就会造成变量多处定义。
>
> static变量、函数放在头文件中，可以被导入它的.c文件使用的原因是因为#inlcude就是单纯的展开。
>
> 若要全局定义变量（即该变量可以被任意文件修改，那么就用extern来修饰，且不能赋初值）。
>
> 参考资料 https://blog.csdn.net/weibo1230123/article/details/83000786



##### 代码错误汇总：

1.寄存器内存间址是()而非[]

2.在jos里的汇编operand用r修饰不能超过5个。

3.要打印固定字宽必须是"%08x"而非"%8x"(不会补0)

4.jos中把warn也当成错误，变量需要初始化。

5.低级错误之“=r”而非"r="

我的

![image-20210920220327323](C:\Users\95395\AppData\Roaming\Typora\typora-user-images\image-20210920220327323.png)

别人正解代码的：

![image-20210920220534878](C:\Users\95395\AppData\Roaming\Typora\typora-user-images\image-20210920220534878.png)

可以发现错误还在于 ebp前面没有空格以及前面没有加Stack backtrace:

最终代码

```c
int
mon_backtrace(int argc, char **argv, struct Trapframe *tf)
{
	// Your code here.
	int ebp=0;
	asm volatile(
		"movl %%ebp,%0"
		:"=r"(ebp)		
	);
	cprintf("Stack backtrace:\n");
	while(1){
		cprintf("  ebp %08x",ebp);
		
		int eip=*((int*)ebp+1);		
		cprintf("  eip %08x",eip);
		
		cprintf("  args");
		for(int i=2;i<=6;i++){
			cprintf(" %08x",*((int*)ebp+i));
		}
		cprintf("\n");
		
		ebp=*((int*)ebp);
		if(ebp==0){
			break;
		}
	}
	return 0;
}
```

#### lab1 练习12知识学习链条

> C语言根据变量的存储类型，分为auto、static、register(将变量放在CPU内寄存器中，速度更快)、extern

> extern 存于全局静态存储区（内存分配给进程的一部分，程序分给进程的部分有堆、栈（记录函数调用和返回及存放局部变量）、text区、存放初始化的全局变量区及该全局静态存储区(bss，会被初始化成0)）
>
> what:说明 “**此变量/函数是在别处定义的，要在此处引用**”。例如：extern int a;是声明a这个变量而非定义,extern int a=100或int a;则是定义一个整数变量a。声明无需占用空间，定义需要。extern int a就和先声明函数然后函数的实现放在后面一样。
>
> why:在多文件编程中，有的变量、函数没有定义就需要使用。这时候就需要先声明。
>
> how:
>
> ![image-20210921154942474](C:\Users\95395\AppData\Roaming\Typora\typora-user-images\image-20210921154942474.png)

##### 符号表 stab(symbol table)

参考资料：https://www.jianshu.com/p/bda60193808d

what why:符号表在代码链接时发挥作用，将符号（包括变量和函数）映射成虚拟地址，使得汇编指令可以运行（因为汇编只认识内存地址、寄存器，而不认识函数名和变量名等编程“助记符”）。更具体一点，符号表中的内容还包括定义域和类型。

注意：局部变量分配在栈中，不会在函数外部被引用，因此不是符号定义。

how it work：符号定义的本质是指这个符号被分配了存储空间。如果是函数名则指代码所在区；如果是变量名则指其所在的静态数据区。所有定义的符号的值就是其目标所在的首地址。因此，符号的解析就是将符号引用和符号定义建立关联后，将引用符号的地址重定位为相关联的符号定义的地址。**所以，之所以可以通过eip获取函数名，是因为在符号表中记录着函数名（一个符号）的空间范围，只要eip在某个表项记录的符号定义的范围内，就可以知道当前eip是在哪个函数内了。**

specific rules:符号分为强符号（函数和已经初始化的全局变量）和弱符号（未初始化的全局变量）。

how to ues: 符号表是一个结构体数组。每个表项是一个结构体。

##### 根据实验提供的线索查看 _stab _

1.查看kern/kernel.ld（链接脚本，用来控制链接器）以求寻找到stab。可以看到里面有stab和stabstr的begin和end。经过查找资料得知这两个分别是符号表和符号表相关字符串的开始和结束的地方。

###### 为了进一步理解，需要学习ld文件的语法。

参考资料：https://blog.csdn.net/daydayup654/article/details/78630341

链接脚本用于控制如何将输入文件（obj（编译源文件得到）和链接脚本）中各section合成并放入到输出文件中（可执行或者obj）。链接器本身带有链接脚本，可以改用自己写的脚本。GNU/linux下的目标文件为EFI格式（文件格式意味着怎么组织里面的数据和元数据，比如设置什么元数据，元数据和数据放的位置等），其中section有名称、大小等信息，还有类型。常为可装载（运行时将数据装入内存）和可分配类型（运行时分配空间并常初始化为0），如果两者都不是，则一般是含有调试信息的。每个可装入或可分配的section一般都有LMA和VMA，一般二者相同，嵌入式设备中由于代码烧录在ROM中会导致不一样。二者的映射可以通过符号表来完成。LMA是段被装入的实际地址（所谓的装入程序就是将别的程序mov到别的地方的程序），而VMA是实际运行过程中，当需要用到这个段时，段在命令中的起始地址。

脚本格式：

1.PROVIDE(符号=.)：如果该符号在目标文件内被引用，但没有在任何目标文件内被定义，那么该符号的地址就指向所在section的相应位置。

2.AT:AT(...) gives the load address of this section, which tells the boot loader where to load the kernel in physical memory.

3BYTE:BYTE(0) /* Force the linker to allocate space for this section */

![image-20210921172545546](C:\Users\95395\AppData\Roaming\Typora\typora-user-images\image-20210921172545546.png)

上面ld脚本中的意思为，将所有输入文件的stab段合成为一个，放在前面seciton的后面。如果_STAB_BEGIN没有定义，引用它时，就指向该段的最前面。剩下的类似。

##### 使用一系列objdum命令查看kernel文件

-h 获取section头部摘要信息。-G 获取调试信息 更多选项https://blog.csdn.net/q2519008/article/details/82349869

###### 第一个 **objdump -h obj/kern/kernel**

![image-20210921182426595](C:\Users\95395\AppData\Roaming\Typora\typora-user-images\image-20210921182426595.png)

###### 第二个**objdump -G obj/kern/kernel**

参考资料1：https://blog.csdn.net/hfyinsdu/article/details/104359578

参考资料2：https://blog.csdn.net/roger__wong/article/details/8623941（针对本实验）

GNU/linux下，.c文件编译生成.S, .S汇编成.o（这个过程中可以生成调试信息） , .o链接成可执行文件。

所谓的调试，就是可以让程序暂时停止在某处（这可用中断手段实现），并打印出中断时程序的一些相关信息，这些相关信息便是所谓的调试信息，当前内存某处的值（这可以通过汇编语言来实现），显示程序变量的类型和值（这就需要用到stab表，因为里面记录着相关信息）。

所以这里用-G查看调试信息本质上查看的就是stab表内部的信息。

![image-20210921192955250](C:\Users\95395\AppData\Roaming\Typora\typora-user-images\image-20210921192955250.png)

###### 第三个 init.c生成汇编文件  查看bootloader是否把stab装入内存？

看汇编文件并不是很懂。有非常多的伪命令。.stabs 用来产生符号，后面要跟五个字段，分别对应五个值，具体间https://blog.csdn.net/nancygreen/article/details/14445829。

由于objdump -h 中查看到的两个section有lvm且具有大小，显然会装入内存（废话，不然这题怎么做）。

##### 完善代码kdebug.c代码

阅读代码注释可知stab_binsearch为二分查找代码（由于stab的组织），传入stabs表（代码帮我们初始化了，其实就是在stab_begin那）、left、right、addr就可以得到表项相应的left和right，之后就可以用stabs[left]访问表项中相应的字段（stab表项是结构体，属性在stab.h中可看到）给info的相应字段（在kdebug.h中）赋值。

##### 在montrace函数中调用以打印函数名等信息

创建kdebug头文件提供的结构体并memset初始化，传入对应参数入函数即可。

返回的Eipdebuginfo结构体的eip_fn_name字段除了函数名外还有一段尾巴，比如`test_backtrace:F(0,25)`，需要将":F(0,25)"去掉，可以使用`printf("%.*s", length, string)`来实现。其中"%.*s"的 *为一个整数，表示所要输出的字符串的宽度。

最后一个字段是当前eip举例函数起始地址的差。

由于编译器做了优化，所以有些函数使用了内联，那么它们就无需进入栈帧调用，故打印出来会没有这些函数的信息。可以在编译时去掉-02(这样编译器就不会优化)

> 编译器负责生成代码。链接器负责符号、地址（不仅包括符号和地址的映射还包括vma lma section，以便于装入）、调试(如stab)。

###### 所犯小错：

C的结构体前面需要加struct.

##### 在JOS的shell中插入该函数

kern/monitor.c中的结构体中数组中加入相应条目

#### 小插曲

看完monitor代码后发现可以自己注册一个shell接口函数（参数是固定的，返回值也需是int），返回<0可以让monitor停止。

报错![image-20210922001414883](C:\Users\95395\AppData\Roaming\Typora\typora-user-images\image-20210922001414883.png)

原因是忘记在monitor头文件中声明。

成功之后，发现退出了monitor后即break后。。。又重新回来，可能是qemu重新启动？

