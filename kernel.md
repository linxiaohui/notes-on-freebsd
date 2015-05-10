# 内核相关

## 重新编译内核

### 备份内核
```
cp -Rp /boot/kernel /boot/kernel.good
```
   * -p 保留原属性信息

### 调整配置文件
   * 复制配置文件
```
cd /usr/src/sys/amd64/conf
mkdir /root/kernels
cp GENERIC /root/kernels/MYKERNEL
ln -s /root/kernels/MYKERNEL
```
     * dmesg |grep 'not found'		(看看GENERIC 内核是不是捕捉到了所有 硬件)
     * dmesg |grep -i cpu		(K8为CPU架构)
```
CPU: Intel(R) Core(TM) i7 CPU	Q740	@ 1.73GHz (1728.97-MHz K8-class CPU)
```
     * 判断内核配置 CPU 选项如何填写：
```
less /usr/src/sys/amd64/conf/NOTES	(找到相应架构的目录)
CPU	OPTIONS
cpu			HAMMER		# aka K8, aka Opteron & Athlon64
```


   * 方法一：根据 GENERIC 调整配置

```
#MYKERNEL
# Basic Options
cpu			HAMMER		(CPU 架构类型)
ident			MYKERNEL	(命名)
makeoptions 	DEBUG=-g		# Build kernel with gdb debug symbols
options 		SCHED_ULE	# ULE scheduler
options		PREEMPTION 		# Enable kernel thread preemption
options		 INET 			# InterNETworking
……
...	...	...
# Device Drivers
# Floppy drives
# device fdc		(注释掉没有的设备)
```

   * 方法二：Include GENERIC 

```
#MYKERNEL
ident 	MYKERNEL		(命名)
include	GENERIC			(包含 GENERIC)
options	CPU_SOEKRIS		(添加 选项)
nooption	MSDOSFS		(撤销 选项)
nodevice	fdc		(撤销 设备)
```

关于options参考文件
   * /usr/src/syc/conf/NOTES  和平台无关的参数
   * /usr/src/syc/i386/conf/NOTES 特定的平台的参数

### 编译内核
```
cd /usr/src
make buildkernel KERNCONF=MYKERNEL
make installkernel KERNCONF=MYKERNEL
```
**备注** 根据`less /usr/src/UPDATING` make的时候最好不要使用参数 `-j`

### 安装测试
   * 方法一：安装后重启运行新的内核
   * 方法二：nextboot  
根据`less /usr/src/UPDATING`若只想执行新内核一次
```
make installkernel KERNCONF=YOUR_KERNEL_HERE KODIR=/boot/testkernel
nextboot -k testkernel
```
或者可以 在安装了新内核后
```
mv /boot/kernel /boot/kernel.test
mkdir /boot/kernel
cp /boot/kernel.good/* /boot/kernel/	(将老版本Kernel 拉回来)
nextboot -t kernel.test			(下次启动运行kernel.test,仅一次)
reboot
mv /boot/kernel /boot/kernel.previous	(测试后保存 老版本 kernel)
mv /boot/kernel.test /boot/kernel	(将kernel.test 改为默认 kernel)
```
   * 方法三：Loader Prompt
```
  ok unload			(erase the loaded kernel and all modules from memory)
  ok load /boot/kernel.test/kernel
  ok load /boot/kernel.test/acpi.ko
  ok boot			(使用 kernel.test 重新启动)
```


## 内核模块

### bsd.kmod.mk
这个makefile位于/usr/src/share/mk/bsd.kmod.mk，
它极大的简化了内核模块的开发。
开发内核模块需要设置两个变量：
   * 通过`KMOD`变量设置内核模块名；
   * 通过`SRCS`变量设置源文件；
然后使用include <bsd.kmod.mk>来构建模块。
这个简明的设置使得只需要用以下makefile模板并简单地通过make即可着手构建的内核模块。

入门的内核模块的Makefile类似于这样的内容：
```makefile
# Note: It is important to make sure you 
include the <bsd.kmod.mk> makefile after declaring the KMOD and SRCS variables.
# Declare Name of kernel module
KMOD    =  hello_fsm
# Enumerate Source files for kernel module
SRCS    =  hello_fsm.c
 # Include kernel module makefile
.include <bsd.kmod.mk>
```
在$HOME下新建一个叫做kernel的目录，并将Makfile复制到该目录下。

### 创建一个模块
运行中的内核插入和移除一个模块的机制：
内核模块允许动态功能添加到一个运行中的内核中。
当一个内核模块被插入，load事件被触发。
当一个内核模块被移除时，unload事件被触发。
内核模块需要处理这些事件。

运行中的内核将传入/usr/include/sys/module.h头文件中所定义的符号
常量框架中的事件给内核模块。目前关注MOD_LOAD以及MOD_UNLOAD这两个事件。

模块使用DECLARE_MODULE宏配置回调函数以供内核调用（并将事件类型作为参数）

DECLARE_MODULE（<sys/module.h>）的参数：
   1. name. 定义名字
   2. data. 指定moduledata_t结构的名字（<sys/module.h>）
   3. sub. 设定子系统接口，它定义了模块的类型。
   4. order. 在被定义的子系统中定义模块初始化顺序。
   
moduledata结构包括了一个char类型的名字和modeventhand_t（<sys/module.h>）的事件处理器。此外，moduledata结构拥有一个void*的指针。
新建一个hello_fsm.c的文件：
```c
#include <sys/param.h>
#include <sys/module.h>
#include <sys/kernel.h>
#include <sys/systm.h>
```
接着将要实现event_handler函数。
This is what the kernel will call and pass either 
MOD_LOAD or MOD_UNLOAD to via the event parameter
```c
/* The function called at load/unload. */
static int event_handler(struct module *module, int event, void *arg) {
        int e = 0; /* Error, 0 for normal return status */
        switch (event) {
        case MOD_LOAD:
                uprintf("Hello FreeBSD! \n");
                break;
        case MOD_UNLOAD:
                uprintf("Bye Bye FreeBSD!\n");
                break;
        default:
                e = EOPNOTSUPP; /* Error, Operation Not Supported */
                break;
        }
        return(e);
}
```

定义DECLARE_MODULE宏的第二个参数：它是moduledata_t类型。
This is where you set the name of the module and expose 
the event_handler routine to be called when loaded and unloaded from the kernel
```c
/* The second argument of DECLARE_MODULE. */
static moduledata_t hello_conf = {
    "hello_fsm",    /* module name */
     event_handler,  /* event handler */
     NULL            /* extra data */
};
```
最后，使用DECLARE_MODULE定义模块。
```c
DECLARE_MODULE(hello_fsm, hello_conf, SI_SUB_DRIVERS, SI_ORDER_MIDDLE);
```
### 构建模块
make

### 装载和卸载模块
   * 装载模块
```bash
sudo kldload ./hello_fsm.ko
或者
sudo make load
```
现在应该在控制台可以看到打印的信息了。kldstat查看已经装载的模块。

   * 卸载模块
```bash
sudo kldunload hello_fsm
或者
sudo make unload
```
