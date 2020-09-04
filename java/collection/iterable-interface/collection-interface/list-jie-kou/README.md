# List接口

* `Arraylist`：`Object[]`数组
* `Vector`：`Object[]`数组
* `LinkedList`： 双向链表\(JDK1.6 之前为循环链表，JDK1.7 取消了循环\)

## ArrayList和Vector的区别?

* ArrayList 是List的主要实现类，底层使用Object\[ \]存储，适用于频繁的查找工作，线程不安全 ；
* Vector是 List 的古老实现类，底层使用Object\[ \]存储，线程安全的。

## ArrayList与LinkedList区别?

1. **是否保证线程安全：**`ArrayList`和`LinkedList`都是不同步的，也就是不保证线程安全；
2. **底层数据结构：** `Arraylist` 底层使用的是**`Object`数组**；`LinkedList`底层使用的是**双向链表**数据结构\(JDK1.6 之前为循环链表，JDK1.7取消了循环。注意双向链表和双向循环链表的区别，下面有介绍到！\)
3. **插入和删除是否受元素位置的影响：** ① **`ArrayList`采用数组存储，所以插入和删除元素的时间复杂度受元素位置的影响。** 比如：执行`add(E e)`方法的时候， `ArrayList`会默认在将指定的元素追加到此列表的末尾，这种情况时间复杂度就是O\(1\)。但是如果要在指定位置 i 插入和删除元素的话`add(int index, E element)`时间复杂度就为 O\(n-i\)。因为在进行上述操作的时候集合中第i和第i个元素之后的\(n-i\)个元素都要执行向后位/向前移一位的操作。 ②**`LinkedList`采用链表存储，所以对于`add(E e)`方法的插入，删除元素时间复杂度不受元素位置的影响，近似 O\(1\)，如果是要在指定位置`i`插入和删除元素的话`(add(int index, E element)`时间复杂度近似为`o(n)`因为需要先移动到指定位置再插入。**
4. **是否支持快速随机访问：** `LinkedList`不支持高效的随机元素访问，而`ArrayList`支持。快速随机访问就是通过元素的序号快速获取元素对象\(对应于`get(int index)`方法\)。
5. **内存空间占用：** ArrayList 的空 间浪费主要体现在在list列表的结尾会预留一定的容量空间，而LinkedList的空间花费则体现在它的每一个元素都需要消耗比ArrayList更多的空间\(因为要存放直接后继和直接前驱以及数据\)。

## 源码

```java
package java.util;

import java.util.function.UnaryOperator;

public interface List<E> extends Collection<E> {

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
    
    boolean addAll(int index, Collection<? extends E> c);
    
    boolean removeAll(Collection<?> c);
    
    boolean retainAll(Collection<?> c);
    
    default void replaceAll(UnaryOperator<E> operator) {
        Objects.requireNonNull(operator);
        final ListIterator<E> li = this.listIterator();
        while (li.hasNext()) {
            li.set(operator.apply(li.next()));
        }
    }
    
    default void sort(Comparator<? super E> c) {
        Object[] a = this.toArray();
        Arrays.sort(a, (Comparator) c);
        ListIterator<E> i = this.listIterator();
        for (Object e : a) {
            i.next();
            i.set((E) e);
        }
    }
    
    void clear();
    
    boolean equals(Object o);
    
    int hashCode();
    
    E get(int index);
    
    E set(int index, E element);
    
    void add(int index, E element);
    
    E remove(int index);
    
    int indexOf(Object o);
    
    int lastIndexOf(Object o);
    
    ListIterator<E> listIterator();
    
    ListIterator<E> listIterator(int index);

    List<E> subList(int fromIndex, int toIndex);
    
    @Override
    default Spliterator<E> spliterator() {
        if (this instanceof RandomAccess) {
            return new AbstractList.RandomAccessSpliterator<>(this);
        } else {
            return Spliterators.spliterator(this, Spliterator.ORDERED);
        }
    }
    
    static <E> List<E> of() {
        return ImmutableCollections.emptyList();
    }
    
    static <E> List<E> of(E e1) {
        return new ImmutableCollections.List12<>(e1);
    }
    
    static <E> List<E> of(E e1, E e2) {
        return new ImmutableCollections.List12<>(e1, e2);
    }
    
    static <E> List<E> of(E e1, E e2, E e3) {
        return new ImmutableCollections.ListN<>(e1, e2, e3);
    }
    
    static <E> List<E> of(E e1, E e2, E e3, E e4) {
        return new ImmutableCollections.ListN<>(e1, e2, e3, e4);
    }
    
    static <E> List<E> of(E e1, E e2, E e3, E e4, E e5) {
        return new ImmutableCollections.ListN<>(e1, e2, e3, e4, e5);
    }
    
    static <E> List<E> of(E e1, E e2, E e3, E e4, E e5, E e6) {
        return new ImmutableCollections.ListN<>(e1, e2, e3, e4, e5, e6);
    }
    
    static <E> List<E> of(E e1, E e2, E e3, E e4, E e5, E e6, E e7) {
        return new ImmutableCollections.ListN<>(e1, e2, e3, e4, e5, e6, e7);
    }
    
    static <E> List<E> of(E e1, E e2, E e3, E e4, E e5, E e6, E e7, E e8) {
        return new ImmutableCollections.ListN<>(e1, e2, e3, e4, e5, e6, e7, e8);
    }
    
    static <E> List<E> of(E e1, E e2, E e3, E e4, E e5, E e6, E e7, E e8, E e9) {
        return new ImmutableCollections.ListN<>(e1, e2, e3, e4, e5, e6, e7, e8, e9);
    }
    
    static <E> List<E> of(E e1, E e2, E e3, E e4, E e5, E e6, E e7, E e8, E e9, E e10) {
        return new ImmutableCollections.ListN<>(e1, e2, e3, e4, e5, e6, e7, e8, e9, e10);
    }
    
    static <E> List<E> of(E... elements) {
        switch (elements.length) { // implicit null check of elements
            case 0:
                return ImmutableCollections.emptyList();
            case 1:
                return new ImmutableCollections.List12<>(elements[0]);
            case 2:
                return new ImmutableCollections.List12<>(elements[0], elements[1]);
            default:
                return new ImmutableCollections.ListN<>(elements);
        }
    }
    
    static <E> List<E> copyOf(Collection<? extends E> coll) {
        return ImmutableCollections.listCopy(coll);
    }
}
```



