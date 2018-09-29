title: Java源码阅读-ThreadLocal
categories: Java
tags: Java
date: 2015-09-17 15:53:15
---

##### Class Overview
> Implements a thread-local storage, that is, a variable for which each thread has its own value. All threads share the same ThreadLocal object, but each sees a different value when accessing it, and changes made by one thread do not affect the other threads. The implementation supports null values.

##### Summary
- Public Constructors
	ThreadLocal() Creates a new thread-local variable.
- Public Methods
	T get() Returns the value of this variable for the current thread.(如果线程是第一次调用该方法,则创建并初始化此副本.)
    ```
   /**
     * Returns the value in the current thread's copy of this
     * thread-local variable.  If the variable has no value for the
     * current thread, it is first initialized to the value returned
     * by an invocation of the {@link #initialValue} method.
     *
     * @return the current thread's value of this thread-local
     */
    public T get() {
        Thread t = Thread.currentThread();
        ThreadLocalMap map = getMap(t);
        if (map != null) {
            ThreadLocalMap.Entry e = map.getEntry(this);
            if (e != null)
                return (T)e.value;
        }
        return setInitialValue();
    }
    ```
    void remove() Removes the entry for this variable in the current thread.(移除此线程局部变量的值,这可能有助于减少线程局部变量的存储需求.)
    ```
    /**
     * Removes the current thread's value for this thread-local
     * variable.  If this thread-local variable is subsequently
     * {@linkplain #get read} by the current thread, its value will be
     * reinitialized by invoking its {@link #initialValue} method,
     * unless its value is {@linkplain #set set} by the current thread
     * in the interim.  This may result in multiple invocations of the
     * <tt>initialValue</tt> method in the current thread.
     *
     * @since 1.5
     */
     public void remove() {
         ThreadLocalMap m = getMap(Thread.currentThread());
         if (m != null)
             m.remove(this);
     }
    ```
    void set(T value) Sets the value of this variable for the current thread.
    ```
    /**
     * Sets the current thread's copy of this thread-local variable
     * to the specified value.  Most subclasses will have no need to
     * override this method, relying solely on the {@link #initialValue}
     * method to set the values of thread-locals.
     *
     * @param value the value to be stored in the current thread's copy of
     *        this thread-local.
     */
    public void set(T value) {
        Thread t = Thread.currentThread();
        ThreadLocalMap map = getMap(t);
        if (map != null)
            map.set(this, value);
        else
            createMap(t, value);
    }
    ```
- Protected Methods
	T initialValue() Provides the initial value of this variable for the current thread.(最多在线程第一次调用get()方法访问变量时调用此方法一次.如果线程先于get()方法调用set(T)方法,则不会在线程中再调用initialValue()方法.)

ThreadLocal是从JDK1.2版本开始提供的,它不是一个线程,而是一个线程的本地化对象.当某个变量在使用ThreadLocal进行维护时,ThreadLocal为使用改变量的每一个线程分配了一个独立的变量副本,每个线程可以自行操作自己对应的变量副本,而不会影响其他线程的变量副本.
从线程角度看,每个线程都保持一个对其线程局部变量副本的隐式引用,只要线程是活动的并且ThreadLocal实例是可访问的;在线程消失之后,其线程局部实例的所有副本都会被垃圾回收(除非存在对这些副本的其他引用).
- ThreadLocal与Thread同步机制的比较
在同步机制中,通过对象锁机制保证同一个时间只能有一个线程访问变量.而ThreadLocal则从另一个角度解决多线程的并发访问.
概括起来说,对于多线程资源共享的问题,同步机制采用了"以时间换空间"的方式,而ThreadLocal采用了"以空间换时间"的方式.前者仅提供一份变量,让不同的线程排队访问,而后者为一个线程都提供了一份变量,因此同时访问而互不影响.
- final域
final域是不可修改的(尽管如果final域指向的对象是可变的,这个对象仍然可被修改),然而它再Java存储模型中还有着特殊的语义.final域使得确保初始化安全性(initialization safety)成为可能,初始化安全性让不可变性对象不需要同步就能自由地被访问和共享.
> 正如"将所有的域声明为私有的,除非它们需要更高的可见性"一样,"将所有的域声明为final型,除非它们是可变的",也是一条良好的实践.


















