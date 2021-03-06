# Vector

## 源码

### 数据结构

```java
public class Vector<E>
    extends AbstractList<E>
    implements List<E>, RandomAccess, Cloneable, java.io.Serializable {
    
    protected Object[] elementData; // 数据存储
    
    protected int elementCount; // 元素个数计算
    
    protected int capacityIncrement; // 每次容量增加的幅度
    
    private static final long serialVersionUID = -2767605614048989439L;
    
    public Vector(int initialCapacity, int capacityIncrement) {
        super();
        if (initialCapacity < 0)
            throw new IllegalArgumentException("Illegal Capacity: "+ initialCapacity);
        this.elementData = new Object[initialCapacity];
        this.capacityIncrement = capacityIncrement;
    }
    
    public Vector(int initialCapacity) {
        this(initialCapacity, 0);
    }
    
    public Vector() {
        this(10); // 初始容量为10
    }
    
    public Vector(Collection<? extends E> c) {
        elementData = c.toArray();
        elementCount = elementData.length;
        // defend against c.toArray (incorrectly) not returning Object[]
        // (see e.g. https://bugs.openjdk.java.net/browse/JDK-6260652)
        if (elementData.getClass() != Object[].class)
            elementData = Arrays.copyOf(elementData, elementCount, Object[].class);
    }
```

### 添加元素

```java
public synchronized void addElement(E obj) {
    modCount++; // 增加操作次数
    add(obj, elementData, elementCount);
}

public synchronized boolean add(E e) {
    modCount++; // 增加操作次数
    add(e, elementData, elementCount);
    return true;
}

private void add(E e, Object[] elementData, int s) {
    if (s == elementData.length) // 判断元素个数是否已经达到容量
        elementData = grow(); // 元素个数达到容量后进行扩容
    elementData[s] = e; // 当前元素的个数就是下一个元素的存放位置
    elementCount = s + 1; 
}

private Object[] grow() {
    return grow(elementCount + 1); // 要多加一个新插入的
}

private Object[] grow(int minCapacity) {
    return elementData = Arrays.copyOf(elementData, newCapacity(minCapacity));
}

private int newCapacity(int minCapacity) {
    // overflow-conscious code
    int oldCapacity = elementData.length;
    int newCapacity = oldCapacity + ((capacityIncrement > 0) ?
                                     capacityIncrement : 
                                     oldCapacity); // 没有明确扩容幅度的情况下，默认翻倍
    if (newCapacity - minCapacity <= 0) {
        if (minCapacity < 0) // overflow
            throw new OutOfMemoryError();
        return minCapacity;
    }
    return (newCapacity - MAX_ARRAY_SIZE <= 0)
        ? newCapacity
        : hugeCapacity(minCapacity);
}

private static int hugeCapacity(int minCapacity) {
    if (minCapacity < 0) // overflow
        throw new OutOfMemoryError();
    return (minCapacity > MAX_ARRAY_SIZE) ?
        Integer.MAX_VALUE :
        MAX_ARRAY_SIZE;
}
```

### 删除元素

```java
public synchronized boolean removeElement(Object obj) {
        modCount++;
        int i = indexOf(obj);
        if (i >= 0) {
            removeElementAt(i);
            return true;
        }
        return false;
    }
    
    public boolean remove(Object o) {
        return removeElement(o);
    }
    
    public synchronized E remove(int index) {
        modCount++;
        if (index >= elementCount)
            throw new ArrayIndexOutOfBoundsException(index);
        E oldValue = elementData(index);

        int numMoved = elementCount - index - 1;
        if (numMoved > 0)
            System.arraycopy(elementData, index+1, elementData, index, numMoved);
        elementData[--elementCount] = null; // Let gc do its work

        return oldValue;
    }
```

