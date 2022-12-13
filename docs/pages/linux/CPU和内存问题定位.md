## Java服务，CPU100%问题如何快速定位

假设，服务器上部署了若干 Java 站点服务，以及若干 Java 微服务，突然收到运维的 CPU 异常告警。如何定位是**哪个服务进程**导致 CPU 过载，**哪个线程**导致 CPU 过载，**哪段代码**导致 CPU 过载？

简要步骤如下：

（1）找到最耗 CPU 的进程；

（2）找到最耗 CPU 的线程；

（3）查看堆栈，定位线程在干嘛，定位对应代码；

**步骤一、找到最耗 CPU 的进程**

**工具**：`top`

**方法**：

- 执行 `top -c` ，显示进程运行信息列表
- 键入 `P` (大写 p)，进程按照 CPU 使用率排序

图示：

![img](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/8071583329e14910b619fb7bf921f748~tplv-k3u1fbpfcp-zoom-1.image)

如上图，最耗 CPU 的进程 PID 为 10765。

**步骤二：找到最耗 CPU 的线程**

**工具**：top

**方法**：

- `top -Hp 10765` ，显示一个进程的线程运行信息列表
- 键入 `P` (大写 p)，线程按照 CPU 使用率排序

图示：

![img](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/3b96a404abd84ee8a108beab98b18be2~tplv-k3u1fbpfcp-zoom-1.image)

如上图，进程 10765 内，最耗 CPU 的线程 PID 为 10804。

**步骤三：查看堆栈，定位线程在干嘛，定位对应代码
 **首先，将线程 PID 转化为 16 进制。

**工具**：`printf`

**方法**：`printf "%x\n" 10804`

图示：

![img](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/f3cef4c2bc82451f9e0f167d76247e44~tplv-k3u1fbpfcp-zoom-1.image)

如上图，10804 对应的 16 进制是 0x2a34，当然，这一步可以用计算器。

之所以要转化为 16 进制，是因为堆栈里，线程 id 是用 16 进制表示的。

接着，查看堆栈，找到线程在干嘛。

**工具**：`jstack`

**方法**：`jstack 10765 | grep '0x2a34' -C5 --color`

- 打印进程堆栈
- 通过线程 id，过滤得到线程堆栈

图示：

![img](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/dc822d5f57294c26b3af1332ae77462a~tplv-k3u1fbpfcp-zoom-1.image)

如上图，找到了耗 CPU 高的线程对应的线程名称 “AsyncLogger-1”，以及看到了该线程正在执行代码的堆栈。
 最后，根据堆栈里的信息，找到对应的代码，搞定！





## Java服务，内存OOM问题如何快速定位

OOM的问题，印象中之前写过，这里再总结一些相对通用的方案，希望能帮助到Java技术栈的同学。

某Java服务（假设PID=10765）出现了OOM，最常见的原因为：

- 有可能是内存分配确实过小，而正常业务使用了大量内存
- 某一个对象被频繁申请，却没有释放，内存不断泄漏，导致内存耗尽
- 某一个资源被频繁申请，系统资源耗尽，例如：不断创建线程，不断发起网络连接



**画外音：无非“本身资源不够”“申请资源太多”“资源耗尽”几个原因。**

更具体的，可以使用以下工具逐一排查。



**一、确认是不是内存本身就分配过小**

方法：`jmap -heap 10765`

![图片](https://mmbiz.qpic.cn/mmbiz_png/YrezxckhYOxiacnickoCpdKk29UN25JFBibgib9oCicicufurPH61ZgM9p7gDqtcdnvggavWQXeYPrWM2EtC5jTNFiaqw/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

如上图，可以查看新生代，老生代堆内存的分配大小以及使用情况，看是否本身分配过小。

 

**二、找到最耗内存的对象**

方法：`jmap -histo:live 10765 | more`

![图片](https://mmbiz.qpic.cn/mmbiz_png/YrezxckhYOxiacnickoCpdKk29UN25JFBibuWcaZKwIrAjpSscVoy7FXF4RLGhdiakUkQdqPlnHsMBsI9myvBxo58Q/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

如上图，输入命令后，会以表格的形式显示存活对象的信息，并按照所占内存大小排序：

- **实例数**
- **所占内存大小**
- **类名**

是不是很直观？对于实例数较多，占用内存大小较多的实例/类，相关的代码就要针对性review了。

 

上图中占内存最多的对象是RingBufferLogEvent，共占用内存18M，属于正常使用范围。



如果发现某类对象占用内存很大（例如几个G），很可能是类对象创建太多，且一直未释放。例如：

- 申请完资源后，未调用close()或dispose()释放资源
- 消费者消费速度慢（或停止消费了），而生产者不断往队列中投递任务，导致队列中任务累积过多

*画外音：线上执行该命令会强制执行一次fgc。另外还可以dump内存进行分析。**

*

**三、确认是否是资源耗尽**

工具：

- `pstree`
- `netstat`

查看进程创建的线程数，以及网络连接数，如果资源耗尽，也可能出现OOM。

 

这里介绍另一种方法，通过

- `/proc/${PID}/fd`
- `/proc/${PID}/task`

可以分别查看句柄详情和线程数。

 

例如，某一台线上服务器的sshd进程PID是9339，查看

- `ll /proc/9339/fd`
- `ll /proc/9339/task`

![图片](https://mmbiz.qpic.cn/mmbiz_png/YrezxckhYOxiacnickoCpdKk29UN25JFBibBAiaMFLiaqPQtiah5SUuKhJ5VRcL01Q6TYOecnYPB943xZo7DN8Ejd3rg/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

如上图，sshd共占用了四个句柄

- 0 -> 标准输入
- 1 -> 标准输出
- 2 -> 标准错误输出
- 3 -> socket（容易想到是监听端口）

 

sshd只有一个主线程PID为9339，并没有多线程。



所以，只要

- `ll /proc/${PID}/fd | wc -l`
- `ll /proc/${PID}/task | wc -l （效果等同pstree -p | wc -l）`

就能知道进程打开的句柄数和线程数。