---
title: ArrayList浅析
date: 2017-07-02
categories: Java基础
author: Sunny
tags:
    - Java基础
    - ArrayList源码
cover_picture: https://upload-images.jianshu.io/upload_images/5231076-94c9e05f6757e5aa.jpg?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240
---

![](https://upload-images.jianshu.io/upload_images/5231076-94c9e05f6757e5aa.jpg?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

#### 一.前言
怎么说，ArrayList的源码还是相对简单的，数据结构也不是特别复杂，那我们直接上代码分析，先从一个简单的应用开始讲起...
注：本文的源码为JDK1.8

#### 二.ArrayList的初始化
ArrayList可以说是我们平常开发遇见的最多的一种集合类了，一般，我们都会这么使用：
```java
List<Person> persons = new ArrayList<>();//注释1
Person person = new Person();
persons.add(person);
...
```

注意到注释1了嘛，在这里我们会实例化一个ArrayList对象，在注释1处调用的是ArrayList的无参构造器，我们进入到ArrayList的源码中看看ArrayList究竟有几种构造器：

```java
public ArrayList(int initialCapacity)；

public ArrayList();

public ArrayList(Collection<? extends E> c)
```

一共有三种构造方法，那么很自然的每种构造方法的实现都是不一样的，我们在前面的例子中使用的无参构造方法，那我们就先以这个构造方法为例进行分析：

```java
public ArrayList() {
    super();
    this.elementData = EMPTY_ELEMENTDATA;
}
```

在这个无参构造方法中先是调用了父类的无参构造方法，进入到super()中后我们发现父类是一个抽象类AbstractList，并且这个抽象类实现了另一个抽象集合类AbstractCollection以及实现了List接口，这两个类在这里我们先不用过多关注，它们的存在无非就是对集合类操作的一种抽象以及提供了一些默认实现。好，我们再往下看这行代码`this.elementData = EMPTY_ELEMENTDATA`，看到这个全大写字母的名称之后，我们的第一直觉就是这可能是一个静态常量，ctrol+鼠标左键跳转到变量的声明处，我们发现果然是一个静态常量，并且它还是一个Object数组：
```java
private static final Object[] EMPTY_ELEMENTDATA = {};

transient Object[] elementData;
```

这是一个空数组，到这里，我们就大概知道了ArrayList的无参构造方法在初始化的时候做的操作：把一个静态空数组的引用赋值给elementData数组变量，我想这样做的目的是减少new数组的次数，总不能每次初始化ArrayList的时候内部存放对象的数组都是空引用吧，这对接下来的许多操作都会造成不便，那我们自然想到的是给这个elementData初始化一个数组，我们可以选择`this.elementData = new Object[]{}`或者直接给elementData初始化`this.elementData = {}`，但是这样做会有一定的性能问题，就是我们每次在`new ArrayList<>()`的时候都要初始化一次elementData数组，而且这个数组的大小还是0，在接下来的数组扩容过程中又得new一个数组赋值给elementData，很明显前面数组的初始化是没有必要的，那么ArrayList采用这样的策略来避免数组无用的初始化：使用在类加载的时候就初始化好的静态数组来给实例数组变量elementData赋值，因为EMPTY_ELEMENTDATA只需要初始化一次，以后每次new ArrayList的时候给elementData赋一个引用就行了（从这里可见ArrayList的作者是多么用心，mark,学习~）。

transient关键字在这里就不做过多阐述，主要是防止elementData数组参与到序列化中。

好，到这里，我们先简单总结一下：
**ArrayList的无参构造方法的作用就是实例化ArrayList对象，并给内部存放对象元素的数组赋一个初始的数组引用，这个初始的数组对象是静态常量，数组的长度为0。**

#### 三.ArrayList的add方法
我们再引用一下前面的示例代码：
```java
List<Person> persons = new ArrayList<>();
Person person = new Person();
persons.add(person);
...
```

初始化完ArrayList之后，我们就可以往ArrayList中添加元素了，比如上面代码这样，我们进入到add()方法的源码：

```java
public boolean add(E e) {
        ensureCapacityInternal(size + 1);  // Increments modCount!!
        elementData[size++] = e;
        return true;
    }
```

在这段源码中，size的值为0，因为ArrayList刚刚初始化，默认值为0，因为要添加一个元素到elementData数组中，首先我们应该确保elementData这个数组的大小是能够放下这个元素的，很明显是放不下的，它的初始数组的大小是0，我们进入`ensureCapacityInternal()`方法中：
```java
private void ensureCapacityInternal(int minCapacity) {
        //如果elementData和EMPTY_ELEMENTDATA的引用一样就走if中的语句
        if (elementData == EMPTY_ELEMENTDATA) {
            //取DEFAULT_CAPACITY和minCapacity大的为数组最小容量
            //minCapacity在这里值为1,而DEFAULT_CAPACITY值为10,自然取                     //DEFAULT_CAPACITY的值
            minCapacity = Math.max(DEFAULT_CAPACITY, minCapacity);
        }
        //这里的minCapacity的值为10
        ensureExplicitCapacity(minCapacity);
    }
```

具体的原理我写在上面的注释中了，我们继续往下看进入`ensureExplicitCapacity(minCapacity)`方法：

```java
private void ensureExplicitCapacity(int minCapacity) {
        //这个变量是用来记录ArrayList修改次数的
        //仔细看ArrayList中其他方法的源码都会发现这行代码
        modCount++;

        // overflow-conscious code
        //minCapacity为10，而elementData是个大小为0的空数组，自然if中的表达式
        //结果是成立的，grow()方法，走你...
        if (minCapacity - elementData.length > 0)
            grow(minCapacity);
    }
```

我们再进入grow()方法：

```java
private void grow(int minCapacity) {
        // overflow-conscious code
        //获取当前elementData数组的长度，很明显这里是0
        int oldCapacity = elementData.length;
        //自然这里newCapacity也是0了，oldCapacity>>1这个表达式的意思是对原数组
        //的长度除以2，为什么要这么写呢？效率高啊，哈哈哈...
        //在这里我们知道了ArrayList内部数组的扩容原来是这样的，就是原来数组长度
        //的1.5倍，这跟StringBuilder内部char数组的扩容策略还是不一样的(人家是
        //指数扩容)
        int newCapacity = oldCapacity + (oldCapacity >> 1);
        //如果扩容后的数组大小还是小于我实际需要的数组大小，那么就干脆使用我需要
        //的数组大小吧，咱目前的情况就属于这种情况:minCapacity为10，而newCapacity为0，那么到这里后，newCapacity的值就变为10了
        if (newCapacity - minCapacity < 0)
            newCapacity = minCapacity;
        //如果扩容后的数组大小比MAX_ARRAY_SIZE还要大，那么就可能有问题了，我们需要做额外的边界处理，这里是扩容后的数组大小比MAX_ARRAY_SIZE大，并不意味着我们实际需要的数组大小就比MAX_ARRAY_SIZE大，所以接下来我们还要继续判断我们需要的实际数组的大小是否比MAX_ARRAY_SIZE大
        if (newCapacity - MAX_ARRAY_SIZE > 0)
            newCapacity = hugeCapacity(minCapacity);
        // minCapacity is usually close to size, so this is a win:
        elementData = Arrays.copyOf(elementData, newCapacity);
    }
```

分析到`newCapacity = hugeCapacity(minCapacity)`了，我们需要进入hugeCapacity()方法看一下这个方法究竟是干嘛的：

```java
private static int hugeCapacity(int minCapacity) {
        //minCapacity小于0,越界,这种情况还是比较少见的吧
        if (minCapacity < 0) // overflow
            throw new OutOfMemoryError();
        //如果我们需要的实际数组大小比MAX_ARRAY_SIZE大，ArrayList最大只能提供到Integer的最大值，如果不是的话就把数组的大小设置成MAX_ARRAY_SIZE大小，当然，有这个结果的前提是扩容后的数组大小比MAX_ARRAY_SIZE大，因为扩容后的数组比MAX_ARRAY_SIZE大，意味着可能实际需要的数组大小比MAX_ARRAY_SIZE大，所以这里要进来再判断一下，即使实际需要的数组大小小于MAX_ARRAY_SZIE，但也差不了多少了，干脆直接设为MAX_ARRAY_SIZE，省的下次再扩容。
        return (minCapacity > MAX_ARRAY_SIZE) ?
            Integer.MAX_VALUE :
            MAX_ARRAY_SIZE;
    }
```

好，我们再回到上次的代码处`newCapacity = hugeCapacity(minCapacity)`,到这行代码后，就确定了最新数组的长度了，那么接下来最重要的一步来了：
```java
elementData = Arrays.copyOf(elementData, newCapacity);
```

这个方法是不是非常熟悉，没错，这是Arrays中拷贝数组的一个非常常见的静态方法，我们具体看看它的实现：
```java
public static <T> T[] copyOf(T[] original, int newLength) {
        //这里original.getClass()获取的是Object类对象
        return (T[]) copyOf(original, newLength, original.getClass());
    }
    
public static <T,U> T[] copyOf(U[] original, int newLength, Class<? extends T[]> newType) {
        T[] copy = ((Object)newType == (Object)Object[].class)
            //很明显上面比较的结果为true，因此这里重新new了一个长度为最新长度的Object数组
            ? (T[]) new Object[newLength]
            : (T[]) Array.newInstance(newType.getComponentType(), newLength);
        //然后把原数组中的元素拷贝到新数组中，在这里由于原来数组中就没有元素，这里也只是拷贝了空的元素到新数组中
        System.arraycopy(original, 0, copy, 0,
                         Math.min(original.length, newLength));
        //最后返回这个扩容后的新数组，这里也就是长度为10的数组
        return copy;
    }
```

最后再回到`elementData = Arrays.copyOf(elementData, newCapacity);`这行代码，那么ArrayList内部的elementData数组就指向了一个新的扩容后的数组了，再返回上层方法的调用处，最后来到了这里：
```java
public boolean add(E e) {
        ensureCapacityInternal(size + 1);  // Increments modCount!!
        elementData[size++] = e;
        return true;
    }
```
`ensureCapacityInternal(size + 1);`这行代码的所有逻辑就执行完了，我们得到了一个长度为10的elementData数组，但是size的值还是0,代码继续向下执行，`elementData[size++] = e;`，把添加进来的元素引用赋值给elementData[size]，也就是elementData[0]，然后size的值再加1，表示内部数组已经有一个元素了，这里需要注意size的值和elementData数组长度的区别，elementData数组的长度在这里是10，而size的值为1，也就是size的值表示的是elementData数组中实际有的元素个数。到这儿，我们就知道了原来我们每次在使用ArrayList的size()方法的时候取得就是这个size值：
```java
public int size() {
        return size;
    }
```

好，我们再来小小总结一下：
- **ArrayList内部数组的扩容策略是：如果需要扩容，每次扩容的大小为原来数组长度的1.5倍，注意和StringBuilder内部数组的扩容策略相比较；**
- **size()方法返回的是数组实际拥有的元素个数，和内部数组的长度没有直接关系。**

前面只分析了ArrayList的无参构造方法，下面我们再来看看另外两个构造方法，先看int类型的有参构造方法：
```java
public ArrayList(int initialCapacity) {
        super();
        if (initialCapacity < 0)
            throw new IllegalArgumentException("Illegal Capacity: "+
                                               initialCapacity);
        this.elementData = new Object[initialCapacity];
    }
```

从上面的代码中我们可以发现给内部elementData数组重新new了一个固定长度的数组对象，从这里我们也可以知道如果我们事先知道List中会存放多少个元素就尽量使用这个构造方法，减少重新分配内存的次数，提高效率，效率的提升是在哪儿呢？比如这样：
```java
//代码段1
long startTime1 = System.currentTimeMillis();
List<Integer> intList1 = new ArrayList<>();
for (int i = 0; i < 100000000; i++) {
    intList1.add(i);
}
System.out.println(System.currentTimeMillis() - startTime1);

//代码段2
long startTime2 = System.currentTimeMillis();
List<Integer> intList2 = new ArrayList<>(10000);
for (int i = 0; i < 100000000; i++) {
    intList2.add(i);
}
System.out.println(System.currentTimeMillis() - startTime2);
```
上面代码段1和代码段2的作用都是相同的，没什么区别，但是代码段1的执行效率要比代码段2低，为什么？原因如下：
- 首先代码段1在初始化ArrayList的时候没有指定内部存放元素的数组长度，因此在for循环中第一次add元素的时候，内部对数组进行了一次扩容，也就是重新new了一个长度为10的新数组,当add到第11个元素的时候，又需要进行一次扩容，这次数组的大小为上次的1.5倍，也就是15，又重新new了一个长度为15的新数组，当add到第16个元素的时候，又需要扩容...就这样ArrayList内部需要进行很多次重新new数组，拷贝数组的过程，这些过程是非常影响性能。
- 代码段2在ArrayList初始化的时候就指定了内部数组的大小，这样只要add的元素个数在100以内，ArrayList内部的数组都不会进行扩容操作，相交于代码段1处的性能会有很大的提升。

我们把上面代码段1和代码段2执行一下，最后结果为：
```java
48316
38853
```
可见，性能差异还是非常大的，因此，在平常的开发中，我们建议使用带默认长度的构造方法初始化ArrayList。

我们接下来再看看另一个有参构造方法：
```java
public ArrayList(Collection<? extends E> c) {
        elementData = c.toArray();
        size = elementData.length;
        // c.toArray might (incorrectly) not return Object[] (see 6260652)
        if (elementData.getClass() != Object[].class)
            elementData = Arrays.copyOf(elementData, size, Object[].class);
    }
```

从上面的构造方法内部实现来看，把外部传入的Collection集合对象转成数组后赋值给elementData,如果elementData的类对象不是Object[]类对象，那么ArrayList会重新生成一个Object[]数组并赋值给elementData。

到这里，ArrayList的初始化以及add操作就分析完了，其实add还有很多其他方法，比如：
```java
public void add(int index, E element);

public boolean addAll(Collection<? extends E> c);

public boolean addAll(int index, Collection<? extends E> c);
```

内部实现原理和add差不多，这里就不做阐述了。

#### 四.ArrayList的remove方法
接下来我们再分析常用的remove操作，直接上代码：
```java
public E remove(int index) {
        //如果index大于等于size的值则说明索引到了数组没值得区域，这里抛出了索引越界的异常
        if (index >= size)
            throw new IndexOutOfBoundsException(outOfBoundsMsg(index));

        //记录ArrayList修改的次数
        modCount++;
        //取出elementData中对应索引的元素
        E oldValue = (E) elementData[index];

        //numMoved是指排在索引后面的元素的个数
        int numMoved = size - index - 1;
        if (numMoved > 0)
            //如果这个值大于0，就把索引后面的元素拷贝到以索引为开始的位置，这样索引位置的元素就被覆盖了，就相当于被移除了
            System.arraycopy(elementData, index+1, elementData, index,
                             numMoved);
        //移除了一个元素之后，最后一个元素和倒数第二个元素是重复的，需要把最后一个元素置空以便GC回收
        elementData[--size] = null; // clear to let GC do its work

        //最后把移除的这个元素的引用返回
        return oldValue;
    }
```

上面是分析了remove(int index)方法，在ArrayList中还有一个有关remove操作的实现也非常重要，就是removeAll(Collection<?> c)方法，我们来看代码：
```java
public boolean removeAll(Collection<?> c) {
        return batchRemove(c, false);
    }
    
private boolean batchRemove(Collection<?> c, boolean complement) {
        final Object[] elementData = this.elementData;
        int r = 0, w = 0;
        boolean modified = false;
        try {
            for (; r < size; r++)
                if (c.contains(elementData[r]) == complement)
                    elementData[w++] = elementData[r];
        } finally {
            // Preserve behavioral compatibility with AbstractCollection,
            // even if c.contains() throws.
            if (r != size) {
                System.arraycopy(elementData, r,
                                 elementData, w,
                                 size - r);
                w += size - r;
            }
            if (w != size) {
                // clear to let GC do its work
                for (int i = w; i < size; i++)
                    elementData[i] = null;
                modCount += size - w;
                size = w;
                modified = true;
            }
        }
        return modified;
    }
```

这里我们要说一下batchRemove()方法，这个方法主要用于两个地方，当complement为false的时候用于removeAll()方法中，当complement为true则用于retainAll()方法中，retainAll()方法我们接下来再详细分析，这里只需关注complement为false的情况。下面我们来具体分析这种情况，主要看batchRemove()方法：
```java
private boolean batchRemove(Collection<?> c, boolean complement) {
        //把原数组的引用赋值给方法体内的elementData变量，这里只考虑complement为false的情况
        final Object[] elementData = this.elementData;
        //r和w的值都为0，r的值用来循环遍历elementData数组内的元素，w用来标识不在传进来的集合c中元素个数，简单来说就是elementData数组中有，c集合中没有的元素，因为这部分元素是要保留的，c中的元素是要被移除的
        int r = 0, w = 0;
        //初始化modified变量，用来标识是否移除成功
        boolean modified = false;
        try {
            for (; r < size; r++)
                //这段代码在complement为false时的作用是把数组elementData中有，c集合中没有的元素重新从elementData[0]处开始赋值，并用w标记这样的元素共有多少个
                if (c.contains(elementData[r]) == complement)
                    elementData[w++] = elementData[r];
        } finally {
            // Preserve behavioral compatibility with AbstractCollection,
            // even if c.contains() throws.
            //正常情况来讲，r的值是等于size的值得，如果出现两者不相等的情况，则说明上面try块中出现了异常，那么这段代码的作用就是把r后面的元素重新赋值到w后面
            if (r != size) {
                System.arraycopy(elementData, r,
                                 elementData, w,
                                 size - r);
                w += size - r;
            }
            if (w != size) {
                // clear to let GC do its work
                //这段代码就是把w后面的元素全部清空，因为经过上面代码的筛选之后，w后面的元素就是c集合中拥有的元素，是需要被移除的
                for (int i = w; i < size; i++)
                    elementData[i] = null;
                modCount += size - w;
                //剩下的元素的个数就是w个
                size = w;
                //移除成功
                modified = true;
            }
        }
        return modified;
    }
```

接下来，我们再分析一下这个remove(Object o)方法，这个方法是用来移除某个指定元素的：
```java
public boolean remove(Object o) {
        if (o == null) {
            for (int index = 0; index < size; index++)
                if (elementData[index] == null) {
                    fastRemove(index);
                    return true;
                }
        } else {
            for (int index = 0; index < size; index++)
                if (o.equals(elementData[index])) {
                    fastRemove(index);
                    return true;
                }
        }
        return false;
    }
    
private void fastRemove(int index) {
        modCount++;
        int numMoved = size - index - 1;
        if (numMoved > 0)
            System.arraycopy(elementData, index+1, elementData, index,
                             numMoved);
        elementData[--size] = null; // clear to let GC do its work
    }
```
remove(Object o)中的方法逻辑还是非常好理解的，由于ArrayList中存放元素的是一个数组，因此，ArrayList中的元素能够重复并且能够为空，当出现多个重复元素的时候，我们每次只能移除第一个元素，注意到`o.equals(elementData[index]))`方法了嘛，这里比较的结果完全取决于对象的equals方法。

当ArrayList中出现多个null值，remove(Object o)方法肯定移除不了所有的null值，那么我们可以使用removeAll(Collection)这个方法，由remove(Collection)方法的源码我们知道，它能移除一个ArrayList中的所有null值，我们看下面的Demo：
```java
        List<Integer> integerList = new ArrayList<>();
        List<Integer> nullList = new ArrayList<>();
        nullList.add(null);
        integerList.add(null);
        integerList.add(null);
        integerList.add(null);
        integerList.add(1);
        integerList.removeAll(nullList);
        for (Integer integer : integerList) {
            System.out.println(integer);
        }

输出结果为：
1
```
从结果可以看出，确实达到了我们想要的结果。

#### 五.ArrayList的clear方法
同样的，我们直接上代码：
```java
public void clear() {
        modCount++;

        // clear to let GC do its work
        for (int i = 0; i < size; i++)
            elementData[i] = null;

        size = 0;
    }
```

从上面的实现来看，ArrayList的clear操作是通过循环遍历数组内的元素，并逐个置空来实现的，其他也没有什么好讲的。

#### 六.ArrayList的retainAll方法
我们来看看ArrayList中的retainAll()方法，这个方法中调用的是和removeAll一样的方法，也就是batchRemove(Collection,boolean):
```java
public boolean retainAll(Collection<?> c) {
        return batchRemove(c, true);
    }

private boolean batchRemove(Collection<?> c, boolean complement) {
        final Object[] elementData = this.elementData;
        int r = 0, w = 0;
        boolean modified = false;
        try {
            for (; r < size; r++)
                //当complement为true的时候，只要elementData中的元素在形参集合c中存在，就会被保存下来，c中没有的，则会被移除，换句话说，这个方法最终的结果是两个集合的交集
                if (c.contains(elementData[r]) == complement)
                    elementData[w++] = elementData[r];
        } finally {
            // Preserve behavioral compatibility with AbstractCollection,
            // even if c.contains() throws.
            if (r != size) {
                System.arraycopy(elementData, r,
                                 elementData, w,
                                 size - r);
                w += size - r;
            }
            if (w != size) {
                // clear to let GC do its work
                for (int i = w; i < size; i++)
                    elementData[i] = null;
                modCount += size - w;
                size = w;
                modified = true;
            }
        }
        return modified;
    }
```
从源码中也不难看出，这个方法的作用就是取两个集合的交集，这个交集的结果最终存放在ArrayList中，返回的boolean值代表取交集的操作是否成功，什么情况下回返回false呢?比如这样：
```java
        List<Integer> integerList1 = new ArrayList<>();
        List<Integer> integerList2 = new ArrayList<>();
        integerList2.add(1);
        integerList2.add(2);
        integerList2.add(3);
        integerList2.add(1);

        integerList1.add(2);
        integerList1.add(3);
        integerList1.add(1);
        integerList1.add(1);
        System.out.println(integerList1.retainAll(integerList2));
        for (Integer integer : integerList1) {
            System.out.println(integer);
        }
        
输出结果：
false
2
3
1
1
```
当两个ArrayList中包含的元素完全一样的时候就会返回false，而且这跟它们的顺序无关

#### 七.ArrayList的contains方法
contains(Object o)的作用是用来判断ArrayList中是否包含有指定元素，源码如下：
```java
public boolean contains(Object o) {
        return indexOf(o) >= 0;
}

public int indexOf(Object o) {
        if (o == null) {
            for (int i = 0; i < size; i++)
                if (elementData[i]==null)
                    return i;
        } else {
            for (int i = 0; i < size; i++)
                //注意这里！！使用的equals方法来比较对象
                if (o.equals(elementData[i]))
                    return i;
        }
        return -1;
}
```

从源码中我们可以知道，对于元素对象的比较使用的equals方法，那么euqals的结果就取决于对象的equals实现了，我们可以通过覆写euqals方法来实现我们想要业务逻辑。