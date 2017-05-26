---
title: "java-tips"
date: 2017-05-05 10:27
---


# Java 打包成 jar
注意 manifest 文件的写法, 必须是 `property:+space+value+\n`; 另外, 类名需要写全名才行!
```java
Manifest-Version: 1.0
Premain-Class: test.ObjectSizeFetcher

```

## 打包代理 agent jar
比如打包 test 包下面的 ObjectSizeFetcher 类, 需要 manifest 文件内容如下: 
```
Manifest-Version: 1.0
Premain-Class: test.ObjectSizeFetcher

```
执行 `java -javaagent:test/ObjectSizeFetcherAgent.jar test.ObjectSizeTest` 即可检查对象自身(不包含成员引用出来的对象的整个图中的对象)大小;

### 参考
 - [Package java.lang.instrument Description](http://docs.oracle.com/javase/8/docs/api/java/lang/instrument/package-summary.html)


# Java String Format
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

# java Sorted Index

Concise way of achieving this with Java 8 Stream API,

```java
final String[] strArr = {"France", "Spain", "France"};
int[] sortedIndices = IntStream.range(0, strArr.length)
                .boxed().sorted((i, j) -> strArr[i].compareTo(strArr[j]) )
                .mapToInt(ele -> ele).toArray();
```
map
```java
TreeMap<String,Int> map = new TreeMap<String,Int>();
for( int i : indexes ) {
    map.put( stringarray[i], i );
}
```

comparator 交换的是索引, 但是比较标准是原数组

```java
class ArrayIndexComparator implements Comparator<Integer>
{
    private final double[] array;

    public ArrayIndexComparator(double[] array)
    {
        this.array = array;
    }

    public Integer[] createIndexArray()
    {
        Integer[] indexes = new Integer[array.length];
        for (int i = 0; i < array.length; i++)
        {
            indexes[i] = i; // Autoboxing
        }
        return indexes;
    }

    @Override
    public int compare(Integer index1, Integer index2)
    {
         // Autounbox from Integer to int to use as array indexes
    	if (array[index1] < array[index2]) return -1;
    	else if (array[index1] > array[index2]) return 1;
    	else return 0;
    }
}


ArrayIndexComparator arrayIndexerComparator = new ArrayIndexComparator(ratios);
Integer[] indexes = arrayIndexerComparator.createIndexArray();
Arrays.sort(indexes, arrayIndexerComparator);
//System.out.println(Arrays.toString(indexes));
```

# Java 线程CPU占用率
查看线程cpu核占用率
```
top -H -p [pid]
```

## Java Exception
三种类型的 exceptions

 1. checked exception: Catch or Specify Requirement
 2. unchecked exception
  - Errors:
  - Runtime Exception: 一般不捕获处理, 因为属于程序逻辑错误.所以最好不要抛出继承自 RuntimeException 的异常.

### 使用异常的好处

 1. 分离错误处理代码和正常的逻辑代码;
```java
 readFile {
    open the file;
    determine its size;
    allocate that much memory;
    read the file into memory;
    close the file;
}
```
```java
errorCodeType readFile {
    initialize errorCode = 0;
    
    open the file;
    if (theFileIsOpen) {
        determine the length of the file;
        if (gotTheFileLength) {
            allocate that much memory;
            if (gotEnoughMemory) {
                read the file into memory;
                if (readFailed) {
                    errorCode = -1;
                }
            } else {
                errorCode = -2;
            }
        } else {
            errorCode = -3;
        }
        close the file;
        if (theFileDidntClose && errorCode == 0) {
            errorCode = -4;
        } else {
            errorCode = errorCode and -4;
        }
    } else {
        errorCode = -5;
    }
    return errorCode;
}
```
```java
readFile {
    try {
        open the file;
        determine its size;
        allocate that much memory;
        read the file into memory;
        close the file;
    } catch (fileOpenFailed) {
       doSomething;
    } catch (sizeDeterminationFailed) {
        doSomething;
    } catch (memoryAllocationFailed) {
        doSomething;
    } catch (readFailed) {
        doSomething;
    } catch (fileCloseFailed) {
        doSomething;
    }
}
```
 2. 通过调用栈传递错误异常
```
method1 {
    call method2;
}

method2 {
    call method3;
}

method3 {
    call readFile;
}

// if else
method1 {
    errorCodeType error;
    error = call method2;
    if (error)
        doErrorProcessing;
    else
        proceed;
}

errorCodeType method2 {
    errorCodeType error;
    error = call method3;
    if (error)
        return error;
    else
        proceed;
}

errorCodeType method3 {
    errorCodeType error;
    error = call readFile;
    if (error)
        return error;
    else
        proceed;
}

// try catch
method1 {
    try {
        call method2;
    } catch (exception e) {
        doErrorProcessing;
    }
}

method2 throws exception {
    call method3;
}

method3 throws exception {
    call readFile;
}
```
 - [Advantages of Exceptions](http://docs.oracle.com/javase/tutorial/essential/exceptions/advantages.html)

### catch 捕获异常

```java
try {

} catch (IndexOutOfBoundsException e) {
    System.err.println("IndexOutOfBoundsException: " + e.getMessage());
} catch (IOException e) {
    System.err.println("Caught IOException: " + e.getMessage());
}
```
JDK1.7 映引入, 一个 catch 可以传入多种 Exception
```java
catch (IOException|SQLException ex) {
    logger.log(ex);
    throw ex;
}
```
Finally 不管异常是否发生,都会执行!一下最后返回2; 前一个return 的值被后面的覆盖了!

```java
    public static int get() {
		try {
			return 1;
		} finally {
			return 2;
		}
	}
```
### 例子
```java
public void writeList() {
    PrintWriter out = null;

    try {
        System.out.println("Entering" + " try statement");

        out = new PrintWriter(new FileWriter("OutFile.txt"));
        for (int i = 0; i < SIZE; i++) {
            out.println("Value at: " + i + " = " + list.get(i));
        }
    } catch (IndexOutOfBoundsException e) {
        System.err.println("Caught IndexOutOfBoundsException: "
                           +  e.getMessage());
                                 
    } catch (IOException e) {
        System.err.println("Caught IOException: " +  e.getMessage());
                                 
    } finally {
        if (out != null) {
            System.out.println("Closing PrintWriter");
            out.close();
        } 
        else {
            System.out.println("PrintWriter not open");
        }
    }
}
```

### specify 申明某函数会抛出异常
```java
public void writeList() throws IOException, IndexOutOfBoundsException {

//Remember that IndexOutOfBoundsException is an unchecked exception; including it in the throws clause is not mandatory. You could just write the following.

public void writeList() throws IOException {
```

### chained exceptions


访问栈调用轨迹

```java
catch (Exception cause) {
    StackTraceElement elements[] = cause.getStackTrace();
    for (int i = 0, n = elements.length; i < n; i++) {       
        System.err.println(elements[i].getFileName()
            + ":" + elements[i].getLineNumber() 
            + ">> "
            + elements[i].getMethodName() + "()");
    }
}
```