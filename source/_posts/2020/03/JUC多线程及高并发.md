---
title: JUC多线程及高并发
author: Ambi
avatar: 'https://cdn.jsdelivr.net/gh/AmbitionLover/cdn/img/custom/avatar.jpg'
authorLink: 'https://sakura.ambitlu.work/'
authorAbout: 一个好奇的人
authorDesc: 一个好奇的人
comments: true
photos: 'https://cdn.jsdelivr.net/gh/honjun/cdn@1.4/img/banner/client.jpg'
categories: 技术
keywords: volatile
abbrlink: 72d52ff4
date: 2020-03-26 10:34:03
tags:
description:
---

## 对volatile的理解
### 1.volatile是java虚拟机提供得轻量级同步机制
保证可见性、不保证原子性、禁止指令重排
### 2.谈谈JMM
JMM（Java内存模型，简称JMM）本身是一种抽象的概念**并不真实存在**
，它描述的是一组规则或规范，通过这组规范定义了程序中各个变量（包括实例字段，静态字段和构成数组对象的元素）的访问方式。

JMM关于同步的规定：
1. 线程解锁前，必须把共享变量的值刷新回主内存。
2. 线程加锁前，必须读取主内存的最新值到自己的工作内存。
3. 加锁解锁是同一把锁。

由于JVM运行程序的实体是线程，而每个线程创建时JVM都会为其创建一个工作内存（有些地方称为栈空间），工作内存是每个线程的私有数据区域，而Java内存模型中规定所有变量都存储到
主内存，主内存是共享内存区域，所有线程都可以访问，==但线程对变量的操作（读取、复制等）必须在工作内存中进行，首先要将变量从主内存拷贝到自己的工作内存空间，然后对变量进行操作，操作完成后再将变量写回主内存==，不能直接操作主内存中的变量，各个线程中的工作内存中存储着主内存中的变量副本拷贝
，因此不同的线程间无法访问对方的工作内存，线程间的通信（传值）必须通过主内存来完成，其简要访问过程如下图：

![](20200326105952.png)

#### 2.1可见性
通过前面对JMM的介绍，我们知道
各个线程对主内存中共享变量的操作都是各个线程各自拷贝到自己的工作内存进行操作后再写回主内存中的。

这就可能存在一个线程A修改了共享变量X的值但还未写回主内存时，另一个线程B又对准内存中同一个共享变量X进行操作，但此时A线程工作内存中共享变量X对线程B来说并不是可见，这种工作内存与主内存同步存在延迟现象就造成了可见性问题。

案例：Volatile保证可见性
``` java
class MyData{
    volatile int number = 0;    //设置为可见。试着不加volatile运行看看。
    void addTo60(){
        this.number = 60;
    }
}

public class VolatileDemo {
    public static void main(String[] args) {
        MyData myData = new MyData();
        new Thread(() -> {
            System.out.println(Thread.currentThread().getName()+"\t come in" );
            try{
                TimeUnit.SECONDS.sleep( 3 );} catch (InterruptedException e) {e.printStackTrace();}
            myData.addTo60();
            System.out.println(Thread.currentThread().getName()+"\t  value:" + myData.number);
        },"AAA").start();
        while (myData.number == 0){

        }
        System.out.println("value : "+myData.number);
    }
}
```

#### 2.2原子性

案例：volatile不保证原子性

```java
class MyData{
    volatile int number = 0;    //volatile不保证原子性
    void addPlusPlus(){ //加synchronized可以保障原子，不建议使用。杀鸡焉用牛刀。
        number++;
    }
    //以下保证原子性
    AtomicInteger atomicInteger = new AtomicInteger();
    void addAtomic(){
        atomicInteger.getAndIncrement();
    }
}

public class VolatileDemo {
    public static void main(String[] args) {
        MyData myData = new MyData();
        //20个线程进行计算
        for (int i = 1;i <= 20; i++){
            new Thread(() -> {
                for (int j = 0; j < 1000; j++) {
                    myData.addPlusPlus();
                    myData.addAtomic();
                }
            },String.valueOf(i)).start();
        }//按理说是1000*20 = 20000

        //需要等待上面20个线程全部计算完成后，在用main线程取得最后结看看。
        while (Thread.activeCount() > 2){   //后台至少两个线程，mian和Gc线程
            Thread.yield();
        }

        System.out.println(Thread.currentThread().getName()+"\t int Type finally number : "+myData.number);
        System.out.println(Thread.currentThread().getName()+"\t Atomic Type finally number : "+myData.atomicInteger);
    }
}
```
#### 2.4有序性
计算机在执行程序时，为了提高性能，编译器和处理器常常会对指令做重排，一般分一下3种：


