---
title: "java-concurrency-interview"
date: 2017-05-05 20:22
---
[TOC]

# Java Basics
## 1. equals/'=='/hashCode()
**`==` 其实是真正比较等式两边的值，值无非就是基本数据类型，或者是指针即引用．**使用 `==` 比较对象引用其实就是比较的对象地址，因此两个不同的对象　`==` 必定为false;

```java
System.out.println("------------------string--------------");
String s1 = "abc";
String s2 = "abc";
String s3 = new String("abc");
String s4 = new String("abc");
String s5 = s4.intern();

System.out.println(s1 == s2); // true
System.out.println(s1 == s3); // false
System.out.println(s3 == s4); // false
System.out.println(s4 == s5); // false
System.out.println(s1 == s5); // true
System.out.println(s3.hashCode() == s4.hashCode()); // true
System.out.println(new String("abc").hashCode() == new String("abc").hashCode()); // true
```
`o1.equals(o2)` 方法必须要是非空对象才能调用，通过他来比较两个对象非常灵活，你可以随便定义怎么比较．Object 类是直接比较的对象地址．
由于 java 对于一些基础类重写，因此有些类使用的时候就不是比较地址了，比如 **String 类，基本数据类型对应的类Byte/Character/Short/Integer/Long/Float/Double 都重写了 `equals`**

```java
public boolean equals(Object obj) {
	return (this == obj);
}
```
```java
System.out.println(s3.equals(s4)); //true
```
**`hashCode()` 方法和`equals`一样可以被重写，Object 类的hashCode()并不直接粗暴的返回对象的内存地址，但基本可以这么认为．实际上, `equals()` 和　`hashCode()` 必须要保持一致性，因此重写　equals 的必须重写　hashCode().**
String/Byte/Character/Short/Integer/Long/Float/Double　都重写了． **hashCode 的存在，在　Set/Map 中非常重要，对于集合中元素相等或者key相等的概念，Map 就是先通过 hashCode 计算key 的hash,然后到对应的桶中在根据 equals　方法比较是否相等,
才确定key是否相同.**

```java
System.out.println("------------------integer--------------");
Integer a1 = 10;
Integer a2 = 10;
Integer b1 = 128;
Integer b2 = 128;

System.out.println(a1 == a2); // true
System.out.println(b1 == b2); // false
HashMap<Integer, Integer> maps = new HashMap<Integer, Integer>();
Integer i1 = 1;
Integer i2 = 2048;
Integer i3 = 2;
maps.put(i1, 1);
maps.put(2, 2);
maps.put(2048, 2048); //　Integer 的　hashCode 就返回的是里面的 intValue, equals 方法当然也比较的是 intValue
System.out.println(maps.get(i1)); // 1
System.out.println(maps.get(i2)); // 2048
System.out.println(maps.get(i3)); // 2
```

## 2. compareble 和 comparator 区别
Comparable是排序接口；若一个类实现了Comparable接口，就意味着“该类支持排序”。
而Comparator是比较器；我们若需要控制某个类的次序，可以建立一个“该类的比较器”来进行排序。

我们不难发现：Comparable相当于“内部比较器”，而Comparator相当于“外部比较器”。

## 3. 深度拷贝和浅层拷贝
深度拷贝: 指的是对象中成员变量引用的对象也拷贝一份新的.
浅层拷贝: 指的是只拷贝对象的第一层次的数据, 对象中成员变量引用的对象不做拷贝, Java 中基类Object的 clone 默认就是浅层拷贝, 并且这是一个本地方法.
```java
protected native Object clone() throws CloneNotSupportedException;
```
并且, 一个对象要想进行clone, 必须申明实现 Cloneable 接口, 否则会抛出异常. `First, if the class of this object does not implement the interface Cloneable, then a CloneNotSupportedException is thrown.`
```java
class Student implements Cloneable {
 public Object clone() {
	    	Object o = null;
	    	try {
				o = super.clone();
			} catch (CloneNotSupportedException e) {
				// TODO Auto-generated catch block
				e.printStackTrace();
			}
	    	return o;
	}
}
```
### Java中实现深度拷贝的两种方式: 
改写clone, 递归的进行浅层次复制. Cloneable 接口也是个空接口. 
```java
class Student implements Cloneable {
	    private String name;
	    private int age;
	    private Professor pro;
	    public Student(){}
	    public Student(String name,int age,Professor pro){
	        this.name=name;
	        this.age=age;
	        this.pro=pro;
	    }
	    public Object clone(){
	        Student o=null;
	        try {
	            //Object中的clone()识别出你要复制的是哪一个对象。
	            o=(Student)super.clone();
	        } catch (CloneNotSupportedException e) {
	            System.out.println(e.toString());
	        }
	        o.pro = (Professor)pro.clone();
	        return o;
	    }
}
```
使用 Serializable 进行. Java 中 Serialization 和 Deserialization 会默认进行深层拷贝, 只要对象申明实现 Serializable 接口, 这个接口是个空接口, 没有实现. 主要通过 Object Stream 中的 `writeObject()` 和 `readObject()` 改写.
```java
class Student implements Serializable {
    private String name;
    private int age;
    Professor pro;
    public Student(){}
    public Student(String name,int age,Professor pro){
        this.name=name;
        this.age=age;
        this.pro=pro;
    }
    public Object deepClone() throws IOException, ClassNotFoundException{
        //将对象写到流中
        ByteArrayOutputStream bo=new ByteArrayOutputStream();
        ObjectOutputStream oo=new ObjectOutputStream(bo);
        oo.writeObject(this);
        //从流中读出来
        ByteArrayInputStream bi=new ByteArrayInputStream(bo.toByteArray());
        ObjectInputStream oi=new ObjectInputStream(bi);
        return oi.readObject();
    }
}

class Professor implements Serializable{
    private String name;
    private int age;
}
```
对象并不用实现 Serializable 接口中任何方法, 因为这是一个空接口, 但是如果需要序列化, 必须申明实现该接口. 原因就是 `readObject()` 和 `writeObject()` 需要对象支持该接口. 如果想要改变 序列化 时的默认行为, 也必须改写这两个方法.

在Java语言里深层复制一个对象，常常可以先使对象实现`Serializable`接口(或者其子接口如 `Externalizable`)，然后把对象（实际上只是对象的一个拷贝）写到一个流中，再从流中读出来，便可以重建对象。

这样做的前提是对象以及对象内部所有引用到的对象都是可串行化的，否则，就需要仔细考察那些不可串行化的对象是否设成transient，从而将之排除在复制过程之外.另外, 静态成员是属于类的,不可串行化.

## 4. try catch finally
对于 try 和 finally都有 return语句的时候，返回值覆盖问题的讨论。主要区别就在修改的是引用类型的对象，还是值类型。

```java
public class TryCatchFinally {
	
	public static int test() {
		int num = 10;
		try {
			return num += 10;
		} finally {
			num = 80;
			return num;
		}
	}
	
	public static int test1() {
		int num = 10;
		try {
			return num += 10;
		} finally {
			num = 80;
		}
	}
	
	public static int test2() {
		Number n = new Number(10);
		try {
			return n.num;
		} finally {
			n.num = 80;
		}
	}
	
	public static Number test3() {
		Number n = new Number(10);
		try {
			return n;
		} finally {
			n.num = 80;
		}
	}
	
	
	
	static class Number {
		int num = 0;
		public Number(int i) {
			this.num = i;
		}
	}
	
	public static void main(String[] args) {
		System.out.println(test());
		System.out.println(test1());
		System.out.println(test2());
		System.out.println(test3().num);
	}

}

// 80
// 20
// 10
// 80

```
 
 - 情况一：如果finally中有return语句，则会将try中的return语句”覆盖“掉，直接执行finally中的return语句，得到返回值，这样便无法得到try之前保留好的返回值。
 - 情况二：如果finally中没有return语句，也没有改变要返回值，则执行完finally中的语句后，会接着执行try中的return语句，返回之前保留的值。
 - 情况三：如果finally中没有return语句，但是改变了要返回的值，这里有点类似与引用传递和值传递的区别，分以下两种情况，：
    1. 如果return的数据是基本数据类型或文本字符串，则在finally中对该基本数据的改变不起作用，try中的return语句依然会返回进入finally块之前保留的值。
    2. 如果return的数据是引用数据类型，而在finally中对该引用数据类型的属性值的改变起作用，try中的return语句返回的就是在finally中改变后的该属性的值。

## 5. Java i=i++ 的坑
```java
i = 0;
i = i++;
System.out.println(i);
```
这个实际上i 最后还是0.

# Java Collection Framework
## 1. 那些集合的容量理论(假设内存足够)上是无限的, 哪些是有限制的?
内部基于数组实现的集合, 理论上容量不超过 Integer.MAX_VALUE, 因为数组下标必须是 int 类型, 这些集合包括 ArrayList, ArrayQueue, EnumMap, PriorityQueue

```java
/**
     * The maximum size of array to allocate.
     * Some VMs reserve some header words in an array.
     * Attempts to allocate larger arrays may result in
     * OutOfMemoryError: Requested array size exceeds VM limit
     */
    private static final int MAX_ARRAY_SIZE = Integer.MAX_VALUE - 8;
``` 
内部基于带指针数据结构实现的集合, 比如链表, 树等, 理论上容量是无限的. LinkedList, TreeSet, HashMap由于使用了链地址法避免冲突, 因此理论上也是无限的, HashSet 背后是 HashMap.

