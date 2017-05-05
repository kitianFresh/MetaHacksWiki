---
title: "ConcurrencyModel"
date: 2017-03-28 22:33
---

# 并行计算
两大类并行

## 多进程或多线程并行
race condition dead lock

## 流水线并行
non-blocking io event-driven system

## 函数式并行(Functional Parallelism)
单个任务可以被拆分成多个部分来处理，每个部分都是一个函数，数据经过这些函数的时候是并行处理的，其实就是流水线
mapreduce; 但是对于本来就运行多个不同类型的任务的系统，比如 web server database server 都是不适合用函数式并行的，因为cpu本来就在处理多任务


# Asignment1

## 1. Sum An Array By Multi-Threads
数组求和实验, 采用串行和多线程的形式进行比较.实验的测试代码:
```Java
	public static void main(String[] args) {
		int size = 400000;
		int[] a = new int[size];
		for (int i = 0; i < size; i++) {
			a[i] = i;
		}
		
		int sum = 0;
		double t = 0;
		long start, end;
		int loops = 5;
		int cpus = Runtime.getRuntime().availableProcessors();
		int[] threads = new int[]{cpus/4, cpus/2, cpus, cpus + 2, 2 * cpus, 2*cpus+2, 3*cpus, 3*cpus + 2, 4*cpus};
		double[] ts = new double[loops];
		String str0 = String.format("%20s\t%15d\t%15d\t%15d\t%15d\t%15d\t%15s\n", "Threads\\Loops",1,2,3,4,5,"Average(ms)");
		System.out.println(str0);
		try {
			for (Integer cores : threads) {
				t = 0;
				for (int i = 0; i < loops; i++) {
					start = System.currentTimeMillis();
					sum = ArraySumer.sum(a, cores);
					end = System.currentTimeMillis();
					ts[i] = end - start;
					t += ts[i];
					
				}
				t = t / loops;
				String str1 = String.format("%20s\t%15f\t%15f\t%15f\t%15f\t%15f\t%15f\n", cores+"", ts[0], ts[1], ts[2], ts[3], ts[4], t);
				System.out.println(str1);
			}
			
			
		} catch (InterruptedException e) {
			// TODO Auto-generated catch block
			e.printStackTrace();
		}
	}
```

### 1.1 one thread (sequential)
单线程就是一种串行求和, 设置 threadSize = 1即可!

### 1.2 different thread nums compares (parallel)
本实验**对每一种线程个数(Threads)做了五次实验(Loops=5), 然后求出五次的平均性能(Average).**

|Threads\Loops|              1|	              2|	            3|	             4|	             5|	    Average(ms)|
|------------:|--------------:|---------------:|----------------:|---------------:|--------------:|---------------:|
|    1	      | 4.000000      |   5.000000     |  0.000000       |     1.000000	  | 0.000000      |   2.000000     |
|    2	      | 0.000000      |   0.000000     |  1.000000       |     0.000000	  | 0.000000      |   0.200000     |
|    4	      | 0.000000      |   0.000000     |  1.000000       |     0.000000	  | 0.000000      |   0.200000     |
|    6	      | 0.000000      |   1.000000     |  0.000000       |     1.000000	  | 1.000000      |   0.600000     |
|    8	      | 1.000000      |   0.000000     |  1.000000       |     0.000000	  | 1.000000      |   0.600000     |
|   10	      | 1.000000      |   1.000000     |  0.000000       |     1.000000	  | 5.000000      |   1.600000     |
|   12	      |10.000000      |   3.000000     |  1.000000       |     0.000000	  | 1.000000      |   3.000000     |
|   14	      | 1.000000      |   1.000000     |  0.000000       |     1.000000	  | 0.000000      |   0.600000     |
|   16	      | 1.000000      |   1.000000     |  8.000000       |     1.000000	  | 0.000000      |   2.200000     |

从以上实验结果可以看出, **对于多线程求和, 线程数目并不是越多性能越好, 一般设置为可用处理器个数或者可用处理器个数的2倍性能最佳!**