1. 源代码->编译器优化的重排->指令并行的重排->内存系统的重排->最终执行的指令
2. 单线程环境里面确保程序最终执行结果和代码顺序执行的结果一致。
3. 处理器在进行重排序时必须考虑指令之间的数据依赖性。

多线程环境中线程交替执行，由于编译器优化重排的存在，两个线程中使用的变量能否保证一致性是无法确定的，结果无法预测。

#####  小总结
volatile实现禁止指令重排优化，从而避免多线程环境下程序出现乱序执行的现象。


先了解一个概念，内存屏障又称内存栅栏，是一个CPU指令，它的作用有两个：
1. 是保证特定操作的执行顺序
2. 是保证某些变量的内存可见性（利用该特性实现volatile的内存可见性）

由于编译器和处理器都能执行指令重排优化。如果在指令间插入一条Memory Barrier则告诉编译器和CPU，不管什么指令都不能和这条Memory Barrier指令重新排序，也就是说
通过插入内存屏障禁止在内存屏障前后的指令执行重排序优化
。内存屏障另外一个作用是强制刷出各种CPU的缓存数据，因此任何CPU上的线程都能读取到这些数据的最新版本。


#### 3.1 单例模式在多线程下可能存在安全问题


```java
public class SingletonDemo {

    public static SingletonDemo instance  = null;

    public SingletonDemo(){
        System.out.println(Thread.currentThread().getName()+"\t 我是构造函数");
    }
    
    public static SingletonDemo getInstance() {
        if (instance == null){
            instance = new SingletonDemo();
        }
        return instance;
    }

    public static void main(String[] args) {
//        System.out.println(SingletonDemo.getInstance() == SingletonDemo.getInstance());
//        System.out.println(SingletonDemo.getInstance() == SingletonDemo.getInstance());
//        System.out.println(SingletonDemo.getInstance() == SingletonDemo.getInstance());
        for(int i=1;i<=10;i++){
            new Thread(()->{
                SingletonDemo.getInstance();
            },String.valueOf(i)).start();
        }
    }
    输出：
    <!--10	 我是构造函数-->
    <!--9	 我是构造函数-->
    <!--2	 我是构造函数-->
    <!--4	 我是构造函数-->
    <!--5	 我是构造函数-->
    <!--7	 我是构造函数-->
    <!--3	 我是构造函数-->
    <!--8	 我是构造函数-->
    <!--6	 我是构造函数-->
    <!--1	 我是构造函数-->
}

```

将上面代码的getInstance方法换成下面方法。采用双端加锁形式：DCL(Double check lock)
```java
public static SingletonDemo getInstance() {
        if (instance == null){
            synchronized (SingletonDemo.class){
                if (instance == null){
                    instance = new SingletonDemo();
                }
            }
        }
        return instance;
    }
```
这个时候运行出现结果


```
1	 我是构造函数
```
#### 3.2 单例模式volatile分析
DCL（双端检锁）机制不一定线程安全，原因是有指令重排序的存在，加入volatile可以禁止指令重排。

原因在于某一个线程执行到第一个检测，读取到的instance不为null时，instance的引用对象**可能没有完成初始化**

```
memory = allocate();    //1.分配对象内存空间
instance(memory);       //2.初始化对象
instance = memory;      //3.设置instance指向刚分配得内存地址、此时instance ！=null

步骤2和步骤3不存在数据依赖关系。而且无论重排前后执行结果都是在单线程中并没有改变。因此这种重排优化是允许得。
memory = allocate()  //1.分配对象内存空间
instance = memory;  //3.设置instance指向刚分配得内存地址、此时instance ！=null
instance(memory);    //2.初始化对象
```

指令重排只会保证串行语义的执行一致性（单线程），但并不会关心多线程间的语义一致性。

所以当一条线程访问instance不为null时，由于instance实例未必已初始化完成，也就造成了线程安全问题。

解决方法：
```java
public static volatile SingletonDemo instance  = null;
```
## 谈谈CAS
### CAS是什么?
CAS : compareAndSet //比较并交换
```java
public class CASDemo {
    public static void main(String[] args) {
        AtomicInteger atomicInteger = new AtomicInteger(9);
        System.out.println(atomicInteger.compareAndSet(9, 4)+"\t"+atomicInteger.get());
        System.out.println(atomicInteger.compareAndSet(9, 2020)+"\t"+atomicInteger.get());
    }
}
//结果
true	4
false	4
```
总结：期望值和真实值相同。修改这个值。不同就不修改。

