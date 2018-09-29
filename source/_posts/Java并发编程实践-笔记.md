title: Java并发编程实践-笔记
date: 2015-09-16 22:01:15
categories: Java
tags: 
  - Java
  - 并发
---
Java Concurrency in Practice
- 为什么要并发编程？
    - 多核时代！
    - 大部分程序只是碰巧是对的.
> “Java 底层机制与设计层必要的策略之间存在着抽象的不匹配.为了解决这个问题,我们呈现了一个经过简化的规则集,用来编写并发程序.”
- [代码](http://www.javaconcurrencyinpractice.com "")

## 介绍
- 基本概念：Socket, signal handlers, shared memory, semaphores
线程允许程序控制流(control flow)的多重分支同时存在于一个进程.它们共享进程范围内的资源,比如内存和文件句柄,但是每个线程都有自己的程序计数器(program counter),栈(stack)和本地变量.
- NPTL线程包,作为Linux发布的一部分,设计它的目的是用来支持数十万甚至更多的线程.
- AWT和Swing工具集用事件派发线程(event dispatch thread,EDT)取代了主事件循环.当一个用户界面事件发生时,比如按下一个按钮,事件线程调用程序定义的事件处理器.
- 安全风险
    - 多线程中各个操作的顺序是不可预测的
    - 竞争条件(race condition)
- 活跃度危险
    - 活跃度失败(liveness failure): 一个活动进入某种它永远无法再继续执行的状态.
    - 死锁(deadlock),饥饿(starvation),活锁(livelock)

## 第一部分　线程安全(Thread Safety)
编写线程安全的代码,本质上就是管理对** 状态 **的访问,而且通常都是** 共享的,可变的 **状态.
我们讨论的线程安全性好像是关于代码的,但是真正要做的是在不可控制的并发访问中保护数据.
** synchronized:独占锁 **
线程安全性：一个类是线程安全的,是指在被多个线程访问时,类可以持续进行正确的行为.
无状态对象永远是线程安全的.
- **竞争条件**
当计算的正确性依赖于运行时相关的时序或者多线程的交替时,会产生竞争条件.
竞争条件的典型例子:
    - 计数器递增(读-改-写)
    - 惰性初始化(检查再运行)
竞争条件和数据竞争
原子操作
synchronized块的两部分：锁对象的引用,以及这个锁保护的代码块
每个java对象都可以隐式地扮演一个用于同步的锁的角色；这些内置的锁被称作内部锁(intrinsic locks)或监控器锁(monitor locks).执行线程进入synchronized块之前会自动获得锁；而无论通过正常控制路径退出,还是从块中抛出异常,线程都在放弃对synchronized块的控制时自动释放锁.获得内部锁的唯一途径是：进入这个内部锁保护的同步块或方法.
内部锁在java中扮演了互斥锁(mutual exclusion lock,也称作mutex)的角色,意味着至多只有一个线程可以拥有锁.
- ** 重进入(Reentrancy) **
当一个线程请求其他线程已经占有的锁时,请求线程将被阻塞.然而内部锁是可重进入的.因此,线程在试图获得它自己占有的锁时,请求会成功.
重进入意味着锁的请求是基于** 每线程 **的,而不是基于** 每调用 **的.
重进入的实现是通过每个锁关联一个请求计数(acquisition count)和一个占有它的线程.当计数为０时,认为锁是未被占有的.线程请求一个未被占有的锁时,JVM将记录锁的占有者,并且将请求计数置为１.如果同一线程再次请求这个锁,计数将递增；每次占用线程退出同步块,计数值将递减.直到计数器到０时,锁被释放.
如果内部锁不是可重入的,下面代码将产生死锁.  

```java
public class Widget {  
    public synchronized void doSomething(){
        ...
    }
}
public class LoggingWidget extends Widget{
    public synchronized void doSomething(){
        System.out.println(toString()+": calling doSomething");
        supper.doSomething();
    }
}
```

- 锁保护: 对于每个可被多个线程访问的可变状态变量,如果所有访问它的线程在执行时都占有** 同一个锁 **,这种情况下,我们称这个变量是由这个锁保护的.
Vector是线程安全的
对于每一个涉及多个变量的不变约束,需要同一个锁保护其所有的变量.
弱并发(poor concurrency)

## 共享对象(Sharing Ojects)
- ** 重排序 **
重排序通常是编译时或运行时环境为了优化程序性能而采取的对指令进行重新排序执行的一种手段.重排序分为两类:** 编译期重排序 **和** 运行期重排序 **,分别对应编译时和运行时环境.
在并发程序中,程序员会特别关注不同进程或线程之间的数据同步,特别是多个线程修改同一变量时,必须采取可靠的同步或其他措施保障数据被正确的修改.一条重要的原则是:**不要假设指令执行的顺序,你无法预知不同线程之间的指令会以何种顺序执行.**
在单线程程序中,通常假设指令是** 顺序执行 **的.
编译期重排序:编译期重排序的典型就是通过调整指令顺序,在不改变程序语义的前提下,尽可能减少寄存器的读取,存储次数,充分复用寄存器的存储值.
内存可见性
当读两个线程同时访问同一个变量时,synchronized用于确保写线程更新变量后,读线程再访问该变量时可以读取到该变量最新的值.
java存储模型中的重排序
在java的存储模型(Java Memory Model,JMM)中,重排序是十分重要的一节,特别是并发编程中.JMM通过** happens-before **法则保证顺序执行语义,如果想要让执行操作B的线程观察到执行操作A的线程的结果,那么A和B就必须满足happens-before原则,否则,JVM就会对它们进行任意排序以提高程序性能.
** volatile ** 关键字可以** 保证变量的可见性 **,因为对volitile的操作都在MainMemory中,而Main Memory是被所有线程所共享的,这里的代价是牺牲了性能,无法利用寄存器或Cache,因为它们都不是全局的,无法保证** 可见性 **,可能产生脏读.
volatile还有一个作用就是** 局部阻止重排序的发生 **,对volatile变量的操作指令都不会被重排序,因为重排序,又可能产生可见性问题.
如果一个字段声明为volatile型,线程对这个字段写入后,在执行后续的内存访问之前,线程必须刷新这个字段,且让这个字段对其他线程可见(即该字段立即刷新).每次对volatile字段的读访问,都要重新装载字段的值.
在保证可见性方面,锁(包括显示锁,对象锁)以及原子变量的读写都可以保证变量的可见性.
当一个线程在没有同步的情况下读取变量,它可能会得到一个过期值.但是至少它可以看到某个线程在那里设定的一个真实数值,而不是一个凭空而来的值.这样的安全保证被称为** 最低限的安全性(out-of-thin-air safety) **
最低限的安全应用与所有变量,除了一个例外:没有声明为volatile的64位数值变量(double和long).Java存储模型要求获取和存储操作都为原子的,但是对于非volatile的long和double变量,
JVM允许将64位的读或写划分为两个32位的操作.如果读和写发生在不同的线程,这种情况读取一个非volatile类型long就可能出现得到一个高32位和另一个值的低32位.
锁不仅仅是关于同步与互斥 的,也是关于内存可见的.为了保证所有线程都能看到共享的,可变变量的最新值,读取和写入线程必须使用公共的锁进行同步.
volatile变量的操作不会加锁,也就不会引起执行线程的阻塞,这使得volatile变量相对于synchronized而言,只是轻量级的同步机制.
volatile的语义不足以使自增操作(count++)原子化,除非你能保证只有一个线程对变量执行写操作.原子变量提供了"读-改-写"原子操作的支持,而且常被用作"更优的volatile变量",**加锁可以保证可见性与原子性,volatile变量只能保证保证可见性.**
- **调试提示**
对于服务器应用程序,确保无论是在开发阶段还是测试阶段,启动JVM时都使用-server命令行选项,server模式的JVM会比client模式的JVM执行更多的优化,比如把没有在循环体中修改的变量提升到循环体外部;在开发环境(client模式的JVM)中可以工作的代码,可能会在部署环境(server环境的JVM)中失败.
- **发布和逸出**
**发布**(publishing)一个对象的意思是使它能够被当前范围之外的代码所使用.
**逸出**是指一个对象在尚未准备好时就将它发布.
使用工厂方法防止this引用再构造期间逸出
```java
public class SafeListener{
	private final EventListener listener;
    private SafeListener(){
    	listener = new EventListener(){
            public void onEvent(Event e){
        		doSomething(e);
        };
    }
    public static SafeListener newInstance(Event e){
    	SafeListener safe = new SafeListener();
        source.registerListener(safe.listener);
        return safe;
    }
}
```
- Ad-hoc 线程限制
**Ad-hoc线程限制**是指维护线程限制性的任务全部落在实现上的这种情况.因为没有可见性修饰符与本地变量等语言特性协助将对象限制在目标线程上,所以这种方式是非常容易出错的.(Ad-hoc的言外之意是"非正式的")
- 栈限制
栈限制是线程限制的一种特例,在栈限制中,只能通过本地变量才能触及对象.正如封装使不变约束更容易被保持,本地变量使对象更容易被限制在线程本地中.本地变量本身就被限制再执行线程中,它们存在于执行线程栈.其他线程无法访问这个栈.
- **ThreadLocal**
一种维护贤臣限制的更加规范的方式是使用ThreadLocal,它允许你将每个线程与持有数值的对象关联在一起.ThreadLocal提供了get与set访问器,为每个使用它的线程维护一份单独的拷贝.所以get总是返回由当前执行线程通过set设置的最新值.
线程本地(ThreadLocal)变量通常用于防止在基于可变的单体(Singleton)或全局变量的设计中,出现不正确的共享.
概念上可以将ThreadLocal<T>看做map<Thread,T>,它存储了与线程相关的值,不过事实上它并非这样实现的.与线程有关的值存储在线程对象自身中,线程终止后,这些值会被垃圾回收.
> 不理解对象的不变约束和后验条件,你就不能保证线程安全性.要约束状态变量的有效值或者状态转换,就需要原子性与封装性.

- 私有锁保护状态
```java
public class PrivateLock {
	private final Object myLock = new Object();
    Widget widget;

    void someMethod() {
    	synchronized(myLock) {
        	// 访问或修改Widget的状态
        }
    }
}
```
**使用私有锁对象,而不是对象内部锁(或其他可公共访问的锁)的好处:**
私有的锁对象可以封装锁,这样客户代码无法得到它.而可公共访问的锁允许客户代码涉足它的同步策略--正确或不正确地.客户不正确地得到另一个对象的锁,会引起**活跃度**方面的问题.另外要验证程序是否正确地使用着一个可公共访问的锁,需要检查完整的程序,而不是一个单独的类.
(**Liveness**,A concurrent application's ability to excute in a timely manner is known as its liveness.deadlock,starvation and livelock are the most common kind of liveness problem.)
> 关于volatile变量的一条规则:当且仅当一个变量没有参与那些涉及其他状态变量的不变约束时,才适合声明为volatile类型.
- 客户端加锁
```java
// 非线程安全的"缺少即加入"实现
// !!!问题:这样的设计,外部能获得list的引用吗?如果不能,其他线程能同时修改list内容吗?
public class ListHelper<E> {
	public List<E> list = Collections.synchronizedList(new ArrayList<E>());
    ...
    public synchronized boolean putIfAbsent(E x) {
    boolean absent = !list.contains(x);
    if(absent)
    	list.add(x);
    return absent;
    }
}
```
这里的问题在于同步行为发生在**错误的锁**上.即使一个list的操作全部声明为synchronized,但是使用了不同锁,将意味着putIfAbsent对于List的其他操作而言,并不是原子化的.所以当putIfAbsent执行时,不能保证另一个线程不会修改list.
```java
// 使用客户端加锁的"缺少即加入"实现
public class ListHelper<E> {
	public List<E> list = Collections.synchronizedList(new ArrayList<E>());
    ...
    public boolean putIfAbsent(E x) {
    	synchronized(list){
            boolean absent = !list.contains(x);
            if(absent)
                list.add(x);
            return absent;
        }
}
```
```java
// 使用组合(composition)实现"缺少即加入"
// 使用Java监视器模式有效地封装了一个已有的List,而且只要我们的类持有底层List的唯一外部引用,那么就能保证提供线程安全性.
// !!!貌似有问题,这里的list是通过外部传参进来的,这样很难保证外部不拥有list的引用.
public class ImprovedList<T> implements List<T> {
	private final List<T> list;
    public ImprovedList(List<T> list) {this.list = this;}
    public synchronized boolean putIfAbsent(T x) {
    	boolean contains = list.contains(x);
        if(contains)
        	list.add(x);
        return !contains;
    }

    public synchronized void clear() {list.clear();}
    // ... similarly delegate other list methods
}
```
#### 构建块(Building Blocks)
- ConcurrentMap 接口
```java
public interface ConcurrentMap<K, V> extends Map<K, V> {

    V putIfAbsent(K key, V value);

    boolean remove(Object key, Object value);

    boolean replace(K key, V oldValue, V newValue);

    V replace(K key, V value);
}
```
- CopyOnWriteArrayList
"写入时复制(copy-on-write)"容器的迭代保留一个底层基础数组(the backing array)的引用.这个数组作为迭代器的起点,永远不会被修改,因此多个线程可以对这个容器进行迭代,并且不会受到另一个或多个想要修改容器的线程带来的干涉."写入时复制"容器返回的迭代器不会抛出ConcurrentModificationException,并且返回的元素严格与迭代器创建时相一致,不会考虑后续的修改.
```java
	/**
     * Appends the specified element to the end of this list.
     *
     * @param e element to be appended to this list
     * @return <tt>true</tt> (as specified by {@link Collection#add})
     */
    public boolean add(E e) {
        final ReentrantLock lock = this.lock;
        lock.lock();
        try {
            Object[] elements = getArray();
            int len = elements.length;
            Object[] newElements = Arrays.copyOf(elements, len + 1);
            newElements[len] = e;
            setArray(newElements);
            return true;
        } finally {
            lock.unlock();
        }
    }
	/**
     * Sets the array.
     */
    final void setArray(Object[] a) {
        array = a;
    }
    /**
     * Returns <tt>true</tt> if this list contains all of the elements of the
     * specified collection.
     *
     * @param c collection to be checked for containment in this list
     * @return <tt>true</tt> if this list contains all of the elements of the
     *         specified collection
     * @throws NullPointerException if the specified collection is null
     * @see #contains(Object)
     */
    public boolean containsAll(Collection<?> c) {
        Object[] elements = getArray();
        int len = elements.length;
        for (Object e : c) {
            if (indexOf(e, elements, 0, len) < 0)
                return false;
        }
        return true;
    }
```
- 生产者-消费者模式
生产者和消费者可以并发地执行,如果一个受限于I/O,另一个受限于CPU,那么并发执行的全部产出会高于顺序执行的产出(想起我写过的loader).
- 双端队列和窃取工作
正如阻塞队列适用于生产者-消费者模式一样,双端队列使它们自身与一种叫做**窃取工作(work stealing)**的模式相关联.在生产者-消费者模式中,所有消费者只共享一个工作队列;在窃取工作的设计中,每一个消费者都有一个自己的双端队列.如果一个消费者完成了自己双端队列中的全部工作,它可以偷取其他消费者的双端队列中的**末尾**任务.因为工作者线程并不会竞争一个共享的任务队列,所以**窃取工作模式**比传统的生产者-消费者设计有更好的可伸缩性;大多数时候它们访问自己的双端队列,减少竞争.当一个工作者必须要访问另一个队列时,它会从尾部截取,而不是从头部,从而进一步降低对双端队列的竞争.
- 闭锁
**闭锁(latch)**是一种Synchronizer,它可以延迟线程的进度直到线程到达**终止(terminal)**状态.一个闭锁工作起来就像一道大门: 直到闭锁达到终点状态之前,门一直是关闭的,没有线程能够通过,在终点状态到来的时候,门开了,允许所有线程通过.一旦闭锁到达了终点状态,它就不能再改变状态了,所以它会永远保持敞开状态.
- FutureTask
FutureTask同样可以作为闭锁.(FutureTask的实现描述了一个抽象的可携带结果的计算).
FutureTask的计算是通过Callable实现的,它等价于一份可携带结果的Runnable,并且有3个状态:等待,运行和完成.
```java
public class FutureTask<V> implements RunnableFuture<V>;
public interface RunnableFuture<V> extends Runnable, Future<V>;
```
状态
```java
    /**
     * The run state of this task, initially NEW.  The run state
     * transitions to a terminal state only in methods set,
     * setException, and cancel.  During completion, state may take on
     * transient values of COMPLETING (while outcome is being set) or
     * INTERRUPTING (only while interrupting the runner to satisfy a
     * cancel(true)). Transitions from these intermediate to final
     * states use cheaper ordered/lazy writes because values are unique
     * and cannot be further modified.
     *
     * Possible state transitions:
     * NEW -> COMPLETING -> NORMAL
     * NEW -> COMPLETING -> EXCEPTIONAL
     * NEW -> CANCELLED
     * NEW -> INTERRUPTING -> INTERRUPTED
     */
    private volatile int state;
    private static final int NEW          = 0;
    private static final int COMPLETING   = 1;
    private static final int NORMAL       = 2;
    private static final int EXCEPTIONAL  = 3;
    private static final int CANCELLED    = 4;
    private static final int INTERRUPTING = 5;
    private static final int INTERRUPTED  = 6;
```
FutureTask.get依赖与任务的状态.如果它已经完成,get可以立刻得到返回结果,否则会被阻塞直到任务转入完成状态,然后完后结果或者抛出异常.
```java
    public V get() throws InterruptedException, ExecutionException {
        int s = state;
        if (s <= COMPLETING)
            s = awaitDone(false, 0L);
        return report(s);
    }
```
- 信号量(semaphore)
计数信号量(Counting semaphore)用来控制能够同时访问某特定资源的活动的数量或者同时执行某一给定操作的数量.计数信号量可以用来实现资源池或者给一个容器限定边界.

关卡(barrier)和闭锁
关卡与闭锁关键的不同在于,所有线程必须同时达到关卡点,才能继续处理.闭锁等待的是事件;关卡等待是其他线程.
Exchanger是关卡的另一种形式,它是一种两步关卡,在关卡点会交换数据.当两方进行的活动不对称时,Exchanger是非常有用的,比如当一个线程向缓冲写入一个数据,这时另一个线程充当消费者使用这个数据;这些线程可以使用Exchanger进行会面,并用完整的缓冲与空缓冲进行交换.当两个线程通过Exchanger交换对象时,交换为双方的对象建立了一个安全的发布.
> 在计算问题中,如果不涉及I/O或者访问共享数据,N<sub>cpu</sub>或者N<sub>cpu</sub>+1个线程产生最优吞吐量;更多的线程也不会有任何帮助,并可能在某种程度上降低性能,因为线程会竞争CPU和内存等资源.

















































