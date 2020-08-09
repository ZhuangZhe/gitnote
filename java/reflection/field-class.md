# Field类

`Field`类用来描述一个类里面的属性或者叫成员变量，通过Field可以获取属性的注解信息、泛型信息，获取和设置属性的值等等。Field的主要方法如下：

| 方法 | 功能 |
| :--- | :--- |
| String getName\(\) | 返回该属性的名称 |
| int getModifiers\(\) | 返回该属性的修饰符 |
| `Class<?>` getType\(\) | 返回该属性的类型Class |
| `Class<?>` getParameterizedType\(\) | 返回该属性的泛型类型Type |
| boolean isSynthetic\(\) | 该属性是否复合的 |
| boolean isEnumConstant\(\) | 该属性是否枚举类型的元素 |
| Object get\(Object obj\) | 通过对象实例获取该属性的值 |
| void set\(Object obj,Object value\) | 通过对象实例设置该属性的值 |
| void setAccessible\(boolean flag\) | 抑制属性访问修饰符的权限判断 |

这里忽略了注解以及Field实现了`FieldAccessor`接口中的`getBoolean`、`setBoolean`等方法。

## 例子

```java
public class Main12 {

    public static void main(String[] args) throws Exception {
        Class<Supper> supperClass = Supper.class;
        Supper supper = supperClass.newInstance();
        Method sayHello = supperClass.getDeclaredMethod("sayHello");
        sayHello.setAccessible(Boolean.TRUE);
        Field name = supperClass.getDeclaredField("name");
        name.setAccessible(Boolean.TRUE);
        name.set(supper,"throwable");
        System.out.println("Field get-->" + name.get(supper));
        sayHello.invoke(supper);
        name.set(supper, "throwable-10086");
        System.out.println("Field get-->" + name.get(supper));
        sayHello.invoke(supper);
    }

    public static class Supper {

        private String name;

        private void sayHello() {
            System.out.println(String.format("%s say hello!", name));
        }
    }
}
```

### 输出结果

```bash
Field get-->throwable
throwable say hello!
Field get-->throwable-10086
throwable-10086 say hello!
```