他的功能是判断内存中的某个存储位置的值是否为预期值，这个过程是原子的。

CAS并发原语体现在JAVA语言中就是sun.misc.Unsafe类中的各个方法。调用Unsafe类中的CAS方法，JVM会帮我们实现出CAS汇编指令。这是一种完全依赖于硬件的功能，通过它实现原子操作。再次强调，由于CAS足一种系统原语，原语属于操作系统用语范畴，是由若干条指令组成的，用于完成某个功能的一个过程，**并非原语的执行必须是连续的，在执行过程中不允许被终端，也就是说CAS足一条CPU的原子指令，不会造成所谓的数据不一致问题。**

#### getAndDecrement在高并发下安全的原因

```java
private volatile int value;
/**
     * Atomically decrements the current value,
     * with memory effects as specified by {@link VarHandle#getAndAdd}.
     *
     * <p>Equivalent to {@code getAndAdd(-1)}.
     *
     * @return the previous value
     */
    public final int getAndDecrement() {
        return U.getAndAddInt(this, VALUE, -1);
    }
```
点开getAndAddInt方法
```java
 @HotSpotIntrinsicCandidate
    public final int getAndAddInt(Object o, long offset, int delta) {
        int v;
        do {
            v = getIntVolatile(o, offset);
        } while (!weakCompareAndSetInt(o, offset, v, v + delta));
        return v;
    }
    //offset 表示，该变量值在内存中的偏移地址。
```

##### 1.U来自于Unsafe类。

是CAS的核心类，由于Java方法无法直接访问底层系统，需要通过本地（native）方法来访问，Unsafe相当与一个后面。基与该类，可以直接操作特定内存的数据，Unsage类存在与sun.misc包中，其内部方法可以像C一样直接操作内存，因为java中CAS操作的执行依赖于Unsafe的方法。

**注意Unsafe类中的所有方法都是native修饰的，也就是Unsafe类中的方法都直接调用操作系统底层资源执行相应任务。**

对上面的进行小总结。{% fb_img TIM截图20200327135011.png [weakCompareAndSetInt的简单解释] %}

### CAS缺点
#### 1.循环时间长开销大
循环时间长开销大
#### 2.只能保证一个共享变量的原子操作
当对一个共享变量执行操作时，我们只能使用循环CAS的方式来保证原子操作，但是，对多个共享变量操作时，循环CAS就无法保证操作的原子性，这个时候就可以用锁来保证原子性。
#### 3.引出来ABA问题？？？
如何产生ABA问题。：

有两个线程A，B。操作一个资源R=10. 线程A将资源R=10读到自己的工作空间 ，同时线B将资源R =10读到自己的工作空间 ，B将R=20修改后返回到主内存，然后再次修改R=10.
线程A此时在访问R=10.和期望值一样，认为没有线程修改过。于是产生了ABA问题（==只管结果，不管过程==）。

#### 解决ABA问题。
通过原子引用+版本机制解决ABA问题

下面实例理解下什么是原子引用。
```java
@Getter
@AllArgsConstructor
@ToString
class User{
    String userName;
    int age;
}

public class AtomicReferenceDemo {
    public static void main(String[] args) {
        User z3 = new User("zs",22);
        User ll = new User("ll",24);
        AtomicReference<User> atomicReference = new AtomicReference<>();
        atomicReference.set(z3);
        System.out.println(atomicReference.compareAndSet(z3, ll)+"\t"+atomicReference.get().toString());
        System.out.println(atomicReference.compareAndSet(z3, ll)+"\t"+atomicReference.get().toString());
    }
}

//结果
true	User(userName=ll, age=24)
false	User(userName=ll, age=24)
```
时间戳原子引用

