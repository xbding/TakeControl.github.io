---
layout:     post
title:      "内部类"
subtitle:   " \"内部类\""
date:       2017-08-06 12:00:00
author:     "ShiYu"
catalog: true
tags:
    - Java
---

> 将一个类的定义放在另一个类的定义内部，这就是内部类

### 1、为什么使用内部类
如果只是需要一个对接口的引用，直接定义一个正常的类实现接口即可以满足需求，当这样做可以满足需求时，完全没必要考虑使用内部类。但是考虑这种情况：*如果你想实现一个接口，这个中的一个方法和你自己构建的类中的一个方法拥有相同的名称和相同的参数*，这种情况下，我们就可以考虑使用内部类来实现这个接口，由于内部类对外部类的所有内容都是可以访问的，所以这样做可以完成直接实现这个接口的所需的功能。

使用内部类最吸引人的地方是：**每个内部类都能独立的地继承一个接口的实现，所以无论外围类是否已经继承了某个接口的实现，对内部类都没有影响**。内部类的这个特性，加上它可以访问外部类的所有成员（包括私有变量），可以实现真正意义上的多重继承，而不是利用接口曲线救国。

```java
class A{}
abstrat class B{}
class C extends A{
	B getB(){ 
		return new B(){};//匿名内部类
	}
}
	
public class MulteImplementation{
	static void testA(A a){}
	static void testB(B b){}
	public static void main(String[] args){
		C c=new C();
		testA(c);
		testB(c.getB());
	}
}
```
*上述代码通过内部类，实现类C类对A类和B类对多重继承，不再局限在只能多重实现接口，即使是普通类或抽象类，也可以通过内部类来实现它们的多重继承*

如果不需要解决**多重继承**问题，那么自然可以用别动方式编码，而不需要使用内部类。如果使用内部类，我们将可以获得其他一些特性：

* 内部类可以有多个实例，每个实例都有自己的状态信息，并且与其外围类对象的信息相互独立。
* 在单个外围类中，可以让多个内部类以不同的方式实现同一个接口，或继承同一个类。
* 创建内部类对象的时刻并不依赖于外围类对象的创建。
* 内部类没有令人迷惑的**is-a**关系，它就是一个独立的实体，

### 2、内部类的使用
>内部类有很多难以理解的语法，例如，可以在一个方法里面或任意作用域内定义内部类，可以定义静态的内部类等。这里将内部类分为3种：*普通内部类，局部内部类，静态内部类*。

#### 2.1 普通内部类
>普通内部类是直接定义外部类中的内部类

```java
package top.shixiaoyu.innerClass;

/**
 * Created by ShiYu on 2017/7/24.
 */
public class Test {

    private String outerPrivate="outerPrivate";

    public String outerPublic="outerPublic";

    private void print(String msg){
        System.out.println("外部类方法,"+msg);
    }

    private String outerLabel(String msg){
        return msg;
    }

    class A{//内部类A,演示普通内部类的创建
        private int i=11;
        public int value(){
            return i;
        }
    }
    class B{//内部类B，演示.this用法

        /**
         * 调用外部类方法
         */
        public void outerMethod(){
            Test.this.print("内部类调用");
        }
        /**
         * @return 返回外部类对象对引用
         */
        public Test outer(){
            return Test.this;
        }
    }
    class C{//内部类C,演示内部类拥有外部类所有元素访问权，包括privae元素
        public void test(){
            System.out.println("访问外部类private元素："+outerPrivate);
            System.out.println("访问外部类public元素："+outerPublic);
            System.out.println("访问外部类方法："+Test.this.outerLabel("调用外部类方法"));
        }
    }

    public void test(){
        A a=new A();
        a.value();//演示外部类调用内部类方法

        B b=new B();
        b.outer().print("利用内部类返回对外部类对象调用外部类方法");
        b.outerMethod();

        C c=new C();
        c.test();
    }

    public static void main(String[] args) {
        Test test=new Test();
        Test.A a=test.new A();//演示.new用法，用于在外部创建内部类对象

        test.test();
    }

}

输出结果：
外部类方法,利用内部类返回对外部类对象调用外部类方法
外部类方法,内部类调用
访问外部类private元素：outerPrivate
访问外部类public元素：outerPublic
访问外部类方法：调用外部类方法
```

普通内部类的使用可总结为以下几点：

* 内部类对象与制造它的外部类对象之间有一种联系，使得内部类拥有外部类的所有元素访问权。
* 内部类通过**外部类.this**来获取外部类对象的引用
* 内部类通过**外部类.this.外部类方法**来调用外部类的方法
* 外部类通过内部类的实例才能访问内部类的方法
* 通过**外部类实例.new**来在外部实例化内部类对象，为什么需要使用外部类对象来实例化内部类，因为内部类对象生成的同时，它与外部类存在联系，能访问外部类元素。
  

