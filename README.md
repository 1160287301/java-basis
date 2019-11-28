# java基础知识

复习java基础知识的笔记   

#### 进程和线程:

    进程和线程主要区别在于他们是操作系统不同的资源管理方式.
  
    进程是程序的一次执行过程(运行中的程序),是系统运行程序的基本单位.
    一个进程至少包含一个线程(main),
    
    可以包含多个线程.(换言之,线程是进程内的执行单元)
  
    线程与进程相似,它是比进程更小的执行单位.一个进程在执行过程中可以产生多个线程.
    同类线程有共享的堆和方法区(jdk8之后的元空间(MetaSpace)),
    每个线程又有自己的程序计数器,虚拟机栈,本地方法栈.
    系统在各个线程之间的切换工作要比进程负担低,因此线程又被称为轻量级进程

#### 线程的几种状态:(见:jdk Thread类源码中的state枚举类)
      NEW,RUNNABLE,BLOCKED,WAITING,TIMED_WAITING,TERMINATED

#### 并发和并行:
    并发是指计算机在同一时间段内处理多任务的执行. 
    如我和小明同时访问淘宝网站,那么淘宝服务器就同时处理我和小明的访问请求
    
    并行是指多任务同时执行,但是任务之间没有任何关系,不涉及共享资源.
    比如我一边看电视一边喝水,2件事互不干扰

#### 公平锁: 
      指根据线程在队列中的优先级获取锁,比如线程优先加入阻塞队列,那么线程就优先获取锁

#### 非公平锁:
      指在获取锁的时候,每个线程都会去争抢,并且都有机会获取到锁,无关线程的优先级 

#### 可重入锁:
       一个线程获取到锁后,如果继续遇到被相同锁修饰的资源或方法,那么可以继续获取该锁.
       对synchronized来说,每个锁都有线程持有者和锁计数器,每次线程获取到锁,会记录下
       改线程,并且锁的计数器就+1,当线程退出synchronized代码块的时候,线程计数就会-1,
       当锁计数为0的时候,就释放锁.      
     
#### 独占锁:
      　锁一次只能被一个线程占有使用,Synchronized和ReetrantLock都是独占锁
    
#### 共享锁:
       锁可以被多个线程持有,对于ReentrantReadWriteLock而言,它的读锁是共享锁,
       写锁是独占锁      

#### 偏向锁:
      偏向锁会偏向第一个获取它的线程.它适合只有一个线程竞争共享资源的情况,
      当一个线程获取偏向锁后,如果再次进入同步代码的时候,只需判断该
      对象的对象头的Mark Word的偏向锁标识是否指向它.
   
####　轻量级锁:
     偏向锁适用于单线程竞争资源,如果遇到其他线程获取该锁对象,
     那么偏向锁将升级为轻量级锁.
     如果多线程获取同一个锁,但是它们之间可能并没有竞争,这就是
     轻量级锁的好处,它使用CAS操作,无需向操作系统申请
     互斥量(Synchronized是互斥锁),可以避免从用户态到内核态的
     转换.　      
     

#### 自旋锁:
     指当锁被获取后,其他线程并不会停止获取,而是一直去尝试获取.这样做的好处是减少上下文开销,
     缺点是增加cpu消耗. 
     CAS底层就使用了自旋操作(不是自旋锁,而是如果预期值和原值比较不成功就会一直比较) 
            
#### synchronized

##### 谈谈 synchronized 关键字
     synchronized关键字是jdk提供的jvm层面的同步锁.
     它解决的是多线程之间访问共享资源的同步性,它保证了
     在被它修饰的方法或代码块只有一个线程执行.
     
     java6之前的synchronized属于重量锁,性能极差.
     它的原理是基于操作系统的Mutex Lock互斥量实现的
     因为java线程是映射到操作系统的线程之上的,所以
     暂停或唤醒线程都需要操作系统帮忙,而操作系统实现
     线程之间的切换需要从用户态转换为内核态,这段
     转换时间消耗较长.
     
     java6之后jvm团队对synchronized做出了非常大的
     优化.(下面写)
     
