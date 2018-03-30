---
title: Java UML类图关系说明
date: 2018.3.30
categories: Java基础
author: Sunny
tags:
    - Java继承
    - UML
cover_picture: https://upload-images.jianshu.io/upload_images/5231076-4e389e75f56a24fd.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240
---

### 1.概览

类与类之间大体分为五种关系，分别为依赖关系，关联关系，聚合关系，组合关系以及继承关系。

![UML类图关系.png](https://upload-images.jianshu.io/upload_images/5231076-4e389e75f56a24fd.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

### 2. 依赖关系

依赖关系是单向的，表示一个类依赖于另一个类的定义，是“use a"的关系，如果A依赖于B，则B表现为A的局部变量，方法参数，静态方法调用等。

```java
public class Person{
  public void doSomething1(){
    Car car = new Car();//局部变量
    ...
  }
  
  public void doSomething2(Car car){//方法参数
    ...
  }
  
  public void doSomething3(){
    int price = Car.do();//静态方法调用
  }
}
```

### 3. 关联关系

单向或者双向的关系，通常我们应该避免双向关联关系，它是一种”has a“的关系。如果A单向关联B，则A has a B，一般类中的表现为全局变量。

```java
public class Person{
  private Phone phone;//全局变量
  
  public void setPhone(Phone phone){
    this.phone = phone;
  }
  
  public Phone getPhone(){
    return this.phone;
  }
}
```

### 4. 聚合关系

它是一种单向的关系，是关联关系的一种，与关联关系的区别仅表现在语义上，处于关联关系的两个对象通常是平等的，但是聚合关系的两个对象是一种整体与局部的感觉，一般也是表现为全局变量，与关联关系在实现上区别不大。

```java
public class Team{
  private Person person;
  
  public void setPerson(Person person){
    this.person = person;
  }
}
```

一个团队是由人组成的(一个人或者多个人)，但是团队解散了，人还可以加入别的组；两个对象的生命周期是不同的，可能Person的生命比Team更长，因为它是从外部传入的，当Team销毁时，外部可能还持有Person的引用，那么Person这个对象就还存活着。

### 5. 组合关系

单向，它是一种强依赖的特殊聚合关系，比如一个人由头，手，脚组成，一旦主题不存在，局部也将消亡。

```java
public class Person{
  private Head head;
  private Leg leg;
  private Body body = new Body();//对象在类内部实例化且时全局变量引用
  
  public Person(){
    this.head = new Head();//同理
    this.Leg = new Leg();
  }
}
```

### 6. 继承关系

类实现接口，类继承抽象类，类继承父类都属于继承关系；细分可以分为：实现，类实现接口；泛化，是”is a”的关系，类继承抽象类，类继承普通父类都属于这种关系，只能单继承。