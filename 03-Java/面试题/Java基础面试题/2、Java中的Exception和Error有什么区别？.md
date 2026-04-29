## 回答重点

Exception和Error都是Throwable类的直接子类，只有继承了Throwable的对象才能被throw和catch

### 核心区别

1. Exception代表程序运行过程中可以预料、可以恢复的异常情况，属于业务逻辑范畴，应该也能够被合理处理，比如NullPointEception、IOException等
2. Error代表JVM本身或者系统环境的严重错误，程序通常无法恢复，出现后往往意味着进程需要终止或重启，比如OutOfMemoryError、StackOverflowError等

简单说，==Exception是可以被处理的程序异常，Error是系统级的不可恢复错误==

Exception又分为Checked Exception 和 Unchecked Exception

1. Checked Exception：继承自Exception但不继承RuntimeException，编译时必须显式处理，要么try-catch要么throws声明抛出，比如IOException、SQLException
2. Unchecked Exception：继承自RuntimeException，不需要显式捕获，运行时才抛出，比如NullPointException、IndexOutBoundsException

![[Pasted image 20260428220216.png]]

## 扩展知识

### 异常处理的六个避坑点

**1、捕获异常要精准，别一把梭Exception**

软件工程是一门协作的艺术，代码要让别人一眼看出你想捕获什么。如果什么异常都用Exception接着，别的开发同事根本看不出这段代码实际想捕获啥，而且还会把本该往上抛的异常也给吞了

**2、别把异常吞了**

捕获了异常既不抛出也不写日志，线上出了bug就会莫名其妙找不到任何信息，不知道哪里出错、为什么出错

有些喜欢catch之后用 e.printStackTrace() ，这个方法输出的是标准错误流，在分布式系统中根本找不到stacktrace。==最好的做法是输出到日志里，用自定义格式把详细信息打到日志系统，方便排查==
![[Pasted image 20260428220833.png]]

**3、异常要尽早处理，别拖**

比如有个方法参数是name，方法内部调了好几个方法，name传的是null值，但没有在方法入口就处理这个情况，而是掉调了好几层之后才爆出空指针。本来堆栈信息只需要一点点就能定位，结果变成了一坨堆栈信息

**4、try-catch范围能小择小**

只在必要的代码段使用try-catch，别不分青红皂白try住一坨代码。try-catch本身的性能开销几乎可以忽略，但是被包裹的代码可能会影响JVM对代码的优化，比如指令重排序

**5、别用异常来控制程序流程**

能用if / else判断的，比如null值检查，就别用异常。异常比条件语句低效得多，条件语句有CPU分支预测优化。而且每实例化一个Exception都会对栈进行快照，这是个比较重得操作，数量多了开销就不能忽略了

**6、别再finally里处理返回值或直接return**

在finally中return或者处理返回值会发生很诡异得事情，比如覆盖了try中得return，或者屏蔽了异常

![[Pasted image 20260428221628.png]]

## 面试官追问

**1、为什么说Error不应该被捕获？你在什么场景下会考虑捕获Error？**

Error代表的是JVM或系统层面的严重问题，比如OutOfMemoryError、StackOverflowError，这时候JVM的状态已经不可靠了，即使捕获了也做不了什么有意义的恢复操作。只不过在一些特殊场景下会考虑捕获，比如在做框架开发时，为了保证主线程不挂掉，可能会catch Throwable然后记录日志；又比如在做内存敏感的批处理任务时，捕获OutOfMemoryError后可以清理部分缓存、释放资源，尝试让后续任务能继续跑

**2、Checked Exception和Unchecked Exception的设计初衷是什么？你更倾向用哪种？**

Checked Exception的设计初衷是强制调用方处理可能出现的异常，比如IO操作、数据库操作，编译器会逼着你要么try-catch要么throws。Unchecked Exception通常是编程错误导致的，比如空指针、数组越界，这种让程序员自己保证代码正确性，不需要编译器强制检查

实际项目中更倾向于用Unchecked Exception，因为Checked Exception会导致异常声明满天飞，代码可读性变差。Spring、Hibernate这些框架也都把Checked Exception包装成Unchecked Exception抛出

**3、try-catch对性能的影响有多大？什么情况下需要关注这个问题？**

try-catch对性能的开销几乎可以忽略，JIT编译器会优化掉没有异常抛出时的开销。真正影响性能的是异常的抛出和捕获过程，因为每次new一个异常对象都要对当前调用栈做一次快照，这个操作比较重。如果在热点代码里频繁抛异常，比如在循环里用异常来控制流程，QPS高的时候性能影响就很明显了。正常业务逻辑里偶尔抛个异常完全不用担心

