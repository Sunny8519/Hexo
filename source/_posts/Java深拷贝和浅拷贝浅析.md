---
title: Java深拷贝和浅拷贝浅析
date: 2017-06-18
categories: Java基础
author: Sunny
tags:
    - Java基础
    - 深拷贝
    - 浅拷贝
cover_picture: https://upload-images.jianshu.io/upload_images/5231076-b8e5d2b7a4968535.jpg?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240
---

![](https://upload-images.jianshu.io/upload_images/5231076-b8e5d2b7a4968535.jpg?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

#### 一.Java语言创建对象的的三种方式

在Java中创建对象共有三种方式：

- new关键字
- clone一个对象
- 反射

clone一个对象的第一步是分配和原对象大小一样的内存空间，然后再调用clone方法在这个内存中创建新的对象。

#### 二.对象的拷贝
对象的拷贝必须要实现Cloneable接口，然后覆写Object类中的clone方法，这是在Object类的clone方法中要求的：

```java
protected Object clone() throws CloneNotSupportedException {
        //当前对象不是Cloneable对象或者Cloneable子类对象就抛异常
        if (!(this instanceof Cloneable)) {
            throw new CloneNotSupportedException("Class " + getClass().getName() + " doesn't implement Cloneable");
        }

        return internalClone();
    }
```

其实Cloneable接口是一个空接口，里面没有抽象方法，它是一种约定：如果要实现对象的拷贝就必须要实现这个接口。

#### 二.浅拷贝
浅拷贝会把当前对象在内存中拷贝一份，当前对象中的属性拷贝要分两种情况：1.属性为基本数据类型，那么在拷贝的新对象中也会把这个基本数据类型的值拷贝过来；2.属性为引用类型，那么在拷贝的新对象中只会把这个引用拷贝过来，引用所指向的具体对象并不会拷贝。
举例：
```java
    public class Person implements Cloneable {
        private int age;
        private Head head;

        public Person() {

        }

        public Person(int age, Head head) {
            this.age = age;
            this.head = head;
        }

        @Override
        protected Object clone() throws CloneNotSupportedException {
            return super.clone();
        }
    }

    public class Head {
        private Face face;

        public Head() {

        }

        public Head(Face face) {
            this.face = face;
        }
    }

    private class Face{

    }

Person person = new Person(25, new Head());
Person personClone = (Person) person.clone();
System.out.println(person);
System.out.println(personClone);
System.out.println(person.getAge() == personClone.getAge());
System.out.println(person.getHead() == personClone.getHead());

输出结果：
com.sunny.demo.scrolldemo.ExampleUnitTest$Person@5e8c92f4
com.sunny.demo.scrolldemo.ExampleUnitTest$Person@61e4705b
true
true
```

从结果可以看出覆写了Object的clone方法的确可以实现对象的拷贝,而且这种写法的结果符合浅拷贝的要求，即基本数据类型把值拷贝过来，引用类型把引用拷贝过来，因此，Object类中默认实现的clone方法只能实现浅拷贝，那么深拷贝又是什么呢？

#### 三.深拷贝
深拷贝就是在拷贝当前对象的基础上还要拷贝它的所有属性对象，包括基本数据类型。

还以前面的例子，代码稍作改动：
```java
    public class Person implements Cloneable {
        private int age;
        private Head head;

        public Person() {

        }

        public Person(int age, Head head) {
            this.age = age;
            this.head = head;
        }

        public int getAge() {
            return age;
        }

        public Head getHead() {
            return head;
        }

        @Override
        protected Object clone() throws CloneNotSupportedException {
            Person person = (Person) super.clone();
            person.head = (Head) head.clone();
            return person;
        }
    }

    public class Head implements Cloneable{
        private Face face;

        public Head() {

        }

        public Head(Face face) {
            this.face = face;
        }

        @Override
        protected Object clone() throws CloneNotSupportedException {
            return super.clone();
        }
    }

    private class Face{

    }

Person person = new Person(25, new Head());
Person personClone = (Person) person.clone();
System.out.println(person);
System.out.println(personClone);
System.out.println(person.getAge() == personClone.getAge());
System.out.println(person.getHead() == personClone.getHead());

输出结果：
com.sunny.demo.scrolldemo.ExampleUnitTest$Person@5e8c92f4
com.sunny.demo.scrolldemo.ExampleUnitTest$Person@61e4705b
true
false
```

从输出结果我们可以发现两个Person对象中的Head属性已经指向不同的对象了，说明在复制Person的时候把Head也复制了一遍，这就实现了对象的深拷贝，但这是彻底的深拷贝吗？我们再把Face属性的引用打印出来，看看是否一样：
```java
    public class Person implements Cloneable {
        private int age;
        private Head head;

        public Person() {

        }

        public Person(int age, Head head) {
            this.age = age;
            this.head = head;
        }

        public int getAge() {
            return age;
        }

        public Head getHead() {
            return head;
        }

        @Override
        protected Object clone() throws CloneNotSupportedException {
            Person person = (Person) super.clone();
            person.head = (Head) head.clone();
            return person;
        }
    }

    public class Head implements Cloneable{
        private Face face;

        public Head() {

        }

        public Head(Face face) {
            this.face = face;
        }

        public Face getFace() {
            return face;
        }

        @Override
        protected Object clone() throws CloneNotSupportedException {
            return super.clone();
        }
    }

    private class Face{

    }

Person person = new Person(25, new Head(new Face()));
Person personClone = (Person) person.clone();
System.out.println(person);
System.out.println(personClone);
System.out.println(person.getAge() == personClone.getAge());
System.out.println(person.getHead() == personClone.getHead());
System.out.println(person.getHead().getFace());
System.out.println(personClone.getHead().getFace());

输出结果：
com.sunny.demo.scrolldemo.ExampleUnitTest$Person@5e8c92f4
com.sunny.demo.scrolldemo.ExampleUnitTest$Person@61e4705b
true
false
com.sunny.demo.scrolldemo.ExampleUnitTest$Face@50134894
com.sunny.demo.scrolldemo.ExampleUnitTest$Face@50134894
```

从打印的结果来看，两个Head对象中的Face属性还是指向同一个对象，那么这就不是彻底的深拷贝，要做到彻底的深拷贝，还需要拷贝Head中的Face属性，如下代码：
```java
    public class Person implements Cloneable {
        private int age;
        private Head head;

        public Person() {

        }

        public Person(int age, Head head) {
            this.age = age;
            this.head = head;
        }

        public int getAge() {
            return age;
        }

        public Head getHead() {
            return head;
        }

        @Override
        protected Object clone() throws CloneNotSupportedException {
            Person person = (Person) super.clone();
            person.head = (Head) head.clone();
            return person;
        }
    }

    public class Head implements Cloneable{
        private Face face;

        public Head() {

        }

        public Head(Face face) {
            this.face = face;
        }

        public Face getFace() {
            return face;
        }

        @Override
        protected Object clone() throws CloneNotSupportedException {
            Head head = (Head) super.clone();
            head.face = (Face) face.clone();
            return head;
        }
    }

    private class Face implements Cloneable{


        @Override
        protected Object clone() throws CloneNotSupportedException {
            return super.clone();
        }
    }

Person person = new Person(25, new Head(new Face()));
Person personClone = (Person) person.clone();
System.out.println(person);
System.out.println(personClone);
System.out.println(person.getAge() == personClone.getAge());
System.out.println(person.getHead() == personClone.getHead());
System.out.println(person.getHead().getFace());
System.out.println(personClone.getHead().getFace());

输出结果：
com.sunny.demo.scrolldemo.ExampleUnitTest$Person@5e8c92f4
com.sunny.demo.scrolldemo.ExampleUnitTest$Person@61e4705b
true
false
com.sunny.demo.scrolldemo.ExampleUnitTest$Face@50134894
com.sunny.demo.scrolldemo.ExampleUnitTest$Face@2957fcb0
```

从打印结果来看的确复制了Face对象，到这儿为止，才实现了彻底的深拷贝，两个Person对象之间没有任何的交集。

**总结：**
- 浅拷贝是只拷贝当前对象，内部的属性如果是基本数据类型就会把值拷贝过来，如果是引用类型，把对象的引用拷贝过来，这样的结果就是复制过来的对象内部的引用类型的属性还是共用一个对象，它们是共享的，改了一个属性的值会影响到另一个对象内的属性；
- 深拷贝是出了拷贝当前对象之外，还会拷贝内部引用类型的属性对象，但是要做到彻底深拷贝是很难的，尤其是在类关系复杂以及引用第三方类的情况下，对于类关系复杂，实现彻底深拷贝会特别麻烦，每个相关的类都需要实现Cloneable接口，然后覆写clone方法，而对于第三方类来说，它可能没有实现Cloneable接口，那么这个类的对象就不能实现拷贝，即使上层的对象都实现拷贝了，到它之后还是共享的。
- 慎用Obeject的clone方法来拷贝对象