#### 2.2 局部内部类
> 定义在方法里面或者任意的非类作用域内的内部类称做局部内部类

定义局部内部类有两种理由：

1. 实现了某类型的接口，创建并返回对其的引用
2. 要解决一个复杂的问题，想创建一个类来辅助解决该问题，但又不希望这个类是公共可用的。

```java
package top.shixiaoyu.innerClass;

/**
 * Created by ShiYu on 2017/7/26.
 *
 */
interface TestA{}

public class InnerClass {

    //展示类定义在方法内
    public TestA getA(){
        class PTestA implements TestA{}//在方法作用域内的的内部类
        return new PTestA();
    }

    //展示类定义在任意作用域内
    public void test(boolean a){
        if(a){
            class TestB{//定义在任意作用域中的内部类
                private String a(){
                    return "aaa";
                }
            }
            TestB testB=new TestB();
            testB.a();//注意这里可以访问类的私有方法，这是因为对外部类来说，内部类是它的一个属性        }
        //作用域外就不可以使用TestB类
        //!TestB testB=new TestB()
    }
    
    //展示匿名内部类
    public TestA testA(){
        return new TestA() {//匿名内部类
            private int i;
            {
                i=11;//实例初始化，可利用实例初始化实现构造器的效果
            }
        };
    }
}

```

上述代码中，类**TestB**被定义在类**if**语句的作用域中，这并不是说该类的创建是有条件的，实际上它已经与别的类一起编译过了。但是，在定义TestB的作用域之外，它是不可用的。除此之外，它与普通的类一样。

方法**testA**中，展示了匿名内部类的定义，匿名内部类很奇怪，在返回值中定义了一个类，但是它没有名字，这种语法指的是：*创建一个继承自TestA的匿名类对象*，**通过new表达式返回的引用被自动向上转型为对TestA的引用。上述匿名内部类的语法是下述代码的简化形式：

```java
public class InnerClass{
	class MyTestA implements TestA{
		private int i;
		public MyTestA(){
			i=11;
		}
	}
	
	public TestA testA(){
		return new MyTestA();
	}
}
```
在匿名内部类末尾的分号，并不是标记内部类的结束的，它标记的是表达式的结束，只是表达式刚好包含匿名内部类。

#### 2.3 静态内部类
> 如果不需要内部类对象与外部类对象之间有联系，就可以将内部类声明为**static**。静态内部类又常被称为嵌套类。

静态内部类意味着：

* 要创建静态内部类的对象，并不需要其外部类的对象
* 不能从静态内部类的对象中访问非静态的外围类对象

静态内部类与普通内部类还有一个区别：普通内部类的字段与方法，只能放在类的外部层次上，所以普通内部类不能有static数据和static字段，也不能包含静态内部类，但是静态内部类可以包含所有这些东西。

```java
package top.shixiaoyu.innerClass;

/**
 * Created by ShiYu on 2017/7/26.
 */
public class StaticInnerClass {
    protected static class A{//静态内部类
        private int i=1;
        private void print(){
            System.out.println(i);
        }
    }

    public static void main(String[] args) {
        A a=new A();//创建静态内部类实例，不需要任何外部类的对象
        a.print();
    }
}
 

```

### 3 内部类的继承
因为内部类的构造器必须连接到指向其外部类对象的引用，所以在继承内部类的时候，事情会变的复杂。问题在于，那个指向外围类对象的“秘密的”引用，**必须被初始化**，而在导出类中不再存在可连接的默认对象。要解决这个问题，必须用特殊的语法明确说清它们之间的关联：

```java
class WithInner{
	class Inner{}
}

public class InheritInner extends WithInner.Inner{
	InheritInner(WithInner wi){
		wi.super();
	}
	
	public static void main(String[] args){
		WithInner wi=new WithInner();
		InheritInner ii=new InheritInner(wi);
	}
}
```

上述代码可见，必须在构造器内使用如下语法：**eclosingClassReference.super()**；这样才能提供必要的引用，编译才能通过。

### 4 内部类的标识符

每个类都会产生一个.class文件，其中包含了如何创建该类型对象的全部信息，内部类也必须生成一个.class文件以包含它们的class对象信息。这些类文件的命名有严格的规则：**外围了对象的名字，加上"$"，再加上内部类的名字**。如果内部类是匿名的，编译器会简单地产生一个数字作为其标识符，如果是局部内部类，编译器会在"$"和内部类名字之间加上一个数字。



