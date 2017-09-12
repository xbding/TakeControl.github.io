---
layout:     post
title:      "LinkedList实现原理"
date:       2017-09-12 12:00:00
author:     "ShiYu"
catalog: true
tags:
    - Java
---
## LinkedList实现原理

LinkedList底层采用链表结构,LinkedList定义了一个静态内部类Node：

```java
   private static class Node<E> {
        E item;
        Node<E> next;
        Node<E> prev;

        Node(Node<E> prev, E element, Node<E> next) {
            this.item = element;
            this.next = next;
            this.prev = prev;
        }
    }
```

LinkedList内部定义了3个成员变量：

```java
	//list的大小
    transient int size = 0;

    /**
     * Pointer to first node.
     * Invariant: (first == null && last == null) ||
     *            (first.prev == null && first.item != null)
     */
    transient Node<E> first;//头节点

    /**
     * Pointer to last node.
     * Invariant: (first == null && last == null) ||
     *            (last.next == null && last.item != null)
     */
    transient Node<E> last;//尾节点

```

下面我们看一下LinkedList的add方法：

```java
    /**
     * Appends the specified element to the end of this list.
     *
     * <p>This method is equivalent to {@link #addLast}.
     *
     * @param e element to be appended to this list
     * @return {@code true} (as specified by {@link Collection#add})
     */
    public boolean add(E e) {
        linkLast(e);//添加元素到尾部
        return true;
    }
    
     /**
     * Links e as last element.
     */
    void linkLast(E e) {
        final Node<E> l = last;
        final Node<E> newNode = new Node<>(l, e, null);//实例化一个节点
        last = newNode;
        if (l == null)//如果尾部节点是null,说明LinkedList是空的，就将节点赋给头节点,否则将当前尾部节点的next指向新增节点
            first = newNode;
        else
            l.next = newNode;
        size++;
        modCount++;
    }
```

下面看一下向指定位置添加元素会发生什么：
```java
    /**
     * Inserts the specified element at the specified position in this list.
     * Shifts the element currently at that position (if any) and any
     * subsequent elements to the right (adds one to their indices).
     *
     * @param index index at which the specified element is to be inserted
     * @param element element to be inserted
     * @throws IndexOutOfBoundsException {@inheritDoc}
     */
    public void add(int index, E element) {
        checkPositionIndex(index);//检测数组是否越界

        if (index == size)//是否插入到尾部
            linkLast(element);
        else
            linkBefore(element, node(index));
    }
    
    
    /**
     * Inserts element e before non-null Node succ.
     */
    void linkBefore(E e, Node<E> succ) {
        // assert succ != null;
        final Node<E> pred = succ.prev;
        final Node<E> newNode = new Node<>(pred, e, succ);
        succ.prev = newNode;
        if (pred == null)
            first = newNode;
        else
            pred.next = newNode;
        size++;
        modCount++;
    }
```

上述代码可以看出，向指定位置插入节点的开销除了是固定的，不像ArrayList那样会产生很大的开销，所以有频繁删改元素的业务场景应该选用LinkedList。

LinkedList对于增删节点的效率是比较高的，那么随机访问呢，我们看以下代码：
```java

    /**
     * Returns the element at the specified position in this list.
     *
     * @param index index of the element to return
     * @return the element at the specified position in this list
     * @throws IndexOutOfBoundsException {@inheritDoc}
     */
    public E get(int index) {
        checkElementIndex(index);//检测索引是否越界
        return node(index).item;
    }
    
    /**
     * Returns the (non-null) Node at the specified element index.
     */
    Node<E> node(int index) {
        // assert isElementIndex(index);

        if (index < (size >> 1)) {//如果索引小于list大小的一半，则顺序遍历节点找到对应索引的节点，否则反向遍历找到索引对应的节点
            Node<E> x = first;
            for (int i = 0; i < index; i++)
                x = x.next;
            return x;
        } else {
            Node<E> x = last;
            for (int i = size - 1; i > index; i--)
                x = x.prev;
            return x;
        }
    }
```

上述代码可以看出LinkedList查找节点的代价是线性增长的，远不如数组的随机访问速度快，所以对于有频繁随机访问的业务场景，应该选用ArrayList而不是LinkedList。

注意LinkedList同样没有保证操作的原子性，所以它也是线程不安全的。