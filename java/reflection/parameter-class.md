# Parameter类

`Parameter`用于描述`Method`或者`Constructor`的参数，主要是用于获取参数的名称。因为在Java中没有形式参数的概念，也就是参数都是没有名称的。Jdk1.8新增了Parameter用来填补这个问题，使用javac编译器的时候加上`-parameters`参数的话，会在生成的.class文件中额外存储参数的元信息，这样会导致.class文件的大小增加。当你输入`javac -help`的时候，你会看到-parameters这个选项。获取Parameter的方法是Method或者Constructor的父类Executable的getParamaters方法。一般而言，Parameter是用于获取参数名称的后备方案，因为Jdk1.8之前没有这个类，并且即使使用了Jdk1.8如果javac编译器的时候没有加上`-parameters`参数的话，通过Parameter获取到的参数名称将会是"arg0"、"arg1"..."argn"类似的没有意义的参数名称。一般框架中使用其他方法解析方法或者构造器的参数名称，参考Spring的源码，具体是`LocalVariableTableParameterNameDiscoverer`，是使用ASM去解析和读取类文件字节码，提取参数名称。Parameter的主要方法如下：

| 方法 | 功能 |
| :--- | :--- |
| String getName\(\) | 返回该参数的名称 |
| int getModifiers\(\) | 返回该参数的修饰符 |
| `Class<?>` getType\(\) | 返回该参数的类型Class |
| `Class<?>` getParameterizedType\(\) | 返回该参数的泛型类型Type |
| boolean isNamePresent\(\) | 该参数的名称是否保存在class文件中，需要编译时加参数-parameters |
| boolean isImplicit\(\) | 该参数是否隐式声明 |
| boolean isSynthetic\(\) | 该参数是否复合的 |
| boolean isVarArgs\(\) | 该参数是否不定参数 |

## 例子

```java
public class Main11 {

    public static void main(String[] args) throws Exception {
        Class<Supper> supperClass = Supper.class;
        Method sayHello = supperClass.getDeclaredMethod("sayHello", String.class);
        sayHello.setAccessible(Boolean.TRUE);
        Parameter[] parameters = sayHello.getParameters();
        for (Parameter parameter : parameters) {
            System.out.println("isNamePresent->" + parameter.isNamePresent());
            System.out.println("isImplicit->" + parameter.isImplicit());
            System.out.println("getName->" + parameter.getName());
            System.out.println("=====================");
        }

    }

    public static class Supper {

        private void sayHello(String name) {
            System.out.println(String.format("%s say hello!", name));
        }
    }
}
```

### 输出结果

```bash
isNamePresent->true
isImplicit->false
getName->name
```

