---
title: Java异常
date: 2018-01-20
categories: Java基础
author: Sunny
tags:
    - 基础
    - Java异常
cover_picture: http://upload-images.jianshu.io/upload_images/5231076-382f3d5351c2e6a0.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240
---

#### 一.异常的分类

Java中的异常基类为Throwable，Error和Exception都继承自Throwable。

Error是指系统错误或者资源耗尽，通常这些Error都是来自虚拟机，比较常见的Error有内存溢出错误(OutOfMemoryError)，栈溢出错误(StackOverflowError)。

Exception是指代码异常，如果代码中抛出了这种异常就说明应用程序有问题。它分为RunTimeException和CheckTimeException，RunTimeException叫做**运行时异常**，又叫做未受检异常，unchecked exception，这种异常通常是在应用程序运行的过程中发生的，比如最常见的NullPointerException、IllegalArgumentException等等。CheckTimeException叫做**编译时异常**，又叫做受检异常，checked exception，顾名思义，这种异常如果我们不进行人为捕获处理的话，编译器是不会通过的。

![Throwable.png](http://upload-images.jianshu.io/upload_images/5231076-382f3d5351c2e6a0.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

#### 二.异常处理机制

- 当异常出现时，系统会new一个Throwable对象抛出；
- 当前应用程序会中止运行；
- 系统开始寻找异常处理程序，也就是catch语句中的代码，当前方法中没有符合的会由里向外不断向上层调用方法抛，直至主函数，如果有对应的异常处理程序，经过异常处理之后当前应用程序可能会换一种方式运行或者继续执行下去（这取决于软件开发者），如果没有对应的异常处理程序，系统会启动默认异常处理，即在控制台打印出异常信息并且程序退出。

#### 三.使用throw抛出异常

throw总是在方法体中出现，一般都是对某个不符合预期的并且会影响到程序正常运行的条件使用throw抛出对应的异常，让调用者明确程序中出现的问题；或者还有一种情况就是当前方法体中对于这种异常处理不了，需要抛出到上层方法中去处理。

比如方法中传入一个参数，要求：这个参数的值不能大于5，否则会导致程序运行不正常。

```java
public void foo(int var){
  if(var < 5){
    throw new IllegalArgumentException("Argument must less than 5");
  }
  ...
}
```

#### 四.使用throws抛出异常

使用throws抛出异常一般是当前方法没有处理这种异常的能力，需要由调用者方法去处理异常。

用法：在方法声明处使用throws抛出方法中可能出现的异常。

```java
public void foo() throws Exception1,Exception2...{
  //do something
}
```

注意：方法中是不允许抛出未处理的受检异常的，这里的未处理包括两方面：1.在方法声明中用throws抛出；2.使用try/catch进行处理。比如父类中声明了IOException，子类在覆写这个方法的时候抛出了SQLException，这是不允许的，因此，这就要求父类应该尽可能的声明所有可能出现的异常。方法中抛出RunTimeException是不需要声明的。

#### 五.捕获异常

语法形式：

```java
public void foo(){
  try{
    //do something
  }catch(Exception1 e){
    //do something
  }catch(Exception2 e){
    //do something
  }finally{
    //do something
  }
}
```

try块中是可能出现异常的代码，catch块中的是对相应异常的异常处理代码，finally块正如字面意义那样指最后执行的一段代码，大多数情况下finally块中的代码都会执行到，除非出现下面几种情况：

- finally块前面的代码中出现`System.exit()`致使程序退出了；
- 当前代码所在的线程死亡；

**异常的重新抛出**

有时我们在收到某个异常之后，发现我们并没有能力去处理这个异常，那么我们就需要把这个异常重新抛出，交由调用者去处理。或者还有一种情况是获取到的异常信息较少，我们希望在异常中加入更具体的描述信息，也会重新抛出异常。

#### 六.finally

前面我们提过几种情况下会导致finally中的代码不会被执行到，但是这些情况很少遇到，我们可以这样认为不管是否有异常，finally中的代码都会执行，但是有几点是我们容易忽略的：

- 在try/catch/finally语法中，catch和finally都不是必须的，可以是try/catch，也可以是try/finally；

- 如果在try或者catch块中出现return语句，finally中的代码会在return语句执行前执行，但是有一点需要注意的是finally中的代码是不会影响return返回的结果的，比如：

  ```java
  public int foo() {
      int a = 1;
      int b = 2;
      try {
         return a + b;
      }finally {
         a = 3;
         b = 4;
      }
  }

  返回结果：
  3
  ```

  上面方法返回的结果是3而不是7，这是因为`a+b`的结果在return语句执行之前用临时变量保存起来了，当finally代码执行完之后，return会把临时变量的值返回。

- 如果try或者catch块中有return语句，并且finally代码块中也有return语句，那么finally中的return会覆盖try或者catch块中的return，比如：

  ```java
  public int foo(){
    try{
      return 3;
    }finally{
      return 4;
    }
  }

  返回结果：
  4
  ```

- 如果try或者catch块中抛出了异常，并且finally中有return语句，这个return语句会覆盖这些异常，使得上层代码接收不到底层抛出的异常；

- 如果try或者catch块中抛出了异常，并且finally中也抛出了异常，那么finally中的异常会覆盖try或者catch块中异常，比如：

  ```java
  public void foo(){
    try{
      throw new IllegalArgumentException("illegalArgument exception");
    }finally{
      throw new NullPointerException("null exception");
    }
  }
  ```

  上面这种写法在上层代码中只会接收到空指针异常，不合法异常被空指针异常覆盖。

#### 七.什么是异常链

当程序抛出异常时，我们希望在接受到这个异常后重新抛出一个新的异常，并且原来的异常信息保留，通常Throwable的子类都会提供Throwable类型参数的构造方法，方便存储上一个异常信息，这样一层接着一层就构成了异常链。

#### 八.自定义异常

自定义异常需要继承Exception类，如下：

```java
public class CustomException extends Exception{
  private String mImportantMessage;
  
  public CustomException(){}
  
  public CustomException(String exceptionMessage){
    super(exceptionMessage);
  }
  
  public CustomException(Throwable cause){
    super(cause);
  }
  
  public CustomException(String exceptionMessage,Throwable cause){
    super(exceptionMessage,cause);
  }
}
```

#### 九.关于异常常见的几个问题

1.什么是异常？异常的分类有哪些？

异常是指阻碍程序正常执行的错误，分类有Error和Exception，Error一般指系统错误（通常由虚拟机抛出）或资源耗尽，通常是由系统使用，应用程序不应抛出和处理，一般Error有内存溢出错误和栈溢出错误等。Exception又分为运行时异常(RunTimeException)和检查型异常(CheckTimeException)。

2.Error和Exception的区别？

Error一般指系统抛出的错误或资源耗尽，通常都是虚拟机抛出；Exception指代码异常，也就是说程序中抛出了这种异常说明我们的代码出现了问题，Exception又分为运行时异常和检查型异常，运行时异常是在程序执行过程中抛出的异常，检查型异常是编译时就要求开发者针对异常进行处理的异常，换句话说就是这些异常是可预知的，我们知道这块代码可能会出现这样的问题，然后我们可以针对这些问题要一些纠正性的操作。

3.常见运行时异常(RunTimeException)和检查型异常(CheckTimeException)有哪些？

运行时异常：NullPointerException(空指针异常)，ArrayIndexOutOfBoundsException(数组索引越界异常)，IllegalArgumentException(非法参数异常)等等；

检查型异常：IOException(IO类的异常)，NumberFormatException(字符串转换为数字抛出的异常)等等。

4.异常的机制（异常的处理流程）？

- 在代码出现异常时，系统会和new普通对象一样，在Java堆内存空间中中生成一个Throwable对象；
- 当前出现异常的代码停止执行，系统会寻找匹配的异常处理程序，当前方法体中如果没有匹配的则抛向上层代码直至主函数，如果到主函数都没有相应的异常处理代码，系统就会启用默认的异常处理，即打印异常信息到控制台并退出程序，一般在实际应用中会有专门的日志记录打印的异常信息。异常处理程序的目的是纠正出现异常的代码或者说对异常代码产生的后果做下一步操作以让程序能够继续执行下去或者换一种方式执行。

5.怎么自定义异常？

自定义异常一般需要继承Exception类，然后提供三类构造方法。

6.Java异常类有哪些重要的方法？

Java异常类Exception及其子类提供的方法调用很有限，通常都是调用Throwable类中的方法。

```java
//获取异常详细信息
String getMessage();

//获取cause对象
Throwable getCause();

//获取用本地语言描述的详细信息，其实在Throwable的内部该方法调用的是getMessage()
String getLocalizedMessage();

//返回对Throwable的简单描述，如果有异常的详细信息也会一同返回
String toString();

//打印Throwable的调用栈轨迹
void printStackTrace();

//打印异常信息到指定的流
void printStackTrace(PrintStream s);
void printStackTrace(PrintWriter s);

//获取异常栈每一层的信息，StackTraceElement包括文件名，类名，方法名以及行号等信息
StackTraceElement[] getStackTrace();
```

#### 十.参考

[异常上](http://mp.weixin.qq.com/s/SB5d0IU7aj-hRSneg3hE9g)

[异常下](http://mp.weixin.qq.com/s/8XiwZGn8djtO7TvtonAe8w)

[Java基础夯实2：全面了解异常](https://mp.weixin.qq.com/s/q0jsOrBSNezGIBOzozlQzA)



