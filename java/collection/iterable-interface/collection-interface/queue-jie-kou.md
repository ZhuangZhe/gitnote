# Queue接口



## 源码

```java
package java.util;

public interface Queue<E> extends Collection<E> {
    
    boolean add(E e);
    
    boolean offer(E e);
    
    E remove();
    
    E poll();
    
    E element();
    
    E peek();
}

```

