# TreeSet

## 源码

### 数据结构

```java
package java.util;

public class TreeSet<E> extends AbstractSet<E>
    implements NavigableSet<E>, Cloneable, java.io.Serializable {
    
    private transient NavigableMap<E,Object> m; // 存数据用的map，map中的key值则为数据
    
    private static final Object PRESENT = new Object(); // 公用的value值
    
    TreeSet(NavigableMap<E,Object> m) {
        this.m = m;
    }
    
    public TreeSet() {
        this(new TreeMap<>());
    }
    
    public TreeSet(Comparator<? super E> comparator) {
        this(new TreeMap<>(comparator));
    }
    
    public TreeSet(Collection<? extends E> c) {
        this();
        addAll(c);
    }
    
    public TreeSet(SortedSet<E> s) {
        this(s.comparator());
        addAll(s);
    }
}
```

### 添加元素

```java
public boolean add(E e) {
    return m.put(e, PRESENT)==null;
}
```

### 删除元素

```java
public boolean remove(Object o) {
    return m.remove(o)==PRESENT;
}

public E pollFirst() {
        Map.Entry<E,?> e = m.pollFirstEntry();
        return (e == null) ? null : e.getKey();
    }
    
    public E pollLast() {
        Map.Entry<E,?> e = m.pollLastEntry();
        return (e == null) ? null : e.getKey();
    }
```

读元素

```java
public E first() {
    return m.firstKey();
}

public E last() {
    return m.lastKey();
}

public E lower(E e) {
    return m.lowerKey(e);
}

public E floor(E e) {
    return m.floorKey(e);
}

public E ceiling(E e) {
    return m.ceilingKey(e);
}

public E higher(E e) {
    return m.higherKey(e);
}
```

