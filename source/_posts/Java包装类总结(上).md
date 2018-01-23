---
title: Java包装类总结（上）
date: 2018-01-23
categories: Java基础
author: Sunny
tags:
    - Java基础
    - 包装类
cover_picture: http://upload-images.jianshu.io/upload_images/5231076-4fbe8e6ba40c6240.jpg?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240
---

![美女一.jpg](http://upload-images.jianshu.io/upload_images/5231076-4fbe8e6ba40c6240.jpg?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

#### 一.包装类简介

Java中的基本数据类型都有其对应的包装类，包装类中一般包括基本数据类型的变量，静态方法，静态变量以及实例方法等，提供了对基本数据类型的各种操作。Java中基本数据类型对应的包装类如下：

| 基本数据类型  |    包装类    |
| :-----: | :-------: |
|  byte   |   Byte    |
|  short  |   Short   |
|   int   |  Integer  |
|  long   |   Long    |
|  float  |   Float   |
| double  |  Double   |
| boolean |  Boolean  |
|  char   | Character |

#### 二.自动装箱/拆箱的原理

Java1.5之后增加了自动装箱拆箱技术，自动装箱拆箱是在编译时进行的，它是编译器提供的一个功能，其基本原理大概如下：

```java
//Java包装类中都提供了这两类方法：valueOf(),xxxValue()
public static Integer valueOf(int i);
public static Long valueOf(long l);
...
public int intValue();
public long longValue();
...
  
//那么自动装箱拆箱就是利用上面两种方法实现的
Integer a = 2;
int b = a;
//上面两行代码在经过编译器编译时会转换成下面这样
Integer a = Integer.valueOf(2);
int b = a.intValue();	
```

#### 三.我们需要掌握的包装类知识点

##### 1.继承了Object类

为什么要特别的把继承自Object类拿出来说呢？因为有部分需要注意的知识点都来自这里，接下来我们就具体来看一下。

Java中所有包装类都继承了Object类的如下三个方法：

```java
public boolean equals(Object obj);
public int hashCode();
public String toString();
```

**equals()**

首先我们来看看equals()方法，这个方法在Object类中是这样的：

```java
public boolean equals(Object obj){
  return (this == obj);
}
```

从代码中我们就很清楚的看到这里仅仅只是比较了当前对象与传入对象的内存地址，因为在Java中`==`是会比较两个对象的引用是否相等，我们在大多数情况下仅比较对象的引用是否相等是不够的，比如一个类中有多个变量，但是我们想要其中一个变量只要是相等，就判定它们是相等的：

```java
public class Computer{
  private String brand;//品牌
  private String size;//尺寸
  private float weight;//重量
  private String macAddress;//MAC地址
  ...
}
```

我们判定只要是MAC地址一样的电脑，它们就是同一台电脑，如果此时我们还使用Object类中这个默认的equals()方法的话就达不到我们想要的需求，怎么办呢？覆写！我们通过覆写可以把它改写成这样：

```java
public class Computer{
  	private String brand;//品牌
  	private String size;//尺寸
 	private float weight;//重量
  	private String macAddress;//MAC地址
  	...
      
    @Override
    public boolean equals(Object obj){
      	if(this.macAddress == null || obj == null || !(obj instanceOf Computer)){
        	return false;
      	}
      	if(this == obj){
        	return true;
      	}
      	final String mac = ((Computer)obj).macAddress;
      	return macAddress.equals(mac);
  	}
}
```

上面这个例子就是想说明Object类中的equals()方法的功能过于简单，有时候并不能满足我们的需求，包装类就是很好的例子，所有的包装类都对equals()方法进行了覆写，我们具体来看几个包装类：

```java
//Integer类
public boolean equals(Object obj) {
    if (obj instanceof Integer) {
      	//这里比较了包装类中的值
        return value == ((Integer)obj).intValue();
    }
    return false;
}

//Long类
public boolean equals(Object obj) {
    if (obj instanceof Long) {
        return value == ((Long)obj).longValue();
    }
    return false;
}

//Byte类
public boolean equals(Object obj) {
    if (obj instanceof Byte) {
        return value == ((Byte)obj).byteValue();
    }
    return false;
}

//Short类
public boolean equals(Object obj) {
    if (obj instanceof Short) {
        return value == ((Short)obj).shortValue();
    }
    return false;
}

//Float类
public boolean equals(Object obj) {
    return (obj instanceof Float)
               && (floatToIntBits(((Float)obj).value) == floatToIntBits(value));
}

//Double类
public boolean equals(Object obj) {
   return (obj instanceof Double)
               && (doubleToLongBits(((Double)obj).value) ==
                      doubleToLongBits(value));
}

//Boolean类
public boolean equals(Object obj) {
   if (obj instanceof Boolean) {
       return value == ((Boolean)obj).booleanValue();
   }
   return false;
}

//Character类
public boolean equals(Object obj) {
   if (obj instanceof Character) {
       return value == ((Character)obj).charValue();
   }
   return false;
}
```

从上面八种包装类的equals()方法的源码来看，有六种包装类的equals()方法的实现原理是相同的，比较的都是包装类中的基本数据类型变量的值，只有Float和Double的稍有差异，下面我们再具体看一看这两个类的euqals()方法实现：

```java
//Float类
public boolean equals(Object obj) {
   return (obj instanceof Float)
           && (floatToIntBits(((Float)obj).value) == floatToIntBits(value));
}
```

我们发现在equals方法中调用了`floatToIntBits()`方法，查阅JDK文档，文档给出的描述大概是这样的：`根据IEEE 754浮点单一格式位布局，返回指定浮点值的十进制表示形式`，这里可以理解为把float的二进制表示看作int，与该方法对应的是把给定的int值转为float值`intBitsToFloat(int bits)`。因此，Float类的equals方法只有在两个float值的二进制完全一样的时候才会返回true，我们都知道计算机在计算浮点数的时候是不精确的，比如：

```java
float number = 0.1f * 0.1f;
```

我们可能想当然的认为这个结果是0.01，但是计算机算出来的结果却是0.010000001，因此，如果我们把下面两个数比较的话结果就是false：

```java
Float a = 0.01f;
Float b = 0.1f * 0.1f;
System.out.println(a.equals(b));
System.out.println(Float.floatToIntBits(a));
System.out.println(Float.floatToIntBits(b));

输出结果：
false
1008981770
1008981771
```

其实Double类中的equals()方法实现和Float很相似：

```java
public boolean equals(Object obj) {
   return (obj instanceof Double)
           && (doubleToLongBits(((Double)obj).value) ==
                      doubleToLongBits(value));
}
```

这里调用了doubleToLongBits()方法，该方法的作用是把double的二进制表示成long形式，同样的，只要double类型的二进制相同，那么两个数就是相等的，而且double也会存在float类型小数不精确的问题。

**hashCode()**

关于对象hash值的问题在这里需要注意的是哈希值是一个int型的整数，一般是由对象中不变的属性映射得来。**相同对象的哈希值必须相同，但不同对象的哈希值也可以相同**。在Java中一般会有这样一个约定：子类在重写equals方法的时候也必须重写hashCode方法，这是因为Java API中的有很多类依赖这种行为，比如集合类。下面是几个包装类中hashCode方法的实现：

```java
//Integer类
@Override
public int hashCode() {
   return Integer.hashCode(value);
}
public static int hashCode(int value) {
   return value;
}

//Boolean类
@Override
public int hashCode() {
   return Boolean.hashCode(value);
}
public static int hashCode(boolean value) {
   //目前并不清楚为什么会选择1231和1237这两个数
   return value ? 1231 : 1237;
}

//Long类
@Override
public int hashCode() {
   return Long.hashCode(value);
}
public static int hashCode(long value) {
   return (int)(value ^ (value >>> 32));
}	
```

**toString()**

每个包装类都实现了toString()方法，返回的结果都是包装类表示的基本数据类型的字符串形式，每个包装类一般都提供了两个toString()方法，一个是Object的toString()方法，另一个则是静态toString()方法，能把某一个具体的基本数据类型的值转换为字符串类型，如下：

```java
//Boolean类
public String toString() {
   return value ? "true" : "false";
}
public static String toString(boolean b) {
   return b ? "true" : "false";
}
```

##### 2.实现了Comparable接口

每个包装类中用作大小比较的是包装类中的基本数据类型，比如：

```java
//Integer类
public int compareTo(Integer anotherInteger) {
   return compare(this.value, anotherInteger.value);
}
public static int compare(int x, int y) {
   return (x < y) ? -1 : ((x == y) ? 0 : 1);
}
```

特别要提一下的是Boolean类中true值要比false大。

##### 3.包装类与基本数据类型以及字符串之间的互转

包装类都能够实现与基本数据类型以及字符串之间的互转，比如：

```java
//Double类
//基本数据类型转包装类
Double d1 = 0.1;//自动装箱
//或者
Double d2 = Double.valueOf(0.1);

//包装类转基本数据类型
double d3 = d1;//自动拆箱
//或者
double d4 = d2.doubleValue();

//包装类转字符串
String s = d1.toString();

//基本数据类型转字符串
String s1 = Double.toString(0.1);

//字符串转基本数据类型
double d5 = Double.parseDouble("0.1");
```

##### 4.包装类中常用的常量

以下几个包装类中的常量是需要我们掌握的：

Boolean类型：

```JAVA
public static final Boolean TRUE = new Boolean(true);
public static final Boolean FALSE = new Boolean(false);
```

数值类型包装类中的最大值和最小值：

```java
public static final double MAX_VALUE = 0x1.fffffffffffffP+1023;
public static final double MIN_VALUE = 0x0.0000000000001P-1022;
```

Float和Double类中定义的一些特殊值：

```java
//正无穷
public static final float POSITIVE_INFINITY = 1.0f / 0.0f;
//负无穷
public static final float NEGATIVE_INFINITY = -1.0f / 0.0f;
//不是数
public static final float NaN = 0.0f / 0.0f;
```

特别提一下，这个NaN有些变态的面试中可能会问到，比如下面这样一道题：

```java
Java是否存在使得语句i > j || i <= j结果为false的i，j值？
```

这个题目的答案就是NaN，是不是很变态，因为NaN和任何数比较都是false，它并不是一个数(not a number)。

##### 5.继承Number抽象类

Number是一个抽象类，定义了下面几个方法：

```java
public abstract int intValue();
public abstract long longValue();
public abstract float floatValue();
public abstract double doubleValue();
public byte byteValue() {
   return (byte)intValue();
}
public short shortValue() {
   return (short)intValue();
}
```

因此，我们可以知道数值类型的包装类是可以直接利用包装类中的这些方法转为其他5种数值类型所对应的基本数据类型的值，并不需要我们人为的拆箱再强转。

##### 6.包装类的不可变性

包装类的不可变性体现在下面三个方面：

- 所有的包装类都被定义成了final类型的，即包装类不能拥有子类；
- 所有包装类中的基本数据类型都被声明为了final；
- 包装类没有定义setter方法。

#### 四.对于包装类我们是该使用new还是valueOf？

我们建议使用valueOf()方法来构建包装类对象，new每次都会创建新的对象，除了Float和Double的valueOf每次都会创建新的对象外，其他包装类中都有对包装类对象进行缓存，以此来减少对包装类对象的创建。

#### 参考

[剖析包装类上](http://mp.weixin.qq.com/s/ZvJ0ya-mNetZiJBgCTu-7g)
