# JDK源码解读

## 容器

1. ArrayList

   - 与LinkedList区别

     1. 随机访问
     2. 线程安全
     3. 占用空间
     4. add元素
     5. 数据结构

   - 扩容机制

     1. 构造函数（无参、初始容量、Collection）

     2. 是否扩容根据minCapacity与elementData.length、oldCapacity决定

     3. add元素时，假设MAX_ARRAY_SIZE = 20【实际值为Integer.MAX_VALUE - 8】

        - 添加第001个元素，oldCapacity = 0：minCapacity = 10 < MAX_ARRAY_SIZE，因此newCapacity = 10；

        - 添加第11个元素，oldCapacity = 10：minCapacity = 10 < newCapacity = 15 < MAX_ARRAY_SIZE，因此newCapacity = 15；

        - 添加第16个元素，oldCapacity = 15：minCapacity = 16 < newCapacity = 22，newCapacity > MAX_ARRAY_SIZE，

          因此再对minCapacity和MAX_ARRAY_SIZE进行比较

          1. minCapacity > MAX_ARRAY_SIZE：newCapacity = Integer.MAX_VALUE
          2. minCapacity < MAX_ARRAY_SIZE：newCapacity = MAX_ARRAY_SIZE

2. HashMap

   - hash算法

     ```java
     int h;
     /**
     key为null，则直接返回0；
     key.hashcode()的高16位与低16位进行异或计算
     得到的hash值需要对table.length - 1取余：hash & (length - 1)
     hash冲突：hash到了同一个bucket，比较key是否相同，相同则覆盖，不相同则拉链插入
     */
     return (key == null) ? 0 : (h = key.hashCode()) ^ (h >>> 16);
     ```

   - 扩容过程

     1. capacity = 2^n：***hash%length==hash&(length-1)的前提是 length 是 2 的 n 次方***

     2. table数组扩容：

        threshold = loadFactor * capacity；size()达到threshold则触发扩容

        - 无参构造函数：首次put会将table长度扩容为16
        - 有参构造函数：找到最接近的2^n
        - 达到threshold，table变为原来的两倍

     3. ***entry元素迁移***

        1. JDK1.7

        2. JDK1.8

           ![img](https://pic1.zhimg.com/v2-da2df9ad67181daa328bb09515c1e1c8_r.jpg)

           扩容后，n - 1的二进制表现为在高位多了个1，因此只需要看下原来entry的hash值的高一位是0还是1，就可以知道这个entry是在原来的bucket还是去oldIndex + oldSize的bucket了，通过尾插法插入到table里面

   - 红黑树

     https://github.com/6868986/liushixing/blob/main/tech/Java/Java.md#hashmap

3. 线程安全问题

   - HashMap

     1. 环形链表
     2. 数据覆盖
        - 空的bucket，线程1、2同时hash到这个bucket
        - 线程1先获取到CPU时间片，进行hash碰撞的判断，由于bucket为空，因此线程1判断无hash冲突，准备直接插入，此时线程1时间片耗尽被挂起
        - 线程2获取到bucket，也判断出了没有hash冲突，并完成了kv的插入，而后释放CPU
        - 线程1继续执行，由于已经判断过了没有hash冲突，因此不会拉链，而是直接插入bucket，这样就将线程2的数据覆盖掉了

   - HashTable

     通过synchronized锁住整个table

   - ConcurrentHashMap

     1. JDK1.7

        segment继承了ReentrantLock，最多有16个segment，仅有拉链法

     1. JDK1.8

        Node数组 + CAS + synchronized

4. 并发容器：BlockingQueue、ConcurrentLinkedQueue、CopyOnWriteArrayList

5. 阻塞队列BlockingQueue【ThreadPool】

   - 功能点：队列空时，获取元素的线程会等待队列变为非空；队列满时，存储元素的线程会等待队列可用

## 并发

1. 线程状态
   - 初始化——Running——Waiting/Timeout_waiting——Blocking——Terminated

   - Running在OS层面又分为就绪【***获取到锁等其他资源&等待CPU资源***】、运行

     thread.yield()释放CPU但不释放锁

     thread.sleep()释放CPU但不释放锁

     

   - Waiting是线程主动释放锁资源，等待其他线程的动作和唤醒

   - Blocking是线程进入同步方法（synchronized）时没有获取到锁资源，因此被动挂起在内存，释放CPU

2. volatile

3. synchronized

4. Lock

5. threadlocal

   一个thread对应一个threadlocalMap，一个map里面有多个threadlocal变量；简化线程内传参；

   key弱引用，value强引用，与thread生命周期相同，但是threadpool中的core thread的生命周期一般很长，因此core thread的threadlocalMap会一直占用较大内存

## JVM

1. **类文件**被**类加载器**加载到**运行时数据区**，

2. 方法区：

   - 存放信息：类信息、常量、静态变量
   - 垃圾回收：对常量池的回收和对类的卸载
   - 运行时常量池：

3. 堆&栈

   功能：栈用来存储局部变量（基本数据类型）&方法的调用链信息；堆用来存储对象和数组

   共享：栈内存线程私有；堆内存线程共享

   ERROR：SOF、OOM

   物理地址：栈连续，速度快；堆不连续，速度慢

4. GC Roots

   - 虚拟机栈、本地方法栈中的对象引用
   - 方法区中的常量对象引用、静态变量对象引用

5. 类的卸载

   - 类的实例被回收
   - 类加载器被回收
   - 类对象未被使用

6. 堆内存分配策略

   - 何为“大对象”？

     根据参数判断，一般是长字符串、大数组

   - 空间分配担保

   - Full GC触发时机

7. 对象的组成

   - 对象头
     1. markword
        - hashcode
        - GC分代年龄
        - 锁标志位
     2. 指向类信息的指针
     3. 数组长度
   - 实例数据
   - 对齐填充字节



























