# Class类

这里先说一个规律，在Class中，`getXXX()`方法和`getDeclearedXXX()`方法有所区别。注解类型`Annotation`的操作方法例外，因为基于注解的修饰符必定是public的：

* `getDeclaredMethod(s)`：返回类或接口声明的所有方法，包括公共、保护、默认\(包\)访问和私有方法，但不包括继承的方法。对于获取`Method`对象，`Method[] methods = clazz.getDeclaredMethods();`返回的是clazz本类所有修饰符\(public、default、private、protected\)的方法数组，但是不包含继承而来的方法。
* `getMethod(s)`：返回某个类的所有公用\(public\)方法包括其继承类的公用方法，当然也包括它所实现接口的方法。对于获取Method对象，`Method[] methods = clazz.getMethods();`表示返回clazz的父类、父类接口、本类、本类接口中的全部修饰符为public的方法数组。
* `getDeclaredField(s)`和`getField(s)`、`getDeclaredConstructor(s)`和`getConstructor(s)`同上。
* `getDeclaredAnnotation(s)`：返回直接存在于此元素上的所有注解，此方法将忽略继承的注解，准确来说就是忽略@Inherited注解的作用。
* `getAnnotation(s)`：返回此元素上存在的所有注解，包括继承的所有注解。

如果想获取一个类的所有修饰符的方法，包括所有父类中的方法，那么建议递归调用`getDeclaredMethods()`\(所谓递归调用就是一直追溯目标类的父类递归调用`getDeclaredMethods()`方法直到父类为Object类型，这个思路可以参考Spring框架中的相关工具类\)。获取一个类的所有`Field`、`Constructor`也可以类似操作，可以参考或者直接使用Spring中的工具类`ReflectionUtils`的相关方法。`@Inherited`元注解是一个标记注解，`@Inherited`阐述了某个被标注的`Annotation`类型是可以被继承的，详细的在分析`AnnotatedElement`的时候再展开。



