---
layout:     post
title:      "ArrayList实现原理"
date:       2017-09-09 12:00:00
author:     "ShiYu"
catalog: true
tags:
    - Java
---
##ArrayList实现原理

ArrayList,顾名思义，是底层用Array实现的List，查看源码：

```java
    /**
     * The array buffer into which the elements of the ArrayList are stored.
     * The capacity of the ArrayList is the length of this array buffer. Any
     * empty ArrayList with elementData == DEFAULTCAPACITY_EMPTY_ELEMENTDATA
     * will be expanded to DEFAULT_CAPACITY when the first element is added.
     */
    transient Object[] elementData; // non-private to simplify nested class access
```

ArrayList底层就是依赖于这个命名为elementData的数组，所有存入ArrayList的数据，都是存储在这个命名为emlemetData的数组中。

下面来看一下，当添加一个元素到ArrayList的一个实例时，将会发生什么：

```java
    /**
     * Appends the specified element to the end of this list.
     *
     * @param e element to be appended to this list
     * @return <tt>true</tt> (as specified by {@link Collection#add})
     */
    public boolean add(E e) {
    	//确保element的容量能够添加元素
        ensureCapacityInternal(size + 1);  // Increments modCount!!
        //向elementData数组中添加元素
        elementData[size++] = e;
        return true;
    }

```

当add一个元素到ArrayList中时，首先会检查elementData到容量是否满足，不满足则会对elementData进行扩容，检测并扩容到代码如下：

```java
    private void ensureCapacityInternal(int minCapacity) {
    		//如果element为空，即当前数组尚未初始化，则最小容量取默认容量和申请最小容量中较大的值
        if (elementData == DEFAULTCAPACITY_EMPTY_ELEMENTDATA) {
            minCapacity = Math.max(DEFAULT_CAPACITY, minCapacity);
        }

        ensureExplicitCapacity(minCapacity);
    }
    
    private void ensureExplicitCapacity(int minCapacity) {
        modCount++;

        // overflow-conscious code
        //如果需求的最小容量大于当前数组容量，则扩容
        if (minCapacity - elementData.length > 0)
            grow(minCapacity);
    }
    /**
     * Increases the capacity to ensure that it can hold at least the
     * number of elements specified by the minimum capacity argument.
     *
     * @param minCapacity the desired minimum capacity
     */
    private void grow(int minCapacity) {
        // overflow-conscious code
        int oldCapacity = elementData.length;
        //先令新容量等于旧容量1.5倍
        int newCapacity = oldCapacity + (oldCapacity >> 1);
        if (newCapacity - minCapacity < 0)//如果新容量依然小于需求容量，则让新容量直接等于需求的容量
            newCapacity = minCapacity;
        if (newCapacity - MAX_ARRAY_SIZE > 0)//如果新容量大于数组允许分配的最大容量，则尝试分配最大容量
            newCapacity = hugeCapacity(minCapacity);
        // minCapacity is usually close to size, so this is a win:
        elementData = Arrays.copyOf(elementData, newCapacity);
    }
    
    private static int hugeCapacity(int minCapacity) {//令新容量为最大容量
        if (minCapacity < 0) // overflow
            throw new OutOfMemoryError();
        return (minCapacity > MAX_ARRAY_SIZE) ?
            Integer.MAX_VALUE :
            MAX_ARRAY_SIZE;
    }    
```


上述已经介绍了像ArrayList尾部添加一个元素的具体过程，那么如果像ArryList中间添加一个元素会发生什么？

```java 
    /**
     * Inserts the specified element at the specified position in this
     * list. Shifts the element currently at that position (if any) and
     * any subsequent elements to the right (adds one to their indices).
     *
     * @param index index at which the specified element is to be inserted
     * @param element element to be inserted
     * @throws IndexOutOfBoundsException {@inheritDoc}
     */
    public void add(int index, E element) {
        rangeCheckForAdd(index);//首先检测指定索引是否越界

		//同像尾部添加元素一样，检测elemetData容量是否满足，不满足则扩容
        ensureCapacityInternal(size + 1);  // Increments modCount!!
        
        //以index为界，拷贝数组！！index之后所有元素全部重新分配！！
        System.arraycopy(elementData, index, elementData, index + 1,
                         size - index);
                         
        //将新元素插入指定位置
        elementData[index] = element;
        size++;
    }
```

上述代码可以看出，当向指定索引位置添加元素时，会将索引之后位置的的所有元素全部后移一位，以便在指定位置插入元素，可见使用ArryList在随机位置插入元素的开销是十分巨大的，所以当业务场景决定会频繁在List中间添加元素时，不要选用ArrayList。

以上部分介绍了向ArrayList中添加元素的实现原理，删除元素也是同样的道理，当删除指定位置元素时，会产生很大的开销，所以频繁增删的场景千万别使用ArrayList。

ArrayList是线程不安全的，从添加元素的代码可以看出，添加元素是分很多步骤添加的，并且没有做任何同步操作，不满足原子性，所以是线程不安全的。


