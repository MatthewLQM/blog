---
title: Java8 新特性之 lambda 表达式
date: 2017-09-15 15:34:22
categories:
- JAVA
- JAVA8 新特性
tags:
- JAVA8
- lambda
---
本篇文章主要讲了在 Java8 中加入的 Java 语言新特性：lambda 表达式，主要分为四部分：lambda 表达式的语法、作用、使用限制以及使用过程中可能会遇到的一些问题，具体 Java 语言是如何实现 lambda 表达式的，以及其  
在 Java 虚拟机中的表现，在字节码中的表现，我还没有搞太懂，所以在这篇文章中，我们也不会提及。
<!-----more-------->
### Java8 加入 lambda 表达式的主要原因
Java 语言，自从产生开始，就是一种纯面向对象的语言，除了 int, char 等基本数据类型，其他的都是对象，而且 Java 也提供了基本类型的封装类，比如 Integer 等，目的也是为了使其面向对象的特性更加的透彻。这样做虽然
有好处，但是也有其缺点，就是在 Java 中定义的函数（或者说是方法）不可能独立存在，同时方法也不能够作为参数或者是返回一个方法给实例。在出现 lambda 表达式之前，解决这一问题的主要方法是利用匿名类，比如线程的初
始化，在 Java8 之前只能这么写：
```Java
new Thread(new Runnable(){
    @Override
    public void run() {
        // Do Something
    }
}).start
```
其实 thread 中只需要一个 run() 函数，但是必须要声明一个类去包装他，因为在 Java 的世界中，函数无法单独存在。于是，在 Java8 中，提供了一种函数式编程的方法：lambda 表达式。
### lambda 表达式的语法
我们先来看几个 lambda 表达式的示例：
```Java
// 利用 lambda 表达式实现线程的初始化
new Thread(() -> System.out.println("hello world")).start();

// 利用 lambda 表达式在 Swing 中实现监听器
button.addActionListener( (e) -> System.out.println("button was clicked"));

// 利用 lambda 表达式遍历集合类
list.foreach((element) -> {
    System.out.println("Travers the list by lambda expression");
    // Do Sometion
})
```
在 Java 8 中新加入了 lambda 表达式，其主要的语法分为三种：
```Java
(params) -> expression;
(params) -> statement;
(params) -> { statements; }
```
lambda 表达式的主要结构描述如下：
 + 一个 lambda 表达式可以有一个或多个参数；
 + 参数类型可以在代码中显示声明，也可以根据上下文来进行推断；
 + 所有参数要放在圆括号中，通过逗号间隔（和函数参数相同）,括号中内容为空时表示没有参数；
 + 当只有一个参数，并且参数陈类型可以推导时，可以省略圆括号；
 + lambda 表达式的主体可以包含有零个、一个或者多个语句；
 + 当主体只有一个表达式时，花括号可以省略，匿名函数的返回类型与该主体表达式一致；
 + 当主体包含一条以上语句，则表达式必须包含在花括号{}中（形成代码块）。匿名函数的返回类型与代码块的返回类型一致，若没有返回则为空

### 函数式接口
函数式接口指的是只包含一个抽象方法声明的接口，比如 `java.lang.Runnable` 就是一个典型的函数式接口，只包含一个方法 `void run()`。在 Java8 之前，我们需要使用匿名内部类来实例化函数式接口的对象，在 Java 8 增
加了 lambda 表达式以后，可以将这一操作进行简化，使用 lambda 表达式来对函数式接口进行赋值，例如，我们可以使用 lambda 表达式来创建一个 Runnable 的引用：
```Java
Runnable r = () -> System.out.println("hello world");
```
在 Java 8 中，加入了一个新的注解 `@FunctionalInterface`,这个注解可以用来指明一个接口是函数式接口，这个接口只能有一个抽象方法，如果有两个抽象方法，那么编译器会报错。在定义了这个函数式接口以后，我们可以在 
API 中使用他，并且使用 lambda 表达式，下面是一个比较长的例子：
```Java
public class Test {

    //定义一个函数式接口
    @FunctionalInterface
    public interface WorkerInterface {
        public void doSomeWork();
    }

    public class WorkerInterfaceTest {
        public static void execute(WorkerInterface worker) {
        worker.doSomeWork();
    }

    public static void main(String [] args) {

        //invoke doSomeWork using Annonymous class
        execute(new WorkerInterface() {
            @Override
            public void doSomeWork() {
                System.out.println("Worker invoked using Anonymous class");
            }
        });
        
        //invoke doSomeWork using Lambda expression 
        execute( () -> System.out.println("Worker invoked using Lambda expression") );
    }
}
```
运行的输出如下：
```
Worker invoked using Anonymous class 
Worker invoked using Lambda expression
```