```java
public class ABADemo {
    static AtomicReference<Integer> atomicReference = new AtomicReference<>(100);
    static AtomicStampedReference<Integer> integerAtomicStampedReference = new AtomicStampedReference<>(100,1);
    public static void main(String[] args) {
        new Thread(() -> {
            atomicReference.compareAndSet(100,101);
            atomicReference.compareAndSet(101,100);
        },"T1").start();
        new Thread(() -> {
            //暂停1秒钟T2线程。保证上面线程完成一次ABA操作
            try{
                TimeUnit.SECONDS.sleep( 1 );} catch (InterruptedException e) {e.printStackTrace();}
            System.out.println("是否修改成功："+Thread.currentThread().getName()+"\t"+atomicReference.compareAndSet(100, 2019)+"\t"+atomicReference.get());
        },"T2").start();

        new Thread(() -> {
            System.out.println("-----------------T3 和 T4 时间戳原子引用-------------------------");
            int stamp = integerAtomicStampedReference.getStamp();
            System.out.println(Thread.currentThread().getName()+"\t第一次版本号："+stamp);
            try{TimeUnit.SECONDS.sleep( 1 );} catch (InterruptedException e) {e.printStackTrace();}
            integerAtomicStampedReference.compareAndSet(100,101,integerAtomicStampedReference.getStamp(),integerAtomicStampedReference.getStamp()+1);
            System.out.println(Thread.currentThread().getName()+"\t第二次版本号："+integerAtomicStampedReference.getStamp());
            integerAtomicStampedReference.compareAndSet(101,100,integerAtomicStampedReference.getStamp(),integerAtomicStampedReference.getStamp()+1);
            System.out.println(Thread.currentThread().getName()+"\t第三次版本号："+integerAtomicStampedReference.getStamp());
        },"T3").start();

        new Thread(() -> {
            int stamp = integerAtomicStampedReference.getStamp();
            System.out.println(Thread.currentThread().getName()+"\t版本号："+stamp);
            try{TimeUnit.SECONDS.sleep( 3 );} catch (InterruptedException e) {e.printStackTrace();}
            boolean result = integerAtomicStampedReference.compareAndSet(100, 2019, stamp, stamp + 1);
            System.out.println(Thread.currentThread().getName()+"\t是否修改成功："+result+"\t当前版本号："+integerAtomicStampedReference.getStamp());
        },"T4").start();
    }
}

//结果
-----------------T3 和 T4 时间戳原子引用-------------------------
T3	第一次版本号：1
T4	版本号：1
是否修改成功：T2	true	2019
T3	第二次版本号：2
T3	第三次版本号：3
T4	是否修改成功：false	当前版本号：3
```
## ArrayList是线程不安全，编写一个不安全的案例并给出解决方案
同样的有ArrayList、HashSet、HashMap
```java
public class ContainerNotSafeDemo {

    public static void main(String[] args) {
        List<String> list = new ArrayList<>();
        for (int i = 1;i <= 200; i++){
            new Thread(() -> {
                list.add(UUID.randomUUID().toString().substring(0,8));
                System.out.println(list);
            },String.valueOf(i)).start();
        }
    }
}
```

1. 故障现象
    java.util.ConcurrentModificationException
2. 导致原因
    并发争抢修改导致，参考我们的花名册签名情况。
    一个人正在写入，另外一个人过来抢夺。导致数据不一致异常。
3. 解决方案
    1. ArrayList：
        1. new Vector<>();
        2. Collections.synchronizedList(new ArrayList<>());
        3. new CopyOnWriteArrayList<>();   //写时复制。读时不加锁。读写分离
    2. HashSet：
        1. Collections.synchronizedSet(new HashSet<>());
        5. new CopyOnWriteArraySet<>();
    3. HashMap：
        6. new  ConcurrentHashMap<>();
        7. Collections.synchronizedMap(new HashMap<>());
4. 优化建议

##### 小知识。hashset
hashset得底层是hashmap。hashset的add方法是hashmap的put方法。值是一个Object的常量。

```java
// Dummy value to associate with an Object in the backing Map
    private static final Object PRESENT = new Object();
    public boolean add(E e) {
        return map.put(e, PRESENT)==null;
    }
```
## 公平锁/非公平锁/可重入锁/递归锁/自旋锁

### 公平锁/非公平锁
公平锁/非公平锁：并发包中**ReentrantLock**的创建可以指定构造函数的boolean类型来得到公平锁或非公平锁，默认是非公平锁。

关于两者区别：

公平锁：Threads acquire a fair lock in the order in which they requested it.

公平锁，就是很公平，在并发情况下，每个线程在获取锁时会查看此锁维护的等待队列，如果为空，或者当前线程是等待队列的第一个，就占有锁，否则就会加入到等待队列中，以后会按照FIFO的规则从队列中取到自己。

非公平锁：非公平锁比较粗鲁，上来就直接尝试占有锁，如果尝试失败，就再采取类似公平锁那种方式。
#### 题外话
Java ReentrantLock而言，通过构造函数指定该锁是否是公平锁，默认是非公平锁
。非公平锁的优点在于吞吐量比公平锁大。