##### synchronized 使用方法
  ###### 1. 修饰静态方法
       修饰静态方法是给类加锁,会作用于所有对象,因为静态方法属于类,
       而不属于对象,不管有多少个对象,static方法都是共享的.
       
  ###### 2. 修饰实例方法
       修饰实例方法是给对象加锁,会作用于当前类的实例对象.      
 
  ###### 3. 修饰代码块
       修饰代码块,根据代码块给定的对象加锁,线程想要进入代码块,只有获取
       指定的对象的锁.

#####　Synchronized和ReentrantLock的区别
 ######　1. synchronized基于jvm层面,ReentrantLock基于java层面.
  
 ###### 2.ReentrantLock提供了更高级的功能
         1: 相比于Synchronized,ReentrantLock提供了非公平锁.
            而Synchronized只能是非公平锁
         2: jdk提供了wait/notify机制解决线程间通信的问题.
            而ReentrantLock提供了更为灵活的Condition
            接口,进行线程之间的调度.
         ....   
 ######　锁消除
       当编译器运行时,检测到一些不可能被竞争但是被加锁的资源,会
       消除锁.
       
 ######　锁粗化 
        当jvm检测到一连串的零碎的操作都对一个对象加锁的
        时候,会把锁的同步范围扩大到整个操作序列.
    ````java
        StringBuffer stbf = new StringBuffer();
        stbf.appedn("a");
        stbf.appedn("b");
        stbf.appedn("c");
        上面三个操作都是synchronized同步的方法,
        它们每一次操作都要加锁,但是由于都是stbf一个对象,
        于是在这3个操作前后只需要加一次锁就行了.
    ````
       

#### volatile:
     Volatile是JVM提供的轻量级的同步机制
     
  ##### 1: volatile保证内存可见性
        JMM内存模型实现总是线程从主内存(共享内存)读取数据,线程把主存的变量存储到本地,
        在本地进行修改,然后写回主内存,而不是直接在主存中进行操作.
        
        那么这就可能造成可见性问题:假设2个线程从主存读取同一个变量,一个线程修改了它的本地变量,
        并写回了主存,但是另一个线程仍然使用的是之前的值,这就造成了数据的不一致.
        volatile关键字修饰的变量就解决了这个问题:被volatile修饰的变量要求线程使用时,
        都从主内存中读取,而不使用本地的拷贝.
          
  ##### 2:　volatile不保证原子性
          原子性指一个操作的完整性,它是不可分割的,要么同时成功执行,要么同时失败回滚. 
          当多个线程同时对volatie关键字修饰的变量进行非原子性操作(++,--)的时候,
          变量可能会被多++一次,少++一次,多--一次,少--一次.
          volatile并不能保证操作的原子性,最后得到的结果可能不尽人意
          
  ##### 如何解决原子性:
        1: 最简单的方法就是加锁
        2: 使用CAS原子类
  
  ##### 3:　volatile禁止指令重排序   
        指令重排序是编译器和cpu为了尽可能高效的执行程序而采取的一种优化手段,
        它会导致程序实际执行的顺序和代码的顺序不一定相符,
        而volatil就是在执行的代码前后加入屏障,使cpu在执行时
        无法重排序,就按代码的顺序执行
  
#### CAS:
      CAS:CompareAndSet,比较并交换,它将指定内存位置的值与给定值进行比较,
      如果两个值相等,就将内存位置的值
      改为给定值.CAS涉及3个元素:内存地址,期盼值和目标值,
      只有内存地址对应的值和期望的值相同时,才把内存地址对应的值修改为目标值.
 ##### CAS在JAVA中的底层实现(Atomic原子类实现)  
      
 ######     1:Unsafe类:
    Unsafe类是CAS的核心类,由jdk自动加载,它的方法都是native方法.
    因为Java无法像c/c++一样直接使用底层指针操作对象
    内存,Unsafe类的作用就是专门解决这个问题,它可以直接操作对象在内存中的地址.
    具体步骤是:首先获取当前Atomic对象的value在内存中真实的偏移地址,再根据这个偏移地址
    获取value的真实值,然后再重复这个步骤,把两次获取到的值进行比较,
    如果比较成功,则继续操作,否则继续循环比较.
    而获取value在内存中真实的偏移地址和比较设置值方法都是native的.
    
 ######    2: volatile:
    Atomic原子类内部的value值是volatile修饰的,这就保证了value的可见性.
 
 ##### CAS的缺点:
 
 ###### 1: 循环时间开销大
          如果预期的值和当前值比较不成功,那么CAS会一直进行循环.如果长时间比较不成功
          就会一直循环,导致CPU开销过大    
           
 ###### 2:　只能保证一个共享变量的原子操作
          在获取内存地址和设置值的时候都是当前Atomic对象的volatile值,如果要保证多个共享变量,
          那么可以通过加锁来保证线程安全和原子性.
          
 ###### 3: ABA问题
           尽管一个线程CAS操作成功,但并不代表这个过程就是没有问题的.
           假设2个线程读取了主内存中的共享变量,如果一个线程对主内存中的值进行了修改后,
           又把新值改回了原来的值,而此时另一个线程进行CAS操作,发现原值和期盼的值是
           一样的,就顺利的进行了CAS操作.这就是CAS引发的ABA问题.
           
 ###### 4:　解决ABA问题
            juc的atomic包下提供了AtomicStampedReference这个类来解决CAS的原子引用
            更新的ABA问题,它相较于普通的Atomic原子类多增加了一个版本号的字段,
            每次修改引用就更新版本号,这样即使发生ABA问题,也能通过版本号判断引用
            是否被修改过了.                 
                     
