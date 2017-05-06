---
title: "java-tips"
date: 2017-05-05 10:27
---


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