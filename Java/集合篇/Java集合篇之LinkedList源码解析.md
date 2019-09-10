![](https://www.lifeofpix.com/wp-content/uploads/2019/03/0C0A9388-2-1600x1067.jpg)

# 前言

今天我们通过分析LinkedList的源码，来学习一下它内部是如何添加、查询以及删除元素的。同时在阅读源码时，也要思考以下几个问题。

- LinkedList的底层数据结构是什么？
- 与ArrayList有什么区别？
- LinkedList是线程安全的吗？

# 继承关系

```
public class LinkedList<E>
    extends AbstractSequentialList<E>
    implements List<E>, Deque<E>, Cloneable, java.io.Serializable
{
  //......
}
```

首先我们来看一下LinkedList的继承关系，可以看出它继承自AbstractSequentialList。并且实现List、Deque、Cloneable以及Serializable接口，然后我们再来看一下它的核心字段。

# 核心字段

```
//表示LinkedList内部的元素数量
transient int size = 0;
//表示头结点
transient Node<E> first;
//表示尾节点
transient Node<E> last;
```

从源码中可以看出LinkedList中存在两个特殊的节点，分别是头结点和尾节点。那我们是不是就能猜到LinkedList底层结构是一个双向循环链表？到底是不是，我们慢慢分析。

# 构造方法

```
//创建了一个空列表
public LinkedList() {
}
//这个构造方法传入一个集合，然后将该集合的元素全部添加的链表中
public LinkedList(Collection<? extends E> c) {
    this();
    addAll(c);
}
```

可以看出构造方法比较简单，我们再来看一下Node类。

# Node

```
 private static class Node<E> {
        E item; //元素
        Node<E> next; //后节点
        Node<E> prev; //前节点

        Node(Node<E> prev, E element, Node<E> next) {
            this.item = element;
            this.next = next;
            this.prev = prev;
        }
    }
```

Node节点类是LinkedList中一个私有的静态内部类，包含元素、前节点和后节点。

# 添加

## 添加一个节点

```
 public boolean add(E e) {
        linkLast(e);
        return true;
    }
 void linkLast(E e) {
 		//l为尾节点
        final Node<E> l = last;
        //创建一个以l节点为前节点，数据为e，后节点为null的Node节点，此时newNode为尾节点
        final Node<E> newNode = new Node<>(l, e, null);
        //更新尾节点
        last = newNode;
        if (l == null)
            first = newNode;
        else
            l.next = newNode;
        size++;
        modCount++;
    }
```

从源码中可以看出LinkedList添加一个元素是从链表尾部进行添加，添加步骤如下：

- 将尾节点赋值l节点
- 创建一个以l节点为前节点，数据为e，后节点为null的Node节点
- 更新尾节点
- 判断l节点是否为空
- size++、modCount++

## 通过指定index添加

```
  public void add(int index, E element) {
  		//判断index是否越界
        checkPositionIndex(index);
        if (index == size)
        	//此时表示在链表尾部添加
            linkLast(element);
        else
            linkBefore(element, node(index));
    }
  void linkBefore(E e, Node<E> succ) {
        // succ表示index对应的节点，pred表示succ前节点
        final Node<E> pred = succ.prev;
        //newNode的前节点为pred，后节点为succ
        final Node<E> newNode = new Node<>(pred, e, succ);
        //succ的前节点为newNode
        succ.prev = newNode;
        //如果pred为null，说明succ原来是头结点，而现在succ的前节点为newNode，所以现在头结点是newNode
        if (pred == null)
            first = newNode;
        else
        	//pred的后节点为newNode
            pred.next = newNode;
        size++;
        modCount++;
    }

```

# 修改

首先我们来看通过指定索引来修改Node数据，源码如下

```
 public E set(int index, E element) {
 	//检查是否数组越界
    checkElementIndex(index);
    //通过node方法来获得对应得Node节点
    Node<E> x = node(index);
    //保存旧数据
    E oldVal = x.item;
    //赋值新数据
    x.item = element;
    //返回旧数据
    return oldVal;
    }
```

可以看出修改数据主要分为以下几步：

- 获得index对应的Node节点
- 保存Node节点旧数据
- 赋值新数据

# 获取

```
 public E get(int index) {
        checkElementIndex(index);
        return node(index).item;
    }
```

可以看出get()方法内部只有两行代码，我们分别来看一下都是做了什么操作。

## checkElementIndex(index)

```
  private void checkElementIndex(int index) {
        if (!isElementIndex(index))
            throw new IndexOutOfBoundsException(outOfBoundsMsg(index));
    }
  private boolean isElementIndex(int index) {
        return index >= 0 && index < size;
    }
```

可以看出checkElementIndex()方法主要是来判断index是否数组越界，如果越界就抛出对应的异常。

## node(index)

```
  Node<E> node(int index) {        
        if (index < (size >> 1)) {
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

可以看出node()方法通过判断index是处于前半段还是后半段，来查找对应的Node节点。通过折半查找提升了一定的效率。

#  删除

## 删除指定索引对应的节点

```
   public E remove(int index) {
        checkElementIndex(index);
        //传入index对应的Node节点
        return unlink(node(index));
    }
   E unlink(Node<E> x) {
        //获取Node节点数据
        final E element = x.item;
        //获取后节点
        final Node<E> next = x.next;
        //获取前节点
        final Node<E> prev = x.prev;
		//分别对前节点和后节点进行了判断
        if (prev == null) {
            first = next;
        } else {
            prev.next = next;
            x.prev = null;
        }
        if (next == null) {
            last = prev;
        } else {
            next.prev = prev;
            x.next = null;
        }
		//将节点数据置为null
        x.item = null;
        size--;
        modCount++;
        return element;
    }
```

然后我们再来简单看下LinkedList中其他remove()方法，具体如下：

```
 //删除第一个节点数据
 public E removeFirst() {
      final Node<E> f = first;
      if (f == null)
         throw new NoSuchElementException();
      return unlinkFirst(f);
    }
 //删除最后一个节点数据
 public E removeLast() {
      final Node<E> l = last;
      if (l == null)
         throw new NoSuchElementException();
      return unlinkLast(l);
    }
```

# 总结

通过分析LinkedList的源码，我们可以知道LinkedList在插入和删除上有着比较大的优势，这也符合链表的特性。而且通过判断index在前半段还是后半段，使用折半查找的方法来获得对应的Node节点，提升了一定的效率。