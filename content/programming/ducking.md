---
title: "ducking"
date: 2019-05-04 13:16
---

多态的特性在于你必须遵循某些接口和规范，对于符合规范的语法，继承父类或者实现接口这种具有父子关系的类，编译器才允许多态运行。对于静态强类型的语言，比如Java，Java在编译的时候就会进行严格的类型检查。

另外，多态虽然非常方便，并且可以减少代码量，将公共部分抽象出来。但那是为了利用这种特性，必须要求类去实现特定的接口，是一种不必要的代码耦合，这是一种糟糕的设计。从Spring和其他容器框架中吸取的教训之一是，为了提供通用功能，不能将自己绑定到特定的类型层次结构。 您应该能够只编写POJO并让框架解决如何处理它们，而不是拥有扩展抽象类或实现一堆接口的类。 在Spring中，我们使用注释，约定或XML映射文件来告诉框架如何理解特定POJO正在尝试做什么。 这样做的优点是，你不用绑定任何东西运行再Spring容器中。

但是对于动态语言 Python 来说，这是不需要的, Python 是动态类型的。不需要特定的类型层次结构也可以实现多态。如果两个方法拥有同样的签名signature, 这意味着他们可能在做同一件事情。这个概念叫 Duck Typing (鸭子类型). 不用确定 IS-A 的关系，而是 Like-A 的关系。

> rather than determining whether something IS-A duck, we just care that it WALKS-like-a duck and QUACKS-like-a duck.

```python
class Duck:
  def quack(self): return "a loud Quaaaaaaaaaaaaack!"
  def walk(self): return "waddle"

class Person:
  def quack(self): return "a person making a quacking sound."
  def walk(self): return "walk"

def tellAStoryAbout(something):
  print("One day while walking in the forest I heard " + something.quack())
  print("Intrigued, I turned round to see a dark shape " + something.walk() + " off into the bushes.")

tellAStoryAbout(Duck())
tellAStoryAbout(Person())
```

在 Golang 中没有继承的概念，类的层次结构不需要显式申明，只需要实现相同的接口函数就意味着他们拥有相同的父亲。Golang 的接口使用起来非常方便，如果你想让某个类看起来像是某个父类的孩子，你只要实现该接口就行了。

> When I see a bird that walks like a duck and swims like a duck and quacks like a duck, I call that bird a duck.

```Golang
package main

import "fmt"

type Something interface {
  Quack() string
  Walk() string
}

type Duck struct {
}
func (d Duck) Quack() string {
  return "a loud Quaaaaaaaaaaaaack!"
}
func (d Duck) Walk() string {
  return "waddle"
}

type Person struct {
}
func (p Person) Quack() string {
  return "a person making a quacking sound."
}
func (p Person) Walk() string {
  return "walk"
}

func tellAStoryAbout(something Something) {
  fmt.Println("One day while walking in the forest I heard " + something.Quack())
  fmt.Println("Intrigued, I turned round to see a dark shape " + something.Walk() + " off into the bushes.")
}

func main() {
  duck := Duck{}
  person := Person{}
  tellAStoryAbout(duck)
  tellAStoryAbout(person)
}
```

对于 Java 这种强类型语言，提供鸭子类型的支持不会像Python这么直接。当然利用反射机制，我们可以直接调用任何类的方法，但是这么做的话就不那么优雅了。我们可以使用动态代理来代理某个需要代理的鸭子类型的类，这样那些鸭子类型的类只要是含有接口方法实现，我们就可以检查到并使用代理类代替鸭子类型的类来执行方法调用。

```java
package advance.reflection;

interface Something {
	String quack();
	String walk();
}

class Duck {
	public String quack() {
		return "a loud Quaaaaaaaaaaaaack!";
	}
	public String walk() {
		return "waddle";
	}
}

class Person {
	public String quack() {
		return "a person making a quacking sound.";
	}
	public String walk() {
		return "walk";
	}
}

public class DuckTypeTest {
	public static void tellAStoryAbout(Object object) {
		Something something = DuckType.coerce(object).to(Something.class);
		System.out.println("One day while walking in the forest I heard " + something.quack());
		System.out.println("Intrigued, I turned round to see a dark shape " + something.walk() + " off into the bushes.");
	}
	public static void main(String[] args) {
		Duck duck = new Duck();
		Person person = new Person();
		tellAStoryAbout(duck);
		tellAStoryAbout(person);
	}
}
```
以下代码可以在Java中实现鸭子类型：
```java
public class DuckType {
	private final Object objectToCoerce;
	private DuckType(Object objectToCoerce) {
		this.objectToCoerce = objectToCoerce;
	}
	
	private class CoercedProxy implements InvocationHandler {
		public Object invoke(Object proxy, Method method, Object[] args)
				throws Throwable {
			Method delegateMethod = findMethodBySignature(method);
			assert delegateMethod != null;
			return delegateMethod.invoke(DuckType.this.objectToCoerce, args);
		}
	}
	
	/**
	 * Specify the duck typed object to cast/coerce.
	 * @param object the object to coerce.
	 * @return
	 */
	public static DuckType coerce(Object object) {
		return new DuckType(object);
	}
	
	/**
	* Coerce the Duck Typed object to the given interface providing it
	* implements all the necessary methods.
	 * @param <T>
	*
	* @param 
	* @param iface
	* @return an instance of the given interface that wraps the duck typed
	*         class
	* @throws ClassCastException if the object being coerced does not implement
	*             all the methods in the given interface.
	*/
	public <T> T to(Class iface) {
		assert iface.isInterface() : "Cannot coerce object to a class, must be an interface";
		if (isA(iface)) {
			return (T) iface.cast(objectToCoerce);
		}
		if (quacksLikeA(iface)) {
			return generateProxy(iface);
		}
		throw new ClassCastException("Could not coerce object of type " 
						+ objectToCoerce.getClass() + " to " + iface);
	}
	
	public boolean isA(Class iface) {
		return objectToCoerce.getClass().isInstance(iface);
	}
	
	/**
	* Determine whether the duck typed object can be used with
	* the given interface.
	*
	* @param  Type of the interface to check.
	* @param iface Interface class to check
	* @return true if the object will support all the methods in the
	* interface, false otherwise.
	*/
	
	public boolean quacksLikeA(Class iface) {
		for (Method method : iface.getMethods()) {
			if (findMethodBySignature(method) == null) {
				return false;
			}
		}
		return true;
	}
	
	public Method findMethodBySignature(Method method) {
		try {
			return objectToCoerce.getClass().getMethod(method.getName(), method.getParameterTypes());
		} catch (NoSuchMethodException e) {
			return null;
		}
	}
	
	@SuppressWarnings("unchecked")
	private <T> T generateProxy(Class iface) {
		return (T) Proxy.newProxyInstance(iface.getClassLoader(),
					new Class[] {iface}, new CoercedProxy());
	}
}
```

 -[Dependency Injection, Duck Typing, and Clean Code in Go](http://txt.fliglio.com/2015/04/di-duck-typing-and-clean-code-in-go/)