对于Synchronized而言，也是一种非公平锁。
### 可重入锁（也就是递归锁）
指的是同一个线程外层函数获得锁之后，内层递归函数仍然能获取该锁的代码，在同一线程在外层方法获取锁的时候，在进入内层方法会自动获取锁。

也就是说，
线程可以进入任何一个它已经拥有的锁所有同步着的代码块。

++**ReentrantLock/Synchronized就是一个典型的可重入锁**++

**++可重入锁最大的作用是避免死锁++**


```java
class Phone implements Runnable{
    public synchronized void SendSMS()throws Exception{
        System.out.println(Thread.currentThread().getName()+"\t invoked sendSMS()");
        SendEmail();
    }
    public synchronized void SendEmail() throws Exception{
        System.out.println(Thread.currentThread().getName()+"\t###### invoked SendEmail()");
    }
    Lock lock = new ReentrantLock();
    @Override
    public void run() {
        get();
    }

    private void get() {
        lock.lock();
        try {
            System.out.println(Thread.currentThread().getName()+"\t    getlock");
            set();
        }catch (Exception e){
            e.printStackTrace();
        }finally {
            lock.unlock();
        }
    }
    private void set() {
        lock.lock();
        try {
            System.out.println(Thread.currentThread().getName()+"\t   set()");
        }catch (Exception e){
            e.printStackTrace();
        }finally {
            lock.unlock();
        }
    }
}
/**
 * @author Ambi
 * @version 1.0
 * @date 2020/3/28 10:14
 * @description: 可重入锁
 * 指的是同一个线程外层函数获得锁之后，内层递归函数仍然能获取该锁的代码，
 * 在同一线程在外层方法获取锁的时候，在进入内层方法会自动获取锁。
 *
 * 也就是说，
 * 线程可以进入任何一个它已经拥有的锁所有同步着的代码块。
 */
public class ReenterLockDemo {
    public static void main(String[] args) {
        Phone phone = new Phone();
        new Thread(() -> {
            try {
                phone.SendSMS();
            } catch (Exception e) {
                e.printStackTrace();
            }
        },"t1").start();
        new Thread(() -> {
            try {
                phone.SendSMS();
            } catch (Exception e) {
                e.printStackTrace();
            }
        },"t2").start();
        try{
            TimeUnit.SECONDS.sleep( 1 );} catch (InterruptedException e) {e.printStackTrace();}
        Thread t3 = new Thread(phone,"t3");
        Thread t4 = new Thread(phone,"t4");
        t3.start();
        t4.start();
    }
}

//结果
t1	 invoked sendSMS()
t1	###### invoked SendEmail()
t2	 invoked sendSMS()
t2	###### invoked SendEmail()
t3	    getlock
t3	   set()
t4	    getlock
t4	   set()
```

### 自旋锁

是指尝试获取锁的线程不会立即阻塞，而是采用循环的方式去尝试获取锁
，这样的好处是减少线程上下切换的消耗，缺点是循环会消耗CPU。

代码实例：

```java
/**
 * @author Ambi
 * @version 1.0
 * @date 2020/3/29 11:00
 * @description:题目 实现一个自旋锁
 * 自旋锁的好处： 血循环比较获取知道成功
 *
 * 通过Cas操作完成自旋锁，A线程先进来调用MyLock方法自己持有锁5秒钟。B随后进来发现
 * 当前线程持有锁。不是null，所以只能通过自旋等待。直到A释放锁后B随后抢到
 */
public class SpinLockDemo {
    //原子引用线程
    AtomicReference<Thread> atomicReference = new AtomicReference<>();
    public void myLock(){
        Thread thread = Thread.currentThread();
        System.out.println(Thread.currentThread().getName()+"\t  come in ");
        while (!atomicReference.compareAndSet(null,thread)){

        }
    }

    public void myUnLock(){
        Thread thread = Thread.currentThread();
        atomicReference.compareAndSet(thread,null);
        System.out.println(Thread.currentThread().getName()+"\t Unlock");
    }
    public static void main(String[] args) {
        SpinLockDemo spinLockDemo = new SpinLockDemo();
        new Thread(() -> {
            spinLockDemo.myLock();
            try{
                TimeUnit.SECONDS.sleep(5 );} catch (InterruptedException e) {e.printStackTrace();}
            spinLockDemo.myUnLock();
        },"AA").start();
        try{TimeUnit.SECONDS.sleep( 1 );} catch (InterruptedException e) {e.printStackTrace();}
        new Thread(() -> {
            spinLockDemo.myLock();
            try{TimeUnit.SECONDS.sleep( 1 );} catch (InterruptedException e) {e.printStackTrace();}
            spinLockDemo.myUnLock();
        },"BB").start();
    }
}
```