### lambda 表达式的作用
在 Java 8 引入 lambda 表达式以后，我们可以利用其做如下的功能：
 1. 替换匿名类的实现，比如 `Runnable` 和 `Listener`；
 2. 对列表类(`Collections`)进行迭代操作，同时可以加入方法引用（将会在写文章另说）；
 3. 利用函数式接口，简化代码；
 4. 使用流 API 进行 Map 和 Reduce 操作，进行过滤。
其中，lambda 表达式与匿名类实现有所不同的是在 lambda 表达式中，this 关键字指向的是外部类的引，同时 Java 编译器将把 lambda 表达式编译成类的私有方法。

### lambda 表达式的使用限制
 1. lambda 表达式仅能放入如下代码：预定义使用了 @Functional 注释的函数式接口，自带一个抽象函数的方法或者是 SAM（Single Abstract Method）类型。
 2. lambda 表达式内可以使用方法引用，仅当该方法不修改 lambda 表达式提供的参数。
 3. lambda 内部可以使用静态、非静态和局部变量。
 4. lambda 表达式在 Java 中又称为闭包或者是匿名函数。
 5. lambda 表达式有个限制，就是只能引用 final 或 final 局部变量，不能在 lambda 内部修改定义在域外的变量；但是只访问，不修改是可以的。
 
### lambda 表达式在使用过程中的一些问题
 1. lambda 表达式需要一个返回值，如果只有一个表达式，那么编译器会自动返回这个表达式的值，如果有大括号，则需要显式指明返回什么数值。
 2. 不能够在 lambda 内部修改定义在域外的变量，否则会编译错误。
 3. lambda 表达式中的 final 指两种，一种是定义为 final ，另外一种是没有在外部被更改，可以推断成 final 的变量。
对于第三种情况，这里举个例子来详细解释一下：
```Java
final List<Integer> a = new ArrayList();        // a 可以用在 lambda 表达式中，因为是 final 变量
List<Integer> b = new ArrayList();              // b 可以用在 lambda 表达式中，因为 b 可以推断为 final 变量
b = a;                                          // 此时 b 不能用在 lambda 表达式中，因为赋值语句破坏了 b 的 final 属性
b.foreach(inta -> a.add(inta))                  // 这样是允许的，因为列表的 add 操作并不会修改链表本身的值（考虑 Java 虚拟机实现）
final Integer intb;
b.foreach(inta -> intb = inta)                  // 错误，lambda 表达式不能修改外部的值
b.foreach(inta -> inta = intb)                  // 正确，lambda 表达式可以修改内部的值
```

### 总结
lambda 表达式是 Java 8 引入的新特性，但是其并没有影响 Java 语言的实现，说到底，lambda 表达式只是一个语法糖，提高大家写代码的速度和效率，同时为大家提供便利。
lambda 表达式是编程模型上的一种提升，赋予了 Java 缺失的函数式编程的特性。

#### 参考
 1. [深入浅出 Java 8 Lambda 表达式](http://blog.oneapm.com/apm-tech/226.html)
 2. [Java 8: The First Taste of Lambdas](https://zeroturnaround.com/rebellabs/java-8-the-first-taste-of-lambdas/)
 3. [Java8 Lambda表达式](http://www.jianshu.com/p/141cf822cb70)