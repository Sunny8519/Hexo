---
title: String类和StringBuilder类简单分析
date: 2017-06-25
categories: Java基础
author: Sunny
tags:
    - Java基础
    - String类
    - StringBuilder类
cover_picture: https://upload-images.jianshu.io/upload_images/5231076-427ae3543d494163.jpg?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240
---

![](https://upload-images.jianshu.io/upload_images/5231076-427ae3543d494163.jpg?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

### 一.String类简单分析
#### 1.常用方法
```java
//支持+和+=运算符
String name = "Sunny";
String des = ",今天去哪儿";
name += ",你好";
System.out.println(name + des);//输出：Sunny,你好,今天去哪儿

/**判断字符串是否为空
* 这个方法只能判断字符串为空字符串的情况，也就是字符串的length为0,
* 和TextUtils.isEmpty()方法差别在于，TextUtils.isEmpty()方法能够
* 判断字符串对象是否为null
*/
public boolean isEmpty();

//获取字符串长度
public int length();

//截取字符串，返回的是一个新的字符串对象
public String substring(int beginIndex);
public String substring(int beginIndex,int endIndex);

//在字符串中检索指定字符串的位置,返回第一次出现的位置索引，如果没有返回-1
public int indexOf(String str);
//从指定位置开始查找，返回第一次出现的位置索引，如果没有返回-1
public int indexOf(String str,int fromIndex);

//判断字符串中是否包含有指定字符串或字符
public boolean contains(CharSequence s);

//判断字符串是否以指定字符串开头
public boolean startsWith(String prefix);

//判断字符串是否以指定字符串结尾
public boolean endsWith(String suffix)

//比较两个字符串是否相同
public boolean equals(Object anObject);
//忽略大小写比较两个字符串是否相同
public boolean equalsIgnoreCase(String anotherString);

//比较两个字符串的大小
public native int compareTo(String anotherString);
//忽略大小写比较两个字符串的大小
public int compareToIngnoreCase(String str);

//所有字符串转为大写字符,返回新字符串，原字符串不变
public String toUpperCase();
//所有字符串转为小写字符,返回新字符串，原字符串不变
public String toLowerCase();

//字符串连接,返回合并后的字符串，原来的字符串不变
public String concat(String str);

//替换单个字符，返回新的字符串，原字符串不变
public String replace(char oldChar,char newChar);
//替换字符序列，返回新的字符串，原字符串不变
public String replace(CharSequence target,CharSequence replacement);
//替换第一个跟正则表达式匹配的子字符串，正则表达式可以是指定的字符串
public String replaceFirst(String regex, String replacement);
//替换所有跟正则表达式匹配的子字符串，正则表达式可以是指定的字符串
public String replaceAll(String regex, String replacement);

//删除开头和结尾的空格，返回新的字符串，原字符串不变
public String trim();

//分隔字符串,返回分隔后的字符串数组，原字符串不变
public String[] split(String regex);

//返回指定索引的char
public native char charAt(int index);
//返回字符串对应的字符数组
public native char[] toCharArray();

//将当前字符串中指定范围的字符插入到目标字符数组的指定位置
public void getChars(int srcBegin,int srcEnd,char dst[],int dstBegin);
```

#### 2.编码转化
在Java中使用Charset类表示各种编码，String内部是按照UTF-16BE编码来处理字符的，但String类中也提供了很多方法让字符串按照指定的编码来表示。

#### 3.Java中的正则表达式
Java中有专门的类如Pattern和Matcher用于正则表达式。


### 二.StringBuilder原理简单分析
#### 1.StringBuilder和StringBuffer的区别
这两者最大的区别是StringBuilder是线程不安全的，而StringBuffer是线程安全的。

#### 2.StringBuilder的append原理分析
StringBuilder继承自抽象类AbstractStringBuilder，初始化时都会走AbstractStringBuilder中的构造方法：
```java
public final class StringBuilder extends AbstractStringBuilder implements java.io.Serializable, CharSequence{
       
    public StringBuilder() {
        super(16);
    }

    public StringBuilder(int capacity) {
        super(capacity);
    }
    
    public StringBuilder(String str) {
        super(str.length() + 16);
        append(str);
    }
    
    public StringBuilder(CharSequence seq) {
        this(seq.length() + 16);
        append(seq);
    }
    ...
}
```

那我们再来看看AbstractStringBuilder中的构造方法：
```java
abstract class AbstractStringBuilder implements Appendable, CharSequence {
    char[] value;
    int count;

    AbstractStringBuilder() {}
    
    AbstractStringBuilder(int capacity) {
        value = new char[capacity];
    }
    ...
}
```

通过上面的代码我们知道利用无参构造方法new一个StringBuilder对象的时候：
```java
StringBuilder text = new StringBuilder();
```
实际存储字符的数组默认大小为16，当然了，我们也可以利用其他构造方法来新建一个对象，接下来我们再来说一个比较典型的构造方法：
```java
public StringBuilder(String str) {
      super(str.length() + 16);
      append(str);
}
```
举个栗子，我们的代码是这样写的：
```java
public static void main(String[] args){
     StringBuilder text = new StringBuilder("sunny");
     text.append(",nihao");
}
```
从上面的代码中，我们知道`text`对象在new的过程中应该走的是带String类型参数的构造方法，进而再走到父类AbstractStringBuilder类中的`AbstractStringBuilder(int capacity)`构造方法，此时传给父类构造器的参数是`str.length()+16`大小的一个整数，在这个栗子中，整数capacity的值就是21，然后为`value`这个变量new了一个大小为21的char类型数组，`count`变量的值则是默认值0，这样父类AbstractStringBuilder的构造方法就走完了，然后继续走子类StringBuilder的构造方法，也就是`append(str)`这句，我们看看append方法的具体代码：
```java
    @Override
    public StringBuilder append(String str) {
        super.append(str);
        return this;
    }
```
在该方法中我们可以看到实际又调到了父类AbstractStringBuilder类中的append方法，然后把当前对象返回，方便继续操作。我们进入父类AbstractStringBuilder的append方法：
```java
    public AbstractStringBuilder append(String str) {
        if (str == null)
            return appendNull();//如果是空，直接在已存在char数组的基础上加上"null"字符串
        //如果不是空，则获取字符串的长度len
        int len = str.length();
        //根据实际存储字符长度(这里的意思是value字符数组可能没有填满,count表示的是字符数组已经被使用的长度)和追加字符的长度对value字符数组进行扩容，最后的结果就是value数组的大小被扩大了，但是里面的内容没有变化
        ensureCapacityInternal(count + len);
        //把追加的字符串整个copy到value数组中，注意是copy到count以后的位置，不然也不叫追加了
        str.getChars(0, len, value, count);
        //增加实际存储字符的大小
        count += len;
        return this;
    }
```

为什么在字符串频繁修改的情境下建议使用StringBuilder呢？最重要的一个原因就在于内存的分配次数上，**String在每次修改字符串的时候内部都会重新new一个char数组来存放最新更改的字符串，而StringBuilder只有在需要扩容的时候才会重新new一个char数组，而且StringBuilder内对char数组长度的扩展采用的是指数扩展策略，能够有效的减小内存分配的次数**，我们来看一看StringBuilder如何进行的指数扩展：

```java
void ensureCapacityInternal(int minimumCapacity) {
        // overflow-conscious code
        //如果数组已被使用的长度加上追加的字符串的长度大于当前数组的最大长度就需要扩容
        if (minimumCapacity - value.length > 0)
            expandCapacity(minimumCapacity);
    }
    
void expandCapacity(int minimumCapacity) {
        //新数组的长度是在原来数组长度的基础上乘以2再加2
        int newCapacity = value.length * 2 + 2;
        //如果发现新数组的长度还是小于原来字符串加上追加进来的字符串总长度，就让新数组的长度等于原字符串和追加的字符串总长度
        if (newCapacity - minimumCapacity < 0)
            newCapacity = minimumCapacity;
        //如果发现新数组的长度小于0了，就可以断定新数组的长度已经大于int的最大值，因为超过int的最大值就变成负值
        if (newCapacity < 0) {
            //这个时候再判断原字符串和追加的字符串总长度是否小于0，如果小于0，则说明追加的字符串导致字符串占用的内存大小超过了int的最大值，这是一个内存溢出的异常
            if (minimumCapacity < 0) // overflow
                throw new OutOfMemoryError();
            //如果原字符串和追加的字符串总长度大于0，那么就把新数组的长度设置成int的最大值
            newCapacity = Integer.MAX_VALUE;
        }
        //重新分配一个newCapacity长度的字符数组，并把原来数组中的值copy到新数组中
        value = Arrays.copyOf(value, newCapacity);
    }
```
走完上面两个方法说明StringBuilder内的数组扩容已经完成了，接下来就需要把传进来的字符串添加到新数组就行了，也就是`str.getChars(0, len, value, count);`这一步。到这里，一次完整的append就完成了。

当然了，虽然StringBuilder中为我们提供了指数扩容的策略来减少内存分配的次数，但如果我们事先知道这个字符串的长度，可以使用StringBuilder的另一个构造方法直接设置内部存放字符的数组长度：`public StringBuilder(int capacity)`

StringBuilder中append()方法有多种重载形式，内部实现原理基本差不多，这里不再赘述。

#### 3.StringBuilder的insert原理分析
```java
public StringBuilde insert(int offset,String str);
```
该方法的作用就是在指定索引位置offset插入指定的字符串str，比如下面这样：
```java
StringBuilder sb = new StringBuilder();
sb.append("Sunny");
sb.insert(5,"，hello");

打印结果：Sunny,hello
```

我们来看一看insert的源码：
```java
//StringBuilder类
public StringBuilder insert(int offset, String str) {
        super.insert(offset, str);
        return this;
    }

//AbstractStringBuilder类
public AbstractStringBuilder insert(int offset, String str) {
        if ((offset < 0) || (offset > length()))
            throw new StringIndexOutOfBoundsException(offset);
        if (str == null)
            str = "null";
        int len = str.length();
        //和append相同先判断是否需要扩容
        ensureCapacityInternal(count + len);
        //调用System.arraycopy方法将内部字符数组指定索引位置往后的字符移到插入的字符后面，中间空出来的地方留给插入的字符使用
        System.arraycopy(value, offset, value, offset + len, count - offset);
        //把插入的字符串copy到内部字符数组offset的位置，也就是中间空出来的空间
        str.getCharsNoCheck(0, len, value, offset);
        //字符数组已使用的空间加上插入的字符串长度
        count += len;
        return this;
    }
```
这里要说明一下`System.arraycopy()`这个方法,它可以在同一个数组中对元素进行移位操作。

当然这里insert方法也有多种重载形式，实现原理也比较好理解，这里不再赘述。

#### 4.StringBuilder的delete原理
```java
//不包括end位置的字符
public StringBuilder delete(int start,int end);
//删除指定位置的字符
public StringBuilder deleteCharAt(int index);
```

内部实现原理：
```java
public StringBuilder delete(int start, int end) {
        super.delete(start, end);
        return this;
    }
    
public AbstractStringBuilder delete(int start, int end) {
        if (start < 0)
            throw new StringIndexOutOfBoundsException(start);
        if (end > count)
            end = count;
        if (start > end)
            throw new StringIndexOutOfBoundsException();
        int len = end - start;
        if (len > 0) {
            //把end到数组结尾的字符copy到start位置，那么中间从start到end的字符就没有了
            System.arraycopy(value, start+len, value, start, count-end);
            //已使用的字符数组长度要减去删除的字符长度
            count -= len;
        }
        return this;
    }
    

public StringBuilder deleteCharAt(int index) {
        super.deleteCharAt(index);
        return this;
    }
    
public AbstractStringBuilder deleteCharAt(int index) {
        if ((index < 0) || (index >= count))
            throw new StringIndexOutOfBoundsException(index);
        //把要删除的字符后面的字符copy到要删除的字符的位置，这样指定位置的字符就被覆盖删除了
        System.arraycopy(value, index+1, value, index, count-index-1);
        count--;
        return this;
    }
```
从上面的源码来看，delete()方法的实现也是非常简单，不再做过多描述。

#### 5.StringBuilder的replace原理
```java
//替换指定范围内的字符串，如果end-start大于str的字符串长度，end-start范围内的字符都会被str替代
public StringBuilder replace(int start,int end,String str);
//替换指定位置的字符
public void setCharAt(int index,char ch);
```

内部实现原理：
```java
public StringBuilder replace(int start, int end, String str) {
        super.replace(start, end, str);
        return this;
    }
    
public AbstractStringBuilder replace(int start, int end, String str) {
        if (start < 0)
            throw new StringIndexOutOfBoundsException(start);
        if (start > count)
            throw new StringIndexOutOfBoundsException("start > length()");
        if (start > end)
            throw new StringIndexOutOfBoundsException("start > end");

        if (end > count)
            end = count;
        int len = str.length();
        int newCount = count + len - (end - start);
        ensureCapacityInternal(newCount);
        
        //把end位置往后的所有字符copy到start+len的位置，中间(start,start+len)留给str使用
        System.arraycopy(value, end, value, start + len, count - end);
        //把str copy到value数组中
        str.getCharsNoCheck(0, len, value, start);
        //更新已使用的字符个数
        count = newCount;
        return this;
    }
    
//AbstractStringBuilder
public void setCharAt(int index, char ch) {
        if ((index < 0) || (index >= count))
            throw new StringIndexOutOfBoundsException(index);
        value[index] = ch;
    }
```

#### 6.StringBuilder的reverse()方法
```java
public StringBuilder reverse()；
```
reverse()方法对于增补字符的翻转依然有效，有时间可研究字符翻转方面的算法实现。

#### 7.其余常用方法
```java
//返回字符数组的长度
public int capacity();

//返回字符数组实际使用的长度
public int length();

//直接修改实际使用的长度,如果newLength的大小大于count，那么中间位置的字符用'\0'表示
public void setLength(int newLength);

//缩小StringBuilder内部数组占用的内存空间
public void trimToSize() {
        if (count < value.length) {
            //内部重新new一个字符数组，长度为count，并把value数组中的内容copy到新数组中
            value = Arrays.copyOf(value, count);
        }
    }
```

#### 8.与String类似的方法
1.查询指定字符串
```java
public int indexOf(String str)
public int indexOf(String str, int fromIndex)
public int lastIndexOf(String str)
public int lastIndexOf(String str, int fromIndex)
```

2.取子字符串
```java
public String substring(int start)
public String substring(int start, int end)
public CharSequence subSequence(int start, int end)
```

3.获取其中的字符或Code Point
```java
public char charAt(int index)
public int codePointAt(int index)
public int codePointBefore(int index)
public int codePointCount(int beginIndex, int endIndex)
public void getChars(int srcBegin, int srcEnd, char[] dst, int dstBegin)
```

#### 9.String的+和+=背后原理
在Java中，我们都知道对于String能够使用+和+=，其实背后的原理和包装类的自动装箱/拆箱很相似，都是编译器提供的功能，当我们写下下面这段代码：
```java
String name = "Sunny";
name += ",hello";
```
经过编译器变异过后，代码会变成如下这样：
```java
String name = "Sunny"
StringBuilder sb = new StringBuilder(name);
sb.append(",hello");
```

那么这时候就会有同学问了，既然有了+和+=,干嘛还要使用StringBuilder呢，直接都让编译器隐式的去翻译成StringBuilder不就行了嘛，我们来看个例子：
```java
String name = "Sunny";
for(int i = 0;i<1000;i++){
   name += ",hello";
}
```
这样写会有什么样的性能问题？我们来把它翻译成StringBuilder的形式：
```java
String name = "Sunny";
for(int i = 0;i<1000;i++){
   StringBuilder sb = new StringBuilder(name);
   sb.append(",hello");
   name = sb.toString();
}
```
很明显了，这里每个循环都需要new一个StringBuilder对象，循环次数多了会造成内存抖动，所以这种情况下应该在循环外面使用一个StringBuilder，在循环内部append就行了。

### 三.参考
[剖析String](http://mp.weixin.qq.com/s/4FhhAp8CJobMzXBuWhB1zg)
[剖析StringBuilder](http://mp.weixin.qq.com/s/RU7nzqP3L7Zud5O26synTg)