### 独占锁/共享锁
    独占锁：指该锁一次只能被一个线程所持有。对ReentrantLock和Synchronized而言都是独占锁。
    
    共享锁：指该锁可被多个线程所持有。
    对ReentrantReadWriteLock，其读锁是共享锁，其写锁是独占锁。读锁的共享锁可保证并发读是非常高效的，读写，写读，写写的过程是互斥的。

#### synchronized和lock有什么区别
**1.原始构成：**

	synchronized是关键字，属于JVM层面，monitorenter（底层是通过monitor对象来完成，其实wait/notify等方法也依赖于monitor对象只有在同步块或者方法中才能调用wait/notify等方法）
	
	Lock是具体类（java.util.concurrent.locks.lock）是api层面的锁。

**2.使用方法**

    synchronized不需要用户去手动释放锁，当synchronized代码执行完后系统会自动让线程释放对锁的占用。
    ReentrantLock则需要用户去手动释放锁，若没有主动释放锁，就有可能导致出现死锁现象。需要lock()和unlock()方法配合try/finally语句块来完成。

**3.等待是否可中断**

    synchronized不可中断，除非抛出异常或者正常运行完成。
    ReentrantLock可中断，1.设置超时方法 tryLock(long timeout,TimeUnit unit)；2.lockInterruptibly()放代码块中，调用interrupt()方法可中断。

**4.加锁是否公平**

    synchronized非公平锁
    ReentrantLock两者都可以，默认非公平锁，构造方法可以传入boolean值，true为公平锁，false为非公平锁。

**5.锁绑定多个条件Condition**

    synchronized没有
    ReentrantLock用来实现分组唤醒需要唤醒的线程，可以精确唤醒，而不是像synchronized要么随机唤醒一个要么唤醒全部线程。

## 6.CountDownLatch/CyclicBarrier/Semaphore
### CountDownLatch
让一些线程阻塞直到另一个线程完成一系列操作后才被唤醒

CountDownLatch主要有两个方法，当一个或多个线程调用await方法时，调用线程会被阻塞。其他线程调用countDown方法会将计数器减1（调用CountDown方法的线程不会阻塞），当计数器的值变为0时，因调用await方法被阻塞的线程会被唤醒，继续执行。


```java
/**
 * @author Ambi
 * @version 1.0
 * @date 2020/3/30 9:22
 * @description: CountDownLatch 
 * CountryEnum.foreach_CountyEnum(i).getRetMessaeg()自己写的枚举
 */
public class ConuntDownLatchDemo {
    public static void main(String[] args) throws Exception{
        CountDownLatch countDownLatch = new CountDownLatch(6);
        for (int i = 1;i <= 6; i++){
            new Thread(() -> {
                System.out.println(Thread.currentThread().getName()+"\t 同学离开");
                countDownLatch.countDown();
            }, CountryEnum.foreach_CountyEnum(i).getRetMessaeg()).start();
        }
        countDownLatch.await();
        new Thread(() -> {
            System.out.println("班长锁门");
        },"input threadd name").start();
    }

}

```
CountryEnum 类
```java
/**
 * @author Ambi
 * @version 1.0
 * @date 2020/3/30 9:29
 * @description: 六国枚举
 */
public enum  CountryEnum {
    ONE(1,"齐"),TWO(2,"楚"),THREE(3,"赵"),
    FOUR(4,"燕"),Five(5,"韩"),SIX(6,"魏")
    ;
    @Getter
    private Integer index;
    @Getter
    private String retMessaeg;

    CountryEnum(int index, String retMessaeg) {
        this.index = index;
        this.retMessaeg = retMessaeg;
    }

    public static CountryEnum foreach_CountyEnum(Integer index){
        for (CountryEnum anEnum : CountryEnum.values()) {
            if (anEnum.getIndex() == index){
                return anEnum;
            }
        }
        return null;
    }
}
```

### CyclicBarrier
CyslicBarrier的字面意思是可循环（Cyclic）使用的屏障（Barrier）。它要做的事情是，让一组线程到达屏障（也可以叫同步点）时被阻塞，直到最后一个线程到达屏障时，屏障才会打开门，所有被屏障拦截的线程才会继续干活，线程进入屏障通过CyclicBarrier的await（）方法。

