###### 创建线程的两种方法

1. 实现Runnable接口

2. 继承Thread类

3. 实现Callable接口

   通过start()方法启动的线程

   Thread.run()//方法级别调用，将run方法加入main线程栈，并不启动一个新线程

   Thread.start()//启动一个新的线程

Thread.join() 等待所有线程执行完之后，主线程再执行

使用线程池执行10000个线程，用时35ms，不用的话，用时3500ms，差100倍

###### ExecutorService的submit和execute的区别

1. submit有返回值
2. submit()可以进行Exception处理

###### ThreadPoolExecutor构造函数参数

corePoolSize：核心线程数量

maximumPoolSize：非核心数量，如果核心和非核心相同，当核心线程满时，则不创建非核心线程

keepAliveTime：保持时长

unit：时间单位

workQueue：任务阻塞队列

threadFactory：线程工厂

handler：拒绝策略

###### 线程池执行task的4中情况

1. 小于核心线程数时，addwork方法，直接运行task
2. 大于核心线程数时，将task添加到BlockingQueue中，通过getTask获取队列中任务
3. 如果BlockingQueue满时，创建一个非核心线程，执行task任务
4. 如果创建非核心线程失败，执行拒绝策略
5. 当线程池中超过corePoolSize线程，空闲时间达到keepAliveTime时，关闭空闲线程
6. 当设置allowCoreThreadTimeOut(true)是，线程池中的corePoolSize线程空闲时间达到keepAliveTime也将关闭

###### 队列常用方法

add(E)/offer(E)：入队

remove()/poll()/take() ：出队

element()/peek()： 查看对头元素

###### 拒绝策略4种

| RejectedExecutionHandler | 特性及效果                                                   |
| ------------------------ | ------------------------------------------------------------ |
| AbortPolicy              |                                                              |
| DiscardPolicy            | 如果添加失败，则放弃，并且不抛出任何异常                     |
| DiscardOldestPolicy      | 如果添加到线程池失败，会将队列中最早添加的元素移除，再尝试添加，如果失败，则该策略不断重试 |
| CallerRunPolicy          | 如果添加失败，那么主线程会自己调用执行器中的execute方法来执行该任务 |
| 自定义                   | 需要实现RejectedExecutionHandler接口，重写rejectedExecution方法 |

###### 进程

进程是系统资源分配的对立实体，每个进程都拥有独立的地址空间。一个进程无法访问另一个进程的变量和数据结构，如果想让一个进程访问另一个进程的资源，需要使用进程间通信，比如管道、文件、套接字等

###### 线程

CPU调度的最小单位，必须依赖进程存在，每个线程还拥有自己的寄存器和栈

###### C核心数和线程的关系

同时运行是1:1的关系，但有些cpu有超线程技术，可以是1:2

###### CPU时间片轮转机制

cpu轮转分配时间片给线程队列，调度过程需要上下文切换

如果线程在分配的时间片内没有执行完，该线程被放到队列尾，执行下一个队列头的线程，因为线程拥有自己的寄存器和栈，所以这时需要上下文切换，频繁的话，比较浪费时间。

Thread.yield()会交出cpu的时间片，该线程停止执行，进入到就绪runnable状态（该方法基本弃用）

##### sleep()和wait()的区别

sleep释放cpu，但不释放锁；wait释放cpu，释放锁

###### join()和wait()的区别

join是循环调用wait，判断子线程是否存活