# WildcardType

`WildcardType`用于表示通配符\(?\)类型的表达式的泛型参数，例如`<? extends Number>`等。根据`WildcardType`注释提示：现阶段通配符表达式仅仅接受一个上边界或者下边界，这个和定义类型变量时候可以指定多个上边界是不一样。但是为了保持扩展性，这里返回值类型写成了数组形式。实际上现在返回的数组的大小就是1。

## 源码

```java
public interface WildcardType extends Type {

    Type[] getUpperBounds();

    Type[] getLowerBounds();
}
```

* `Type[] getUpperBounds()`：获取泛型通配符的上限类型`Type`数组，实际上目前该数组只有一个元素，也就是说只能有一个上限类型。
* `Type[] getLowerBounds()`：获取泛型通配符的下限类型`Type`数组，实际上目前该数组只有一个元素，也就是说只能有一个下限类型。

## 例子

```java
public class Main16 {

    public static void main(String[] args) {
        Class<Main16> clazz = Main16.class;
        Method[] methods = clazz.getMethods();
        for (Method method : methods) {
            if ("print".equals(method.getName())) {
                Type[] genericParameterTypes = method.getGenericParameterTypes();
                for (Type type : genericParameterTypes) {
                    if (type instanceof ParameterizedType) {
                        ParameterizedType parameterizedType = (ParameterizedType) type;
                        Type[] actualTypeArguments = parameterizedType.getActualTypeArguments();
                        for (Type actualType : actualTypeArguments) {
                            if (actualType instanceof WildcardType) {
                                WildcardType wildcardType = (WildcardType) actualType;
                                System.out.println("WildcardType --> " + wildcardType + " getUpperBounds--> "
                                        + Arrays.toString(wildcardType.getUpperBounds()) + " getLowerBounds--> " + Arrays.toString(wildcardType.getLowerBounds()));
                            } else {
                                System.out.println("Not WildcardType --> " + actualType);
                            }
                        }

                    }
                }
            }
        }
    }

    interface Person {

    }

    public static void print(List<? extends Number> list, Set<? super Person> persons) {

    }
}
```

### 输出结果

```bash
WildcardType --> ? extends java.lang.Number getUpperBounds--> [class java.lang.Number] getLowerBounds--> []
WildcardType --> ? super org.throwable.inherited.Main16$Person getUpperBounds--> [class java.lang.Object] getLowerBounds--> [interface org.throwable.inherited.Main16$Person]
```

这里注意的是`List<? extends Number> list`这个参数整体来看是`ParameterizedType`类型，剥掉第一次List之后的`? extends Number`是`WildcardType`类型。

