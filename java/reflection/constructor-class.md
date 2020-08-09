# Constructor类

`Constructor`用于描述一个类的构造函数。它除了能获取到构造的注解信息、参数的注解信息、参数的信息之外，还有一个很重要的作用是可以抑制修饰符进行实例化，而Class的实例化方法`newInstance`只能实例化修饰符为public的类。Constructor的主要方法如下：

| 方法 | 功能 |
| :--- | :--- |
| `Class<T>` getDeclaringClass\(\) | 获取当前构造的定义类 |
| String getName\(\) | 获取当前构造的名称 |
| int getModifiers\(\) | 获取当前构造的修饰符 |
| String toGenericString\(\) | 返回描述此构造的字符串，其中包括类型参数的字面量 |
| `TypeVariable<Constructor<T>>[]` getTypeParameters\(\) | 获取类定义泛型参数的类型变量 |
| `Class<?>[]` getExceptionTypes\(\) | 获取当前构造异常类型数组，如果不存在则返回一个长度为0的数组 |
| Type\[\] getGenericExceptionTypes\(\) | 获取当前构造异常类型数组的泛型类型，如果不存在则返回一个长度为0的数组 |
| Type\[\] getGenericParameterTypes\(\) | 获取当前构造参数的泛型类型，如果不存在则返回一个长度为0的数组 |
| Annotation\[\]\[\] getParameterAnnotations\(\) | 获取当前构造参数的注解数组，这里是二维数组的原因是一个参数可以使用多个注解 |
| int getParameterCount\(\) | 获取当前构造参数的数量 |
| `Class<?>[]` getParameterTypes\(\) | 获取当前构造参数的Class数组 |
| boolean isSynthetic\(\) | 当前构造是否复合的 |
| boolean isVarArgs\(\) | 当前构造是否使用不定参数 |
| T newInstance\(Object...initargs\) | 使用此构造对象表示的构造方法来创建该构造方法的声明类的新实例，并用指定的初始化参数初始化该实例 |
| Parameter\[\] getParameters\(\) | 返回此构造对象的参数Parameter数组，如果没有则返回一个长度为0的数组 |
| void setAccessible\(boolean flag\) | 抑制构造访问修饰符的权限判断 |

## 例子

```java
public class Main8 {

    public static void main(String[] args) throws Exception{
        Class<Supper> supperClass = Supper.class;
        Constructor<Supper> constructor = supperClass.getDeclaredConstructor();
        constructor.setAccessible(Boolean.TRUE);
        Supper supper = constructor.newInstance();
        supper.sayHello("throwable");
    }

    private static class Supper {

        public void sayHello(String name) {
            System.out.println(String.format("%s say hello!", name));
        }
    }
}
```

### 输出结果

```text
throwable say hello!
```