## 2. add/offer, remove/poll, element/peek 区别?
A collection designed for holding elements prior to processing. Besides basic Collection operations, queues provide additional insertion, extraction, and inspection operations. Each of these methods exists in two forms: one throws an exception if the operation fails, the other returns a special value (either null or false, depending on the operation). The latter form of the insert operation is designed specifically for use with capacity-restricted Queue implementations; in most implementations, insert operations cannot fail.

| Operations\Type   |Throws exception	|Returns special value|
|------------------:|------------------:|--------------------:|
|Insert	            |add(e)	            |offer(e)             |
|Remove	            |remove()	        |poll()               |
|Examine	        |element()	        |peek()               |


# JVM 相关知识
## 1. static 和 non-static field 区别?
 1. 全局唯一，任何一次的修改都是全局性的影响
 2. 只加载一次，优先于非静态
 3. 使用方式上不依赖于实例对象
 4. 生命周期属于类级别，从JVM 加载开始到JVM卸载结束

## 2. 类加载机制?
**加载, 连接(验证/准备/解析), 初始化, 使用, 卸载. 初始化的时候,按照static block 和 static statement出现的顺序一次执行语句. 对于采用类加载机制的单例模式, 如果私有构造器需要对静态成员变量进行赋值, 我们一定要把 构造单列的语句放在最后面!**
```java
class SingleTon {
	
	private static SingleTon singleTon = new SingleTon();
	public static int count1; //在准备阶段value初始值为0
	public static int count2 = 3; //在准备阶段value初始值为0
	
	//private static SingleTon singleTon = new SingleTon();// 顺序不一样, 结果不一致偶! 
	//类的加载机制! 加载, 连接(验证/准备/解析), 初始化, 使用, 卸载. 初始化的时候,按照static block 和 static statement出现的顺序一次执行语句.
	// 初始化之前, singleton = null, count1=count2=0, 这是虚拟机规范.由于是先执行 singleton = new Singleton(), 因此先执行
	// 私有构造函数, 然后 count1 = 1. count2 = 1. 然后又继续执行 count1没有变化, count2 = 3. 因此最后, count1 = 1, count2 = 3;
	// 对于这种形式的单例模式, 如果私有构造器需要对静态成员变量进行赋值, 我们一定要把 构造单列的语句放在最后面!
	
	private SingleTon() {
		count1++;
		count2++;
	}

	public static SingleTon getInstance() {
		return singleTon;
	}
}

public class Test {
	public static void main(String[] args) {
		SingleTon singleTon = SingleTon.getInstance();
		System.out.println("count1=" + singleTon.count1);
		System.out.println("count2=" + singleTon.count2);
	}
}
```

## 3. AutoBox/unbox技术?
```java
Integer a1 = 10;
Integer a2 = 10;
Integer b1 = 128;
Integer b2 = 128;
System.out.println(a1 == a2); // true
System.out.println(b1 == b2); // 不设置JVM参数的情况下是 false
```
把 `int` 和 `Integer` 之间的隐式转换其实就是 `Integer.valueOf(param)` 和 `Integer.intValue()`; 而在java中, Integer.valueOf() 方法带有缓存机制, 即默认情况下, 对于 -128~127 的整数, 返回的是同一个对象! 对于缓存的上限 128 其实可以通过设置 JVM 参数 `-XX:AutoBoxCacheMax=?` 来更改!

## 4. JVM 内存布局和模型
<div align="center"><img src="/static/images/Java/jvm_memory_arch.png" style="width:700px;height:500px;">
<caption><center> JVM 内存整体布局 </center></caption></div>

<div align="center"><img src="/static/images/Java/stack_heap_info.png" style="width:700px;height:500px;">
<caption><center> JVM stack & heap 详情 </center></caption></div>

<div align="center"><img src="/static/images/Java/java_object.png" style="width:700px;height:500px;">
<caption><center> JVM 内存堆内对象存储 </center></caption></div>

## 5. 方法块和静态语句执行顺序
Java虚拟机在看到左边的类出现的时候，就回去寻找该类的代码，**首先初始化该类的类属性，也就是static属性(包括static variable，static blocks)，按照在代码中出现的先后顺序，依次执行可以执行的初始化语句或者代码块**。

这也是为啥**static里面无法访问在后面顺序申明的静态成员变量**，并且更加不能访问非静态变量即实例变量，因为这不符合**先有类后有类实例的原则**；

并且类在jvm加载一次初始化完成之后就不会再执行初始化了，也就是**所有的static在类被首次load到jvm时仅仅执行一次**.

对于类的实例，则是当java虚拟机扫描到new关键字的时候，就会执行构造函数创建类的实例，此时**非static{}代码块**会被**插入到类的构造函数最前面**(其实就等价于在{}里面的代码写到构造函数最前面一样)。

这也是为啥jvm在扫描类的时候不管{}代码块的原因，因为他是类实例的，和类无关，并且此时的实例变量都是默认值，因为这个实例还没有完全构造出来，{}只是插在构造函数前面的一段代码.
 
 1. jvm解释执行代码时，遇到新的类时，首先加载这个类；
 2. 类初始化过程中，按static顺序执行静态成员初始化语句或static blocks；且只执行一次；
 3. 先有类，后才有类实例原则，静态成员属于类，实例成员属于实例-->一系列规则 
    - 静态方法中可以存取静态变量和其他静态方法，不可以存取实例变量和方法
    - 非静态方法可以存取一切静态方法和静态成员变量
    - 类实例可以调用一切静态方法
 4. 实例从new中产生，必定由构造函数构造出；
 5. 类实例的初始化不管时成员初始化private a = 3;还是{}代码块初始化，实际上都是按照在代码中出现的先后顺序被插入到构造函数最前面；


# 多线程并发初级主题
## 1. <del>什么是线程? 为何使用线程?</del>
线程是操作系统调度执行的基本单位, 进程是操作系统进行资源分配的基本单位. 一个进程内至少有一个主线程作为执行单位执行; 线程比进程更加轻量级, 创建时间更短, 资源消耗更少.

## 2. <del>线程和进程的区别?</del>
线程是操作系统调度执行的基本单位, 进程是操作系统进行资源分配的基本单位. 多个线程共享同一个进程内部的资源, 不同进程之间地址空间相互独立,不共享内存. 进程创建比线程更加耗费资源, 但进程通常更加稳定, Nginx 就采用了多进程的形式, 当然, 多线程环境下, 一个线程的阻塞并不会影响其他线程的执行, 提高的运行效率.

## 4. <del>Java 中如何实现线程? `Runable` 和 `Thread` 的区别?</del>
`Runnable` 是接口, `Thread` 是实现了 `Runnable` 接口的类. Java 中实现线程, 既可以 实现`Runnable ` 接口, 然后讲其丢给`Thread` 启动起来, 也可以直接继承 `Thread` 类而成为一个线程类直接运行. 由于 Java 没有多重继承, 因此, 一般需要继承其他类的线程, 我们直接使用 `Runnable` 接口更好.

## 5. <del>`Thread` 中的 `start()` 和 `run()` 方法区别?</del>
`start()` 方法是启动线程, 而 `run()` 是线程运行的代码, `start()` 方法会调用`run()` 方法; 如果只是调用 `run()` 方法, 那就没有启动线程, 只是普通的方法调用, 即还是在原来的线程之中. 而 `start()` 才会开创一个新线程并在新线程中执行`run()`.

## 6. Java 中的 `Runnable` 和 `Callable` 有何不同?
Runnable和Callable都代表那些要在不同的线程中执行的任务。Runnable从JDK1.0开始就有了，Callable是在JDK1.5增加的。它们的主要区别是**Callable的 call() 方法可以返回值和抛出异常**，而**Runnable的run()方法没有这些功能。Callable可以返回装载有计算结果的Future对象.**

