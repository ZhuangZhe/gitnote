# Class类

这里先说一个规律，在Class中，`getXXX()`方法和`getDeclearedXXX()`方法有所区别。注解类型`Annotation`的操作方法例外，因为基于注解的修饰符必定是public的：

* `getDeclaredMethod(s)`：返回类或接口声明的所有方法，包括公共、保护、默认\(包\)访问和私有方法，但不包括继承的方法。对于获取`Method`对象，`Method[] methods = clazz.getDeclaredMethods();`返回的是clazz本类所有修饰符\(public、default、private、protected\)的方法数组，但是不包含继承而来的方法。
* `getMethod(s)`：返回某个类的所有公用\(public\)方法包括其继承类的公用方法，当然也包括它所实现接口的方法。对于获取Method对象，`Method[] methods = clazz.getMethods();`表示返回clazz的父类、父类接口、本类、本类接口中的全部修饰符为public的方法数组。
* `getDeclaredField(s)`和`getField(s)`、`getDeclaredConstructor(s)`和`getConstructor(s)`同上。
* `getDeclaredAnnotation(s)`：返回直接存在于此元素上的所有注解，此方法将忽略继承的注解，准确来说就是忽略@Inherited注解的作用。
* `getAnnotation(s)`：返回此元素上存在的所有注解，包括继承的所有注解。

如果想获取一个类的所有修饰符的方法，包括所有父类中的方法，那么建议递归调用`getDeclaredMethods()`\(所谓递归调用就是一直追溯目标类的父类递归调用`getDeclaredMethods()`方法直到父类为Object类型，这个思路可以参考Spring框架中的相关工具类\)。获取一个类的所有`Field`、`Constructor`也可以类似操作，可以参考或者直接使用Spring中的工具类`ReflectionUtils`的相关方法。`@Inherited`元注解是一个标记注解，`@Inherited`阐述了某个被标注的`Annotation`类型是可以被继承的，详细的在分析`AnnotatedElement`的时候再展开。

`Class`实现了`Serializable`、`GenericDeclaration`、`Type`、`AnnotatedElement`接口，它提供了类型判断、类型实例化、获取方法列表、获取字段列表、获取父类泛型类型等方法。下面主要介绍一下它的主要方法：

| 方法 | 功能 |
| :--- | :--- |
| `Class<?>` forName\(String className\) | 传入全类名创建Class实例 |
| T newInstance\(\) | 通过当前的Class实例进行实例化对象，返回的就是新建的对象 |
| int getModifiers\(\) | native方法，返回当前Class的修饰符 |
| String getName\(\) | 返回类名称，虚拟机中类名表示 |
| String getCanonicalName\(\) | 返回类名称，便于理解的类名表示 |
| String getSimpleName\(\) | 返回类名称，源代码中给出的底层类的简单名称 |
| Package getPackage\(\) | 返回类的包属性 |
| String getPackageName\(\) | 返回类的包路径名称 |
| String toGenericString\(\) | 返回描述此Class的字符串，其中包括类型参数的字面量 |
| `TypeVariable<Class<T>>[]` getTypeParameters\(\) | 获取类定义泛型的类型变量 |
| `Class<?>[]` getClasses\(\) | 获取所有的修饰符为public的成员Class，包括父类 |
| `Class<?>[]` getDeclaredClasses\(\) | 获取本类所有修饰符的成员Class，不包括父类 |
| `Constructor<?>[]` getConstructors\(\) | 获取所有的修饰符为public的构造器，包括父类 |
| `Constructor<T>` getConstructor\(`Class<?>... parameterTypes`\) | 获取参数类型匹配的修饰符为public的构造器，包括父类 |
| `Constructor<?>[]` getDeclaredConstructors\(\) | 获取本类所有修饰符的构造器，不包括父类 |
| `Constructor<T>[]` getDeclaredConstructor\(`Class<?>... parameterTypes`\) | 获取本类参数类型匹配的所有修饰符的构造器，不包括父类 |
| Method\[\] getMethods\(\) | 获取本类所有的修饰符为public的方法列表，包括父类 |
| Method\[\] getDeclaredMethods\(\) | 获取本类所有修饰符的方法列表，不包括父类 |
| Method getMethod\(String name, `Class<?>... parameterTypes`\) | 通过指定方法名和参数类型获取本类修饰符为public的方法，包括父类 |
| Method getDeclaredMethod\(String name, `Class<?>... parameterTypes`\) | 通过指定方法名和参数类型获取本类不限修饰符的方法，不包括父类 |
| Field\[\] getFields\(\) | 获取本类所有的修饰符为public的属性列表，包括父类 |
| Field\[\] getDeclaredFields\(\) | 获取本类所有修饰符的属性列表，不包括父类 |
| Field getField\(String name\) | 通过指定属性名名获取本类修饰符为public的属性，包括父类 |
| Field getDeclaredField\(String name\) | 通过指定属性名获取本类不限修饰符的属性，不包括父类 |
| `Class<?>[] getInterfaces()` | 获取类实现的所有接口的Class数组 |
| Type\[\] getGenericInterfaces\(\) | 获取类实现的所有泛型参数接口的Type数组 |
| `Class<? super T>` getSuperclass\(\) | 获取当前类的父类的Class，如果当前类是Object、接口、基本数据类型\(primitive\)或者void，则返回null |
| Type getGenericSuperclass\(\) | 获取当前类的泛型参数父类的Type，如果当前类是Object、接口、基本数据类型\(primitive\)或者void，则返回null |
| native boolean isInstance\(Object obj\) | 判断传入的object是否当前类的实例 |
| native boolean isAssignableFrom\(`Class<?> cls`\) | 判断传入的Class对象是否和当前类相同，或者是否当前类的超类或超接口 |
| native boolean isInterface\(\) | 判断当前类是否接口 |
| native boolean isArray\(\) | 判断当前类是否数组 |
| native boolean isPrimitive\(\) | 判断当前类是否基本数据类型 |
| boolean isAnnotation\(\) | 判断当前类是否注解类型 |
| boolean isSynthetic\(\) | 判断当前类是否复合 |
| native `Class<?>` getComponentType\(\) | 如果当前类是数组，返回数组元素的类型 |
| `Class<?>` getEnclosingClass\(\) | 返回一个类，当前类\(一般是成员类\)在这个类\(封闭类，相对于内部类的外部类或者说外面一层\)中定义 |
| `Constructor<?>` getEnclosingConstructor\(\) | 返回构造器，当前类是在这个构造函数中定义 |
| Method getEnclosingMethod\(\) | 返回方法，当前类是在这个方法中定义 |
| Module getModule\(\) | 返回模块，JDK9新增方法 |







