#### JVM运行时数据区

虚拟机栈，本地方法区，程序计数器是线程独占

堆，元空间是线程共享的

类的每个方法，对应一个栈帧，每个栈帧包含局部变量表，操作数栈，方法出口，动态链接

| 栈帧区域                 | JVM指令集                          | java      | 说明                                                         |
| ------------------------ | ---------------------------------- | --------- | ------------------------------------------------------------ |
| 操作数栈<br />局部变量表 | 0  :  iconst_1<br />1  :  istore_1 | int a = 1 | 将常量1放入操作数栈<br />将常量1弹出操作数栈，赋给局部变量表的a<br />程序计数器=1，标记执行引擎执行程序的步骤 |
| 方法出口                 | 3  : ireturn                       | return a  | 将a赋给调用该方法的方法栈帧的局部变量表中                    |
| 动态链接                 | reference                          |           | 局部变量表中对象的引用，指向堆中的变量地址                   |
|                          |                                    |           |                                                              |

元空间存储类的基本信息，堆中的类对象的头信息，存储指向元空间的类的hashcode。

对象头信息存储锁的状态（无锁态01,轻量级锁00,重量级锁10），分代年龄，线程ID

#### GC过程

1. 在Eden区创建对象
2. Eden区满后，触发minorGC，清除Eden区无效对象，将存活对象拷贝到From区，并将存活对象age+1
3. Eden区再次满后，重复步骤2，From区的无效对象清除，再将From区的存活对象拷贝到To区，将From区变成To区（空），To区变成From区
4. Eden区再次满后，重复步骤3，当对象的年龄达到15（默认）的时候，会将该对象拷贝到老年代。
5. 如果老年代满的话，执行FullGC，触发Stop The World。
6. 如果来年代无法放置对象的时候，会方法OOM