### 1.3 shared and local memory in my code
以下是实验中使用的  `ArraySumer` 类的代码.
```java
public class ArraySumer extends Thread{
	public int[] a;
	public int low;
	public int high;
	public int result = 0;
    public void run() {
		for (int i = this.low; i <= this.high; i++) {
			result += a[i];
		}
	}
	public static int sum(int[] arr, int threadSize) throws InterruptedException {
		int len = arr.length;
		int result = 0;
		ArraySumer[] as = new ArraySumer[threadSize];
		for (int i = 0; i < threadSize; i++) {
			as[i] = new ArraySumer(arr, i*len/threadSize, (i+1)*len/threadSize-1);
			as[i].start();
		}
		for (int i = 0; i < threadSize; i++) {
			as[i].join();
			result += as[i].result;
		}
		return result;
		
	}
	
}
```
从代码中可以看出, 所有线程共享数组变量 `a`. 主线程和从线程都只对 `a` 进行读取操作. 主线程通过局部变量的形式将参数 `low`, `high` 传递给从线程, 从线程会写入自己的成员变量 `result`, 而主线程会读取所有从线程的成员变量 `result`. 因此 race condition 可能会发生在这里, 但是这个数组求和程序, 由于从线程的 `as[i].join()` 操作, 主线程会等待所有的从线程都做完(写完 result)之后才开始读取, 因此并没有数据竞争.

### 1.4 preformance analysis
**对于多线程 parallel, 并不是选择越多的线程越好, 因为创建线程本身也是耗费资源(时间和空间资源)的.** 极端的例子, 比如现在有处理器P个,P >> n, 我们对于每一个数,都创建一个线程读取, 然后让主线程合并, 这反而没有串行执行的快. 

**因此要想使得多线程 performance 达到最大, 需要选取合适的 threadSize. 从以上的实验可以得出一个经验性的结论就是, 线程数目和当前可得处理器的个数有关系, 一般我们选取cpus或者 2\*cpus比较合理.**

## 2. Divide-and-Conquer method to write a parallel program for computing the max number of an array
采用分治法计算数组的最大(最小)数, 分治思想: 主线程将任务分解成两个子任务, 左子线程寻找左半边数组的最大值, 右子线程寻找右半边数组的最大值, 当左右任务完成之后, 主线程合并左右结果(比较左右结果的较大者返回), 左右线程进行同样的递归分解操作, 分解终止的条件由 `SEQUENTIAL_CUTOFF` 来控制.

### 2.1 start/run method
根据分治思想, 代码如下:
```java
public class MaxNumber extends Thread{
	int low;
	int high;
	int max = 0;
	int[] numbers;
	int SEQUENTIAL_CUTOFF;
	MaxNumber(int[] numbers, int low, int high, int cutoff) {
		this.numbers = numbers;
		this.low = low;
		this.high = high;
		this.SEQUENTIAL_CUTOFF = cutoff;
	}
	
	public void run() {
		if (high - low < SEQUENTIAL_CUTOFF) {
			for (int i = low; i <= high; i++) {
				max = Math.max(max, numbers[i]);
			}
		}
		else {
			int mid = (low + high) / 2;
			MaxNumber left = new MaxNumber(numbers, low, mid, SEQUENTIAL_CUTOFF);
			MaxNumber right = new MaxNumber(numbers, mid+1, high, SEQUENTIAL_CUTOFF);
			left.start();
			right.start();
			
			try {
				left.join();
				right.join();
			} catch (InterruptedException e) {
				// TODO Auto-generated catch block
				e.printStackTrace();
			}
			max = Math.max(left.max, right.max);
		}
	}
	
	public static int max(int[] numbers, int cutoff) {
		MaxNumber mn = new MaxNumber(numbers, 0, numbers.length-1,cutoff);
		mn.run();
		return mn.max;
	}
}
```
采用覆盖`run()` 方法的缺陷就是无法返回一个值, 只能通过成员变量的形式存储. 另外, 由于每一次递归创建子任务的时候, 主线程没有做任何计算任务, 必须要等待左右都完成之后,才开始计算, 这是一种资源的浪费, 因此一种形式的优化是,让主线程来做右子树的任务! 这样线程数目可以减少一般!
```java
int mid = (low + high) / 2;
MaxNumber left = new MaxNumber(numbers, low, mid, SEQUENTIAL_CUTOFF);
MaxNumber right = new MaxNumber(numbers, mid+1, high, SEQUENTIAL_CUTOFF);
left.start();
//right.start();
right.run();

try {
	left.join();
	//right.join();
} catch (InterruptedException e) {
	// TODO Auto-generated catch block
	e.printStackTrace();
}
```
### 2.2 Fork/Join framework
以下是改用fork/join framework 的代码:
```java
public class MaxNumberForkJoin extends RecursiveTask {
	int low;
	int high;
	int[] numbers;
	int SEQUENTIAL_CUTOFF;
	
	MaxNumberForkJoin(int[] numbers, int low, int high, int cutoff) {
		this.numbers = numbers;
		this.low = low;
		this.high = high;
		this.SEQUENTIAL_CUTOFF = cutoff;
	}
	
	@Override
	protected Integer compute() {
		int max = 0;
		if (high - low < SEQUENTIAL_CUTOFF) {
			for (int i = low; i <= high; i++) {
				max = Math.max(max, numbers[i]);
			}
			return max;
		}
		else {
			int mid = (low + high) / 2;
			MaxNumberForkJoin left = new MaxNumberForkJoin(numbers, low, mid, SEQUENTIAL_CUTOFF);
			MaxNumberForkJoin right = new MaxNumberForkJoin(numbers, mid+1, high, SEQUENTIAL_CUTOFF);
			left.fork();
			int rightMax = right.compute();
			int leftMax = (int) left.join();
			max = Math.max(rightMax, leftMax);
			return max;
		}
	}

	static final ForkJoinPool fjPool = new ForkJoinPool();
	public static int max(int[] numbers, int cutoff) {
		return fjPool.invoke(new MaxNumberForkJoin(numbers, 0, numbers.length-1, cutoff));
	}
	
}
```
### 2.3 sequential checker
以下是串行运算的部分代码:
```
for (int i = 0; i < size; i++) {
    max = Math.max(max, numbers[i]);
}
```

