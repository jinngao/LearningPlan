![](https://www.lifeofpix.com/wp-content/uploads/2019/03/0C0A9135-1600x1067.jpg)

# 前言

今天我们来学习一下CopyOnWriteArrayList这个比ArrayList更安全的集合，同时带着以下几个问题去分析源码。

- CopyOnWriteArrayList如何保证线程安全？
- CopyOnWriteArrayList有什么优缺点吗？
- CopyOnWriteArrayList与ArrayList有什么区别吗？
- 了解CopyOnWriteArraySet吗？

# 继承关系

```
public class CopyOnWriteArrayList<E>
                 implements List<E>, RandomAccess, Cloneable,                    java.io.Serializable {
     //......
     }
```

首先我们来看一下CopyOnWriteArrayList的继承关系，从源码中可以看出它分别实现了List、RandomAccess、Cloneable以及Serializable，对于我们比较熟悉的应该是List和Serializable了。

# 核心字段

```
  //可以看出这是重入锁
  final transient ReentrantLock lock = new ReentrantLock();
  //底层数组，可以看出用了volatile
  private transient volatile Object[] array;  
```

通过以上两个核心字段，可以猜出CopyOnWriteArrayList内部进行了加锁来保证线程安全。那么它到底是如何实现的呢？我们继续往下分析。

# 构造方法

```
 /**
 * 默认构造方法，初始化了一个空数组
 */
 public CopyOnWriteArrayList() {
       setArray(new Object[0]);
   }
```

再来简单看一下其他两个构造方法，如下：

```
 public CopyOnWriteArrayList(Collection<? extends E> c) {
        Object[] elements;
        if (c.getClass() == CopyOnWriteArrayList.class)
            elements = ((CopyOnWriteArrayList<?>)c).getArray();
        else {
            elements = c.toArray();
            // c.toArray might (incorrectly) not return Object[] (see 6260652)
            if (elements.getClass() != Object[].class)
                elements = Arrays.copyOf(elements, elements.length, Object[].class);
        }
        setArray(elements);
    }
  public CopyOnWriteArrayList(E[] toCopyIn) {
      setArray(Arrays.copyOf(toCopyIn, toCopyIn.length, Object[].class));
  }

```

# 修改

```
  public E set(int index, E element) {
  	   //进行加锁
       final ReentrantLock lock = this.lock;
       lock.lock();
       try {
           //获取旧数组
           Object[] elements = getArray();
            //获取旧数据
           E oldValue = get(elements, index);
            //如果旧数据与插入的新数据不同
           if (oldValue != element) {
               int len = elements.length;
               //创建了一个新数组
               Object[] newElements = Arrays.copyOf(elements, len);
               newElements[index] = element;
               //设置array为newElements
               setArray(newElements);
           } else {
               // Not quite a no-op; ensures volatile write semantics
               setArray(elements);
           }
           return oldValue;
       } finally {
          //解锁
           lock.unlock();
       }
   }
```

可以看出在写入数据时通过加锁来保证线程安全，并且通过复制一个新数组。然后修改新数组中index对应的Value值，最后将新数组赋值给CopyOnWriteArrayList的底层数组array，也可以说是将array数组指向新数组，如下图：

![](<https://raw.githubusercontent.com/jinngao/Picture/master/CopyOnWriteArrayList_set.png>)

对于CopyOnWriteArrayList的set()方法，我们可以这样理解。

- 获取旧数组
- 通过复制旧数组创建新数组
- 在新数组中进行修改
- array指向新数组

# 添加

```
 public boolean add(E e) {
      final ReentrantLock lock = this.lock;
      lock.lock();
      try {
          Object[] elements = getArray();
          int len = elements.length;
          Object[] newElements = Arrays.copyOf(elements, len + 1);
          newElements[len] = e;
          setArray(newElements);
          return true;
      } finally {
          lock.unlock();
        }
    }
```

可以看出与set()方法类似，也是通过加锁的方式来保证线程安全。然后创建一个新数组来添加元素，最后将新数组赋值给array。

# 获取

```
public E get(int index) {
       return get(getArray(), index);
    }
@SuppressWarnings("unchecked")
private E get(Object[] a, int index) {
       return (E) a[index];
    }
```

从get()方法源码中可以看出 ，在从CopyOnWriteArrayList获取元素时并没有进行加锁。那么我们就要思考一个问题了，如果当写入数据时，在没有写完数据的情况下。再去读取数据，那么我们能准确的获得数据吗？而且重点是在get()方法中，是直接对底层数组array进行操作的。那就意味着在我们读取数据时，有可能获得的是旧数据。也就是说CopyOnWriteArrayList并不能保证数据的实时性，主要原因可以从以下几点来思考。

- 在get()方法并没有进行加锁
- 读取的是底层数组array的数据

# 删除

```
  public E remove(int index) {
        //加锁
        final ReentrantLock lock = this.lock;
        lock.lock();
        try {
            Object[] elements = getArray();
            int len = elements.length;
            E oldValue = get(elements, index);
            int numMoved = len - index - 1;
            if (numMoved == 0)
                setArray(Arrays.copyOf(elements, len - 1));
            else {
                Object[] newElements = new Object[len - 1];
                System.arraycopy(elements, 0, newElements, 0, index);
                System.arraycopy(elements, index + 1, newElements, index,
                                 numMoved);
                setArray(newElements);
            }
            return oldValue;
        } finally {
            //解锁
            lock.unlock();
        }
    }
```

可以看出remove()方法也是一样的，通过加锁来保证线程安全。

# 其他方法

```
 /**
 * 获取底层数组
 */
 final Object[] getArray() {
      return array;
    }
 /**
 * 设置底层数组
 */
 final void setArray(Object[] a) {
      array = a;
    }
 /**
 * 获取数组大小
 */
 public int size() {
      return getArray().length;
    }
 /**
 * 判断数组是否为空
 */   
 public boolean isEmpty() {
       return size() == 0;
    }
```

# 优点与缺点

通过查看源码，我们可以看出CopyOnWriteArrayList最大的优点就是线程安全，也是最突出的优点。它在读取操作时没有加锁，只有当进行写入操作时才会加锁。这也体现出了CopyOnWriteArrayList的一个缺点，那就是无法读取实时数据。因为可能当写操作还没有完成时，就进行了读取操作，那么读取的将是旧数据。

# CopyOnWriteArraySet

接下来我们来简单了解一下CopyOnWriteArrayList的表兄弟CopyOnWriteArraySet，主要源码如下。

```
 private final CopyOnWriteArrayList<E> al;
 
 public CopyOnWriteArraySet() {
        al = new CopyOnWriteArrayList<E>();
    }
```

通过构造方法就可以看出初始化了一个CopyOnWriteArrayList，也就是说CopyOnWriteArraySet内部是用CopyOnWriteArrayList来实现的，我们再来看一下其他几个方法。

```
public boolean add(E e) {
        return al.addIfAbsent(e);
    }
public boolean remove(Object o) {
        return al.remove(o);
    }
```

可以看出add()和remove()都是通过CopyOnWriteArrayList来实现。

# 总结

通过分析源码，我们了解了CopyOnWriteArrayList的内部实现，以及它是如何保证线程安全的，还了解一下CopyOnWriteArraySet。