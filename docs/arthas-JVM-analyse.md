# arthas分析jvm进程线程——工具用法

###### 快速启动调试工具
`curl -O https://arthas.aliyun.com/arthas-boot.jar && java -jar arthas-boot.jar`

## 常用命令
###### jvm
打印当前jvm虚拟机状态
###### thread -n 10
打印吃CPU最大的10个线程信息
###### thread --all
打印所有线程信息
###### thread -b
分析死锁的线程
###### thread $pid
打印进程ID的堆栈信息


一. 背景介绍
Arthas 是 Alibaba 在 2018 年 9 月开源的 Java 诊断工具。支持 JDK6+， 采用命令行交互模式，提供 Tab 自动补全，可以方便的定位和诊断线上程序运行问题。得益于 Arthas 强大且丰富的功能，让 Arthas 能做很多的事情，比如以下场景：

是否有一个全局视角来查看系统的运行状况？
为什么 CPU 又升高了，到底是哪里占用了 CPU ？
运行的多线程有死锁吗？有阻塞吗？
程序运行耗时很长，是哪里耗时比较长呢？如何监测呢？
这个类从哪个 jar 包加载的？为什么会报各种类相关的 Exception？
遇到问题无法在线上 debug，难道只能通过加日志再重新发布吗？
二. 安装和启动
在服务器上输入以下安装命令：
`wget https://arthas.aliyun.com/arthas-boot.jar`
![](assets/17241538861737.jpg)


系统会下载arthas-boot.jar， 要启动arthas就直接运行这个jar包：
`java -jar arthas-boot.jar`
arthas-boot是Arthas的启动程序，它启动后，会列出所有的Java进程，用户可以选择需要诊断的目标进程。
![](assets/17241538916693.jpg)


attach成功后，会看到Arthas的logo，启动成功：
![](assets/17241538979064.jpg)


快速退出某个命令：Q或者Ctrl+C
退出Arthas: exit或者quit， 退出当前session，Arthas server还在目标进程中运行。
彻底退出: stop. 用完一定要stop哦，避免Arthas server依然运行占用系统资源。

三. 使用Arthas trace命令定位代码耗时
性能测试过程中，经常会碰到接口请求耗时长，但是又不知道具体是哪个环节哪段代码耗时长。这个时候Arthas的trace命令的作用就体现出来了，可以方便快捷从方法表层顺着调用链路一步步往下追踪，最终找出具体耗时长的代码块，是性能测试优化的神器。

举例：假设用例列表页有性能问题，加载列表耗时长，下面介绍如何使用Arthas一步一步定位到具体是哪段代码耗时长：




1.通过浏览器F12查看network,找到请求URL为: /getList，反查代码，可以知道Controller层面的方法名：getList：
![](assets/17241539162360.jpg)



1.回到服务器上，使用trace命令：`trace com.xxxx.controller.DubboCaseController getList
‘#cost > 2’`
com.xxxx.controller.DubboCaseController 是controller的路径和类名 getList是方法名 '#cost > 2' 表示查找耗时大于2ms的方法

![](assets/17241539511558.jpg)
![](assets/17241539916280.jpg)



1.根据提示，是DubboCaseService下的getList()方法耗时长。那就继续往下查：
`trace com.xxxx.service.DubboCaseService getList ‘#cost > 2’`
得到getListWithoutCount方法耗时长

![](assets/17241540046455.jpg)


继续往下：`trace com.xxxx.service.impl.DubboCaseServiceImpl getListWithoutCount ‘#cost > 2’`
得到getAllByMavenId方法耗时长

![](assets/17241540127198.jpg)

![](assets/17241540247158.jpg)


到此已经定位到是这段SQL耗时比较长，可以针对性的优化SQL。当然上述举例是代码逻辑比较简单，所以最终反映是在SQL上耗时长。如果代码逻辑复杂，那可能定位到的就是前面某个代码方法的逻辑耗时长了，那就可以针对那个代码方法做优化。

四、其他常用命令使用介绍
1. thread命令：查看线程
   用thread命令列出线程的信息:

![](assets/17241540310859.jpg)


如果发现某个线程CPU使用过高，通过thread加线程id输出该线程的栈信息：

![](assets/17241540363452.jpg)


从输出结果可以看到这个线程处于运行状态，在执行的具体某个方法，方法中的代码行号。这样就可以去找到对应的代码块，定位问题。

`thread -n 3` 查看CPU使用率top n线程的栈:

![](assets/17241540417887.jpg)



thread -b 找出当前阻塞其他线程的线程。有时候我们发现应用卡住了， 通常是由于某个线程拿住了某个锁， 并且其他线程都在等待这把锁造成的。 为了排查这类问题， arthas提供了thread -b， 一键找出那个罪魁祸首:

