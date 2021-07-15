# pimbt_driver.h

头文件：`#include <linux/ioctl.h>`作用， ioctl是设备驱动程序中对设备的I/O通道进行管理的函数。

`long (*unlocked_ioctl) (struct file *文件结构体指针, unsigned int指令,unsigned long接收数据的地址)`

作用：此函数指针原型位于struct file_operations结构体当中，配合应用层ioctl函数实现指令传递的功能

以` _IOR (魔数， 基数， 变量型)`说明用法：

内核中_IO _IOR _IOW _IOWR的用法：主要之前linux内核中会区分命令码，让命令码驱动外设操作，但是程序员很难使用，所以ioctl中会提供宏来方便操作，其中的参数叫

1. 魔数(魔数范围为 0~255 。通常，用英文字符 "A" ~ "Z" 或者 "a" ~ "z" 来表示。设备驱动程序从传递进来的命令获取魔数，然后与自身处理的魔数想比较，如果相同则处理，不同则不处理。魔数是拒绝误使用的初步辅助状态)
2. 基数(基数用于区别各种命令。通常，从0开始递增，相同设备驱动程序上可以重复使用该值。例如，读取和写入命令中使用了相同的基数，设备驱动程序也能分辨出来，原因在于设备驱动程序区分命令时使用 switch ，且直接使用命令变量cmd值。创建命令的宏生成的值由多个域组合而成，所以即使是相同的基数，也会判断为不同的命令**（这里有点抽象 就相当于 基数+cmd值？然后就能辨别是读还是写 _IOR(MAJOR_NUM,0) _IOW(MAJOR_NUM,0) 虽然基数相同但是cmd不同 所以知道读写的区别 但是是不是同一个CMD中同一个基数就分辨不出来？）**
3. 变量型：变量型使用 arg 变量指定传送的数据大小，但是不直接代入输入，而是代入变量或者是变量的类型，原因是在使用宏创建命令，已经包含了 sizeof() 编译命令。比如这样：`#define IOCTL_START_TRAVERSE  _IOR(MAJOR_NUM, 1, unsigned long *)`，而不是这样`#define IOCTL_START_TRAVERSE  _IOR(MAJOR_NUM, 1, sizeof(unsigned long))`

# run_kernel_module.sh

# test-driver.c

一些头文件，还有自己设置的pimbt_driver.h。有宏定义定义pim设备的地址和大小，

**c语言中static的作用：**变量全局和控制函数的可见性。具体内存分配过程是：static会让编译的时候把那个global符号去掉，然后就不会全局链接。**链接的时候优先本文件进行链接，对于其他文件后链接。**

**c语言中const对于变量和指针的作用：**