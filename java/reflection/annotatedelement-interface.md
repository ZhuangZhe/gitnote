# AnnotatedElement接口

定义的方法主要和注解操作相关，例如用于判断注解的存在性和获取注解等等。

| 方法 | 功能 |
| :--- | :--- |
| `boolean isAnnotationPresent(Class<? extends Annotation> annotationClass)` | 判断指定的注解类型在当前的实例上是否存在 |
| `<T extends Annotation> T getAnnotation(Class<T> annotationClass)` | 获取当前实例上指定注解类型的注解实例，不存在时返回null |
| `Annotation[] getAnnotations()` | 获取当前实例上所有注解实例，包括继承获得的注解，不存在则返回长度为0的数组 |
| `<T extends Annotation> T getDeclaredAnnotation(Class<T> annotationClass)` | 获取当前实例上指定注解类型的注解实例，不包括继承获得的注解，不存在则返回长度为0的数组 |
| `<T extends Annotation> T[] getDeclaredAnnotations(Class<T> annotationClass)` | 获取当前实例上所有的注解实例，不包括继承获得的注解，不存在则返回长度为0的数组 |
| `<T extends Annotation>` T\[\] getDeclaredAnnotationsByType\(`Class<T>` annotationClass\) | 在不使用@Repeatable的时候，功能和getDeclaredAnnotations方法一致，如果使用了@Repeatable，则合并解析@Repeatable后的结果 |
| `<T extends Annotation>` T\[\] getAnnotationsByType\(`Class<T>` annotationClass\) | 如果指定annotationClass注解类型可继承\(使用了@Inherited\)，那么递归调用getDeclaredAnnotationsByType |





## 源码

```java
package java.lang.reflect;

import java.lang.annotation.Annotation;
import java.lang.annotation.AnnotationFormatError;
import java.lang.annotation.Repeatable;
import java.util.Arrays;
import java.util.LinkedHashMap;
import java.util.Map;
import java.util.Objects;
import java.util.function.Function;
import java.util.stream.Collectors;
import sun.reflect.annotation.AnnotationSupport;
import sun.reflect.annotation.AnnotationType;

public interface AnnotatedElement {
    
    default boolean isAnnotationPresent(Class<? extends Annotation> annotationClass) {
        return getAnnotation(annotationClass) != null;
    }
    
    <T extends Annotation> T getAnnotation(Class<T> annotationClass);
    
    Annotation[] getAnnotations();
    
    default <T extends Annotation> T[] getAnnotationsByType(Class<T> annotationClass) {

         T[] result = getDeclaredAnnotationsByType(annotationClass);

         if (result.length == 0 && this instanceof Class &&
             AnnotationType.getInstance(annotationClass).isInherited()) { 
             
             Class<?> superClass = ((Class<?>) this).getSuperclass();
             if (superClass != null) {
                 result = superClass.getAnnotationsByType(annotationClass);
             }
         }

         return result;
     }
     
     default <T extends Annotation> T getDeclaredAnnotation(Class<T> annotationClass) {
         Objects.requireNonNull(annotationClass);
         for (Annotation annotation : getDeclaredAnnotations()) {
             if (annotationClass.equals(annotation.annotationType())) {
                 return annotationClass.cast(annotation);
             }
         }
         return null;
     }
     
     default <T extends Annotation> T[] getDeclaredAnnotationsByType(Class<T> annotationClass) {
        Objects.requireNonNull(annotationClass);
        return AnnotationSupport.
            getDirectlyAndIndirectlyPresent(Arrays.stream(getDeclaredAnnotations()).
                                            collect(Collectors.toMap(Annotation::annotationType,
                                                                     Function.identity(),
                                                                     (
                                                                         (first,second) -> first),
                                                                         LinkedHashMap::new)
                                                                     ),
                                                                     annotationClass);
    }
    
    Annotation[] getDeclaredAnnotations();
}
```

