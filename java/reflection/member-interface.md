# Member接口

Member接口注解提供成员属性的一些描述。

| 方法 | 功能 |
| :--- | :--- |
| `Class<?>` getDeclaringClass\(\) | 获取声明的Class对象，也就是获取当前Member实例的来源Class对象 |
| String getName\(\) | 获取实例的名称，对于Constructor返回全类名，对于Method返回方法名，对于Field返回属性名 |
| int getModifiers\(\) | 获取实例的修饰符 |
| boolean isSynthetic\(\) | 是否合成的 |

synthetic总的来说，是由编译器引入的字段、方法、类或其他结构，主要用于JVM内部使用，为了遵循某些规范而作的一些小技巧从而绕过这些规范，有点作弊的感觉，只不过是由编译器光明正大为之，一般开发者是没有权限的\(但事实上有时候还是能被利用到的\)。

## 源码

```java
package java.lang.reflect;

public interface Member {

    public static final int PUBLIC = 0;

    public static final int DECLARED = 1;

    public Class<?> getDeclaringClass();

    public String getName();

    public int getModifiers();

    public boolean isSynthetic();
    
}
```

