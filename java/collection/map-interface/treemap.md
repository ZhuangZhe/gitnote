# TreeMap



## 源码

### 数据结构

```java
public class TreeMap<K,V>
    extends AbstractMap<K,V>
    implements NavigableMap<K,V>, Cloneable, java.io.Serializable {
    
    private final Comparator<? super K> comparator;

    private transient Entry<K,V> root;
    
    private transient int size = 0;
    
    private transient int modCount = 0;
    
    public TreeMap() {
        comparator = null;
    }
    
    public TreeMap(Comparator<? super K> comparator) {
        this.comparator = comparator;
    }
    
    public TreeMap(Map<? extends K, ? extends V> m) {
        comparator = null;
        putAll(m);
    }
    
    public TreeMap(SortedMap<K, ? extends V> m) {
        comparator = m.comparator();
        try {
            buildFromSorted(m.size(), m.entrySet().iterator(), null, null);
        } catch (java.io.IOException | ClassNotFoundException cannotHappen) {
        }
    }
    
    // ...
   
}
```

### 添加元素