### 2.4 performance comparison
这里, 我设置的 `SEQUENTIAL_CUTOFF` 的值是 2048; 还是一样, 对于每一种情况(Serialize Start/Run 和 Fork/Join) 都采用五次测试取平均来比较平均时间性能. 数组规模是 2^25 次方.

|Pattern\Loops           |	            1|	             2|              3|              4|	              5|    Average(ms)|
|-----------------------:|--------------:|---------------:|--------------:|--------------:|---------------:|--------------:|
|Serialize Baseline	     |   29.673285	 |    30.963514	  |    26.244178  |      26.037493|	      25.921920|      27.768078|
|Parallelize Start/Run	 |13103.625879	 |  9753.860262	  |  9774.133012  |   10805.342403|	    9349.867925|   10557.365896|
|Parallelize Fork/Join	 |   34.131773	 |    20.526604	  |    15.806599  |      10.757907|	      10.060238|      18.256624|

对于2^26 次方的时候, Start/Run 模式会出现 `Exception in thread "Thread-28661" java.lang.OutOfMemoryError: unable to create new native thread` 这种 `OutOfMemory` 的错误. 但是 Fork/Join 可以轻松应对! 

**猜测 Fork/Join framework 内部实现了线程池, 因此可以保证不会因为创建过多线程而导致 JVM 爆出内存错误. 因为每一个线程都有私有线程栈, 线程创建的越多, 需要的内存就越多, 当内存不够的时候, 就无法再创建线程了! **

这里**要么设置更小的线程栈以争取创建更多线程的可能, 要么扩充JVM内存.** 但是栈大小本身 JVM 也是有限制的, 设置过小, JVM 就无法启动而报错!
```
The stack size specified is too small, Specify at least 228k
Error: Could not create the Java Virtual Machine.
Error: A fatal exception has occurred. Program will exit.
```

**但是当线程数数目达到一定程度的时候, 创建线程的开销反而大于任务本身了, 这是适得其反的! 另外, 线程栈过小的话, 如果有递归, 这里分治法就是递归, 压栈就不可能很深, 就会限制递归的深度, 限制递归的深度就会限制能解决问题的规模数目!**

去掉 Start/Run 方法之后, 我们设置数组规模到JVM堆可以容纳的最大值, **运行的时候你可以设置JVM 的运行参数 `-Xmx4096M -Xms2000M -Xmn500M`, 因为JVM默认最大堆内存是机器物理内存的 1/4, 而我机器内存8G, 也就是最大是2G, 但是JVM最大堆内存的限制实际上取决于操作系统和物理内存空间, 我的机器最大堆内存可以设置到8G. **

                 Pattern\Loops	              1	              2	              3	              4	              5	    Average(ms)
            Serialize Baseline	     188.076303	     206.252412	     208.105703	     203.449220	     203.056148	     201.787957
         Parallelize Fork/Join	     145.812311	      85.910176	      85.791367	      90.719006	      81.187196	      97.884011