#### 线程池的好处:
     池化技术屡见不鲜:数据库连接池,Http连接池,线程池都是这种思想．
     池化技术的好处非常明显,以往单个的new Thread,不易于线程之间的管理,
     而池化技术把所有线程都放在一个池子里,要用就取出,用完就回收,这样非常易于管理,
     并且可以降低资源的消耗,使线程可以重复利用,提高任务的响应速度,当任务到来时,就可以
     处理.

#### 线程池构造参数:
````
 ThreadPoolExecutor
(int corePoolSize,
 int maximumPoolSize, 
 long keepAliveTime,
 TimeUnit unit,
 BlockingQueue<Runnable> workQueue,
 ThreadFactory threadFactory,
 RejectedExecutionHandler handler)
````

##### corePoolSize:
      线程池的核心线程数(常驻线程数),也就是线程池的最小线程数,这部分线程不会被回收.
      
##### maximumPoolSize:
      线程池最大线程数,线程池中允许同时执行的最大线程数量
      
##### keepAliveTime:
      当线程池中的数量超过 corePoolSize(最小线程池数量),并且此时没有新的任务执行,那么
      会保持    keepAliveTime 的时间才会回收线程
      
##### unit:
      keepAliveTime的时间单位
  
##### workQueue:
      任务队列,当有新任务来临时,如果核心线程数corePoolSize被用完,此时如果workQueue有空间,
      任务就会被放入workQueue            

##### threadFactory:
      创建工作线程的工厂,也就是如何创建线程的,一般采用默认的

##### handler:
      拒绝策略. 如果线程池陷入一种极端情况:工作队列满了,无法再容纳新的任务,最大工作线程也到达限制了,
      此时线程池如何处理这种极端情况.
      ThreadPoolExecutor 提供了四种策略:
   ###### AbortPolicy(是线程池的默认拒绝策略): 
         如果还有新任务到来,那么拒绝,并抛出RejectedExecutionException异常
   ###### CallerRunsPolicy: 
         这种策略不会拒绝执行新任务,但是由发出任务的线程执行,也就是说当线程池无法
         执行新任务的时候,就由请求线程自己执行任务
   ###### DiscardPolicy:
          这种策略会拒绝新任务,但是不会抛出异常
   ###### DiscardOldestPolicy:
         这种策略不会拒绝策略,他会抛弃队列中等待最久那个任务,来执行新任务      
 
##### 阿里巴巴开发者手册不建议开发者使用Executors创建线程池:
      newFixedThreadPool和newSingleThreadExecutor:
      会创建固定数量线程的线程池和单线程线程池,尽管二者线程池数量有限,
      但是它会创建长度为Integer.MAX_VALUE长度的阻塞队列,这样可能会导致阻塞队列
      的任务过多而导致OOM(OutOfMemoryError)
      newCachedThreadPool和newScheduledThreadPool:
      会创建缓存线程池和周期任务线程池,二者线程池的最大线程为Integer.MAX_VALUE,
      也可能会导致OOM   
    
