# AccessibleObject类



## 源码

```java
package java.lang.reflect;

import java.lang.annotation.Annotation;
import java.lang.invoke.MethodHandle;
import java.security.AccessController;

import jdk.internal.misc.VM;
import jdk.internal.module.IllegalAccessLogger;
import jdk.internal.reflect.CallerSensitive;
import jdk.internal.reflect.Reflection;
import jdk.internal.reflect.ReflectionFactory;
import sun.security.action.GetPropertyAction;
import sun.security.util.SecurityConstants;

public class AccessibleObject implements AnnotatedElement {

    static void checkPermission() {
        SecurityManager sm = System.getSecurityManager();
        if (sm != null) {
            sm.checkPermission(SecurityConstants.ACCESS_PERMISSION);
        }
    }

    @CallerSensitive
    public static void setAccessible(AccessibleObject[] array, boolean flag) {
        checkPermission();
        if (flag) {
            Class<?> caller = Reflection.getCallerClass();
            array = array.clone();
            for (AccessibleObject ao : array) {
                ao.checkCanSetAccessible(caller);
            }
        }
        for (AccessibleObject ao : array) {
            ao.setAccessible0(flag);
        }
    }

    @CallerSensitive
    public void setAccessible(boolean flag) {
        AccessibleObject.checkPermission();
        setAccessible0(flag);
    }

    boolean setAccessible0(boolean flag) {
        this.override = flag;
        return flag;
    }

    @CallerSensitive
    public final boolean trySetAccessible() {
        AccessibleObject.checkPermission();

        if (override == true) return true;

        if (!Member.class.isInstance(this)) {
            return setAccessible0(true);
        }

        Class<?> declaringClass = ((Member) this).getDeclaringClass();
        if (declaringClass == Class.class && this instanceof Constructor) {
            return false;
        }

        if (checkCanSetAccessible(Reflection.getCallerClass(),
                                  declaringClass,
                                  false)) {
            return setAccessible0(true);
        } else {
            return false;
        }
    }

    void checkCanSetAccessible(Class<?> caller) {
        // do nothing, needs to be overridden by Constructor, Method, Field
    }

    final void checkCanSetAccessible(Class<?> caller, Class<?> declaringClass) {
        checkCanSetAccessible(caller, declaringClass, true);
    }

    private boolean checkCanSetAccessible(Class<?> caller,
                                          Class<?> declaringClass,
                                          boolean throwExceptionIfDenied) {
        if (caller == MethodHandle.class) {
            throw new IllegalCallerException();   // should not happen
        }

        Module callerModule = caller.getModule();
        Module declaringModule = declaringClass.getModule();

        if (callerModule == declaringModule) return true;
        if (callerModule == Object.class.getModule()) return true;
        if (!declaringModule.isNamed()) return true;

        String pn = declaringClass.getPackageName();
        int modifiers;
        if (this instanceof Executable) {
            modifiers = ((Executable) this).getModifiers();
        } else {
            modifiers = ((Field) this).getModifiers();
        }

        // class is public and package is exported to caller
        boolean isClassPublic = Modifier.isPublic(declaringClass.getModifiers());
        if (isClassPublic && declaringModule.isExported(pn, callerModule)) {
            // member is public
            if (Modifier.isPublic(modifiers)) {
                logIfExportedForIllegalAccess(caller, declaringClass);
                return true;
            }

            // member is protected-static
            if (Modifier.isProtected(modifiers)
                && Modifier.isStatic(modifiers)
                && isSubclassOf(caller, declaringClass)) {
                logIfExportedForIllegalAccess(caller, declaringClass);
                return true;
            }
        }

        if (declaringModule.isOpen(pn, callerModule)) {
            logIfOpenedForIllegalAccess(caller, declaringClass);
            return true;
        }

        if (throwExceptionIfDenied) {
            // not accessible
            String msg = "Unable to make ";
            if (this instanceof Field)
                msg += "field ";
            msg += this + " accessible: " + declaringModule + " does not \"";
            if (isClassPublic && Modifier.isPublic(modifiers))
                msg += "exports";
            else
                msg += "opens";
            msg += " " + pn + "\" to " + callerModule;
            InaccessibleObjectException e = new InaccessibleObjectException(msg);
            if (printStackTraceWhenAccessFails()) {
                e.printStackTrace(System.err);
            }
            throw e;
        }
        return false;
    }

    private boolean isSubclassOf(Class<?> queryClass, Class<?> ofClass) {
        while (queryClass != null) {
            if (queryClass == ofClass) {
                return true;
            }
            queryClass = queryClass.getSuperclass();
        }
        return false;
    }

    private void logIfOpenedForIllegalAccess(Class<?> caller, Class<?> declaringClass) {
        Module callerModule = caller.getModule();
        Module targetModule = declaringClass.getModule();
        // callerModule is null during early startup
        if (callerModule != null && !callerModule.isNamed() && targetModule.isNamed()) {
            IllegalAccessLogger logger = IllegalAccessLogger.illegalAccessLogger();
            if (logger != null) {
                logger.logIfOpenedForIllegalAccess(caller, declaringClass, this::toShortString);
            }
        }
    }

    private void logIfExportedForIllegalAccess(Class<?> caller, Class<?> declaringClass) {
        Module callerModule = caller.getModule();
        Module targetModule = declaringClass.getModule();
        // callerModule is null during early startup
        if (callerModule != null && !callerModule.isNamed() && targetModule.isNamed()) {
            IllegalAccessLogger logger = IllegalAccessLogger.illegalAccessLogger();
            if (logger != null) {
                logger.logIfExportedForIllegalAccess(caller, declaringClass, this::toShortString);
            }
        }
    }

    String toShortString() {
        return toString();
    }

    @Deprecated(since="9")
    public boolean isAccessible() {
        return override;
    }

    @CallerSensitive
    public final boolean canAccess(Object obj) {
        if (!Member.class.isInstance(this)) {
            return override;
        }

        Class<?> declaringClass = ((Member) this).getDeclaringClass();
        int modifiers = ((Member) this).getModifiers();
        if (!Modifier.isStatic(modifiers) &&
                (this instanceof Method || this instanceof Field)) {
            if (obj == null) {
                throw new IllegalArgumentException("null object for " + this);
            }

            if (!declaringClass.isAssignableFrom(obj.getClass())) {
                throw new IllegalArgumentException("object is not an instance of "
                                                   + declaringClass.getName());
            }
        } else if (obj != null) {
            throw new IllegalArgumentException("non-null object for " + this);
        }

        if (override) return true;

        Class<?> caller = Reflection.getCallerClass();
        Class<?> targetClass;
        if (this instanceof Constructor) {
            targetClass = declaringClass;
        } else {
            targetClass = Modifier.isStatic(modifiers) ? null : obj.getClass();
        }
        return verifyAccess(caller, declaringClass, targetClass, modifiers);
    }

    protected AccessibleObject() {}

    boolean override;

    static final ReflectionFactory reflectionFactory =
        AccessController.doPrivileged(new ReflectionFactory.GetReflectionFactoryAction());


    public <T extends Annotation> T getAnnotation(Class<T> annotationClass) {
        throw new AssertionError("All subclasses should override this method");
    }

    @Override
    public boolean isAnnotationPresent(Class<? extends Annotation> annotationClass) {
        return AnnotatedElement.super.isAnnotationPresent(annotationClass);
    }

    @Override
    public <T extends Annotation> T[] getAnnotationsByType(Class<T> annotationClass) {
        throw new AssertionError("All subclasses should override this method");
    }

    public Annotation[] getAnnotations() {
        return getDeclaredAnnotations();
    }

    @Override
    public <T extends Annotation> T getDeclaredAnnotation(Class<T> annotationClass) {
        return getAnnotation(annotationClass);
    }
    
    @Override
    public <T extends Annotation> T[] getDeclaredAnnotationsByType(Class<T> annotationClass) {
        return getAnnotationsByType(annotationClass);
    }

    public Annotation[] getDeclaredAnnotations()  {
        throw new AssertionError("All subclasses should override this method");
    }

    volatile Object securityCheckCache;

    final void checkAccess(Class<?> caller, 
                           Class<?> memberClass,
                           Class<?> targetClass, 
                           int modifiers) throws IllegalAccessException {
        if (!verifyAccess(caller, memberClass, targetClass, modifiers)) {
            IllegalAccessException e = Reflection.newIllegalAccessException(
                caller, memberClass, targetClass, modifiers);
            if (printStackTraceWhenAccessFails()) {
                e.printStackTrace(System.err);
            }
            throw e;
        }
    }

    final boolean verifyAccess(Class<?> caller, 
                               Class<?> memberClass,
                               Class<?> targetClass, 
                               int modifiers) {
        if (caller == memberClass) {  // quick check
            return true;             // ACCESS IS OK
        }
        Object cache = securityCheckCache;  // read volatile
        if (targetClass != null // instance member or constructor
            && Modifier.isProtected(modifiers)
            && targetClass != memberClass) {
            if (cache instanceof Class[]) {
                Class<?>[] cache2 = (Class<?>[]) cache;
                if (cache2[1] == targetClass &&
                    cache2[0] == caller) {
                    return true;     // ACCESS IS OK
                }
            }
        } else if (cache == caller) {
            // Non-protected case (or targetClass == memberClass or static member).
            return true;             // ACCESS IS OK
        }
        return slowVerifyAccess(caller, memberClass, targetClass, modifiers);
    }

    private boolean slowVerifyAccess(Class<?> caller, 
                                     Class<?> memberClass,
                                     Class<?> targetClass, 
                                     int modifiers) {
        if (!Reflection.verifyMemberAccess(caller, memberClass, targetClass, modifiers)) {
            return false;
        }

        logIfExportedForIllegalAccess(caller, memberClass);

        Object cache = (targetClass != null
                        && Modifier.isProtected(modifiers)
                        && targetClass != memberClass)
                        ? new Class<?>[] { caller, targetClass }
                        : caller;

        securityCheckCache = cache;         // write volatile
        return true;
    }

    private static volatile boolean printStackWhenAccessFails;

    private static volatile boolean printStackPropertiesSet;

    private static boolean printStackTraceWhenAccessFails() {
        if (!printStackPropertiesSet && VM.initLevel() >= 1) {
            String s = GetPropertyAction.privilegedGetProperty(
                    "sun.reflect.debugModuleAccessChecks");
            if (s != null) {
                printStackWhenAccessFails = !s.equalsIgnoreCase("false");
            }
            printStackPropertiesSet = true;
        }
        return printStackWhenAccessFails;
    }

    AccessibleObject getRoot() {
        throw new InternalError();
    }
}

```



