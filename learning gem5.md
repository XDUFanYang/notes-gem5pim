http://learning.gem5.org/

http://github.com/powerjg/learning_gem5

介绍不同课程的介绍内容

# 第一课

如何build gem5 确定os版本 然后就可以git clone build  大概150MB 然后git切换版本 进行编译(大概8分钟)

然后那个解释scons build/x86/gem5.opt -j5是啥

scons:http://scons.org/

build/x86/gem5.opt:gem5 binary to run

x86:

opt:

gem5的结构：由simobject组成，基本所有的c++对象都由simobject继承；这些类表示了真实的物理系统单元

gem5是一个离散事件模拟器：有一个事件队列，先从事件队列里面拿出第一个事件，然后执行模拟这个事件，这个事件可能会再生成一些事件，然后将事件入队。于此同时，其他事件也可以入队。

gem5离散事件模拟的调度：？

gem5的configuring:user interface python脚本定义系统；c++类都暴露给python？

先定义系统类 然后系统类里面有clock voltage memory cpu bus mem_controller process...然后初始化系统。

```
.../gem5.opt .../simple.py
```

然后脚本会提示初始化完成

## port interface

通过master slave来连接 master->slave(req) slave->master(res) 

## simulation mode

se mode/full mode：full mode类似于qemu

se mode:系统调用 不用os run

## add cache

介绍了更改cache参数的文件

做了个小实验，通过更改l1 cache size 可以得到那个模拟的效果，可以看出提升l1cachesize的效果

## understanding the output

config.ini:告诉你模拟了什么参数

config.json:json形式的上一个

stats.txt:模拟输出

## 关于checkpoint



# 第二课

## debug --help --flag=DRAM

告诉你模拟的模块信息

--flag=DRAM 会直接打印其中的dram信息

## debugflag

修改hello.cc 其中修改DPRINT(Hello,"create the simobject") 就可以在scons中调用 debugflag(Hello)

## event-driven programming

--debugflag=Exec 就是模拟微操作

修改hh cc 注册然后增加函数 然后recompile

simple event callback: eventfunctionwrapper event 简单事件的便捷类

## sim parameters

在scons中添加HelloObject()，指定参数？是在scons中指定吗？

how to build simboject? how to schedule event?debug statements in gem5?add parameter to simobject?

## MemObject

需要master slave/包有请求包、数据包、命令包...

讨论了请求包成功和不成功的情况

master 和 slave 中有很多数据

现场写代码： 类中套类

这个很复杂...

## Cache

如何模拟数据存储 标签 相联模式以及数据访存延迟 锁

## More on events

schedule();什么的

# 第三课

## ruby概念

cpu dma others dramcc 这些是经典端口

很多控制器 需要片上通信

ruby组成：控制器 连接模式 网络模型

控制器模型由SLICC编译

永远不要修改一致性文件！

# 第四课

## packet构造

可以支持更多的包构造

删除或者不删除packet

## gem5的执行模型 ISA和CPU

支持很多ISA指令集 ISA指令的设置

分为static instr

cpu的类别 以及与其相匹配的内存