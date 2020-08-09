# Proxy类



源码

```java
package java.lang.reflect;

import java.lang.module.ModuleDescriptor;
import java.security.AccessController;
import java.security.PrivilegedAction;
import java.util.Arrays;
import java.util.Collections;
import java.util.HashMap;
import java.util.HashSet;
import java.util.IdentityHashMap;
import java.util.List;
import java.util.Map;
import java.util.Objects;
import java.util.Set;
import java.util.concurrent.atomic.AtomicInteger;
import java.util.concurrent.atomic.AtomicLong;
import java.util.stream.Collectors;
import java.util.stream.Stream;

import jdk.internal.loader.BootLoader;
import jdk.internal.module.Modules;
import jdk.internal.misc.Unsafe;
import jdk.internal.misc.VM;
import jdk.internal.reflect.CallerSensitive;
import jdk.internal.reflect.Reflection;
import jdk.internal.loader.ClassLoaderValue;
import sun.reflect.misc.ReflectUtil;
import sun.security.action.GetPropertyAction;
import sun.security.util.SecurityConstants;

import static java.lang.module.ModuleDescriptor.Modifier.SYNTHETIC;

public class Proxy implements java.io.Serializable {

    private static final long serialVersionUID = -2222568056686623797L;
    
    private static final Class<?>[] constructorParams = { InvocationHandler.class };
    
    private static final ClassLoaderValue<Constructor<?>> proxyCache = new ClassLoaderValue<>();

    protected InvocationHandler h;
    
    private Proxy() {}
    
    protected Proxy(InvocationHandler h) {
        Objects.requireNonNull(h);
        this.h = h;
    }
    
    @Deprecated
    @CallerSensitive
    public static Class<?> getProxyClass(ClassLoader loader,
                                         Class<?>... interfaces)
        throws IllegalArgumentException
    {
        Class<?> caller = System.getSecurityManager() == null
                              ? null
                              : Reflection.getCallerClass();

        return getProxyConstructor(caller, loader, interfaces)
            .getDeclaringClass();
    }
    
    private static Constructor<?> getProxyConstructor(Class<?> caller,
                                                      ClassLoader loader,
                                                      Class<?>... interfaces)
    {
        // optimization for single interface
        if (interfaces.length == 1) {
            Class<?> intf = interfaces[0];
            if (caller != null) {
                checkProxyAccess(caller, loader, intf);
            }
            return proxyCache.sub(intf).computeIfAbsent(
                loader,
                (ld, clv) -> new ProxyBuilder(ld, clv.key()).build()
            );
        } else {
            final Class<?>[] intfsArray = interfaces.clone();
            if (caller != null) {
                checkProxyAccess(caller, loader, intfsArray);
            }
            final List<Class<?>> intfs = Arrays.asList(intfsArray);
            return proxyCache.sub(intfs).computeIfAbsent(
                loader,
                (ld, clv) -> new ProxyBuilder(ld, clv.key()).build()
            );
        }
    }
    
    private static void checkProxyAccess(Class<?> caller,
                                         ClassLoader loader,
                                         Class<?> ... interfaces)
    {
        SecurityManager sm = System.getSecurityManager();
        if (sm != null) {
            ClassLoader ccl = caller.getClassLoader();
            if (loader == null && ccl != null) {
                sm.checkPermission(SecurityConstants.GET_CLASSLOADER_PERMISSION);
            }
            ReflectUtil.checkProxyPackageAccess(ccl, interfaces);
        }
    }
    
    // private static final class ProxyBuilder { ... }
    
    @CallerSensitive
    public static Object newProxyInstance(ClassLoader loader,
                                          Class<?>[] interfaces,
                                          InvocationHandler h) {
        Objects.requireNonNull(h);

        final Class<?> caller = System.getSecurityManager() == null
                                    ? null
                                    : Reflection.getCallerClass();

        Constructor<?> cons = getProxyConstructor(caller, loader, interfaces);

        return newProxyInstance(caller, cons, h);
    }

    private static Object newProxyInstance(Class<?> caller, // null if no SecurityManager
                                           Constructor<?> cons,
                                           InvocationHandler h) {

        try {
            if (caller != null) {
                checkNewProxyPermission(caller, cons.getDeclaringClass());
            }

            return cons.newInstance(new Object[]{h});
        } catch (IllegalAccessException | InstantiationException e) {
            throw new InternalError(e.toString(), e);
        } catch (InvocationTargetException e) {
            Throwable t = e.getCause();
            if (t instanceof RuntimeException) {
                throw (RuntimeException) t;
            } else {
                throw new InternalError(t.toString(), t);
            }
        }
    }
    
    private static void checkNewProxyPermission(Class<?> caller, Class<?> proxyClass) {
        SecurityManager sm = System.getSecurityManager();
        if (sm != null) {
            if (ReflectUtil.isNonPublicProxyClass(proxyClass)) {
                ClassLoader ccl = caller.getClassLoader();
                ClassLoader pcl = proxyClass.getClassLoader();

                // do permission check if the caller is in a different runtime package
                // of the proxy class
                String pkg = proxyClass.getPackageName();
                String callerPkg = caller.getPackageName();

                if (pcl != ccl || !pkg.equals(callerPkg)) {
                    sm.checkPermission(new ReflectPermission("newProxyInPackage." + pkg));
                }
            }
        }
    }
    
    private static ClassLoader getLoader(Module m) {
        PrivilegedAction<ClassLoader> pa = m::getClassLoader;
        return AccessController.doPrivileged(pa);
    }
    
    public static boolean isProxyClass(Class<?> cl) {
        return Proxy.class.isAssignableFrom(cl) && ProxyBuilder.isProxyClass(cl);
    }
    
    @CallerSensitive
    public static InvocationHandler getInvocationHandler(Object proxy)
        throws IllegalArgumentException
    {
        /*
         * Verify that the object is actually a proxy instance.
         */
        if (!isProxyClass(proxy.getClass())) {
            throw new IllegalArgumentException("not a proxy instance");
        }

        final Proxy p = (Proxy) proxy;
        final InvocationHandler ih = p.h;
        if (System.getSecurityManager() != null) {
            Class<?> ihClass = ih.getClass();
            Class<?> caller = Reflection.getCallerClass();
            if (ReflectUtil.needsPackageAccessCheck(caller.getClassLoader(),
                                                    ihClass.getClassLoader()))
            {
                ReflectUtil.checkPackageAccess(ihClass);
            }
        }

        return ih;
    }

    private static final Class<?>[] EMPTY_CLASS_ARRAY = new Class<?>[0];
    private static final String PROXY_PACKAGE_PREFIX = ReflectUtil.PROXY_PACKAGE;
    
}
```

