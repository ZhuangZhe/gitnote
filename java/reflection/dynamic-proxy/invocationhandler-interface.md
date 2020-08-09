# InvocationHandler接口



## 源码

```java
package java.lang.reflect;

public interface InvocationHandler {

    public Object invoke(Object proxy, Method method, Object[] args) throws Throwable;

}
```

