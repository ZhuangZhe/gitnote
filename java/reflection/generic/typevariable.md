# TypeVariable

`TypeVariable`，也就是类型变量，它是各种类型变量的公共父接口，它主要用来表示带有上界的泛型参数的信息，它和`ParameterizedType`不同的地方是，`ParameterizedType`表示的参数的最外层一定是已知具体类型的\(如`List<String>`\)，而`TypeVariable`面向的是K、V、E等这些泛型参数字面量的表示。常见的`TypeVariable`的表示形式是`<T extends KnownType-1 & KnownType-2>`。

## 源码

```java
public interface TypeVariable<D extends GenericDeclaration> extends Type {

   //获得泛型的上限，若未明确声明上边界则默认为Object
    Type[] getBounds();
    
    //获取声明该类型变量实体(即获得类、方法或构造器名)
    D getGenericDeclaration();
    
    //获得名称，即K、V、E之类名称
    String getName();
    
    //获得注解类型的上限，若未明确声明上边界则默认为长度为0的数组
    AnnotatedType[] getAnnotatedBounds()
}
```

* `Type[] getBounds()`：获得该类型变量的上限\(上边界\)，若无显式定义\(extends\)，默认为Object，类型变量的上限可能不止一个，因为可以用&符号限定多个（这其中有且只能有一个为类或抽象类，且必须放在extends后的第一个，即若有多个上边界，则第一个&之后的必为接口）。
* `D getGenericDeclaration`：获得声明\(定义\)这个类型变量的类型及名称，会使用泛型的参数字面量表示，如`public void club.throwable.Main.query(java.util.List<club.throwable.Person>)`。
* `String getName()`：获取泛型参数的字面量名称，即K、V、E之类名称。
* `AnnotatedType[] getAnnotatedBounds()`：Jdk1.8新增的方法，用于获得注解类型的上限，若未明确声明上边界则默认为长度为0的数组。

## 例子

```java
public class Main14 {

    public static void main(String[] args) throws Exception {
        Class<Supper> subClass = Supper.class;
        TypeVariable<Class<Supper>>[] typeParameters = subClass.getTypeParameters();
        for (TypeVariable<Class<Supper>> typeVariable : typeParameters) {
            System.out.println("getBounds --> " + Arrays.toString(typeVariable.getBounds()));
            System.out.println("getGenericDeclaration  --> " + typeVariable.getGenericDeclaration());
            System.out.println("getName --> " + typeVariable.getName());
            AnnotatedType[] annotatedBounds = typeVariable.getAnnotatedBounds();
            StringBuilder stringBuilder = new StringBuilder("getAnnotatedBounds --> ");
            for (AnnotatedType annotatedType : annotatedBounds) {
                java.lang.annotation.Annotation[] annotations = annotatedType.getAnnotations();
                for (java.lang.annotation.Annotation annotation : annotations) {
                    stringBuilder.append(annotation).append(",");
                }
            }
            System.out.println(stringBuilder.toString());
            System.out.println("===================");
        }
    }

    @Target(ElementType.TYPE)
    public @interface Annotation {

    }

    interface InterFace {

    }

    public static class Person {

    }

    public static abstract class Supper<T extends Person & InterFace, E extends Annotation> {

    }
}
```

### 输出结果

```bash
getBounds --> [class org.throwable.inherited.Main14$Person, interface org.throwable.inherited.Main14$InterFace]
getGenericDeclaration  --> class org.throwable.inherited.Main14$Supper
getName --> T
getAnnotatedBounds -->
===================
getBounds --> [interface org.throwable.inherited.Main14$Annotation]
getGenericDeclaration  --> class org.throwable.inherited.Main14$Supper
getName --> E
getAnnotatedBounds -->
===================
```