### ProxyBuilder

```java
private static final class ProxyBuilder {
    private static final Unsafe UNSAFE = Unsafe.getUnsafe();

    private static final String proxyClassNamePrefix = "$Proxy";

    private static final AtomicLong nextUniqueNumber = new AtomicLong();

    private static final ClassLoaderValue<Boolean> reverseProxyCache =
        new ClassLoaderValue<>();

    private static Class<?> defineProxyClass(Module m, List<Class<?>> interfaces) {
        String proxyPkg = null;     // package to define proxy class in
        int accessFlags = Modifier.PUBLIC | Modifier.FINAL;

        for (Class<?> intf : interfaces) {
            int flags = intf.getModifiers();
            if (!Modifier.isPublic(flags)) {
                accessFlags = Modifier.FINAL;  // non-public, final
                String pkg = intf.getPackageName();
                if (proxyPkg == null) {
                    proxyPkg = pkg;
                } else if (!pkg.equals(proxyPkg)) {
                    throw new IllegalArgumentException(
                            "non-public interfaces from different packages");
                }
            }
        }

        if (proxyPkg == null) {
            proxyPkg = m.isNamed() ? PROXY_PACKAGE_PREFIX + "." + m.getName()
                                   : PROXY_PACKAGE_PREFIX;
        } else if (proxyPkg.isEmpty() && m.isNamed()) {
            throw new IllegalArgumentException(
                    "Unnamed package cannot be added to " + m);
        }

        if (m.isNamed()) {
            if (!m.getDescriptor().packages().contains(proxyPkg)) {
                throw new InternalError(proxyPkg + " not exist in " + m.getName());
            }
        }

        long num = nextUniqueNumber.getAndIncrement();
        String proxyName = proxyPkg.isEmpty()
                                ? proxyClassNamePrefix + num
                                : proxyPkg + "." + proxyClassNamePrefix + num;

        ClassLoader loader = getLoader(m);
        trace(proxyName, m, loader, interfaces);

        byte[] proxyClassFile = ProxyGenerator.generateProxyClass(
                proxyName, interfaces.toArray(EMPTY_CLASS_ARRAY), accessFlags);
        try {
            Class<?> pc = UNSAFE.defineClass(proxyName, proxyClassFile,
                                             0, proxyClassFile.length,
                                             loader, null);
            reverseProxyCache.sub(pc).putIfAbsent(loader, Boolean.TRUE);
            return pc;
        } catch (ClassFormatError e) {
            throw new IllegalArgumentException(e.toString());
        }
    }
    
    static boolean isProxyClass(Class<?> c) {
        return Objects.equals(reverseProxyCache.sub(c).get(c.getClassLoader()),
                              Boolean.TRUE);
    }

    private static boolean isExportedType(Class<?> c) {
        String pn = c.getPackageName();
        return Modifier.isPublic(c.getModifiers()) && c.getModule().isExported(pn);
    }

    private static boolean isPackagePrivateType(Class<?> c) {
        return !Modifier.isPublic(c.getModifiers());
    }

    private static String toDetails(Class<?> c) {
        String access = "unknown";
        if (isExportedType(c)) {
            access = "exported";
        } else if (isPackagePrivateType(c)) {
            access = "package-private";
        } else {
            access = "module-private";
        }
        ClassLoader ld = c.getClassLoader();
        return String.format("   %s/%s %s loader %s",
                c.getModule().getName(), c.getName(), access, ld);
    }

    static void trace(String cn,
                      Module module,
                      ClassLoader loader,
                      List<Class<?>> interfaces) {
        if (isDebug()) {
            System.err.format("PROXY: %s/%s defined by %s%n",
                              module.getName(), cn, loader);
        }
        if (isDebug("debug")) {
            interfaces.forEach(c -> System.out.println(toDetails(c)));
        }
    }

    private static final String DEBUG =
        GetPropertyAction.privilegedGetProperty("jdk.proxy.debug", "");

    private static boolean isDebug() {
        return !DEBUG.isEmpty();
    }
    private static boolean isDebug(String flag) {
        return DEBUG.equals(flag);
    }

    private final List<Class<?>> interfaces;
    private final Module module;
    ProxyBuilder(ClassLoader loader, List<Class<?>> interfaces) {
        if (!VM.isModuleSystemInited()) {
            throw new InternalError("Proxy is not supported until "
                    + "module system is fully initialized");
        }
        if (interfaces.size() > 65535) {
            throw new IllegalArgumentException("interface limit exceeded: "
                    + interfaces.size());
        }

        Set<Class<?>> refTypes = referencedTypes(loader, interfaces);

        validateProxyInterfaces(loader, interfaces, refTypes);

        this.interfaces = interfaces;
        this.module = mapToModule(loader, interfaces, refTypes);
        assert getLoader(module) == loader;
    }

    ProxyBuilder(ClassLoader loader, Class<?> intf) {
        this(loader, Collections.singletonList(intf));
    }

    Constructor<?> build() {
        Class<?> proxyClass = defineProxyClass(module, interfaces);
        final Constructor<?> cons;
        try {
            cons = proxyClass.getConstructor(constructorParams);
        } catch (NoSuchMethodException e) {
            throw new InternalError(e.toString(), e);
        }
        AccessController.doPrivileged(new PrivilegedAction<Void>() {
            public Void run() {
                cons.setAccessible(true);
                return null;
            }
        });
        return cons;
    }

    private static void validateProxyInterfaces(ClassLoader loader,
                                                List<Class<?>> interfaces,
                                                Set<Class<?>> refTypes)
    {
        Map<Class<?>, Boolean> interfaceSet = new IdentityHashMap<>(interfaces.size());
        for (Class<?> intf : interfaces) {

            ensureVisible(loader, intf);

            if (!intf.isInterface()) {
                throw new IllegalArgumentException(intf.getName() + " is not an interface");
            }

            if (interfaceSet.put(intf, Boolean.TRUE) != null) {
                throw new IllegalArgumentException("repeated interface: " + intf.getName());
            }
        }

        for (Class<?> type : refTypes) {
            ensureVisible(loader, type);
        }
    }

    private static Set<Class<?>> referencedTypes(ClassLoader loader,
                                                 List<Class<?>> interfaces) {
        var types = new HashSet<Class<?>>();
        for (var intf : interfaces) {
            for (Method m : intf.getMethods()) {
                if (!Modifier.isStatic(m.getModifiers())) {
                    addElementType(types, m.getReturnType());
                    addElementTypes(types, m.getSharedParameterTypes());
                    addElementTypes(types, m.getSharedExceptionTypes());
                }
            }
        }
        return types;
    }

    private static void addElementTypes(HashSet<Class<?>> types,
                                        Class<?> ... classes) {
        for (var cls : classes) {
            addElementType(types, cls);
        }
    }

    private static void addElementType(HashSet<Class<?>> types,
                                       Class<?> cls) {
        var type = getElementType(cls);
        if (!type.isPrimitive()) {
            types.add(type);
        }
    }

    private static Module mapToModule(ClassLoader loader,
                                      List<Class<?>> interfaces,
                                      Set<Class<?>> refTypes) {
        Map<Class<?>, Module> modulePrivateTypes = new HashMap<>();
        Map<Class<?>, Module> packagePrivateTypes = new HashMap<>();
        for (Class<?> intf : interfaces) {
            Module m = intf.getModule();
            if (Modifier.isPublic(intf.getModifiers())) {
                // module-private types
                if (!m.isExported(intf.getPackageName())) {
                    modulePrivateTypes.put(intf, m);
                }
            } else {
                packagePrivateTypes.put(intf, m);
            }
        }

        if (packagePrivateTypes.isEmpty() && modulePrivateTypes.isEmpty()) {
            return loader != null ? loader.getUnnamedModule()
                                  : BootLoader.getUnnamedModule();
        }

        if (packagePrivateTypes.size() > 0) {

            if (packagePrivateTypes.size() > 1 &&
                    (packagePrivateTypes.keySet().stream()  // more than one package
                             .map(Class::getPackageName).distinct().count() > 1 ||
                     packagePrivateTypes.values().stream()  // or more than one module
                             .distinct().count() > 1)) {
                throw new IllegalArgumentException(
                        "non-public interfaces from different packages");
            }

            Module target = null;
            for (Module m : packagePrivateTypes.values()) {
                if (getLoader(m) != loader) {
                    throw new IllegalArgumentException(
                            "non-public interface is not defined by the given loader");
                }
                target = m;
            }

            for (Class<?> intf : interfaces) {
                Module m = intf.getModule();
                if (m == target) continue;

                if (!target.canRead(m) || !m.isExported(intf.getPackageName(), target)) {
                    throw new IllegalArgumentException(target + " can't access " + intf.getName());
                }
            }
            
            return target;
        }

        Module target = getDynamicModule(loader);

        Set<Class<?>> types = new HashSet<>(interfaces);
        types.addAll(refTypes);
        for (Class<?> c : types) {
            ensureAccess(target, c);
        }
        return target;
    }

    private static void ensureAccess(Module target, Class<?> c) {
        Module m = c.getModule();
        if (!target.canRead(m)) {
            Modules.addReads(target, m);
        }
        String pn = c.getPackageName();
        if (!m.isExported(pn, target)) {
            Modules.addExports(m, pn, target);
        }
    }

    private static void ensureVisible(ClassLoader ld, Class<?> c) {
        Class<?> type = null;
        try {
            type = Class.forName(c.getName(), false, ld);
        } catch (ClassNotFoundException e) {
        }
        if (type != c) {
            throw new IllegalArgumentException(c.getName() +
                    " referenced from a method is not visible from class loader");
        }
    }

    private static Class<?> getElementType(Class<?> type) {
        Class<?> e = type;
        while (e.isArray()) {
            e = e.getComponentType();
        }
        return e;
    }

    private static final ClassLoaderValue<Module> dynProxyModules =
        new ClassLoaderValue<>();
    private static final AtomicInteger counter = new AtomicInteger();

    private static Module getDynamicModule(ClassLoader loader) {
        return dynProxyModules.computeIfAbsent(loader, (ld, clv) -> {
            // create a dynamic module and setup module access
        String mn = "jdk.proxy" + counter.incrementAndGet();
        String pn = PROXY_PACKAGE_PREFIX + "." + mn;
        ModuleDescriptor descriptor =
            ModuleDescriptor.newModule(mn, Set.of(SYNTHETIC))
                            .packages(Set.of(pn))
                            .build();
        Module m = Modules.defineModule(ld, descriptor, null);
        Modules.addReads(m, Proxy.class.getModule());
        // java.base to create proxy instance
        Modules.addExports(m, pn, Object.class.getModule());
        return m;
        });
    }
}
```