示例代码：
```java
/**
 * @author Ambi
 * @version 1.0
 * @date 2020/3/30 9:48
 * @description: CyclicBarrierDemo
 * 召唤神龙
 */
public class CyclicBarrierDemo {
    public static void main(String[] args) {
        CyclicBarrier cyclicBarrier = new CyclicBarrier(7,()->{
            System.out.println("召唤神龙");
        });
        for (int i = 1;i <= 7; i++){
            final int tempInt = i;
            new Thread(() -> {
                System.out.println(Thread.currentThread().getName()+"\t 收集到第："+tempInt+"龙珠");
                try {
                    cyclicBarrier.await();
                } catch (InterruptedException e) {
                    e.printStackTrace();
                } catch (BrokenBarrierException e) {
                    e.printStackTrace();
                }
            },String.valueOf(i)).start();
        }
    }
}
//结果：
2	 收集到第：2龙珠
6	 收集到第：6龙珠
3	 收集到第：3龙珠
7	 收集到第：7龙珠
1	 收集到第：1龙珠
4	 收集到第：4龙珠
5	 收集到第：5龙珠
召唤神龙
```

### Semaphore
信号量主要用于两个目的，一个是用于多个共享资源的互斥使用，另一个用于并发线程数的控制。


```java
/**
 * @author Ambi
 * @version 1.0
 * @date 2020/3/30 9:57
 * @description: 信号灯机制。
 */
public class SemaphoreDemo {
    public static void main(String[] args) {
        Semaphore semaphore = new Semaphore(3);
        for (int i = 1;i <= 6; i++){    //模拟6个车
            new Thread(() -> {
                try {
                    semaphore.acquire();    //占有一个信号
                    System.out.println(Thread.currentThread().getName()+"\t 抢到车位");
                    try{
                        TimeUnit.SECONDS.sleep( 3 );} catch (InterruptedException e) {e.printStackTrace();}
                    System.out.println(Thread.currentThread().getName()+"\t 停3秒后离开车位");
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }finally {
                    semaphore.release();    //释放一个信号
                }
            },String.valueOf(i)).start();
        }
    }
}
//结果
6	 抢到车位
1	 抢到车位
2	 抢到车位
6	 停3秒后离开车位
4	 抢到车位
2	 停3秒后离开车位
1	 停3秒后离开车位
5	 抢到车位
3	 抢到车位
3	 停3秒后离开车位
4	 停3秒后离开车位
5	 停3秒后离开车位
```
## 7.阻塞队列
阻塞队列，首先它是一个队列，而一个阻塞队列在数据结构中所起的作用大致是：线程1往阻塞队列中添加元素，而线程2从阻塞队列中移除元素。

当阻塞队列是空时，从队列中获取元素的操作将被**阻塞**。
当阻塞队列是满时，往队列里添加元素的操作将被**阻塞**。

### 为什么用？有什么好处？
在多线程领域：所谓阻塞，在某些情况下会挂起线程（即阻塞），一旦满足条件，被挂起的线程又会自动被唤醒。

**为什么需要BlockingQueue？**

好处是我们不需要关心什么时候需要阻塞线程，什么时候需要唤醒线程，因为这一切BlockingQueue都给你包办了。

在concurrent包发布之前，在多线程环境下，我们每个程序员都必须去自己控制这些细节，尤其还要兼顾效率和线程安全，而这会给程序带来不小的复杂度。
#### BlockingQueue的核心方法
BlockingQueue和list都是Collections的接口。

    ArrayBlockingQueue:由数组结构组成的有界阻塞队列
    LinkedBlockingQueue:由链表结构组成的有界（但大小默认值为Integer.MAX_VALUE）阻塞队列
    PriorityBlockingQueue:支持优先级排序的无界阻塞队列。
    DelayQueue:使用优先级队列实现的延迟无界阻塞队列。
    LinkedBlockingDeque:由链表结构组成的双向阻塞队列。
    LinkedTransferQueue:由链表结构组成的无界阻塞队列。
    SynchronousQueue:不存储元素的阻塞队列，也即单个元素的队列。
    
    
    SynchronousQueue没有容量。
    与其他BlockingQueue不同，SynchronousQueue是一个不存储元素的BlockingQueue。
    每一个put操作必须要等待一个take操作，否则不能继续添加元素，反之亦然。
    
#### BlockingQueue的使用