但是**你最好不要设置成最大内存 8G, 除非你的服务器只运行一个 JAVA 程序, 如果你设置成最大物理内存8 G, 然后申请 1<<30 个int整形数组,也即是占用 4G 内存, 然后运行程序,机器会非常卡, 因为处理器和内存资源全部都给 JAVA 程序了, 其他进程就无法很愉快的运行和响应了! 如果设置的 `SEQUENTIAL_CUTOFF` 和数组规模以及处理器个数不太符合的话,由于线程数目过多, Fork/Join 的性能就比不上 Serialize 了.** `SEQUENTIAL_CUTOFF = 1<<11`, 将得到一下结果;

                 Pattern\Loops	              1	              2	              3	              4	              5	    Average(ms)
            Serialize Baseline	     952.887980	    1345.387388	     777.596211	     773.802849	     774.879831	     924.910852
         Parallelize Fork/Join	    3157.547799	    1058.703371	    2999.615653	     274.054247	     245.031633	    1546.990541

因此, 我再次提升 `SEQUENTIAL_CUTOFF = 1<<15`, 此时 Fork/Join 的性能将会更好.

                 Pattern\Loops	              1	              2	              3	              4	              5	    Average(ms)
            Serialize Baseline	     781.488606	     794.344170	     761.915751	     750.881256	     746.211944	     766.968345
         Parallelize Fork/Join	     398.081041	     257.000682	     215.312012	     228.940126	     233.489181	     266.564608



## 4. PrimeNumber By Concurrency And Parallel

### 4.1 Parallel
```java
public class PrimeNumberParallel extends Thread{
	int low;
	int high;
	List<Long> primes; // 使用 vector 线程安全的
	
	PrimeNumberParallel(List<Long> primes, int low, int high) {
		this.primes = primes;
		this.low = low;
		this.high = high;
	}
	
	public void run() {
		
		for (long i = low; i <= high; i++) {
			if (isPrime(i)) {
				primes.add(i);
//				System.out.println(i);
			}
		}
	}
	
	public static boolean isPrime(long number) {
		if (number <= 1) return false;
		for (long i = 2; i <= (long)Math.sqrt(number); i++) {
			if (number % i == 0) return false;
		}
		return true;
	}
	
	public static List<Long> primeNumbers(int threads, int size) throws InterruptedException {
		List<Long> primes = new Vector<Long>();
		PrimeNumberParallel[] pns = new PrimeNumberParallel[threads];
		for (int i = 0; i < threads; i ++) {
			pns[i] = new PrimeNumberParallel(primes, i*size/threads, (i+1)*size/threads);
			pns[i].start();
		}
		
		for (int i = 0; i < threads; i ++) {
			pns[i].join();
		}
		return primes;
	}
}

```

### 4.2 Concurrency
```java
// concurrency for prime
public class PrimeNumberConcurrency extends Thread {
	
	Counter counter;
	long SIZE = 1000;
	List<Long> primes; // 使用 vector 线程安全的
	
	PrimeNumberConcurrency(List<Long> primes, Counter counter, int size) {
		this.counter = counter;
		this.primes = primes;
		this.SIZE = size;
	}
	
	public void run() {
		long number;
		while ((number = counter.getAndIncrement()) <= SIZE) {
			if (isPrime(number)) {
				primes.add(number);
//				System.out.println(number);
			}
		}
	}
	
	public static boolean isPrime(long number) {
		if (number <= 1) return false;
		for (long i = 2; i <= (long)Math.sqrt(number); i++) {
			if (number % i == 0) return false;
		}
		return true;
	}
	
	public static List<Long> primeNumbers(Counter counter, int threads, int size) throws InterruptedException {
		List<Long> primes = new Vector<Long>();
		PrimeNumberConcurrency[] pns = new PrimeNumberConcurrency[threads];
		for (int i = 0; i < threads; i ++) {
			pns[i] = new PrimeNumberConcurrency(primes, counter, size);
			pns[i].start();
		}
		
		for (int i = 0; i < threads; i ++) {
			pns[i].join();
		}
		return primes;
	}
}
```
### 4.3 PrimeNumber concurrency and parallel perfomance comparison


```java
/ better: do
SumThread left  = …
SumThread right = …
// order of next 4 lines
// essential – why?
left.start();
right.run();
left.join(); 
ans=left.ans+right.ans;
```



```java
public static String padRight(String s, int n) {
     return String.format("%1$-" + n + "s", s);  
}

public static String padLeft(String s, int n) {
    return String.format("%1$" + n + "s", s);  
}

...

public static void main(String args[]) throws Exception {
 System.out.println(padRight("Howto", 20) + "*");
 System.out.println(padLeft("Howto", 20) + "*");
}
```