#### JVM运行时内存分区:.
     JDK8之前:线程私有的部分有:程序计数器(PC寄存器),JAVA虚拟机栈,
     本地方法栈(native),线程共享部分有: GC堆,方法区(永久代包含运行时常量池)
     JDK8之后:线程私有的部分不变, 线程共享部分的方法区改为了元空间(MetaSpace),
     运行时常量池也移动到了 heap空间
     
##### 程序计数器:
      程序计数器又称PC寄存器,它是记录着当前线程执行的字节码的行号指示器.
      cpu是通过时间片轮换制度执行每个线程的任务,而在多个线程之间切换时,
      程序计数器就记录当前线程执行的位置,当cpu又开始执行此线程的时候,它需要知道
      上次运行的位置,那么就是通过程序计数器直到上次字节码的执行位置,
      所以每个线程都会有属于自己的程序计数器
      
##### Java虚拟机栈:
       Java虚拟机栈描述的是方法执行的内存模型,每个方法在执行时会在虚拟机栈中创建一个
       栈帧,每个方法的执行,就对应着栈帧在虚拟机栈中的入栈和出栈.
       栈帧由局部变量表,操作数栈,方法出口,动态链接等数据组成.
    
#### Java虚拟机的错误:    
  
   ##### StackOverflowError:
          当Java虚拟机栈无法动态扩容的时候,当前线程执行或请求的栈的大小超过了Java
          虚拟机栈的最大空间(比如递归嵌套调用太深),那么抛出StackOverflowError错误
           
   ##### OutOfMemoryError:
           1: 当Java虚拟机栈允许动态扩容的时候,当前虚拟机栈执行请求的栈的大小
              仍然超过了扩容之后的最大空间,无法继续为栈分配空间(堆内存分配空间过小),
              那么抛出OutOfMemoryError错误
             
           2: Java堆存放对象实例,当需要为对象分配内存时,而堆空间大小已经达到最大值,
              无法为对象实例继续分配空间时,抛出 OutOfMemoryError错误
              
           3: GC时间过长可能会抛出OutOfMemoryError.也大部分的时间都用在GC上了,并
             且每次回收都只回收一点内存,而清理的一点内存很快又被消耗殆尽,这样就恶性循环,
             不断长时间的GC,就可能抛出GC Overhead limit,但是这点在我的机器上测试不出来,
             可能与jdk版本或gc收集器或Xmx分配内存的大小有关,一直抛出的是java heap sapce
    
           4: 因为jvm内存依赖于本地物理内存,那么给程序分配超额的物理内存,而堆内存充足,
              那么GC就不会执行回收,DirectByteBuffer对象就不会被回收,如果继续分配
              直接物理内存,那么可能会出现DirectBufferMemoryError
                
           5: 一个应用程序可以创建的线程数有限,如果创建的线程的数量达到相应平台的上限,
              那么可能会出现 unable to create new native thread 错误   
              
           6:jdk8之后的Metaspace元空间也有可能抛出OOM,Metasapce受限于物理内存,它存储
             着类的元信息,当Metaspace里的类的信息过多时,Metaspace可能会发生OOM.
             这里是可以使用cglb的字节码生成类的技术测试的.   
   
#### 虚拟机栈栈的动态扩容:
           上面说过如果当虚拟机栈允许动态扩容,当动态扩容的空间都不够用的时候,就抛出OutOfMemory异常.
           虚拟机栈的动态扩容是指在栈空间不够用的时候,自动增加栈的内存大小.
           
           动态扩容栈有2种方法:
           Stack Copying: 就是分配一个更大的栈空间,把原来的栈拷贝到新栈空间去.
           Segmented Stack: 可以理解为一个双向链表把多个栈链接起来,一开始只分配一个栈,当
                            这个空间不够时.再分配一个栈空间,用链表链接起来.
           
  ##### 局部变量表:存放编译期可知的各种数据类型(常见的8大数据类型)和引用类型(reference,可以是指向对象的指针,也可以是句柄)
  
  ##### 操作数栈:与局部变量类似,它也可以存放任意类型的数据,但它是作为数据在计算时的临时存储空间,入站和出栈
  
  ##### 动态链接:因为每个方法在运行时都会创建相应的栈帧,那么栈帧也会保存一份方法的引用,栈帧保存方法的引用是为了支持方法调用过程中的动态链接.
  
      动态链接是指将符号引用转为直接引用的过程.
      如果方法调用另一个方法或者调用另一个类的成员变量就需要知道其名字,
      符号引用就相当于名字,运行时就将这个名字解析成相应的直接引用.
  
  ###### 方法出口:方法执行后,有2种方式退出: 1: 执行方法的过程中遇到了异常; 2: 遇到正常的返回字节码指令.
        无论何种方式退出,在方法退出后,都需要回到方法被调用时的位置,
        方法在返回时就需要在栈帧中保存一些信息,用来帮助它恢复上层方法的执行状态.