```java
class MyResource{
    private volatile boolean FLAG = true;//默认开启，进行生产+消费
    private AtomicInteger atomicInteger = new AtomicInteger();

    BlockingQueue<String> blockingQueue = null;
    public MyResource(BlockingQueue<String> blockingQueue) {
        this.blockingQueue = blockingQueue;
        System.out.println(blockingQueue.getClass().getName());
    }

    public void myProd() throws Exception{
        String data = null;
        boolean retValue;
        while(FLAG){
            data = atomicInteger.incrementAndGet()+"";
            retValue = blockingQueue.offer(data,2L, TimeUnit.SECONDS);
            if(retValue){
                System.out.println(Thread.currentThread().getName()+"\t插入队列"+data+"成功");
            }else{
                System.out.println(Thread.currentThread().getName()+"\t插入队列"+data+"失败");
            }
            TimeUnit.SECONDS.sleep(1);
        }
        System.out.println(Thread.currentThread().getName()+"\t生产停止");
    }

    public void myConsumer() throws Exception{
        String result = null;
        while(FLAG){
            result = blockingQueue.poll(2L,TimeUnit.SECONDS);
            if(null==result || result.equalsIgnoreCase("")){
                FLAG = false;
                System.out.println(Thread.currentThread().getName()+"\t 超过2秒，消费退出");
                System.out.println();
                System.out.println();
                return;
            }
            System.out.println(Thread.currentThread().getName()+"\t消费队列"+result+"成功");
        }
    }

    public void stop() throws Exception{
        this.FLAG = false;
    }
}

/**
 * @author Ambi
 * @version 1.0
 * @date 2020/3/31 11:38
 * @description: myProd（）生成资源、myConsumer()消费资源、stop()停止工作
 */
public class ProdConsumer_BlockQueueDemo {
    public static void main(String[] args) throws Exception{
        MyResource myResource = new MyResource(new ArrayBlockingQueue<>(10));

        new Thread(()->{
            System.out.println(Thread.currentThread().getName()+"\t 生产线程启动");

            try{
                myResource.myProd();
            }catch (Exception e){
                e.printStackTrace();
            }
        },"Prod").start();

        new Thread(()->{
            System.out.println(Thread.currentThread().getName()+"\t 消费线程启动");
            System.out.println();
            System.out.println();
            try{
                myResource.myConsumer();
            }catch (Exception e){
                e.printStackTrace();
            }
        },"Consumer").start();

        try{TimeUnit.SECONDS.sleep(5);}catch (InterruptedException e){e.printStackTrace();}

        System.out.println();
        System.out.println();
        System.out.println();

        System.out.println("5秒钟到，main停止");
        myResource.stop();
    }
}
```
## 线程池

### 线程池如何使用？
#### 架构说明
Java中的线程池是通过Executor框架实现的，该框架中用到了Executor，Executors，ExecutorService，ThreadPoolExecutor这几个类。
#### 编码实现
    
    Executors.newScheduledThreadPool()//了解
    Executors.newWorkStealingPool(int)//java8新出
    
Executors.newFixedThreadPool(int)

    主要特点：
    创建一个定长
    线程池，可控制线程最大并发数，超出的线程会在队列中等待。
    newFixedThreadPool创建的线程池corePoolSize和maximumPoolSize值是相等的，它使用的LinkedBlockingQueue。
Executors.newSingleThreadExecutor()

    主要特点：
    创建
    一个单线程化的线程池，它只会唯一的工作线程来执行任务，保证所有任务按照指定顺序执行。
    newSingleThread将corePoolSize和maximumPoolSize都设置为1，它使用的是LinkedBlockingQueue。

Executors.newCachedThreadPool()

    主要特点：
    创建一个
    可缓存线程池，如果线程池长度超过处理需要，可灵活回收空闲线程，若无可回收，则新建线程。
    newCachedThreadPool将corePoolSize设置为0，将maximumPoolSize设置为Integer.MAX_VALUE，使用的SynchronousQueue，也就是说来了任务就创建线程运行，当线程空闲超过60秒，就销毁线程。
```java 
**
 * @author Ambi
 * @version 1.0
 * @date 2020/3/31 15:23
 * @description: 第四种，获得/使用java多线程方式，线程池。
 */
public class MyThreadPoolDemo {

    public static void main(String[] args) {
        //固定数
        ExecutorService threadPool = Executors.newFixedThreadPool(5);
//        ExecutorService threadPool = Executors.newSingleThreadExecutor();//单一
//        ExecutorService threadPool = Executors.newCachedThreadPool();//同步队列


        try {
            for (int i = 0; i <= 10; i++) {
                threadPool.execute(()->{
                    System.out.println(Thread.currentThread().getName() +"\t  办理业务");
                });
            }
        }catch (Exception e){
            e.printStackTrace();
        }finally {
            threadPool.shutdown();
        }
    }
}
```
