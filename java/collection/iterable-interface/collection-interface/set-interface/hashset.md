# HashSet

## 源码

### 数据结构

```java
public class HashSet<E>
    extends AbstractSet<E>
    implements Set<E>, Cloneable, java.io.Serializable {
    
    static final long serialVersionUID = -5024744406713321676L;

    private transient HashMap<E,Object> map; // 用一个map存数据，数据为map的key
    
    private static final Object PRESENT = new Object(); // 数据公用的value
    
    public HashSet() {
        map = new HashMap<>();
    }
    
    public HashSet(Collection<? extends E> c) {
        map = new HashMap<>(Math.max((int) (c.size()/.75f) + 1, 16));
        addAll(c);
    }
    
    public HashSet(int initialCapacity, float loadFactor) {
        map = new HashMap<>(initialCapacity, loadFactor);
    }
    
    public HashSet(int initialCapacity) {
        map = new HashMap<>(initialCapacity);
    }
    
    HashSet(int initialCapacity, float loadFactor, boolean dummy) {
        map = new LinkedHashMap<>(initialCapacity, loadFactor);
    }
    
    // ...

}
```

### 添加元素

```java
public boolean add(E e) {
    return map.put(e, PRESENT)==null;
}
```

### 删除元素

```java
public boolean remove(Object o) {
    return map.remove(o)==PRESENT;
}
```

