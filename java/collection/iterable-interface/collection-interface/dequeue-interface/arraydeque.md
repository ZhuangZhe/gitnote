# ArrayDeque

## 源码

```java
package java.util;

import java.io.Serializable;
import java.util.function.Consumer;
import java.util.function.Predicate;
import java.util.function.UnaryOperator;
import jdk.internal.misc.SharedSecrets;

public class ArrayDeque<E> extends AbstractCollection<E>
                           implements Deque<E>, Cloneable, Serializable {
                           
    transient Object[] elements; // 数据
    
    transient int head; // 数据头
    
    transient int tail; // 数据尾
    
    private static final int MAX_ARRAY_SIZE = Integer.MAX_VALUE - 8;
    
    public ArrayDeque() {
        elements = new Object[16];
    }
    
    public ArrayDeque(int numElements) {
        elements = new Object[(numElements < 1) ? 1 :
                       (numElements == Integer.MAX_VALUE) ? Integer.MAX_VALUE :
                       numElements + 1];
    }
    
    public ArrayDeque(Collection<? extends E> c) {
        this(c.size());
        copyElements(c);
    }
```

### 添加元素

```java
public void addFirst(E e) {
        if (e == null)
            throw new NullPointerException();
        final Object[] es = elements; // 获得数据指针
        es[head = dec(head, es.length)] = e;
        if (head == tail)
            grow(1);
    }
    
    public void addLast(E e) {
        if (e == null)
            throw new NullPointerException();
        final Object[] es = elements;
        es[tail] = e;
        if (head == (tail = inc(tail, es.length)))
            grow(1);
    }
    
    static final int inc(int i, int modulus) {
        if (++i >= modulus) i = 0;
        return i;
    }
    
    private void grow(int needed) {
        // overflow-conscious code
        final int oldCapacity = elements.length;
        int newCapacity;
        // Double capacity if small; else grow by 50%
        int jump = (oldCapacity < 64) ? (oldCapacity + 2) : (oldCapacity >> 1);
        if (jump < needed || (newCapacity = (oldCapacity + jump)) - MAX_ARRAY_SIZE > 0)
            newCapacity = newCapacity(needed, jump);
        final Object[] es = elements = Arrays.copyOf(elements, newCapacity);
        // Exceptionally, here tail == head needs to be disambiguated
        if (tail < head || (tail == head && es[head] != null)) {
            // wrap around; slide first leg forward to end of array
            int newSpace = newCapacity - oldCapacity;
            System.arraycopy(es, head, es, head + newSpace, oldCapacity - head);
            for (int i = head, to = (head += newSpace); i < to; i++)
                es[i] = null;
        }
    }
    
    public boolean offerFirst(E e) {
        addFirst(e);
        return true;
    }
    
    public boolean offerLast(E e) {
        addLast(e);
        return true;
    }
    
    public boolean add(E e) {
        addLast(e);
        return true;
    }
    
    public boolean offer(E e) {
        return offerLast(e);
    }
    
    public void push(E e) {
        addFirst(e);
    }
```

### 删除元素

```java
public E removeFirst() {
        E e = pollFirst();
        if (e == null)
            throw new NoSuchElementException();
        return e;
    }
    
    public E removeLast() {
        E e = pollLast();
        if (e == null)
            throw new NoSuchElementException();
        return e;
    }
    
    public E pollFirst() {
        final Object[] es;
        final int h;
        E e = elementAt(es = elements, h = head);
        if (e != null) {
            es[h] = null;
            head = inc(h, es.length);
        }
        return e;
    }

    public E pollLast() {
        final Object[] es;
        final int t;
        E e = elementAt(es = elements, t = dec(tail, es.length));
        if (e != null)
            es[tail = t] = null;
        return e;
    }
    
    public E remove() {
        return removeFirst();
    }
    
    public E poll() {
        return pollFirst();
    }
    
    public E pop() {
        return removeFirst();
    }
```

