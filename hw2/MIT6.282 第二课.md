## MIT6.282 第二课

#### 指令集

RISC（Reduced Instruction Set Computing）：精简指令集，指令较短，效率高。

CISC（Complex Instruction Set Computing）：复杂指令集，功能强大，效率较低。

#### 架构

x86：南北桥，故有较强的扩展性；追求高性能，能耗高。

arm：用专用接口，扩展性较差。使用RISC，能耗低，用于嵌入式设备、移动通信领域。

#### CPU发展

8086 16位寄存器，20位地址总线，20位的地址总线使得指针的位数非常尴尬。

80386 32位寄存器和地址总线。开机时为16位（实模式），后由BIOS转换为32位保护模式，这是为了向前兼容。有虚存机制。

向64位地址总线发展。

> 指针的字节数：指针指向内存单位，本质上是一个内存单元地址，故内存地址总线为几位，指针就应该为几位。

#### IO

8086只有1024 IO地址。以及slide中提到的打印机代码。可以知道，硬件编程本质上就是往控制器（更深层目的是寄存器）中写读数据。

![image-20210922173300035](C:\Users\95395\AppData\Roaming\Typora\typora-user-images\image-20210922173300035.png)



内存内存IO地址的特点：像寻址内存一般，但并不像内存一样。不用别的指令。读的结果会因为事件而改变。具体的硬件由控制器去路由。

#### x86指令集

数据相关（如mov、出入栈）、算数相关、I/O相关（in out）、流程控制相关（jmp）、字符串操作相关（movsb）、系统相关（INT）

#### gcc调用方式

![image-20210922175159623](C:\Users\95395\AppData\Roaming\Typora\typora-user-images\image-20210922175159623.png)

caller：调用结束后，eax保存返回值。ecx和edx可能被破坏。

callee：ebx, esi , edi, ebp需保存，不能破坏。

esp可能改变。

#### 虚拟PC QEMU

内存的虚拟本质上就是用数组来模拟寄存器和内存。对于x86分段内存来说，要分段处理。

![image-20210922180323905](C:\Users\95395\AppData\Roaming\Typora\typora-user-images\image-20210922180323905.png)

CPU虚拟的本质就是用软件的方式模拟CPU的执行步骤，1.获取指令 2.解析指令的操作码 3.根据指令的操作码执行相应的流程：对寄存器和内存（也就是上面的数组）执行相应操作。3.改变eip。

对于硬件的模拟：用一个文件模拟硬盘；用print的方式模拟VGA；用主机键盘的API模拟键盘；用主机的时钟模拟时钟。

#### 作业

1.git xv6后进入文件夹make

2.nm kernel | grep _start   ，得到：

```
$ nm kernel | grep _start
8010a48c D _binary_entryother_start
8010a460 D _binary_initcode_start
0010000c T _start
```

其中nm命令可以查看文件中包含的符号，包括值、类型（指出了全局和局部）、符号名。T说明符号在text段，D说明在data段。

![image-20210922182934175](C:\Users\95395\AppData\Roaming\Typora\typora-user-images\image-20210922182934175.png)

发现无法使用make gdb来连接make qume-gdb启动的服务，原因是因为Makefile中没有加入gdb, 增加gdb: gdb -x .gdbinit即可。

在0x10000c断点，用info reg查看寄存器中的内容，用x/nx $esp查看栈中内容。

![image-20210922221754212](C:\Users\95395\AppData\Roaming\Typora\typora-user-images\image-20210922221754212.png)

要解释栈中的内容，就要看是谁调用并进入此处。所以作业描述上让我们从boot开始看。可以发现ebp指的地方是0，说明上一级ebp为0，说明这里是第二层。ebp往下显然就是上一层也是第一层的返回地址eip 7c4d，查看代码可以发现这里是调用bootmain的下一行地址，符合预期。7db7经过查看是调用main的下一行，故这里也是返回地址，之所以esp指向这里(esp还指向eip)是因为进入entry后还没有执行ebp入栈和esp等于ebp的操作。为了验证这点，找到了kernel.asm中断点所在的地方

![image-20210922230130643](C:\Users\95395\AppData\Roaming\Typora\typora-user-images\image-20210922230130643.png)

找到这个地方的逻辑是：boot肯定是为了启动内核，所以找kernel.asm反汇编。地址是8010000c而非10000c，但这个是运行地址（VMA）。查看kernel的section（objdump -h）发现确实是和10000c（LMA）映射。

#### 总结

1.知道了虚拟PC的一些基本思路。知道了x86指令集中out,in这两个命令。知道了IO在代码层面的基本思路。

2.作业的收获：对于xv6，BIOS将控制权给kernel后，kernel调用bootmain, bootmain调用入口；懂得了查看栈中值基本意义的思路（即先找eip）
