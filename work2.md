
### 作业题：JVM 虚拟机论述题
#### 题目 01- 请你用自己的语言向我介绍 Java 运行时数据区（内存区域）

###### 堆、虚拟机栈、本地方法栈、方法区（永久代、元空间）、运行时常量池（字符串常量池）、直接内存
- 堆： JVM直接管理的内存区域，存放生成的对象
- 虚拟机栈：每一个线程都有一个虚拟机栈，而线程的每一个方法都有一个独立的栈帧，栈帧中有包括局部变量表，操作数栈，动态链接和方法返回地址
- 本地方法栈：本地方法，用native区分，C/C++代码，一般是jvm更底层的实现。
- 方法区： 方法区是逻辑概率，而永久代和元空间的落地实现，里面存放了类加载的元数据
- 运行时常量池：存在类加载的Class对象
- 字符串常量池：特殊的运行时常量词，一个字符串全局只有一个
- 直接内存：非堆内存和栈空间，nio就是直接作用的直接内存，少了JVM内存管理的步骤
######  为什么堆内存要分年轻代和老年代？
答：这设计到分代收集理论的三条假说，认为大多数对象都是朝生夕灭的，而熬过收集次数越多的对象越难消亡，例如局部对象和静态对象。基于三条假说，划分了年轻代和老年代，认为年轻代的对象应该是朝生夕灭的，而from区和to区来标记收集次数，达到阈值后进入老年代，认为该对象就是不易消亡的。

#### 题目 02- 描述一个 Java 对象的生命周期
###### 解释一个对象的创建过程

1. 检查类是否加载
2. 分配内存空间
3. 设置对象头信息并完成链接映射，class文件池，运行时常量池，对象头信息mark work包含锁信息,GC信息，hashcode等, 类型指针指向的是元空间的类元信息
4. 设置初始值，基础类型复制
5. 初始化方法开始，比如常见的构造函数开始

###### 解释一个对象的内存分配

1. 直接分配（如指针指向的是未分配内存的区域首地址，当大量对象需要创建时，就会出现竞争的情况，使用CAS自旋分配）
2. TLAB（本地线程缓存分配），每一个线程都预先分配了一段内存空间。如果内存不够，则使用CAS进行分配。

###### 解释一个对象的销毁过程
这涉及到引用计数和可达性分析，当对象循环引用时不适用引用计数，所以jvm采用可达性分析，当一个对象为不可达状态时，会被标记，在被真正的GC时，如果还是未被引用，则会被GC。

###### 对象的 2 种访问方式是什么？

1. 句柄 句柄池包含了对象的数据指针地址和对象类型数据的指针地址，当频繁GC时，只需要更改句柄池里对象的地址，无需修改对象所用的被引用地址，修改更快，访问慢一层。
2. 直接访问  直接指向对象的数据地址，对象的数据地址包含了对象类型的指针地址。 更快。

###### 为什么需要内存担保？
这让我想起了大对象直接进入老年代的问题，我原以为大对象和内存担保是有一个衡量机制的，比如空间不够用时新生对象的大小为多少直接进入老年代，否则进行内存担保。后来我发现内存担保是eden区在发生minor gc之后，s区无法放下eden存货对象时发生的内存担保，移入老年代。所以内存担保的机制就是为了防止出现这种OOM情况。
#### 题目 03- 垃圾收集算法有哪些？垃圾收集器有哪些？他们的特点是什么？
###### ParNew 收集器
年轻代并行收集器，复制算法，和CMS搭配
###### ParallelScavenge 收集器
jdk1.8默认年轻代收集器，复制算法，吞吐量优先
###### ParallelOld 收集器
jdk1.8默认老年代收集器，标记整理算法，吞吐量优先
###### CMS 收集器

* 老年代并行标记清除算法
* 容易产生内存碎片
* 低延时
* 有内存阈值的概率，达到了就会进行回收
* 2次标记依然会STW

###### G1 收集器
全局收集器，全能而不全优。

* 可设置的回收时间，不合理的设置会导致效率低下，比如设置过小导致回收效率不高，设置过高会加大停顿的时间
* 和CMS一样在最初标记和最终标记阶段都会短暂的STW
* G1虽然依然存在年轻代和老年代的概率，但是并没有将内存区域直接划分开
* 以Region作为独立的块，一般为1M，默认为堆的1/2000


###### ZGC 收集器

低延时的标记整理算法，基于Region，更先进的解决思维。