##### 本地方法栈:
      与Java虚拟机栈类似,Java虚拟机栈是对Java方法执行的描述,
      但是本地方法栈是对jvm的native方法描述,也就是第三方c/c++
      编写的方法,本地方法栈也有相应的 局部变量表,操作数栈,方法出口,动态链接
   
##### 堆:
      堆是jvm内存区域中最大的一块区域.
      
      堆区的作用是为对象分配内存,存储他们,并负责回收它们之中无用的对象.
      堆可以分为新生代和老年代,新生代占堆区的1/3,老年代占堆区的2/3.
      新生代包括eden,from survivor, to survivor三个空间
      其中 eden空间最大占新生代的80%内存,from和to都是1:1.
      
      新生代是GC发生最频繁的区域.
      发生在新生代的GC被称为MinorGC,老年代的GC被称为Major GC / Full GC
      
   
     因为老年代的空间比新生代的空间大,所以通常老年代的GC时间会比新生代的GC时间慢很多.
     对象优先在eden区域分配,当eden区域没有空间时,虚拟机就会发起一次Minor GC
    
#### 判断对象存活的方法

#####  1: 引用计数法:
          给每个对象添加一个引用计数器,当对象被引用的时候,引用计数器就+1,当引用失效时,引用计数器
          就-1,直到引用计数器为0,就代表对象不再被引用.
          引用计数的主要缺陷是很难解决循环引用的问题:也就是当2个对象互相引用的时候,除了彼此,
          就没有其他地方引用这2个对象,那么他们的引用计数都为1,就无法被回收
#####  2: 可达性算法:
          通过一系列被称为GC ROOTS的对象节点往下搜索,节点走过的地方被称为引用链,
          如果一个对象不被任何引用链走过,那么称
          此对象不可达.
          
 ###### 什么是GC Root
        
        上面说通过GC Root对象搜索引用链,那么GC Root对象是什么对象,或者什么样的对象是GC Root对象.
        
        可以作为GC Root对象的有: 
        1: 虚拟机栈和本地方法栈区(native)的引用对象;
        2: 堆区里的静态变量引用的对象;
        3: 堆区里的常量池的常量引用的对象         
        
#### 垃圾回收算法：

  ##### 复制算法:
         将内存分为2块大小的内存空间,每次使用其中一块空间,当一块使用完后,
         将还存活的对象复制到另一块空间去,然后清楚已经使用过的空间.
         根据GC角度来说就是:在新生代的eden空间和From Survivor空间经历过MinorGC后,
         仍然存活的对象采用复制算法复制到To Survivor,
         并将To Survivor(其实就是空间不空闲的那块Survivor区域)的对象年龄+1,
         默认对象年龄撑过15岁,那么进入老年代.
   
         复制算法的缺点就是太耗空间内存.
         
  ##### 标记-清除算法:
         标记出所有仍然存活(可达)的对象,然后统一回收所有未被标记(不可达)的对象.
         
         标记清除算法的最大缺点就是会造成不连续的内存空间,
         也就是内存碎片,因为对象在内存中的分布是不均匀的.
      
  ##### 标记-整理算法:
         是对标记-清除算法做出的改进,标记整理算法也是首先标记出所有仍然存活的对象,
         不同的是,它会使所有仍然存活的对象向空间的一段移动,然后对其它端进行清理.
         
         此算法虽然不会产生内存碎片,但是它的效率会比标记清楚算法慢
         
  ##### 分代收集算法:
         分代收集算法不是一种具体的收集算法.
         因为堆是分为新生代(包括eden空间,from survivor,to survivor)
         老年代的,分代收集算法就是在不同的分代空间采用不同的垃圾回收算法:
         如复制算法应用于新生代,标记清除和标记整理应用于老年代
         