## 7. Java中CyclicBarrier 和 CountDownLatch 有什么不同?
CyclicBarrier 和 CountDownLatch 都可以用来让一组线程等待其它线程。与 CyclicBarrier 不同的是，CountdownLatch 不能重新使用. [教程](http://javarevisited.blogspot.com/2012/07/cyclicbarrier-example-java-5-concurrency-tutorial.html)

CountDownLatch 一般就是一个线程等待所有线程完成之后，主线程开始后续动作；比如服务退出时候，主线程需要等待所有子线程完成任务退出之后才能退出，服务启动的时候，主线程需要等待所有子线程先启动子服务之后才能继续启动；应用非常广泛，python 中可以自己实现；
```java
import java.util.Date;
import java.util.concurrent.CountDownLatch;
import java.util.logging.Level;
import java.util.logging.Logger;

/**
 * Java program to demonstrate How to use CountDownLatch in Java. CountDownLatch is
 * useful if you want to start main processing thread once its dependency is completed
 * as illustrated in this CountDownLatch Example
 * 
 * @author Javin Paul
 */
public class CountDownLatchDemo {

    public static void main(String args[]) {
       final CountDownLatch latch = new CountDownLatch(3);
       Thread cacheService = new Thread(new Service("CacheService", 1000, latch));
       Thread alertService = new Thread(new Service("AlertService", 1000, latch));
       Thread validationService = new Thread(new Service("ValidationService", 1000, latch));
      
       cacheService.start(); //separate thread will initialize CacheService
       alertService.start(); //another thread for AlertService initialization
       validationService.start();
      
       // application should not start processing any thread until all service is up
       // and ready to do there job.
       // Countdown latch is idle choice here, main thread will start with count 3
       // and wait until count reaches zero. each thread once up and read will do
       // a count down. this will ensure that main thread is not started processing
       // until all services is up.
      
       //count is 3 since we have 3 Threads (Services)
      
       try{
            latch.await();  //main thread is waiting on CountDownLatch to finish
            System.out.println("All services are up, Application is starting now");
       }catch(InterruptedException ie){
           ie.printStackTrace();
       }
      
    }
  
}

/**
 * Service class which will be executed by Thread using CountDownLatch synchronizer.
 */
class Service implements Runnable{
    private final String name;
    private final int timeToStart;
    private final CountDownLatch latch;
  
    public Service(String name, int timeToStart, CountDownLatch latch){
        this.name = name;
        this.timeToStart = timeToStart;
        this.latch = latch;
    }
  
    @Override
    public void run() {
        try {
            Thread.sleep(timeToStart);
        } catch (InterruptedException ex) {
            Logger.getLogger(Service.class.getName()).log(Level.SEVERE, null, ex);
        }
        System.out.println( name + " is Up");
        latch.countDown(); //reduce count of CountDownLatch by 1
    }
  
}

Output:
ValidationService is Up
AlertService is Up
CacheService is Up
All services are up, Application is starting now
```

CyclicBarrier 一般是线程之间相互等待，如果其他线程没有抵达，那么自己就会阻塞在栅栏上，直到所有线程都到达某一个点，之后再共同进行下一步！其实　CyclicBarrier 可以完成 CountDownLatch 的工作; 比如主线程开始等待，从线程也等待，所有从线程也抵达栅栏的时候，主线程也就可以继续进行了，所有从线程可以同时进行后续工作或者同时退出．
CyclicBarrier 可以用来实现多线程的mapreduce 操作, 当然　CountDownLatch 也可以；
```java
import java.util.concurrent.BrokenBarrierException;
import java.util.concurrent.CyclicBarrier;
import java.util.logging.Level;
import java.util.logging.Logger;

/**
 * Java program to demonstrate how to use CyclicBarrier in Java. CyclicBarrier is a
 * new Concurrency Utility added in Java 5 Concurrent package.
 *
 * @author Javin Paul
 */
public class CyclicBarrierExample {

    //Runnable task for each thread
    private static class Task implements Runnable {

        private CyclicBarrier barrier;

        public Task(CyclicBarrier barrier) {
            this.barrier = barrier;
        }

        @Override
        public void run() {
            try {
                System.out.println(Thread.currentThread().getName() + " is waiting on barrier");
                barrier.await();
                System.out.println(Thread.currentThread().getName() + " has crossed the barrier");
            } catch (InterruptedException ex) {
                Logger.getLogger(CyclicBarrierExample.class.getName()).log(Level.SEVERE, null, ex);
            } catch (BrokenBarrierException ex) {
                Logger.getLogger(CyclicBarrierExample.class.getName()).log(Level.SEVERE, null, ex);
            }
        }
    }

    public static void main(String args[]) {

        //creating CyclicBarrier with 3 parties i.e. 3 Threads needs to call await()
        final CyclicBarrier cb = new CyclicBarrier(3, new Runnable(){
            @Override
            public void run(){
                //This task will be executed once all thread reaches barrier
                System.out.println("All parties are arrived at barrier, lets play");
            }
        });

        //starting each of thread
        Thread t1 = new Thread(new Task(cb), "Thread 1");
        Thread t2 = new Thread(new Task(cb), "Thread 2");
        Thread t3 = new Thread(new Task(cb), "Thread 3");

        t1.start();
        t2.start();
        t3.start();
      
    }
}

Output:
Thread 1 is waiting on barrier
Thread 3 is waiting on barrier
Thread 2 is waiting on barrier
All parties have arrived at barrier, lets play
Thread 3 has crossed the barrier
Thread 1 has crossed the barrier
Thread 2 has crossed the barrier
```


## 8. <del>Java 内存模型?(<<Java并发编程实践>>第16章)</del>

## 9. Java 的 `volatile` 变量是什么?
**`volatile`修饰符只能修饰成员变量, 他使得多线程环境下一个线程对变量的写操作对其他读线程立即可见!相当于指明变量不经允许缓存cache.** `volatile`变量可以保证下一个读取操作会在前一个写操作之后发生，就是上一题的`volatile`变量规则。仅仅**当只有一个写线程,其他都是读线程的并发环境下,才能只用该变量而不用枷锁来实现线程安全!**

## 10. 什么是线程安全? `Vector` 是一个线程安全类吗?
**多线程在运行同一段代码的时候, 每一次结果都和预期的一致, 就是线程安全的.** 集合包括线程安全和不安全的, `Vector/Stack/HashTable` 都是通过同步方法实现了线程安全的集合.但是**请注意的是, 集合本身线程安全并不代表直接使用他们的程序就是线程安全了.**

## 11. Java中的 race condition 竞争条件?举例说明
竞态条件会导致程序在并发情况下出现一些bugs。多线程对一些资源的竞争的时候就会产生竞态条件，如果首先要执行的程序竞争失败排到后面执行了，那么整个程序就会出现一些不确定的bugs。这种bugs很难发现而且会重复出现，因为线程间的随机竞争。一个例子就是无序处理. 原因就是本来需要是原子操作的部分, 因为无序随机竞争而导致的
```java
count++;
// 拆分成指令
1. read count from memory
2. update count
3. write updated count to memory
```
以上三个部分每一句都可能被打断转而执行另一个线程.

### check and act
经典的单例模式的错误写法. T1 执行比较后进入if, 此时  CPU 切换运行 T2, T2又发现是 null.
```java
public Singleton getInstance() {
    if (_instance == null) {// race condition if two threads sees _instance=null
        _instance = new Singleton();
    }
}
```
### read-modify-update
使用线程安全的集合,不加同步就一定能写出线程安全的代码吗?
```java
if(!hashTable.contains(key)) {
    hashTable.put(key,value);
}
```
虽然** HashTable 的 put 操作和 contains 操作都是线程安全的**, 但是, 这里我们的** 判断+赋值 作为一个整体必须要是 原子性的**, 因此这里就发生了线程不安全. T1 判断该 key 不存在后进入if, 此时CPU切换至 T2 做同样的判断, 又进入 if, 那么两次就会造成无法预期的结果.

## 12. <del>Java 中如何停止一个线程?</del>
现在的Java没有显式的方法停止一个线程, 要么让 `run` 方法执行完, 要么**通过一个 标志变量 让 `run` 方法退出.**

## 13. 线程运行时发生异常会怎么样?
**如果异常没有被捕获该线程将会停止执行.**`Thread.UncaughtExceptionHandler`是用于处理未捕获异常造成线程突然中断情况的一个内嵌接口。**当一个未捕获异常将造成线程中断的时候JVM会使用`Thread.getUncaughtExceptionHandler()`来查询线程的`UncaughtExceptionHandler`并将线程和异常作为参数传递给`handler`的`uncaughtException()`方法进行处理。**

## 14. <del>多个线程如何共享数据?(线程间通讯)</del>
wait/notify/notifyAll

## 15. <del>Java 中 notify 和 notifyAll 的区别?</del>
notify()方法不能唤醒某个具体的线程，所以只有一个线程在等待的时候它才有用武之地。而notifyAll()唤醒所有线程并允许他们争夺锁确保了至少有一个线程能继续运行。[notify vs notifyAll in Java ](http://javarevisited.blogspot.com/2012/10/difference-between-notify-and-notifyall-java-example.html)

## 16. 为何 wait, notify, notifyAll 这些方法不在 Thread 类中, 而是在 Object 类中?
因为 Java 线程锁并不是全局锁,(Python线程使用的就是解释器进程的全局锁GIL, 锁是线程级别的, 每一个线程要想运行需要获得唯一的一把全局锁, 所以只能串行的执行, 导致Python多线程无法真正利用多核, 因为锁的粒度太大了). 每一个线程只是局部需要枷锁, 甚至就是某个对象的访问需要枷锁, 因此 Java 的锁粒度非常细化, 他能细化到一个对象上, 比如 AtomicInteger. 因此, 锁是对象级别的而不是线程级别的, 如果线程需要获得某些锁来访问某些资源(比如对象), 那么每个对象就应该有一把锁. [我的博客]()

## 17. ThreadLocal 变量是什么?
hreadLocal是Java里一种特殊的变量。每个线程都有一个ThreadLocal就是每个线程都拥有了自己独立的一个变量，竞争条件被彻底消除了。它是为创建代价高昂的对象获取线程安全的好方法，比如你可以用ThreadLocal让SimpleDateFormat变成线程安全的，因为那个类创建代价高昂且每次调用都需要创建不同的实例所以不值得在局部范围使用它，如果为每个线程提供一个自己独有的变量拷贝，将大大提高效率。首先，通过复用减少了代价高昂的对象的创建个数。其次，你在没有使用高代价的同步或者不变性的情况下获得了线程安全。线程局部变量的另一个不错的例子是ThreadLocalRandom类，它在多线程环境中减少了创建代价高昂的Random对象的个数。

## 18. 什么是 FutureTask ?(不熟悉)
在Java并发程序中FutureTask表示一个可以取消的异步运算。它有启动和取消运算、查询运算是否完成和取回运算结果等方法。只有当运算完成的时候结果才能取回，如果运算尚未完成get方法将会阻塞。一个FutureTask对象可以对调用了Callable和Runnable的对象进行包装，由于FutureTask也是调用了Runnable接口所以它可以提交给Executor来执行。

### 18.1 异步编程（异步计算和异步IO）
计算机科学和网络中经常会出现异步的概念，这个概念就是双方交互的过程，主动方在发起请求(函数调用)之后，不需要等待这个请求完成就可以继续往后执行。

异步编程提供了一个非阻塞的，事件驱动的编程模型。 这种编程模型利用系统中多核执行任务来提供并行，因此提供了应用的吞吐率。此处吞吐率是指在单位时间内所做任务的数量。 在这种编程方式下， 一个工作单元将独立于主应用线程而执行， 并且会将它的状态通知调用线程：成功，处理中或者失败。

### 18.2 异步IO
最典型的应用就是 基于非阻塞IO的Web 服务器比如tornado, tornado 实际上是一个利用底层非阻塞操作系统调用select/poll/epoll等方式，对于发来的请求（读事件）和发送的响应（写事件）进行循环扫描监听，从而实现对请求的高并发处理。而对于上层业务逻辑的Handler其实就是给Tornado框架的callback, Tornado 知道请求做完之后，需要发送回响应给客户端。
但是回调的方式容易出现　callback hell. 一般在 tornado.ioloop.IOLoop.instance()　启动的时候，需要设置很多服务，这些服务有先后依赖关系，就会出现 在回调函数里面继续使用另一个回调函数的情况，这样有很多服务的话就会出现层层嵌套。
```python
from tornado import ioloop
class ServiceBase():
    def __init__(self):
        self.__http_client = AsyncHTTPClient()
        
    def start_register(self, delay=5):
        io = ioloop.IOLoop.instance()
        self._register_thread = io.add_timeout(delay, self.register)
     
    def register(self):
        self.registerA()
     
    def on_fail(self):
        self.start_register(delay=30)
        
    def registerA(self):
        self.__http_client.fetch('', self.on_registerA_succ, self.on_fail)
    
    def on_registerA_succ(self):
        self.registerB()
        
    def registerB(self):
        self.__http_client.fetch('', self.on_registerB_succ, self.on_fail)
        
    def on_registerB_succ(self):
        self.registerC()
    
    def registerC(self):
        .....
        
    .....
    
    
```
### 18.3 异步计算
异步还可以是异步计算任务的执行，比如多线程中，主线程提交一个CPU密集型的任务给一个线程池，线程池函数立即返回，这样主线程就不会因为等待计算这个CPU密集任务而阻塞了。这里就会看到，　Runable 只是简单的执行一段任务，但是无法获取任务的结果，任务是没有返回值的，而　Callable 是一个泛型接口，是可以获取返回值的，他一般和　Future 一起使用，Future 是一个接口。

```java
public interface Runnable {
    public abstract void run();
}

public interface Callable<V> {
    /**
     * Computes a result, or throws an exception if unable to do so.
     *
     * @return computed result
     * @throws Exception if unable to compute a result
     */
    V call() throws Exception;
}

```

Callable并不像Runnable那样通过Thread的start方法就能启动实现类的run方法，所以它通常利用ExecutorService的submit方法去启动call方法自执行任务，而ExecutorService的submit又返回一个Future类型的结果，因此Callable通常也与Future一起使用.
```java
ExecutorService pool = Executors.newCachedThreadPool();
     Future future = pool.submit(new Callable{
           public void call(){
                   //TODO
           }
    });
```

因为Future只是一个接口，所以是无法直接用来创建对象使用的，因此就有了下面的FutureTask。
```java
public interface Future<V> {
    //
    boolean cancel(boolean mayInterruptIfRunning);
    
    
    boolean isCancelled();
    
    
    boolean isDone();
    //用来获取执行结果，这个方法会产生阻塞，会一直等到任务执行完毕才返回；
    V get() throws InterruptedException, ExecutionException;

    //用来获取执行结果，如果在指定时间内，还没获取到结果，就直接返回null。
    V get(long timeout, TimeUnit unit)
        throws InterruptedException, ExecutionException, TimeoutException;
}
```
FutureTask实现了RunnableFuture接口，提供了即可以使用Runnable来执行任务，又可以使用Future执行任务并取得结果的构造器，所以可以利用FutureTask去封装Runnable或Callable对象，之后再submit任务;

```java
FutureTask(Callable callable) FutureTask(Runnable runnable, V result)
 
FutureTask task = new FutureTask(new Callable{
        public void call(){
              //TODO
        }
  });
 Thead thread = new Thread(task);
 thread.start();
```

Java 7 之前的Future模式的缺点

　- Future虽然可以实现获取异步执行结果的需求，但是它没有提供通知的机制，我们无法得知Future什么时候完成。
 - 要么使用阻塞，在future.get()的地方等待future返回的结果，这时又变成同步操作。要么使用isDone()轮询地判断Future是否完成，这样会耗费CPU的资源。

### 18.4 java 8 CompletableFuture
其实这个东西在 python 里面早就有了，就是　future.add_done_callback(done_callback)，异步计算任务结果拿到之后，可以继续调用这个注册的done_callback 处理计算结果！ 这样就可以实现任务完成之后的通知机制了。
另外，我们所熟悉的 java awt swing 中注册摸个组件的监听器，就是你给某个　Button 注册一个Listener，监听器写的是你的处理逻辑代码（比如通过网络发送用户和密码登录），然后窗口主线程监听到点击事件之后，默认只会在自己的主线程执行你的监听代码，因此如果你的监听代码如果是耗时的，最好是开一个新线程来执行，否则会阻塞主循环。但这个模式是和我们异步编程反过来的，这个一般是主线程（窗口主线程）执行你的给的回调函数（监听器），而在异步编程中，一般是其他的非主线程（如线程池）执行我们主线程给出的回调函数，线程池执行完异步任务之后，可以拿到你给的　done_callback 继续执行任务完成之后的代码。

Java8 中的CompletableFuture能够将回调放到与任务不同的线程中执行，也能将回调作为继续执行的同步函数，在与任务相同的线程中执行。它避免了传统回调最大的问题，那就是能够将控制流分离到不同的事件处理器中。

CompletableFuture弥补了Future模式的缺点。在异步的任务完成后，需要用其结果继续操作时，无需等待。可以直接通过thenAccept、thenApply、thenCompose等方式将前面异步处理的结果交给另外一个异步事件处理线程来处理。


## 19. Java 中的 interrupted 和 isInterrupted 区别?(不熟悉)
**`interrupted()` 和 `isInterrupted()` 的主要区别是前者会将中断状态清除而后者不会。Java多线程的中断机制是用内部标识来实现的，调用Thread.interrupt()来中断一个线程就会设置中断标识为true。**当中断线程调用静态方法Thread.interrupted()来检查中断状态时，中断状态会被清零。而非静态方法isInterrupted()用来查询其它线程的中断状态且不会改变中断状态标识。简单的说就是任何抛出InterruptedException异常的方法都会将中断状态清零。无论如何，一个线程的中断状态有有可能被其它线程调用中断来改变。

## 20. 为何wait 和 notify 方法要在同步块中调用?(不熟悉)
主要是因为**Java API强制要求这样做，如果你不这么做，你的代码会抛出IllegalMonitorStateException异常**。还有一个原因是为了**避免wait和notify之间产生竞态条件。**[Why wait notify and notifyAll called from synchronized block or method in Java](http://javarevisited.blogspot.sg/2011/05/wait-notify-and-notifyall-in-java.html)

## 21. 为什么要***在循环中而不是直接使用一个if else***检查等待条件?
 - 处于等待状态的线程可能会收到**错误警报和伪唤醒**, **如果不在循环中检查等待条件而是直接ifelse,程序就会在没有满足结束条件的情况下退出**
 - 当一个等待着的线程醒来时, 不能认为它原来的等待状态仍然是有效的, **在 notify() 方法调用之后和等待线程醒来之前这段时间它可能会改变状态**

## 22. Java 中的同步集合与并发集合区别?
相同点:
 
 - Synchronized 和 Concurrent 集合都有 线程安全类(thread-safe class)
 - 但是性能和可扩展性以及实现机制都不同

不同点:

 - Synchronized 集合比如 synchronized HashMap, HashTable, HashSet, Vector以及 synchronized ArrayList要比 ConcurrentHashMap, CopyOnWriteArrayList 和 CopyOnWriteHashSet 慢!
 - **枷锁粒度不同, HashSet, Vector 单个操作就锁住的是整个集合,** 而** ConcurrentHashMap 锁住的是部分集合, 他将内部Map分段, 当一个段被操作的时候, 其他段还可以继续被访问. CopyOnWriteArrayList 允许多个多线程同时访问,直到一个写发生,他会复制整个List然后和新的交换.**

### 22.1 锁优化
减少线程之间锁竞争的方法，就是通过缩小锁的冲突域，比如数据结构分段枷锁，锁的分配算法等。最常见的方式是
 - 减小锁的粒度，比如从一个方法枷锁改成某个语句枷锁
 - 锁分离技术，读写锁分开，比如 LinkedBlockingQueue
 - 锁粗话，就是有多个地方不停的获取释放锁，考虑合并成一个大锁
 - 无锁编程，比如采用 CAS 原语，当然　CAS 也有缺点，还是有总线竞争，CPU cache 导致的　false sharing 和轮训浪费CPU的缺点。更高性能的无锁方式最著名的是 [Disruptor](https://martinfowler.com/articles/lmax.html)，基于消除这些缺点之后的 CAS 和　循环缓冲区实现，性能非常强悍，多用在金融交易系统，Storm 也使用了这个框架

## 23. <del>Java 堆和栈区别?</del>

## 24. <del>线程池是什么? 为何使用线程池?</del>
线程池是有一系列**已经创建的线程组成, 他们内部使用一个队列来存储需要执行的任务, 然后调度线程池中的线程从队列中去任务去执行. **

线程虽然比进程轻量, 但是**创建线程还是会耗费时间和资源的, 如果任务到来的时候才创建线程, 响应时间就会边长, 另外, 线程池还可以实现线程的复用, 不用浪费资源.**


# 多线程并发高级主题
## 25. 如何编写代码解决生产者消费者问题?
 低级的办法是 wait/notify 方法, 高级的办法是 Semaphore 或者 BlockingQueue. [教程](http://javarevisited.blogspot.sg/2012/02/producer-consumer-design-pattern-with.html)
 
## 26. 如何避免死锁?
Java多线程中的死锁: 死锁是指两个或两个以上的进程在执行过程中, 因争夺资源而造成的一种相互等待的现象. 死锁会导致程序被挂起而无法执行.

死锁发生的四个条件:
 - 互斥条件: 一个资源每次只能被一个进程使用.
 - 请求与保持条件: 一个进程因为请求资源而阻塞时, 对已获得的资源保持不放.
 - 不剥夺条件: 进程已获得的资源, 在未使用完之前, 不能强行剥夺.
 - 循环等待条件: 若干进程之间形成一种头尾相接的循环等待资源关系.

避免死锁最简单的方法就是阻止循环等待条件.

## 27. Java 中的活锁和死锁有什么区别?
活锁和死锁的主要区别是前者进程的状态可以改变但是却不能继续执行。

## 28. 怎么检测一个线程是否拥有锁？
在`java.lang.Thread`中有一个方法叫`holdsLock()`，它返回true如果当且仅当当前线程拥有某个具体对象的锁.

## 29. 你如何在Java中获取线程堆栈？
对于不同的操作系统，有多种方法来获得Java进程的线程堆栈。当你获取线程堆栈时，JVM会把所有线程的状态存到日志文件或者输出到控制台。在Windows你可以使用`Ctrl + Break`组合键来获取线程堆栈，Linux下用`kill -3`命令。你也可以用`jstack`这个工具来获取，它对线程`id`进行操作，你可以用`jps`这个工具找到`id`。

## 30. <del>JVM中哪个参数是用来控制线程的栈大小的?</del>
这个问题很简单， `-Xss`参数用来控制线程的堆栈大小。你可以查看JVM配置列表`java -XX:+PrintFlagsFinal -version | grep -iE 'HeapSize|PermSize|ThreadStackSize'`来了解这个参数的更多信息。

## 31. Java中 `synchronized` 和 `ReentrantLock` 有什么不同？
Java在过去很长一段时间只能通过 `synchronized`关键字来实现互斥，它有一些缺点。比如你不能扩展锁之外的方法或者块边界，尝试获取锁时不能中途取消等。Java 5 通过Lock接口提供了更复杂的控制来解决这些问题。 ReentrantLock 类实现了 Lock，它拥有与 synchronized 相同的并发性和内存语义且它还具有可扩展性。

## 32. <del>有三个线程T1，T2，T3，怎么确保它们按顺序执行？</del>
在多线程中有多种方法让线程按特定顺序执行，你可以用**线程类的join()方法在一个线程中启动另一个线程，另外一个线程完成该线程继续执行。T3 调用 T2(然后 T2.join()), T2 调用 T1(然后 T1.join()).**

## 33. Thread类中的`yield`方法有什么作用？
Yield方法可以暂停当前正在执行的线程对象，让其它有相同优先级的线程执行。它是一个静态方法而且只保证当前线程放弃CPU占用而不能保证使其它线程一定能占用CPU，执行`yield()`的线程有可能在进入到暂停状态后马上又被执行。

## 34. Java中ConcurrentHashMap的并发度是什么？
**`ConcurrentHashMap`把实际map划分成若干部分来实现它的可扩展性和线程安全。**这种划分是使用并发度获得的，它是ConcurrentHashMap类构造函数的一个可选参数，默认值为16，这样在多线程情况下就能避免争用

## 35. Java中Semaphore是什么？
Java中的`Semaphore`是一种新的同步类，它是一个计数信号。从概念上讲，信号量维护了一个许可集合。如有必要，在许可可用前会阻塞每一个 `acquire()`，然后再获取该许可。每个 `release()`添加一个许可，从而可能释放一个正在阻塞的获取者。但是，不使用实际的许可对象，`Semaphore`只对可用许可的号码进行计数，并采取相应的行动。信号量常常用于多线程的代码中，比如数据库连接池。

## 36. 如果你提交任务时，线程池队列已满。会时发会生什么？
这个问题问得很狡猾，许多程序员会认为该任务会阻塞直到线程池队列有空位。**事实上如果一个任务不能被调度执行那么ThreadPoolExecutor’s submit()方法将会抛出一个`RejectedExecutionException`异常。**

## 37. Java线程池中submit() 和 execute()方法有什么区别？
两个方法都可以向线程池提交任务，execute()方法的返回类型是void，它定义在Executor接口中, 而submit()方法可以返回持有计算结果的Future对象，它定义在ExecutorService接口中，它扩展了Executor接口，其它线程池类像ThreadPoolExecutor和ScheduledThreadPoolExecutor都有这些方法。更多详细信息请点击这里。

## 38. 什么是阻塞式方法？
阻塞式方法是指程序会一直等待该方法完成期间不做其他事情，ServerSocket的accept()方法就是一直等待客户端连接。这里的阻塞是指调用结果返回之前，当前线程会被挂起，直到得到结果之后才会返回。此外，还有异步和非阻塞式方法在任务完成前就返回。更多详细信息请点击这里。

## 39. Java中invokeAndWait 和 invokeLater有什么区别？
这两个方法是Swing API 提供给Java开发者用来从当前线程而不是事件派发线程更新GUI组件用的。InvokeAndWait()同步更新GUI组件，比如一个进度条，一旦进度更新了，进度条也要做出相应改变。如果进度被多个线程跟踪，那么就调用invokeAndWait()方法请求事件派发线程对组件进行相应更新。而invokeLater()方法是异步调用更新组件的。更多详细信息请点击这里。


## 40. 如何在Java中创建Immutable对象？
这个问题看起来和多线程没什么关系， 但不变性有助于简化已经很复杂的并发程序。Immutable对象可以在没有同步的情况下共享，降低了对该对象进行并发访问时的同步化开销。可是Java没有@Immutable这个注解符，要创建不可变类，要实现下面几个步骤：通过构造方法初始化所有成员、对变量不要提供setter方法、将所有的成员声明为私有的，这样就不允许直接访问这些成员、在getter方法中，不要直接返回对象本身，而是克隆对象，并返回对象的拷贝。[教程](http://javarevisited.blogspot.com/2013/03/how-to-create-immutable-class-object-java-example-tutorial.html)

## 41. Java中的ReadWriteLock是什么？
一般而言，**读写锁**是用来提升并发程序性能的**锁分离技术**的成果。Java中的ReadWriteLock是**Java 5 中新增的一个接口，一个ReadWriteLock维护一对关联的锁，一个用于只读操作一个用于写**。**在没有写线程的情况下一个读锁可能会同时被多个读线程持有。写锁是独占的，你可以使用JDK中的ReentrantReadWriteLock来实现这个规则，它最多支持65535个写锁和65535个读锁。**

## 42. 多线程中的忙循环是什么?
忙循环就是程序员用循环让一个线程等待，不像传统方法wait(), sleep() 或 yield() 它们都放弃了CPU控制，而忙循环不会放弃CPU，**它就是在运行一个空循环**。这么做的目的是为了**保留CPU缓存**，在多核系统中，一个等待线程醒来的时候可能会在另一个内核运行，这样会重建缓存。为了避免重建缓存和减少等待重建的时间就可以使用它了。

## 43. volatile 变量和 atomic 变量有什么不同？
Volatile变量可以确保先行关系，即写操作会发生在后续的读操作之前, **但它并不能保证原子性。**例如用volatile修饰count变量那么 `count++` 操作就不是原子性的。而 `AtomicInteger` 类提供的 `atomic` 方法可以让这种操作具有原子性如getAndIncrement()方法会原子性的进行增量操作把当前值加一，其它数据类型和引用变量也可以进行相似操作。

## 44. 如果同步块内的线程抛出异常会发生什么？(88)
这个问题坑了很多Java程序员，若你能想到锁是否释放这条线索来回答还有点希望答对。无论你的同步块是正常还是异常退出的，里面的线程都会释放锁，所以对比锁接口我更喜欢同步块，因为它不用我花费精力去释放锁，该功能可以在finally block里释放锁实现。

## 45. <del>单例模式的双检锁是什么？</del>
[参考](http://javarevisited.blogspot.com/2012/12/how-to-create-thread-safe-singleton-in-java-example.html)

## 46. <del>如何在Java中创建线程安全的Singleton？</del>
这是上面那个问题的后续，如果你不喜欢双检锁而面试官问了创建Singleton类的替代方法，你可以利用JVM的类加载和静态变量初始化特征来创建Singleton实例，或者是利用枚举类型来创建Singleton，我很喜欢用这种方法。[enum](http://javarevisited.blogspot.com/2012/07/why-enum-singleton-are-better-in-java.html)

## 47. 写出3条你遵循的多线程最佳实践

 - **给你的线程起个有意义的名字**
    * 这样可以方便找bug或追踪。OrderProcessor, QuoteProcessor or TradeProcessor 这种名字比 Thread-1. Thread-2 and Thread-3 好多了，给线程起一个和它要完成的任务相关的名字，所有的主要框架甚至JDK都遵循这个最佳实践。
 - **避免锁定和缩小同步的范围**
    * 锁花费的代价高昂且上下文切换更耗费时间空间，试试最低限度的使用同步和锁，缩小临界区。因此相对于同步方法我更喜欢同步块，它给我拥有对锁的绝对控制权。
 - **多用同步类少用wait 和 notify**
    * 首先，CountDownLatch, Semaphore, CyclicBarrier 和 Exchanger 这些同步类简化了编码操作，而用wait和notify很难实现对复杂控制流的控制。其次，这些类是由最好的企业编写和维护在后续的JDK中它们还会不断优化和完善，使用这些更高等级的同步工具你的程序可以不费吹灰之力获得优化。
 - **多用并发集合少用同步集合**
    * 这是另外一个容易遵循且受益巨大的最佳实践，并发集合比同步集合的可扩展性更好，所以在并发编程时使用并发集合效果更好。如果下一次你需要用到map，你应该首先想到用ConcurrentHashMap。我的文章Java并发集合有更详细的说明。

## 48. Java中的fork join框架是什么？
fork join框架是JDK7中出现的一款高效的工具，Java开发人员可以通过它充分利用现代服务器上的多处理器。**它是专门为了那些可以递归划分, 分治并行成许多子模块设计的，目的是将所有可用的处理能力用来提升程序的性能。**fork join框架一个巨大的优势是它使用了工作窃取算法，可以完成更多任务的工作线程可以从其它线程中窃取任务来执行。

## 49. <del>Java多线程中调用wait() 和 sleep()方法有什么不同？</del>
Java程序中wait 和 sleep都会造成某种形式的暂停，它们可以满足不同的需要。**wait()方法用于线程间通信，如果等待条件为真且其它线程被唤醒时它会释放锁**，而**sleep()方法仅仅释放CPU资源或者让当前线程停止执行一段时间，但不会释放锁.**

## 50. notify 会立即释放锁么？
**不会！！ 只有退出 synchronized 同步块之后才会释放锁，这和 wait 是不一样的， wait 调用之后立即释放锁，并挂起**。

## 51. 实现三个线程分别打印ABC,但是打印按照 ABCABCABC循环进行。
同步信号的设计和选择非常重要，如果你选择正确的同步信号，代码会非常容易写。如果选错了，代码非常容易出错。

### 分析
要让三个线程最终按顺序打印，其实最终的执行是串行互斥的，就是线程A 打印的时候，B 和 C 都是不能打印的，这其实是一种互斥。另外，需要一个依赖顺序，对于如何控制这个顺序，就需要靠我们设计了。可以采用一个全局控制变量，控制三个线程的轮换执行，然后相互协调通知，比如 `A=ture,B=false,C=false` 表示目前A可以执行，完了之后设置 `A=false,B=true,C=false` 表示目前B可以执行，以此类推。我开始选择的同步信号不合理，导致代码死活写不出来，
### 版本1(synrhonized/wait/notify)
```java
public class ABCSyncNotifyWell {
	// 实际上三个线程互斥的， 同一个时刻只有一个线程执行。我们的条件变量应该是控制全局状态，每个线程根据这个全局状态决定是等待还是执行。条件控制变量的设计合理性非常重要！
	public static void main(String[] args) {
		Object lockABC = new Object();
		Signal s = new Signal();
		Thread ta = new Thread(new A(lockABC, s));
		Thread tb = new Thread(new B(lockABC, s));
		Thread tc = new Thread(new C(lockABC, s));
		ta.start();
		tb.start();
		tc.start();
	}
	
	private static class Signal {
		boolean A = true;
		boolean B = false;
		boolean C = false;
	}
	
	private static class A implements Runnable {

		private Object lockABC = null;
		private Signal s = null;
		public A(Object lock, Signal s) {
			this.lockABC = lock;
			this.s = s;
		}
		@Override
		public void run() {
			while (true) {
				synchronized(this.lockABC) {
					while (!s.A) {
						try {
							lockABC.wait();
						} catch (InterruptedException e) {
							// TODO Auto-generated catch block
							e.printStackTrace();
						}
					}
					try {
						Thread.sleep(100);
					} catch (InterruptedException e) {
						// TODO Auto-generated catch block
						e.printStackTrace();
					}
					
					System.out.println("A");
					this.s.A = false;
					this.s.B = true;
					this.s.C = false;
					lockABC.notifyAll();
					
				}
			}
			
		}
		
	}

	private static class B implements Runnable {

		private Object lockABC = null;
		private Signal s = null;
		public B(Object lock, Signal s) {
			this.lockABC = lock;
			this.s = s;
		}
		@Override
		public void run() {
			while (true) {
				synchronized(this.lockABC) {
					while (!s.B) {
						try {
							lockABC.wait();
						} catch (InterruptedException e) {
							// TODO Auto-generated catch block
							e.printStackTrace();
						}
					}
					try {
						Thread.sleep(100);
					} catch (InterruptedException e) {
						// TODO Auto-generated catch block
						e.printStackTrace();
					}
					System.out.println("B");
					this.s.A = false;
					this.s.B = false;
					this.s.C = true;
					lockABC.notifyAll();
					
				}
			}
			
		}
		
	}

	private static class C implements Runnable {
		private Object lockABC = null;
		private Signal s = null;
		public C(Object lock, Signal s) {
			this.lockABC = lock;
			this.s = s;
		}
		@Override
		public void run() {
			while (true) {
				synchronized(this.lockABC) {
					while (!s.C) {
						try {
							lockABC.wait();
						} catch (InterruptedException e) {
							// TODO Auto-generated catch block
							e.printStackTrace();
						}
					}
					try {
						Thread.sleep(100);
					} catch (InterruptedException e) {
						// TODO Auto-generated catch block
						e.printStackTrace();
					}
					System.out.println("C");
					this.s.A = true;
					this.s.B = false;
					this.s.C = false;
					lockABC.notifyAll();
				}
			}
			
		}
	}
}

```
### 版本2(Condition/Lock)
这个同步信号设计的是，**通过全局计数器整除3的结果，来判断轮到谁执行。当然全局计数器累加也是互斥的**！ 并且这里采用更加高级的**条件变量，这样允许更多的谓词信号单独发送，也可以说是精准发送和唤醒**。前面使用基本同步原语是无法精确提醒的，因为只能使用单一谓词条件。

```java
public class ABCCondition {
	private static Lock lock = new ReentrantLock();
	private static int conditionVar = 0;
	private static Condition A = lock.newCondition();
	private static Condition B = lock.newCondition();
	private static Condition C = lock.newCondition();

	static class A extends Thread {

		@Override
		public void run() {
			lock.lock();
			try {
				for (int i = 0; i < 10; i ++) {
					while (conditionVar % 3 != 0) {
						A.await();
					}
					System.out.println("A");
					conditionVar ++;
					B.signal();
				}
			} catch (InterruptedException e) {
				e.printStackTrace();
			} finally {
				lock.unlock();
			}
		}
	}
	
	static class B extends Thread {

		@Override
		public void run() {
			lock.lock();
			try {
				for (int i = 0; i < 10; i ++) {
					while (conditionVar % 3 != 1) {
						B.await();
					}
					System.out.println("B");
					conditionVar ++;
					C.signal();
				}
			} catch (InterruptedException e) {
				e.printStackTrace();
			} finally {
				lock.unlock();
			}
		}
	}

	static class C extends Thread {

		@Override
		public void run() {
			lock.lock();
			try {
				for (int i = 0; i < 10; i ++) {
					while (conditionVar % 3 != 2) {
						C.await();
					}
					System.out.println("C");
					conditionVar ++;
					A.signal();
				}
			} catch (InterruptedException e) {
				e.printStackTrace();
			} finally {
				lock.unlock();
			}
		}
	}

	public static void main(String[] args) throws InterruptedException {
	
		new A().start();
		new B().start();
		new C().start();
	}
}

```
### 版本3(Semaphore/Lock)
版本三是比较通用的做法，这种方法可以根据线程执行的图模型，构造出可执行路径，而且写起来非常简单，通过信号量给出的操作原语，在这种线程之间依赖关系执行的时候，显得非常有用。我们知道操作系统中的信号里是另外一种经典的同步互斥原语设计，这个设计能够同时支持

信号量用来控制资源数量大于等于1的资源并发访问，对于n个可以同时访问的资源，最多可以有 n 个线程可以同时并发执行访问各个资源。当再有新线程需要访问资源的时候，会排队挂起，直到有就旧的线程释放资源。前面介绍的资源访问一般就是只有一个共享的资源，这里信号量机制可以做多资源并发处理的能手！

另一个重要特性是，信号量只有1个资源的时候，就实现了互斥锁，并且当初始化为0个资源的时候，可以起到拦截和解决线程执行依赖关系的作用！资源为0的信号量，必须等到别人release一次才能acquire。

看ABC 的依赖关系，A<--B, B<--C, C<--A. 也就是说，我们设置三个信号量，A=1,B=0,C=0。 这里B要等A release，才能顺利 acquire。以此类推。

```java
public class ABCSemaphore {

	private static Semaphore A = new Semaphore(1);
	private static Semaphore B = new Semaphore(0);
	private static Semaphore C = new Semaphore(0);

	static class A extends Thread {

		@Override
		public void run() {
			try {
				for (int i = 0; i < 10; i++) {
					A.acquire();
					System.out.println("A");
					B.release();
				}
			} catch (InterruptedException e) {
				e.printStackTrace();
			}
		}

	}

	static class B extends Thread {

		@Override
		public void run() {
			try {
				for (int i = 0; i < 10; i++) {
					B.acquire();
					System.out.println("B");
					C.release();
				}
			} catch (InterruptedException e) {
				e.printStackTrace();
			}
		}

	}

	static class C extends Thread {

		@Override
		public void run() {
			try {
				for (int i = 0; i < 10; i++) {
					C.acquire();
					System.out.println("C");
					A.release();
				}
			} catch (InterruptedException e) {
				e.printStackTrace();
			}
		}

	}

	public static void main(String[] args) throws InterruptedException {
	
		new A().start();
		new B().start();
		new C().start();
	}
}

```

## 补充知识资深话题
java 现在的并发实现互斥和同步的机制对新手来说看起来非常混乱！新手大多知道 `synchronized/volatile` 关键字可以实现互斥，再辅以　`wait/notify/notifyAll` 就可以实现同步．以上可以认为是JVM内置的原生互斥同步实现．但是在　JDK 5以后，`java.util.concurrent` 包里面又实现了很多额外的机制，比如 Lock(ReentrantLock/ReentrantReadWriteLock),Semaphore,CountDownLatch, AtomicInteger等，其实这个并发包并不是替代内置枷锁的方法，而是当内置枷锁方法不使用的时候，作为一种可选择的高级功能．

## Locks

```java
public class Counter{

  private int count = 0;

  public int inc(){
    synchronized(this){
      return ++count;
    }
  }
}
```
```java

  private Lock lock = new Lock();
  private int count = 0;

  public int inc(){
    lock.lock();
    int newCount = ++count;
    lock.unlock();
    return newCount;
  }
}
```
**并发包里的锁实际上可以通过 synchronized/wait/notify 来模拟实现同样的功能. 那这和直接用 synchronized/wait/notify有什么区别？**
synchronized 是JVM 内置的,采用对象监控器的一种锁实现机制, 你必须控制到代码块的枷锁粒度,并不是非常灵活,而且采用 synchronized 必定会造成阻塞, 因为JVM会把 synchronized 关键字翻译成也不能设置超时. 

Lock 就是更加灵活的选择, 他的实现在JDK中是通过 CAS 来做的, CAS 是cpu原语指令, 他是不需要经过操作系统去调度阻塞的,即 CAS 可以实现非阻塞的锁.

1. The use of synchronized methods or statements provides access to the **implicit monitor lock associated with every object, but forces all lock acquisition and release to occur in a block-structured way**

2. Lock implementations provide additional functionality over the use of synchronized methods and statements by providing a **non-blocking attempt to acquire a lock (tryLock()), an attempt to acquire the lock that can be interrupted (lockInterruptibly(), and an attempt to acquire the lock that can timeout (tryLock(long, TimeUnit)).**

3. A Lock class can also provide behavior and semantics that is quite different from that of the implicit monitor lock, such as **guaranteed ordering, non-reentrant usage, or deadlock detection**

>Locks (and other more advanced synchronization mechanisms) are created using synchronized blocks, so it is not like we can get totally rid of the synchronized keyword.

```java
public class Lock{

  private boolean isLocked = false;

  public synchronized void lock()
  throws InterruptedException{
    while(isLocked){
      wait();
    }
    isLocked = true;
  }

  public synchronized void unlock(){
    isLocked = false;
    notify();
  }
}
```
这是一个**自旋锁的实现！其实就是因为原来程序员直接使用 synchronized/wait/notify 自己实现锁，很容易出bug,并且JVM还有伪唤醒的bug(即在没有调用notify/notifyAll的情况下，wait线程也可能被唤醒，因此必须使用 while 而不是简单的一次 if判断),　自旋锁可以避免这种bug.** 

>For inexplicable reasons it is possible for threads to wake up even if notify() and notifyAll() has not been called. This is known as spurious wakeups. Wakeups without any reason.

其实这个自旋锁如果你仔细观察的话，这不就是生产消费者模型里面的 `while() wait; notify;`,只是这里判断条件是单个bool变量，生产消费者模型中使用的是判断循环队列是空还是满．两者及其相似！
[Locks in Java](http://tutorials.jenkov.com/java-concurrency/locks.html)

### synchronized 和 并发包中的Lock区别（使用场景）
ReentrantLock的优势

 1. 能够中断获取锁过程 
 2. 在等待锁时可以指定超时时间 
 3. 能够创建公平锁 
 4. 获取所有等待锁的线程 
 5. 可以灵活的尝试获取锁而不锁定线程

ReentrantLock的劣势
 
 - ReentrantLock的主要缺点是使用ReentrantLock时必须将ReentrantLock中的方法放到try-finally结构中，这就造成了代码不易读并且隐藏了业务逻辑。

### Consumer & Producer 的两种实现方式
#### 基于 wait/notify 和 synchronized 原语的实现，这个是最基本的实现。
```java
public class PCModel {
	
	public static void main(String[] args) {
		SharedObject so = new SharedObject();
		Producer p = new Producer(so);
		Consumer c1 = new Consumer(so);
		Consumer c2 = new Consumer(so);
		
		new Thread(c1, "c1").start();
		new Thread(c2, "c2").start();
		Thread pt = new Thread(p,"p");
		//pt.setPriority(1);
		pt.start();
		
	}
	
	static class SharedObject {
		private int[] buffer = new int[64];
		private int rindex = 0;
		private int windex = 0;
		private int count = 0;

		public int get() {
			synchronized(this) {
				while (count == 0) {
					try {
//						System.out.println(Thread.currentThread().getName()+ " get() wait");
						this.wait();
					} catch (InterruptedException e) {
						e.printStackTrace();
					}
				}
				int item = this.buffer[rindex];
				rindex = (rindex+1)%64;
				count --;
				this.notify();
				return item;
			}		
		}
		
		public void put(int product) {
			synchronized(this) {
				while (count >= buffer.length) {
					try {
						this.wait();
					} catch (InterruptedException e) {
						e.printStackTrace();
					}
				}
				this.buffer[windex] = product;
				windex = (windex+1)%64;
				count ++;
				this.notifyAll();
			}
		}
	}

	static class Producer implements Runnable {
		SharedObject so = null;
		Producer(SharedObject so) {
			this.so = so;
		}
		@Override
		public void run() {
			int i = 0;
			while(true){
				so.put(++i);
				System.out.println(Thread.currentThread().getName()+"-" + i);
				try {
					Thread.sleep(100);
				} catch (InterruptedException e) {
					e.printStackTrace();
				}
			}
			
		}
		
	}

	static class Consumer implements Runnable {
		SharedObject so = null;
		Consumer(SharedObject so) {
			this.so = so;
		}
		@Override
		public void run() {
			System.out.println(Thread.currentThread().getName() + " started");
			while(true){
				int item = so.get();
				System.out.println(Thread.currentThread().getName()+"-"+item);	
				try {
					Thread.sleep(100);
				} catch (InterruptedException e) {
					e.printStackTrace();
				}
			}
			
		}
		
	}


}


```

#### 基于 Condition 和 ReentrantLock 的实现
利用 `java.util.concurrent` 并发包实现，会更加灵活，同步粒度也会更细。
```java
public class PCModel2 {
	
	
	static class SharedObject {
		private ReentrantLock lock = new ReentrantLock();
		
		private Condition notEmpty = lock.newCondition();
		private Condition notFull = lock.newCondition();
		private int[] buffer = new int[64];
		private int rindex = 0;
		private int windex = 0;
		private int count = 0;
		
		public int get() {
			lock.lock();
			try {
				while (count == 0) {
					notEmpty.await();
				}
				int item = this.buffer[rindex];
				rindex = (rindex+1)%buffer.length;
				count --;
				notFull.signal();
				return item;
			} catch (InterruptedException e) {
				e.printStackTrace();
			} finally {
				lock.unlock();
			}
			return 0;
		}
		
		public void put(int item) {
			lock.lock();
			try {
				while (count >= buffer.length) {
					notFull.await();
				}
				this.buffer[windex] = item;
				count ++;
				windex = (windex+1)%buffer.length;
				notEmpty.signal();
			} catch (InterruptedException e) {
				e.printStackTrace();
			} finally {
				lock.unlock();
			}
		}
	}
	
	static class Producer implements Runnable {
		private SharedObject so = null;

		Producer(SharedObject so) {
			this.so = so;
		}
		@Override
		public void run() {
			int i = 0;
			while(true){
				so.put(++i);
				System.out.println(Thread.currentThread().getName()+"-" + i);
				try {
					Thread.sleep(100);
				} catch (InterruptedException e) {
					e.printStackTrace();
				}
			}
			
		}
		
	}
	
	static class Consumer implements Runnable {
		private SharedObject so = null;
		Consumer(SharedObject so) {
			this.so = so;
		}
		
		public void run() {
			while (true) {
				int item = so.get();
				System.out.println(Thread.currentThread().getName() + "-" + item);
				try {
					Thread.sleep(100);
				} catch (InterruptedException e) {
					e.printStackTrace();
				}
			}
		}
	}
	
	public static void main(String[] args) {
		SharedObject so = new SharedObject();
		Producer p = new Producer(so);
		Consumer c1 = new Consumer(so);
		Consumer c2 = new Consumer(so);
		
		new Thread(c1, "c1").start();
		new Thread(c2, "c2").start();
		Thread pt = new Thread(p,"p");
		//pt.setPriority(1);
		pt.start();
	}
}
```
**wait/notify 有个缺点，就是他们的条件谓词信号只能是同一个，也就是等待和唤醒都是同一个锁，因为 Java 要求 wait/notify 必须是在 synchronized 同步快内部使用，这里我们可以看到基础版本的 生产者消费者模型，当消费者唤醒调用的时候，可能唤醒的还是消费者自己，而不一定是生产者！！导致效率比较低，甚至出现饥饿现象。** 

### Semaphore
>A Semaphore is a thread synchronization construct that can be used either to send signals between threads to avoid missed signals, or to guard a critical section like you would with a lock.

信号量可以解决信号丢失的缺点.可以看成 PV 原语的实现.我们知道Linux下PV原语,P代表减少,V代表释放,是一种实现同步的方式,并且当信号设置成1的时候就是互斥锁了.
简单版本的 Semaphore

```java
public class Semaphore {
  private boolean signal = false;

  public synchronized void take() {
    this.signal = true;
    this.notify();
  }

  public synchronized void release() throws InterruptedException{
    while(!this.signal) wait();
    this.signal = false;
  }

}
``` 
可以记录信号发送的信号个数(Counting Semaphore)

```java
public class CountingSemaphore {
  private int signals = 0;

  public synchronized void take() {
    this.signals++;
    this.notify();
  }

  public synchronized void release() throws InterruptedException{
    while(this.signals == 0) wait();
    this.signals--;
  }
```
带上界的(Bounded Semaphore)

```java
public class BoundedSemaphore {
  private int signals = 0;
  private int bound   = 0;

  public BoundedSemaphore(int upperBound){
    this.bound = upperBound;
  }

  public synchronized void take() throws InterruptedException{
    while(this.signals == bound) wait();
    this.signals++;
    this.notify();
  }

  public synchronized void release() throws InterruptedException{
    while(this.signals == 0) wait();
    this.signals--;
    this.notify();
  }
}
```

PV原语就是有bound个资源,当剩余资源数目不是0的时候,就可以take,当到达bound的时候,后续take会被阻塞.当有线程release的时候,剩余资源又有了,又可以take了.

这里资源的使用是用信号signal表示的,可以理解为使用一个资源,就发射一次信号. [Semaphores](http://tutorials.jenkov.com/java-concurrency/semaphores.html#counting)


### AtomicInteger

AtomicInteger 使用的是　CAS 操作进行的，即自旋＋CAS

```java
//使用volatile关键字保证线程可见性
private volatile int value;
public final int get(){
    return value;
}

public final int incrementAndGet(){
    for (; ; ) {
        int current = get();
        int next = current + 1;
        if (compareAndSet(current, next))
            return next;
    }
}

 public final int getAndIncrement() {
    for (;;) {
        int current = get();
        int next = current + 1;
        if (compareAndSet(current, next))
            return current;
    }
}

public final boolean compareAndSet(int expect, int update) {
    return unsafe.compareAndSwapInt(this, valueOffset, expect, update);
}
```
CAS模拟

```java
class SimulatedCAS {
	private long value;
	
	public synchronized long get() {
		return value;
	}
	
	public synchronized boolean compareAndSwapLong(long expected, long newValue) {
		long oldValue = value;
		if (oldValue == expected) {
			value = newValue;
			return true;
		}
		return false;
	}
	
}

class SimulatedCASCounter {
	private SimulatedCAS value;
	
	public final long getCounter() {
		return value.get();
	}
	
	public long incrementAndGet() {
		for (;;) {
			long expected = value.get();
			long next = expected + 1;
			if (value.compareAndSwapLong(expected, next)) {
				return next;
			}
		}
	}
	
	public long getAndIncrement() {
		for (;;) {
			long expected = value.get();
			long next = expected + 1;
			if (value.compareAndSwapLong(expected, next)) {
				return expected;
			}
		}
	}
	
}
```

使用 `sun.misc.Unsafe` 实现CASCounter
```java
public class CASCounter {
	private Unsafe unsafe;
	private volatile long counter = 0;
	private long offset;
	
	private Unsafe getUnsafe() throws IllegalAccessException, NoSuchFieldException {
		Field f = Unsafe.class.getDeclaredField("theUnsafe");
		f.setAccessible(true);
		return (Unsafe) f.get(null);
	}
	
	public CASCounter() throws Exception {
		unsafe = getUnsafe();
		offset = unsafe.objectFieldOffset(CASCounter.class.getDeclaredField("counter"));
	}
	
	public void increment(){
		long expected = counter;
		while (!unsafe.compareAndSwapLong(this, offset, expected, expected+1)) {
			expected = counter;
		}
	}
	
	public long incrementAndGet(){
		long expected = counter;
		while (!unsafe.compareAndSwapLong(this, offset, expected, expected+1)) {
			expected = counter;
		}
		return expected + 1;
	}
	
	public long getAndIncrement() {
		long expected = counter;
		while (!unsafe.compareAndSwapLong(this, offset, expected, expected+1)) {
			expected = counter;
		}
		return expected;
	}
	public long getCounter() {
		return counter;
	}
}
``` 

### CountDownLatch 和 CyclicBarrier

- [Java concurrency – CountDownLatch Example](https://howtodoinjava.com/core-java/multi-threading/when-to-use-countdownlatch-java-concurrency-example-tutorial/)
- [CountDownLatch](https://docs.oracle.com/javase/7/docs/api/java/util/concurrent/CountDownLatch.html)
- [CyclicBarrier](https://docs.oracle.com/javase/7/docs/api/java/util/concurrent/CyclicBarrier.html)

自制版CountDownLatch
```java
class CountDownLatch {
	volatile int count;
	public CountDownLatch(int cnt) {
		count = cnt;
	}
	public synchronized void countDown() {
		count --;
		if (count == 0) notifyAll();
	}
	public synchronized void await() {
		while (count != 0) {
			wait();
		}
	}
}
```
自制版CyclicBarrier
```java
class Generation {
	boolean broken;
}
class CyclicBarrier {
	int parties;
	int count;
	Runnable event;
	Generation generation = new Generation();
	public CyclicBarrier(int parties, Runnable event) {
		this.parties = parties;
		this.count = parties;
		this.event = event;
	}
	public synchronized int await() {
		Generation g = this.generation;
		int index = --this.count;
		if (index == 0) {
			event.run();
			//reset
			this.count = this.parties;
			this.generation = new Generation();
			
			notifyAll();
			return index;
		}
		while(true) {
			wait();
			if (g != this.generation) {
				return index;
			}
		}
	}
}
```

### sun.misc.Unsafe
- [Guide to sun.misc.Unsafe](http://www.baeldung.com/java-unsafe)
- [Understanding sun.misc.Unsafe](https://dzone.com/articles/understanding-sunmiscunsafe)
- [Java Magic. Part 4: sun.misc.Unsafe](http://mishadoff.com/blog/java-magic-part-4-sun-dot-misc-dot-unsafe/)
- [Java Magic. Part 5: SecurityManager](http://mishadoff.com/blog/java-magic-part-5-securitymanager/)
- [Java Magic. Part 3: Finally](http://mishadoff.com/blog/java-magic-part-3-finally/)