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