#### 垃圾回收器
   
##### Serial 串行收集器:
       它为单线程环境设计,并只使用一个线程进行垃圾回收,会暂停所有用户线程,
       不适用于并发环境.
       但是它在单线程环境中是很高效的,因为它没有多线程切换的消耗     
       
       新生代采用复制算法,老年代采用标记-整理算法. 
       
#####  Serial Old 串行收集器:
       它是 Serial收集器的老年代使用的GC收集器,同样是一个单线程的垃圾收集器. 
       它除了与Serial串行收集器搭配使用,还可作为ParNew + CMS 的备用收集器.   
           
````java
   
/** 开启串行收集器使用 -XX:+UseSerialGC , 这样默认年轻代使用 Serial 收集器,
  * 老年代使用 Serial Old 收集器. 
  *
  * 设置VM参数:
  *
  * -XX:+Xlogs:gc* 打印gc信息
  * -XX:+PrintCommandLineFlags  打印java版本信息
  * -XX:+UseSerialGC 使用串行GC
  */                      

//如果程序正常运行,日志会显示 :
// 新生代的信息为:  def new generation.....
// 老年代的信息为:  tenured generation.....

````           
            
            
##### Parallel Scavenge 并行收集器
       多个垃圾回收线程一起工作,但是仍然会暂停所有用户线程,但是暂停时间会比
       Serial垃圾回收器短,也不适用于并发环境. 
       新生代采用复制算法,老年代采用标记-整理算法.
    
##### Parallel Old 并行收集器
      它是 Parallel Scavenge 的老年代版本,同样是一个并行收集器,使用标记-整理算法.       
  
````java
    /**
     * 
     * 设置 Parallel Scavenge 收集器的参数:
     *
     * -XX:+UseParallelGC
     * 
     * ParallelGC老年代默认使用的 Parallel Old GC 回收器
     * 
     * 并行收集器打印的年轻代的信息为:
     *  PSYoungGen ....
     *  
     *  老年代的信息为:
     *  ParOldGen ....
     * 
     */
````        
        
        
##### ParNew 收集器
      它就是多线程版的Serial收集器,它和 Parallel Scavenge 并行收集器相似,
      同样是并行收集器,他可以和CMS收集器配合工作.
      当使用ParNew收集器时,老年代默认使用 Serial Old单线程收集器
         
      新生代采用复制算法,老年代采用标记-整理算法.
````java
     /**
       * 
       * 设置ParNewGC回收器的参数为:
       * -XX:+UseParNewGC
       * 
       * 需要注意的是jdk10以后就没有ParNewGC回收器了,我使用的是11,
       * 所以在我的机器上测试不出来．
       */
````           
        
##### CMS 并发标记清除收集器
      Concurrent Mark Sweep,并发标记-清除垃圾回收器,是第一款真正意义上的垃圾回收器,
      见名知意,使用的是标记-清除回收算法.它允许垃圾回收线程和用户线程同时工作.
      但是它的缺点也很明显就是标记-清除算法的缺点:会产生不连续的内存碎片,而且并发
      执行用户线程和收集线程,对cpu消耗较大.
      
 ````java
    /**
     *
     * 设置 CMS 收集器参数:
     * -XX:+UseConcMarkSweepGC
     *
     * 使用ConcMarkSweepGC收集器后,它的年轻代使用的是:
     * ParNew收集器.
     *
     * 当ConcMarkSweepGC收集器出现异常时,会将CMS替换成Serial Old收集器
     *
     * CMS回收分为4个阶段:
     *
     * 初始标记:    (Stop the world 暂停用户线程)
     * 标记与GC Root直接可达的对象.      
     *
     * 并发标记:    (并发的,不暂停用户线程)
     * 从第一步标记的可达的对象开始,并发的标记所有可达的对象 
     *
     * 重新标记:    (Stop the world 暂停用户线程)
     * 在第二部的并发标记阶段,由于程序运行导致对象间引用的关系发生变化,就需要重新标记
     *
     * 并发清除:     (并发的,不暂停用户线程)
     * 这个阶段不暂停用户线程,并且并发的去清除未被标记的对象
     * 
     */
