# ParameterizedType

`ParameterizedType`，也就是参数化类型，注释里面说到`ParameterizedType`表示一个参数化类型，例如`Collection<String>`，实际上只要带有参数化\(泛型\)标签`<ClassName>`的参数或者属性，都属于`ParameterizedType`。

例如下面的类型都是`ParameterizedType`：

```java
Set<String> set;
Class<Integer> clazz;
MyClass<String> myClass;
List<String> list;

class MyClass<V>{

}
```

而像下面的忽略泛型参数或者基本数据类型和基本数据类型的包装类都不是`ParameterizedType`：

```java
String name = "throwbale";
int age = 25;
Set set;
List list;

public String method(int age,String name){

}
```

## 源码

```java
package java.lang.reflect;

public interface ParameterizedType extends Type {

    Type[] getActualTypeArguments();

    Type getRawType();

    Type getOwnerType();
}
```

* `Type[] getActualTypeArguments()`：返回这个`ParameterizedType`类型的参数的实际类型`Type`数组，`Type`数组里面的元素有可能是`Class`、`ParameterizedType`、`TypeVariable`、`GenericArrayType`或者`WildcardType`之一。值得注意的是，无论泛型符号`<>`中有几层`<>`嵌套，这个方法仅仅脱去最外层的`<>`，之后剩下的内容就作为这个方法的返回值。
* `Type getRawType()`：返回的是当前这个`ParameterizedType`的原始类型，从`ParameterizedTypeImpl`的源码看来，原始类型rawType一定是一个`Class<?>`实例。举个例子，`List<Person>`通过`getRawType()`获取到的Type实例实际上是`Class<?>`实例，和List.class等价。
* `Type getOwnerType()`：获取原始类型所属的类型，从`ParameterizedTypeImpl`的源码看来，就是调用了原始类型`rawType`的`getDeclaringClass()`方法，而像`rawType`为`List<T>`、`Map<T>`这些类型的`getOwnerType()`实际上就是调用`List.class.getDeclaringClass()`，`Map.class.getDeclaringClass()`，返回值都是null。

## 例子

```java
public class Main13 {

    public static void main(String[] args) throws Exception {
        Class<Sub> subClass = Sub.class;
        Type genericSuperclass = subClass.getGenericSuperclass();
        if (genericSuperclass instanceof ParameterizedType) {
            ParameterizedType parameterizedType = (ParameterizedType) genericSuperclass;
            //获取父类泛型类型数组
            Type[] actualTypeArguments = parameterizedType.getActualTypeArguments();
            for (Type type : actualTypeArguments) {
                System.out.println(type + " is ParameterizedType -> " + (type instanceof ParameterizedType));
            }
        }
        Field field = subClass.getDeclaredField("clazz");
        Type genericType = field.getGenericType();
        System.out.println(genericType + " is ParameterizedType -> " + (genericType instanceof ParameterizedType));
    }

    public static class Person {

    }

    public static abstract class Supper<T, E> {

    }

    public static class Sub extends Supper<String, List<Person>> {

    }
}
```

### 输出结果

```bash
class java.lang.String is ParameterizedType -> false
java.util.List<org.throwable.inherited.Main13$Person> is ParameterizedType -> true
java.lang.Class<?> is ParameterizedType -> true
```







