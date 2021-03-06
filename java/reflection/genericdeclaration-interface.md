# GenericDeclaration接口

`GenericDeclaration`接口继承自`AnnotatedElement`，新增了一个方法`getTypeParameters()`用于返回类型变量`TypeVariable`数组，这里的`TypeVariable`是类型变量，它的定义如下：

```java
public interface TypeVariable<D extends GenericDeclaration> extends Type, AnnotatedElement {
    //获得泛型的类型(Type)上限数组，若未明确声明上边界则默认为Object
    Type[] getBounds();
    //获取声明该类型变量实体(即获得类、方法或构造器名)
    D getGenericDeclaration();
    //获得泛型参数的字面量名称，即K、V、E之类名称
    String getName();
    //获得泛型的注解类型(AnnotatedType)上限数组，若未明确声明上则为长度为0的空数组
    AnnotatedType[] getAnnotatedBounds();
}
```

## 源码

```java
package java.lang.reflect;

public interface GenericDeclaration extends AnnotatedElement {

    public TypeVariable<?>[] getTypeParameters();

}
```

 

