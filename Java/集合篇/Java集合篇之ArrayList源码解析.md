![](https://www.lifeofpix.com/wp-content/uploads/2019/03/dsc-2037-1-1600x1068.jpg)

# 前言

今天我们来分析一下ArrayList的源码实现，还是带着以下几个问题来分析源码。

- ArrayList底层实现？
- ArrayList的扩容机制？
- ArrayList是线程安全吗？如果不是，那么有线程安全的吗？

# 继承关系

```
public class ArrayList<E> extends AbstractList<E>
        implements List<E>, RandomAccess, Cloneable, java.io.Serializable{
        //......
  }
```

可以看出ArrayList继承了AbstractList，并且实现了List、RandomAccess等接口，以及支持序列化。

# 核心字段

```
 //默认数组大小
 private static final int DEFAULT_CAPACITY = 10;
 //一个空数组
 private static final Object[] EMPTY_ELEMENTDATA = {};
 //默认空数组
 private static final Object[] DEFAULTCAPACITY_EMPTY_ELEMENTDATA = {};
 //核心数组
 transient Object[] elementData;
 //数组中元素的实际大小
 private int size;
```

可以看出ArrayList的底层实现是一个动态数组，默认大小为10。

# 构造方法

```
//创建指定大小的ArrayList
 public ArrayList(int initialCapacity) {
        if (initialCapacity > 0) {
            this.elementData = new Object[initialCapacity];
        } else if (initialCapacity == 0) {
            this.elementData = EMPTY_ELEMENTDATA;
        } else {
            throw new IllegalArgumentException("Illegal Capacity: "+
                                               initialCapacity);
        }
    }
 //默认构造方法
 public ArrayList() {
        this.elementData = DEFAULTCAPACITY_EMPTY_ELEMENTDATA;
    }
```

从构造方法可以看出，ArrayList默认创建了一个DEFAULTCAPACITY_EMPTY_ELEMENTDATA的空数组。此时并没有对该数组进行初始化容量，那么什么时候初始化容量呢？我们继续看。

# add()

```
  public void add(int index, E element) {
  	   //判断index是否越界
       rangeCheckForAdd(index);
       //判断是否扩容
       ensureCapacityInternal(size + 1);  // Increments modCount!!
       System.arraycopy(elementData, index, elementData, index + 1,
                         size - index);
       //添加element
       elementData[index] = element;
       size++;
    }
```

##  rangeCheckForAdd(index)

```
 private void rangeCheckForAdd(int index) {
        if (index > size || index < 0)
            throw new IndexOutOfBoundsException(outOfBoundsMsg(index));
    }
```

从源码中可以看出该方法的作用是判断index是否越界。

## ensureCapacityInternal(size + 1)

```
  private void ensureCapacityInternal(int minCapacity) {
      if (elementData == DEFAULTCAPACITY_EMPTY_ELEMENTDATA) {
          minCapacity = Math.max(DEFAULT_CAPACITY, minCapacity);
      }
      ensureExplicitCapacity(minCapacity);
    }
```

方法最后调用了ensureExplicitCapacity()方法，源码如下

```
 private void ensureExplicitCapacity(int minCapacity) {
        modCount++;
        if (minCapacity - elementData.length > 0)
            grow(minCapacity);
    }
```

接着调用了grow()方法

```
  private void grow(int minCapacity) {
        // 获取数组大小
        int oldCapacity = elementData.length;
        //新数组大小为旧数组+旧数组的一半，也就是新数组大小是旧数组的1.5倍
        int newCapacity = oldCapacity + (oldCapacity >> 1);
        //新数组大小小于旧数组大小，
        if (newCapacity - minCapacity < 0)
            newCapacity = minCapacity;
        if (newCapacity - MAX_ARRAY_SIZE > 0)
            newCapacity = hugeCapacity(minCapacity);
        // 将旧数组数据和新数组大小，创新一个新数组
        elementData = Arrays.copyOf(elementData, newCapacity);
    }
```

可以看出ArrayList的扩容机制的核心方法就是grow()方法了，扩容后的数组大小是旧数组的1.5倍。

# get()

```
 public E get(int index) {
        //判断index是否越界
        rangeCheck(index);
        return elementData(index);
    }
```

## rangeCheck(index)

```
 private void rangeCheck(int index) {
        if (index >= size)
            throw new IndexOutOfBoundsException(outOfBoundsMsg(index));
    }
```

主要判断index是否越界

# set()

```
  public E set(int index, E element) {
        //判断是否越界
        rangeCheck(index);
        //获取旧数据
        E oldValue = elementData(index);
        //设置新数据
        elementData[index] = element;
        //返回旧数据
        return oldValue;
    }
```

set()方法比较简单

# remove()

```
 public E remove(int index) {
        //判断是否越界
        rangeCheck(index);
        modCount++;
        //获取旧数据
        E oldValue = elementData(index);
        int numMoved = size - index - 1;
        if (numMoved > 0)
            System.arraycopy(elementData, index+1, elementData, index,
         //置null，方便GC
        elementData[--size] = null;
        return oldValue;
    }
```

# 其他方法

```
//获取size
public int size() {
      return size;
    }
//判断是否为空
public boolean isEmpty() {
      return size == 0;
    }
```

# 总结

我们从ArrayList的构造方法与增删改除，分析了ArrayList的源码实现。了解它的扩容机制和底层实现，那么在开始时我们提到过是否有线程安全的ArrayList呢？答案是有的，CopyOnWriteArrayList就是一种线程安全的ArrayList，有兴趣的同学可以自行去查看源码。