![](assets/17241540471923.jpg)



2. Dashboard命令：查看当前系统的实时面板
   命令：dashboard, 每5秒刷新一次面板

![](assets/17241540511221.jpg)

![](assets/17241540572874.jpg)


比如内存泄露：如果内存使用率在不断上升，而且gc后也不下降，后面还发现gc越来越频繁，很可能就是内存泄漏了。这个时候我们可以直接用heapdump命令：heapdump --live /root/jvm.hprof 把内存快照dump出来，作用和jmap工具一样（jmap -dump:live）：

![](assets/17241540673651.jpg)


下载下来，再使用内存dump分析工具，比如：MAT（ Eclipse Memory Analyzer），分析可能泄露的内存原因：

![](assets/17241540726433.jpg)



3. watch命令：查看指定方法的调用情况
   `watch com.xxxx.xxxxController update “{params,returnObj}” -x 3 -b -s`，查看xxxxController的update方法的返回值：

-x 3是指定输出结果的属性遍历深度，默认为 1
-b方法调用前观察，用于返回方法入参
-s方法调用后观察，用于返回方法返回值

![](assets/17241540814528.jpg)



备注：更多的时候，是用于这个方法代码里面你没输出日志，临时看一下入参&返回结果定位问题。而不用重新去加日志打印的代码，重新发布应用。

4. monitor命令：监控方法的执行情况
   包括：成功次数、失败次数、平均响应时间、失败率

`monitor -c 10 com.xxxx.xxxxController update`

- 监控update这个方法的执行情况
- -c 10 指定统计周期为10秒统计一次，默认是120秒统计一次

![](assets/17241540896969.jpg)



5. tt命令: TimeTunnel 记录下方法执行数据的时空隧道
   `tt -t com.xxxx.xxxxController list`

![](assets/17241540952908.jpg)


INDEX: 时间片段记录编号，每一个编号代表着一次调用，后续tt还有很多命令都是基于此编号指定记录操作，非常重要。
TIMESTAMP: 方法执行的本机时间，记录了这个时间片段所发生的本机时间
COST(ms): 方法执行的耗时
IS-RET: 方法是否以正常返回的形式结束
IS-EXP: 方法是否以抛异常的形式结束
OBJECT: 执行对象的hashCode()，注意，曾经有人误认为是对象在JVM中的内存地址，但很遗憾他不是。但他能帮助你简单的标记当前执行方法的类实体
CLASS: 执行的类名
METHOD: 执行的方法名

通常该命令是用于：根据之前的记录直接重做一次调用。
因为有的时候调用不是那么好触发的，这个时候这个命令就比较有用，可以快速重新调用一次：
`tt -i 1003 -p`   表示重做Index为1003的那次调用

![](assets/17241541031390.jpg)




6. stack命令：监控方法的被执行的路径
   很多时候我们都知道一个方法被执行，但这个方法被执行的路径非常多，或者你根本就不知道这个方法是从那里被执行了，此时你需要的是 stack 命令, 主要用于监控方法被谁调用了:
   `stack com.xxxx.xxxxController list`

7. sm命令：能搜索出所有已经加载了 Class 信息的方法信息
   `sm -d com.xxxx.xxxxController`

8. jad命令：反编译指定已加载类的源码
   `jad com.xxxx.xxxxController`

反编译源码到指定文件：
jad --source-only com.xxxx.xxxxController > /tmp/xxxxController.java
-source-only表示只打印源码，如果不加这个参数，在反编译出的内容头部会携带类加载器的信息，内容太多
-/tmp/xxxxController.java 表示打印到tmp文件夹下xxxxController.java文件中

9. logger命令：实现动态更新logger level
   使用sc命令查看你需要改变的类信息，关注classLoaderHash
   `sc -d com.xxxx.xxxxController`

![](assets/17241541207893.jpg)

logger -c 70ac4376 查看当前的日志级别

![](assets/17241541282710.jpg)


查看具体某一个name的日志级别：`logger -c 70ac4376 --name com.xxxx.xxxx`

将日志级别改为info: `logger -c 70ac4376 --name com.xxxx.xxxx --level info`
10. 其他说明
    Arthas还可以通过使用一系列命令：jad>mc>redefine热更新线上代码，实现不重新发版试验新代码效果。但研发一般不敢在生产环境轻易用这个功能，因为：

黑屏化的操作可能会导致误操作
不符合安全生产的规范，不满足可监控、可回滚、可降级

Arthas中文版指导文档：
> https://arthas.aliyun.com/doc/a