````
##### G1 收集器            
      jdk9开始默认使用的垃圾回收器,它可以独立的管理整个堆区的
      垃圾回收,不需要配合其他收集器使用.
      它从整体看是使用标记-整理算法,从局部看使用复制算法,
      因此它不会产生内存碎片.其他收集器包括CMS收集器,在执行过程中,
      都会有停顿,而G1收集器不仅追求最短的停顿,而且可以预测停顿时间.
      G1从宏观上不再分为年轻代和老年代的概念,而是将内存分为一个个Region分区.
      但是在逻辑上还是保留了分代的概念,这些Region分区就随着G1在运行中不断
      的变化着分代.
      有的Region是Eden区,有的Region是Suvivor区,有的Region是Old区,
      有的Region是存储着大对象(Humongons)的区域
````java

    /**
     *
     * 因为我的机器的jdk版本是11,所以无需指定垃圾回收器
     * 指定G1回收器的参数是: -XX:+UseG1GC
     *
     * G1回收和CMS相似分4个阶段,但和CMS不同的是,G1的所有阶段都不需要停顿:
     * 1:初始标记:
     *   标记所有与GC Root直接可达的对象
     *
     * 2:并发标记
     * 　从第一个阶段标记的对象开始,trace标记
     *
     * 4:重新标记
     * 　在第二步并发标记的阶段,由于程序执行,导致被标记对象之间的引用关系发生变化,所以需要重新调整标记
     *
     * 5:筛选回收:
     *  和CMS的并发回收不一样,这里G1会在较短时间内,优先筛选出Region较大的区域进行回收,
     *  这样可以保证在有限的时间内获得最大的回收率.
     *
     */
````   
         
#### 引用类型    

  ##### 强引用:
        一般常用的new方式创建对象,创建的就是强引用. 只要强引用存在,
        垃圾回收器就不会回收.        
  
  ##### 软引用
         SoftReference , 非必须引用,如果内存足够或正常,就不回收,但是当
         内存不够,快发生OOM的时候就回收掉软引用对象.
  
  ##### 弱引用
        WeakReference , 对于弱引用的对象来说,只要垃圾回收器开始回收,
        无论空间是否充足,都回收弱引用的对象.
        
  ##### 虚引用(幽灵引用)
        和其他几种引用不同,虚引用不会决定对象生命周期,垃圾回收时,无法通过虚引用获取对象
        值.虚引用在任何时候都可能被垃圾回收掉.虚引用必须和引用队列(ReferenceQueue)使用.   
      
  ##### 软引用,弱引用,虚引用在被GC前会被加入到与其关联的引用队列中.     
         
#### 对象在内存中的布局
      对象在内存中的布局虽然有: 对象头,实例数据,对齐填充三部分组成.
      但是实例数据和对齐填充是不固定的,只有对象头是无论如何都会存在的.
    
  ##### 对象头:
        对象头可以分为2部分数据组成.
         (如果是数组,对象头还会保存数组长度)
  ###### 1:Mark Word:
           Mark Word存储着对象运行时的自身的数据:
           1. 哈希码(hashcode)
           2. GC分代年龄(因为它的大小为4bit,所以大小为
              2^4, 0 - 15,15岁后就会进入老年代)
           3. 线程持有的锁 (每次锁被获取,线程就会记录下锁
              持有的线程,线程也保存它获取到的锁)
           4. 偏向线程标识
           5. 偏向时间戳      
 
  ###### 2:Class Pointer:
           Class Pointer存储对象类的元数据的地址,
           用于判断对象属于那个类.它就是一个指向类的指针.
           
           
  ##### 实例数据:
        实例数据存储着对象在程序中被定义的各个字段的数据,也就是对象的字段的
        数据,继承父类的和在子类中定义的都会被记录下来.其中相同宽度的
        数据会被分配在一起,方便取数据.
        如果对象没有字段,也就没有数据.所以这就是我说它是不固定的原因.
        
  ##### 对齐填充
        Java对象的小必须是8的倍数,像13,15这种非8的倍数的对象的大小,不足
        或多余的部分就要使用对齐填充数据补齐.
        如果Java对象大小正好是8的倍数,那么就无需对齐填充数据
        所以我说它是不固定的.　　
      
              
    
      

   
  
 
  
