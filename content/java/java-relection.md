---
title: "java-relection"
date: 2019-02-18 16:55
---

# 反射

静态强类型语言来说，基本上都由

任何语言编写代码的时候，可能都会遇到一种需求，就是希望看到某个类或者结构数据到底长什么样，比如获取类名称，字段名称，方法名称，这些信息都属于代码层面的信息了，也叫元信息。而操作元信息来编写代码的过程被称为元编程。由于任何语言的代码都会先交给编译器或者解释器，因此这个任务可以让编译器或者解释器来完成，比如java编译器，JVM, Python解释器，Go语言运行时环境这些。

在 Java 中实现元编程就靠反射机制了。这是 Java 虚拟机提供的一套接口。因为 Java 的代码编译成字节码后，都是交给 JVM 解释执行的，JVM 能够了解运行中的 Java 代码的一切信息。



## inspect classes
`java.lang.Class` 类是一切类的封装和抽象，因此一切都是从这个类开始的，获得一个 Class 的实例对象有两种方式，一个是编译时，一个是运行时。

编译时的用法，其实就是你的这个类（字节码）在编译的时候就存在且可访问的。

```java
Class myObjectClass = MyObject.class;
```
运行时的用法，就是你可能就不知道类名，或者这个类在编译的时候可能都还没有，必须在运行时才确定。

```java
String className = 'myMapper';
Class klass = Class.forName(className);
```

在获取类对象 Class 以后，可以通过相关函数获取到类的各种元信息。包括 类名， 修饰符，包信息，超类，接口，构造函数，方法，字段，注解信息，范型信息, 模块信息(JAVA 9引入)等。

### 获取类里的范型信息
```java
// return generic type
Method method = Klass.class.getMethod("getPhones", null);
Type returnType = method.getGenericReturnType();
if (returnType instanceof ParameterizedType) {
	ParameterizedType type = (ParameterizedType) returnType;
	Type[] typeArguments = type.getActualTypeArguments();
	for (Type typeArgument : typeArguments) {
		Class typeArgClass = (Class) typeArgument;
		System.out.println("typeArgClass = " + typeArgClass);
	}
}
// argument generic type
method = Klass.class.getMethod("setPhones", List.class);
Type[] genericParameterTypes = method.getGenericParameterTypes();
for (Type genericParameterType : genericParameterTypes) {
	if (genericParameterType instanceof ParameterizedType) {
		ParameterizedType aType = (ParameterizedType) genericParameterType;
		Type[] parameterArgTypes = aType.getActualTypeArguments();
		for (Type parameterArgType : parameterArgTypes) {
			Class parameterArgClass = (Class) parameterArgType;
			System.out.println("parameterArgClass = " + parameterArgClass);
		}
	}
}
// field generic type
Field field = Klass.class.getField("phones");
Type genericFieldType = field.getGenericType();
if (genericFieldType instanceof ParameterizedType) {
	ParameterizedType aType = (ParameterizedType) genericFieldType;
	Type[] fieldArgTypes = aType.getActualTypeArguments();
	for (Type fieldArgType : fieldArgTypes) {
		Class fieldArgClass = (Class) fieldArgType;
		System.out.println("fieldArgClass = " + fieldArgClass);
	}
}
		
```

### 获取数组的类对象
```java
{
/*
		 * int[] intArray = (int[]) Array.newInstance(int.class, 3);
		 * 
		 * Array.set(intArray, 0, 123); Array.set(intArray, 1, 456); Array.set(intArray,
		 * 2, 789);
		 * 
		 * intArray[0] = 1;
		 * 
		 * System.out.println("intArray[0] = " + Array.get(intArray, 0));
		 * System.out.println("intArray[1] = " + Array.get(intArray, 1));
		 * System.out.println("intArray[2] = " + Array.get(intArray, 2));
		 */

//		Class stringArrayClass = String[].class;
		// 运行时获取 int[] 数组，注意是 [I, I 在虚拟机中表示 int,
		Class intArray = Class.forName("[I");
		Class stringArrayClass = Class.forName("[Ljava.lang.String;");
		
		// 通用的获取任何元素类型数组类的方法
		String theClassName = intArray.getName();
		Class theClass = getClass(theClassName);
		Class typeArrayClass = Array.newInstance(theClass, 0).getClass();
		System.out.println("is array: " + typeArrayClass.isArray());

		// 获取数组元素类  ComponentType of an array
		Class stringArrayComponentType = stringArrayClass.getComponentType();
		System.out.println(stringArrayComponentType);
	}

	public static Class getClass(String className) throws ClassNotFoundException {
		System.out.println("className: " + className);
		if ("int".equals(className))
			return int.class;
		if ("long".equals(className))
			return long.class;
		return Class.forName(className);
	}
```

## class loading & reloading
Java 类加载机制遵循一个层级结构，一个创建一个类加载器必须告知它的父类加载器，子加载器加载一个类之前先会委托父加载器加载，只有父加载器无法加载时候，子加载器才进行加载。加载步骤:
1. 检查类是否已经加载。
2. 如果未被加载，委托父加载器加载。
3. 如果父加载器无法加载，则尝试自己加载。

### 动态类加载
```java
public class MainClass {

  public static void main(String[] args){

    ClassLoader classLoader = MainClass.class.getClassLoader();

    try {
        Class aClass = classLoader.loadClass("com.jenkov.MyClass");
        System.out.println("aClass.getName() = " + aClass.getName());
    } catch (ClassNotFoundException e) {
        e.printStackTrace();
    }

}
```
### 动态 类重加载
运行时加载一个类, 重新加载一个类必须实现自己的classloader, 因为Java 内置的类加载器加载一个类都会检查类是否已经加载过。

### 设计类重加载代码
Java中每一个类被加载后都通过全类名fully qualified name(包+类名) 和类加载实例来区分。同一个类，A加载器和B加载器加载出来的就是不一样的。

```java
MyObject object = (MyObject)
    myClassReloadingFactory.newInstance("com.jenkov.MyObject");
```
如果以上代码每次使用不同的类加载器加载MyObject，就无法进行强制(cast)转换了. 因此，良好的设计应该采用接口或者超类来引用被加载的类的实例.

```java
MyObjectInterface object = (MyObjectInterface)
myClassReloadingFactory.newInstance("com.jenkov.MyObject");

MyObjectSuperclass object = (MyObjectSuperclass)
myClassReloadingFactory.newInstance("com.jenkov.MyObject");
``` 


### appendix

```java
package advance.reflection;

import java.util.List;

interface Interface {
	void print();
}
public class Klass implements Interface {
	String name;
	int id;
	public List<String> phones;
	
	public Klass() {
		this("tt", 77);
	}
	
	public Klass(String name, int id) {
		this.name = name;
		this.id = id;
	}
	
	public Klass(String name, int id, List<String> phones) {
		this.name = name;
		this.id = id;
		this.phones = phones;
	}
	
	public List<String> getPhones() {
		return this.phones;
	}
	
	public void setPhones(List<String> phones) {
		this.phones = phones;
	}

	@Override
	public void print() {
		System.out.println(id + ": " + name);
	}
}
```



