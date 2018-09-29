title: JAVA基础
date: 2015-10-27 19:42:03
categories: Java
tags: Java
---
编程的思路: 将大问题分解成小问题,逐个解决.  
**列举出所有可能出现的情况,不能有遗漏.**  
使用数学方法归纳总结!!!  


- java位操作  
Java提供的位运算符有：左移( << )、右移( >> ) 、无符号右移( >>> ) 、位与( & ) 、位或( | )、位非( ~ )、位异或( ^ )，除了位非( ~ )是一元操作符外，其它的都是二元操作符。
- 创建线程的两种方法  
  
```
public
class Thread implements Runnable {
	/* Make sure registerNatives is the first thing <clinit> does. */
    private static native void registerNatives();
    static {
        registerNatives();
    }  

    /* Whether or not the thread is a daemon thread. */
    private boolean     daemon = false;  

	/* What will be run. */
    private Runnable target;  


    
    
}
```