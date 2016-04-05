##lab1 实验报告

###练习1

> ucore.img的生成
```
    首先通过gcc命令编译了kern/init/init.c kern/libs/readline.c kern/libs/stdio.c kern/debug/kdebug.c kern/debug/kmonitor.c kern/debug/panic.c kern/driver/clock.c kern/driver/console.c kern/driver/intr.c kern/driver/picirq.c kern/trap/trap.c kern/trap/trapentry.S kern/trap/vectors.S kern/mm/pmm.c libs/printfmt.c libs/string.c 其命令行参数为-fno-builtin -Wall -ggdb -m32 -gstabs -nostdinc  -fno-stack-protector -Ilibs/，意思分别如下：
  - fno-builtin:不使用c语言的内建函数
  - Wall:生成所有警告信息
  - ggdb:使用gcc为gdb生成丰富的调试信息
  - gstabs:此选项以stabs格式声称调试信息,但是不包括gdb调试信息
  - m32:生成32位机器的汇编代码
  - nostdinc: 不在标准系统目录中搜索头文件,只在-I指定的目录中搜索
  - fno-stack-protector:进行编译器的堆栈保护，防止栈溢出
    编译后生成了相应的.o文件，再利用ld -m  elf_i386 -nostdlib -T tools/kernel.ld -o bin/kernel命令，将生成的.o文件生成kernel。ld命令是GNU的连接器，将目标文件连接为可执行程序，-T参数提供了链接脚本，elf_i386代表了可执行文件的格式，nostdlib代表了链接的时候不使用标准的系统启动文件和系统库。
    进而又编译了boot/bootasm.S boot/bootmain.c tools/sign.c，然后利用其执行文件，生成了ld bin/bootblock，最后通过ld -m    elf_i386 -nostdlib -N -e start -Ttext 0x7C00 obj/boot/bootasm.o obj/boot/bootmain.o -o obj/bootblock.o命令生成了bootblock.o,最后利用dd if=/dev/zero of=bin/ucore.img count=10000命令就生成ucore.img。其中dd命令作用是用指定大小的块拷贝一个文件，并在拷贝的同时进行指定的转换。
```
> 一个被系统认为是符合规范的硬盘主引导扇区的特征是什么
```
   从sign.c中可以看出主引导扇区特征为大小为512字节，最后两个字节为0x55，0xAA
```

###练习二
> 使用make debug命令进行gdb的调用，但是具体的命令要在gdbinit文件中进行更改，然后再进入gdb命令行操作，此时打印出0x7c00地址附近的反汇编命令，发现与bootasm.S与bootblock.asm中的命令相同

###练习三
> 为何开启A20,如何开启A20
```
通过修改A20地址线可以完成从实模式到保护模式的转换,激活32位地址线，使访存空间增大到4G。进行如下操作可以开启A20：
1. 等待8042 Input buffer为空；
2. 发送Write 8042 Output Port （P2）命令到8042 Input buffer；
3. 等待8042 Input buffer为空；
4. 将8042 Output Port（P2）得到字节的第2位置1，然后写入8042 Input buffer；
```
> GDT表如何初始化
```
lgdt gdtdesc命令来初始化GDT表
```
> 如何使能进入保护模式
```
将CR0寄存器的0号bit置成1，就进入了保护模式
```

###练习四
> bootloader如何读取硬盘扇区的？
```
通过readsect函数读取硬盘扇区，主要步骤为
1. 读I/O地址0x1f7，等待磁盘准备好
2. 发出读取扇区的命令（0x20）
3. 读I/O地址0x1f7，等待磁盘准备好
4. 把磁盘扇区数据读到指定内存
```
> bootloader是如何加载ELF格式的OS？
```
在bootmain函数中实现加载ELF格式的OS，首先加载ELF的头，判断其是否为ELF文件（ELFHDR->e_magic = ELF_MAGIC？），然后将描述表的头地址存在ph，按照描述表将ELF文件中数据载入内存，根据ELF头部储存的入口信息，找到内核的入口(不再返回)
```

###练习五
> 见代码 ebp记录了caller的ebp的地址，ebp+4记录了eip的地址。自己写的代码没有调出来，最后参考了答案的代码，发现了几个问题，一是提示中的arguments [0..4] = the contents in address (unit32_t)ebp +2 [0..4]，这一部分没有看懂，所以args的赋值写错了，最后使用了答案中的写法，二是eip  = ss:[ebp+4]和ebp =ss：[ebp]这个部分不理解ss：[]的意义，所以也写错了，也看了答案。

###练习六
> 见代码 中断向量表一个表项占用8字节，其中2-3字节是段选择子，0-1字节和6-7字节拼成位移，入口地址=段选择子+段内偏移量。代码中关于时钟中断的部分自己写出来了，但是idt_init函数不会操作，主要是SETGATE这个宏的参数不知道应该怎样赋值，所以最后参考了答案。

###challenge
> 见代码，主要是参考了答案
