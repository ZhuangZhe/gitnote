# 类实例化和类加载

每个编写的`.java`拓展名类文件都存储着需要执行的程序逻辑，这些`.java`文件经过Java编译器编译成拓展名为`.class`的文件，`.class`文件中保存着Java代码经转换后的虚拟机指令，当需要使用某个类时，虚拟机将会加载它的`.class`文件，并创建对应的`class`对象，将`class`文件加载到虚拟机的内存，这个过程称为类加载：

* `加载(Loading)` → `验证(Verification)` → `准备(Preparation)` → `解析(Resolution)` → `初始化(initialization)`
* `验证`、`准备`和`解析`合称`链接(Linking)`
* `加载`：通过一个类的完全限定查找此类字节码文件，并利用字节码文件创建一个`class`对象。
* `验证`：
  * 目的在于确保`class`文件的字节流中包含信息符合当前虚拟机要求。
  * 主要包括四种验证：
    * `文件格式验证`
    * `元数据验证`
    * `字节码验证`
    * `符号引用验证`
* `准备`：为类变量\(即static修饰的字段变量\)分配内存并且设置该类变量的初始值即0\(如static int i=5;这里只将i初始化为0，至于5的值将在初始化时赋值\)，这里不包含用final修饰的static，因为final在编译的时候就会分配了，注意这里不会为实例变量分配初始化，类变量会分配在方法区中，而实例变量是会随着对象一起分配到Java堆中。
* `解析`：
  * 主要将常量池中的符号引用替换为直接引用的过程。
    * 符号引用就是一组符号来描述目标，可以是任何字面量。
    * 直接引用就是直接指向目标的指针、相对偏移量或一个间接定位到目标的句柄。
  * 有类或接口的解析，字段解析，类方法解析，接口方法解析。
* `初始化`
  * 类加载最后阶段，若该类具有超类，则对其进行初始化，执行静态初始化器和静态初始化成员变量\(如前面只初始化了默认值的static变量将会在这个阶段赋值，成员变量也将被初始化\)。

类加载器的任务是根据一个类的全限定名来读取此类的二进制字节流到JVM中，然后转换为一个与目标类对应的`java.lang.Class`对象实例。

虚拟机提供了3种类加载器：

* `启动(Bootstrap)类加载器`：负责将`<JAVA_HOME>/lib`路径下的核心类库或`-Xbootclasspath`参数指定的路径下的jar包加载到内存中，注意必由于虚拟机是按照文件名识别加载jar包的，如rt.jar，如果文件名不被虚拟机识别，即使把jar包丢到`lib`目录下也是没有作用的。启动类加载器只加载包名为`java`、`javax`、`sun`等开头的类。
* `扩展(Extension)类加载器`：负责加载`<JAVA_HOME>/lib/ext`目录下或者由系统变量`-Djava.ext.dir`指定位路径中的类库，开发者可以直接使用标准扩展类加载器。
* `系统(System)类加载器`：负责加载系统类路径`java -classpath`或`-D java.class.path`指定路径下的类库，也就是我们经常用到的classpath路径，开发者可以直接使用系统类加载器，一般情况下该类加载是程序中默认的类加载器。

## 双亲委派模式

### 双亲委派模式工作原理

除了顶层的`启动类加载器`外，其余的类加载器都应当有自己的父类加载器。请注意双亲委派模式中的父子关系是采用组合关系来复用父类加载器的相关代码。

类加载器间的关系：`启动类加载器` ← `扩展类加载器` ← `系统类加载器` ← `自定义类加载器`

原理：一个类加载器收到了类加载请求，先把这个请求委托给父类的加载器去执行，如果父类加载器还存在其父类加载器，则进一步向上委托，依次递归，请求最终将到达顶层的启动类加载器，如果父类加载器可以完成类加载任务，就成功返回，倘若父类加载器无法完成此加载任务，子加载器才会尝试自己去加载。

类加载器基于`ClassLoader`抽象类，其中几个重要的方法：

* `loadClass(String)`
  * 该方法加载指定全限定名的二进制类型，不建议用户重写但用户可以直接调用该方法。
  * 当类加载请求到来时，先从缓存中查找该类对象，如果存在直接返回，如果不存在则交给该类加载去的父加载器去加载，倘若没有父加载则交给顶级启动类加载器去加载，最后倘若仍没有找到，则使用`findClass()`方法去加载。
* `findClass(String)`
  * 建议把自定义的类加载逻辑写在`findClass()`方法中，从前面的分析可知，`findClass()`方法是在`loadClass()`方法中被调用的，当`loadClass()`方法中父加载器加载失败后，则会调用自己的`findClass()`方法来完成类加载，这样就可以保证自定义的类加载器也符合双亲委托模式。
* `defineClass(byte[] b, int off, int len)`
  * 用来将byte字节流解析成JVM能够识别的`class`对象
* `resolveClass(Class≺?≻ c)`
  * 使用该方法可以使用类的Class对象创建完成也同时被解析。

### 双亲委派模式优势

Java类随着它的类加载器一起具备了一种带有优先级的层次关系，通过这种层级关可以避免类的重复加载，当父亲已经加载了该类时，就没有必要子类加载器再加载一次。

Java核心api中定义类型不会被随意替换，假设通过网络传递一个名为`java.lang.Integer`的类，通过双亲委托模式传递到启动类加载器，而启动类加载器在核心Java API发现这个名字的类，发现该类已被加载，并不会重新加载网络传递的过来的`java.lang.Integer`，而直接返回已加载过的`Integer.class`，这样便可以防止核心API库被随意篡改。



**参考资料：**

* \*\*\*\*[**深入分析Java反射\(五\)-类实例化和类加载**](https://www.cnblogs.com/throwable/p/12272269.html)\*\*\*\*



