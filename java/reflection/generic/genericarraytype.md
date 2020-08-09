# GenericArrayType

`GenericArrayType`，也就是泛型数组，也就是元素类型为泛型类型的数组实现了该接口。它要求元素的类型是`ParameterizedType`或`TypeVariable`\(实际中发现元素是`GenericArrayType`也是允许的\)。举个例子：

```java
List<String>[] listArray; //是GenericArrayType,元素是List<String>类型，也就是ParameterizedType类型
T[] tArray; //是GenericArrayType,元素是T类型，也就是TypeVariable类型

Person[] persons; //不是GenericArrayType
List<String> strings; //不是GenericArrayType
```

## 源码

```java
public interface GenericArrayType extends Type {

    Type getGenericComponentType();
}
```

* `Type getGenericComponentType()`：获取泛型数组中元素的类型。注意无论从左向右有几个`[]`并列，这个方法仅仅脱去最右边的`[]`之后剩下的内容就作为这个方法的返回值。

## 例子

```java
public class Main15<T> {


    public static void main(String[] args) throws Exception {
        Method[] methods = Main15.class.getMethods();
        for (Method method : methods) {
            if ("method".equals(method.getName())) {
                Type[] genericParameterTypes = method.getGenericParameterTypes();
                for (Type type : genericParameterTypes) {
                    if (type instanceof GenericArrayType) {
                        System.out.println("GenericArrayType --> " + type + " getGenericComponentType --> "
                                + ((GenericArrayType) type).getGenericComponentType());
                    } else {
                        System.out.println("Not GenericArrayType --> " + type);
                    }
                }
            }
        }
    }

    public static <T> void method(String[] strings, List<String> ls, List<String>[] lsa, T[] ts, List<T>[] tla, T[][] tts) {

    }
}
```

### 输出结果

```bash
Not GenericArrayType --> class [Ljava.lang.String;
Not GenericArrayType --> java.util.List<java.lang.String>
GenericArrayType --> java.util.List<java.lang.String>[] getGenericComponentType --> java.util.List<java.lang.String>
GenericArrayType --> T[] getGenericComponentType --> T
GenericArrayType --> java.util.List<T>[] getGenericComponentType --> java.util.List<T>
GenericArrayType --> T[][] getGenericComponentType --> T[]
```

* `String[] strings`：数组是`Class`类型。
* `List<String> ls`：列表是`ParameterizedType`类型。
* `List<String>[] lsa`：数组是`GenericArrayType`类型，调用`getGenericComponentType`后返回的类型是`java.util.List<java.lang.String>`，也就是数组元素是`ParameterizedType`类型。
* `T[] ts`：s数组是`GenericArrayType`类型，调用`getGenericComponentType`后返回的类型是T，也就是数组元素是`TypeVariable`类型。
* `List<T>[] tla`：数组是GenericArrayType类型，调用`getGenericComponentType`后返回的类型是`java.util.List<T>`，也就是数组元素是`ParameterizedType`类型。
* `T[][] tts`：数组是`GenericArrayType`类型，调用`getGenericComponentType`后返回的类型`T[]`，也就是数组元素是`GenericArrayType`类型。

