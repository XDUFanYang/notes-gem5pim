# Linux中/dev作用

https://www.cnblogs.com/sdu20112013/p/11313585.html

# .ko文件处理作用

https://blog.csdn.net/qq_38880380/article/details/79227760

# pim_dirver

## makefile

obj-m?:obj-m表示把文件test.o作为"模块"进行编译，不会编译到内核，但是会生成一个独立的 "test.ko" 文件；obj-y表示把test.o文件编译进内核; https://blog.csdn.net/qq_28779021/article/details/78583981

CROSS_COMPILE 交叉工具链名称

## pimbt_driver.h

头文件：`#include <linux/ioctl.h>`作用， ioctl是设备驱动程序中对设备的I/O通道进行管理的函数。

`long (*unlocked_ioctl) (struct file *文件结构体指针, unsigned int指令,unsigned long接收数据的地址)`

作用：此函数指针原型位于struct file_operations结构体当中，配合应用层ioctl函数实现指令传递的功能

以` _IOR (魔数， 基数， 变量型)`说明用法：

内核中_IO _IOR _IOW _IOWR的用法：主要之前linux内核中会区分命令码，让命令码驱动外设操作，但是程序员很难使用，所以ioctl中会提供宏来方便操作，其中的参数叫

1. 魔数(魔数范围为 0~255 。通常，用英文字符 "A" ~ "Z" 或者 "a" ~ "z" 来表示。设备驱动程序从传递进来的命令获取魔数，然后与自身处理的魔数想比较，如果相同则处理，不同则不处理。魔数是拒绝误使用的初步辅助状态)
2. 基数(基数用于区别各种命令。通常，从0开始递增，相同设备驱动程序上可以重复使用该值。例如，读取和写入命令中使用了相同的基数，设备驱动程序也能分辨出来，原因在于设备驱动程序区分命令时使用 switch ，且直接使用命令变量cmd值。创建命令的宏生成的值由多个域组合而成，所以即使是相同的基数，也会判断为不同的命令**（这里有点抽象 就相当于 基数+cmd值？然后就能辨别是读还是写 _IOR(MAJOR_NUM,0) _IOW(MAJOR_NUM,0) 虽然基数相同但是cmd不同 所以知道读写的区别 但是是不是同一个CMD中同一个基数就分辨不出来？）**
3. 变量型：变量型使用 arg 变量指定传送的数据大小，但是不直接代入输入，而是代入变量或者是变量的类型，原因是在使用宏创建命令，已经包含了 sizeof() 编译命令。比如这样：`#define IOCTL_START_TRAVERSE  _IOR(MAJOR_NUM, 1, unsigned long *)`，而不是这样`#define IOCTL_START_TRAVERSE  _IOR(MAJOR_NUM, 1, sizeof(unsigned long))`

## run_kernel_module.sh

加载模块，insmod以及rmmod

## user_code.c

就是开一个pim寄存器 往里面写东西 然后读出来

## test-driver.c

一些头文件，还有自己设置的pimbt_driver.h。有宏定义定义pim设备的地址和大小，

**c语言中static的作用：**变量全局和控制函数的可见性。具体内存分配过程是：static会让编译的时候把那个global符号去掉，然后就不会全局链接。**链接的时候优先本文件进行链接，对于其他文件后链接。**

**c语言中const对于变量和指针的作用：**

定义了多种方法：`device_open`,`device_exit`,`device_release`,`device_read`,`device_write`,`device_ioctl`,`device_mmap`

### mmap的操作以及作用

https://blog.csdn.net/yangle4695/article/details/52139585

### **static** **int** __init device_init(**void**);

函数命名可以中断吗？不是 是一个宏`#define __init      __section(.init.text) __cold notrace`应该是那个控制所在的段 不是堆栈自己分配。

`#if 0 #else`在内核代码中很容易看到，这是什么？**如果设置if 0的代码被默认不编译的话，那为什么还要写呢？**

```c++
static int __init device_init(void)
{
#if 0
    int check;
    check=register_chrdev(major,"/dev/pimbt",&device_fops);
    if(check<0) {
		printk("problem in registering PIM module\n");
		return check;
	}
    printk("PimBT is succefully registered!\n");
#else
	int res;
	dev_t devno = MKDEV(major, 0);//设置要操作的设备地址
	cdev_init(&pim_cdev, &device_fops);//初始化字符设备(pim) 并定义它的操作
	pim_cdev.owner = THIS_MODULE;//定义owner?有啥用

	res = register_chrdev_region(devno, 1, "/dev/pimbt");//注册设备

	if (res) {//注册成功
		printk("Can't get major %d for PIM\n", major);
		return res;
	}

	res = cdev_add(&pim_cdev, devno, 1);//将pim设备添加进来

	if (res) {//添加完需要释放内存？
		printk("Error %d adding PIM\n", res);
		unregister_chrdev_region(devno, 1);
		return res;
	}

#endif

#if 1
	if (! request_region (BASE_PIM_DEV, SIZE_PIM_DEV, "/dev/pimbt")) {
		//在添加完以后还需要申请区域？为啥阿
		printk("request region failed");
		cdev_del(&pim_cdev);
		unregister_chrdev_region(devno, 1);//然后就需要释放设备
		return -ENODEV;
	}
#endif

	//	pim_dev_membase = ioremap(BASE_PIM_DEV, SIZE_PIM_DEV);

	//printk("IO remap address for PIM = %lx\n", (unsigned long)pim_dev_membase);

    return 0;
}
```

**主要疑问有两个：就是注册为什么分两步，为什么不能直接在内存中申请呢？为什么申请了以后还需要释放呢？**

### **void** device_exit(**void**);

内核中的MAJOR和MINOR以及MKDEV：https://blog.csdn.net/stan_linux/article/details/18357991

```c++
void device_exit(void)
{
    release_region(BASE_PIM_DEV, SIZE_PIM_DEV);//释放内存
	cdev_del(&pim_cdev);//消除字符设备
    unregister_chrdev_region(MKDEV(major, 0), 1);//将寄存器销毁
    printk("PimBT is removed! \n");
}
```

**这三者有什么区别？**

### **int** device_open(**struct** inode *inode, **struct** file* filep);

```c++
int device_open(struct inode *inode, struct file* filep)
{
    printk("PimBT is opened!\n");//单纯打印信息
    return 0;
}
```

### **int** device_release(**struct** inode *inode, **struct** file* filep);

```c++
int device_release(struct inode *inode, struct file* filep)
{
    printk("PimBT is closed! \n");//单纯打印释放信息
    return 0;
}
```

### **ssize_t** device_read(**struct** file* filp, **char*** buf, **size_t** count,loff_t* f_pos);

```c++
ssize_t device_read(struct file* filp, char* buf, size_t count, loff_t* f_pos)
{
    printk("PimBT is being read\n");
    memset(buf,'a',count);//将前count个字符用a替换
    //printk("count: %d\n",count);
    return count;
}//很搞，怎么读呢？
```

### **ssize_t** device_write(**struct** file* filp, **const** **char** __user * buf, **size_t** count, loff_t* f_pos);

```c++
ssize_t device_write(struct file* filp, const char __user * buf, size_t count, loff_t* f_pos)
{
    int i, pagenum;
    int *root = (int *)(*(long *)buf);
	unsigned long pa, pgd;
	struct page *phy_page;
	struct vm_area_struct *vma;

	printk("count = %ld, root = (va) %lx, key = %lx\n", 
		   count, (unsigned long)root, *((long *)buf +1 ));
    for (i = 0; i < 8 ; i++)
		printk("data = %d\n", root[i]);
	
	pagenum = get_user_pages(current, current->mm, (unsigned long)root, 1, 0, 0, &phy_page, &vma);//获得进程的也表获取属于指定进程的地址范围内的页列表，并指示覆盖每个页的VMA。但这可能是不可靠的，因为我们可能会终止从复合页增加slab页或辅助页的页数；不允许访问不支持它的vma，例如I/O映射。
	if (pagenum == 0)
		{
			printk("Unable to lock the user page\n");
		}
	else 
		{
			pa = page_to_phys(phy_page);//拿到物理地址
			printk("Physical address is %lx, vma start = %lx, vma end = %lx\n", pa, vma->vm_start, vma->vm_end);
		}
	pgd = virt_to_phys(current->mm->pgd);//这也是拿到物理地址 为什么拿两个？
	printk("pgd address is (va) %lx, (pa) %lx\n", (unsigned long)current->mm->pgd, pgd);

    outl((unsigned int)(unsigned long)root, BASE_PIM_DEV);
    outl((unsigned int)((unsigned long)root >> 32), BASE_PIM_DEV + 4);
    outl((unsigned int)pgd, BASE_PIM_DEV + 8);
    outl((unsigned int)((unsigned long)pgd >> 32), BASE_PIM_DEV + 12);

    return count;//
}
```

**vma是什么？这个物理页为什么拿两次？**

### **long** device_ioctl(**struct** file *file, **unsigned** **int** ioctl_num, **unsigned** **long** ioctl_param);

```c++
long device_ioctl(struct file *file, unsigned int ioctl_num, unsigned long ioctl_param)
{
	unsigned long *buf;
	unsigned long tempbuf[3];
	unsigned long baseaddr = 0;
	int retval = -1;
	unsigned long pgd;
	int done;
	//	unsigned long lo, hi;
	//	printk("ioctl_num %ld, %ld, %ld\n", ioctl_num, IOCTL_START_TRAVERSE, IOCTL_POLL_DONE);

	switch (ioctl_num) {
	case IOCTL_START_TRAVERSE:
		buf = (unsigned long *)ioctl_param;
		if (0 != copy_from_user(tempbuf, buf, sizeof(unsigned long) * 3)) {
			printk("Unable to get buf %lx from user", (unsigned long) buf);
		}
		baseaddr = BASE_PIM_DEV + (buf[2] << 8);
		pgd = virt_to_phys(current->mm->pgd);

		outl((unsigned int)buf[0], baseaddr + RootAddrLo);
		outl((unsigned int)(buf[0] >> 32), baseaddr + RootAddrHi);

		outl((unsigned int)pgd, baseaddr + PageTableLo);
		outl((unsigned int)(pgd >> 32), baseaddr + PageTableHi);

		outl((unsigned int)buf[1], baseaddr + KeyLo);
		outl((unsigned int)(buf[1] >> 32), baseaddr + KeyHi);

		outl(1, baseaddr + StartTrav);

		retval = 0;

		break;

	case IOCTL_POLL_DONE:
		buf = (unsigned long *)ioctl_param;
		get_user(baseaddr, buf);
		convert_to_pa(baseaddr);
		done = 1;
		retval = done;

		break;
	}

	return retval;
}
```

**这个ioctl很懵 没咋看懂**

### **int** device_mmap(**struct** file *file, **struct** vm_area_struct *vma);

```c++
int device_mmap(struct file *file, struct vm_area_struct *vma)
{
	unsigned long size, pgd, baseaddr, i;
	unsigned long physaddr, pfn;

	physaddr = (unsigned long)(BASE_PIM_DEV + 0x2f000000);
	pfn = physaddr >> PAGE_SHIFT;

	printk("Device mmap %lx %lx\n", physaddr, pfn);

	size = vma->vm_end - vma->vm_start;//这个vma是不是虚拟地址？

	if (size > SIZE_PIM_DEV)
		size = SIZE_PIM_DEV;

	vma->vm_flags |= VM_SHARED;
	vma->vm_page_prot = __pgprot_modify(vma->vm_page_prot, PTE_ATTRINDX_MASK, 
										PTE_ATTRINDX(MT_DEVICE_nGnRE) | PTE_PXN | PTE_UXN);//这个状态是什么
	//pgprot_noncached(vma->vm_page_prot);
    //需要设置一些状态

	remap_pfn_range(vma,
					vma->vm_start,
					pfn,
					size,
					vma->vm_page_prot);//remap kernel memory to userspace
    //为什么还需要将内核空间映射到用户空间？

	// Write the base of page table to hardware
	pgd = virt_to_phys(current->mm->pgd);

	for (i = 0; i < 4; i++) {
		baseaddr = BASE_PIM_DEV + (i << 8);

		pgd = virt_to_phys(current->mm->pgd);

		outl((unsigned int)pgd, baseaddr + PageTableLo);
		outl((unsigned int)(pgd >> 32), baseaddr + PageTableHi);
	}

	return 0;
}
```

test-driver.c大致看完，但是还是不知道那个创建的新设备在哪里创建？有几个很关键问题不清楚。

# pim_lib

看完