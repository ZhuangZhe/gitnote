# Set接口

* `HashSet`（无序，唯一）: 基于 `HashMap` 实现的，底层采用 `HashMap` 来保存元素
* `LinkedHashSet`：`LinkedHashSet` 是 `HashSet` 的子类，并且其内部是通过 `LinkedHashMap` 来实现的。有点类似于我们之前说的 `LinkedHashMap` 其内部是基于 `HashMap` 实现一样，不过还是有一点点区别的
* `TreeSet`（有序，唯一）： 红黑树\(自平衡的排序二叉树\)

## 源码

```java
package java.util;

public interface Set<E> extends Collection<E> {

    int size();
    
    boolean isEmpty();
    
    boolean contains(Object o);
    
    Iterator<E> iterator();
    
    Object[] toArray();
    
    <T> T[] toArray(T[] a);
    
    boolean add(E e);
    
    boolean remove(Object o);
    
    boolean containsAll(Collection<?> c);
    
    boolean addAll(Collection<? extends E> c);
    
    boolean retainAll(Collection<?> c);
    
    boolean removeAll(Collection<?> c);
    
    void clear();
    
    boolean equals(Object o);
    
    int hashCode();
    
    @Override
    default Spliterator<E> spliterator() {
        return Spliterators.spliterator(this, Spliterator.DISTINCT);
    }
    
    static <E> Set<E> of() {
        return ImmutableCollections.emptySet();
    }
    
    static <E> Set<E> of(E e1) {
        return new ImmutableCollections.Set12<>(e1);
    }
    
    static <E> Set<E> of(E e1, E e2) {
        return new ImmutableCollections.Set12<>(e1, e2);
    }
    
    static <E> Set<E> of(E e1, E e2, E e3) {
        return new ImmutableCollections.SetN<>(e1, e2, e3);
    }
    
    static <E> Set<E> of(E e1, E e2, E e3, E e4) {
        return new ImmutableCollections.SetN<>(e1, e2, e3, e4);
    }
    
    static <E> Set<E> of(E e1, E e2, E e3, E e4, E e5) {
        return new ImmutableCollections.SetN<>(e1, e2, e3, e4, e5);
    }
    
    static <E> Set<E> of(E e1, E e2, E e3, E e4, E e5, E e6) {
        return new ImmutableCollections.SetN<>(e1, e2, e3, e4, e5, e6);
    }
    
    static <E> Set<E> of(E e1, E e2, E e3, E e4, E e5, E e6, E e7) {
        return new ImmutableCollections.SetN<>(e1, e2, e3, e4, e5, e6, e7);
    }
    
    static <E> Set<E> of(E e1, E e2, E e3, E e4, E e5, E e6, E e7, E e8) {
        return new ImmutableCollections.SetN<>(e1, e2, e3, e4, e5, e6, e7, e8);
    }
    
    static <E> Set<E> of(E e1, E e2, E e3, E e4, E e5, E e6, E e7, E e8, E e9) {
        return new ImmutableCollections.SetN<>(e1, e2, e3, e4, e5, e6, e7, e8, e9);
    }
    
    static <E> Set<E> of(E e1, E e2, E e3, E e4, E e5, E e6, E e7, E e8, E e9, E e10) {
        return new ImmutableCollections.SetN<>(e1, e2, e3, e4, e5, e6, e7, e8, e9, e10);
    }
    
    static <E> Set<E> of(E... elements) {
        switch (elements.length) { // implicit null check of elements
            case 0:
                return ImmutableCollections.emptySet();
            case 1:
                return new ImmutableCollections.Set12<>(elements[0]);
            case 2:
                return new ImmutableCollections.Set12<>(elements[0], elements[1]);
            default:
                return new ImmutableCollections.SetN<>(elements);
        }
    }
    
    static <E> Set<E> copyOf(Collection<? extends E> coll) {
        if (coll instanceof ImmutableCollections.AbstractImmutableSet) {
            return (Set<E>)coll;
        } else {
            return (Set<E>)Set.of(new HashSet<>(coll).toArray());
        }
    }
}
```



