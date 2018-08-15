

# CLASSLOADER详解

### 一、Java类加载流程

#### 1）java自带3个类加载器

1. **Bootstrap ClassLoader**：最顶层的加载类，主要加载核心类库，**<u>%JRE_HOME%/lib</u>**下的rt.jar、resources.jar、charsets.jar和class等。可以通过启动jvm时指定-Xbootclasspath和路径来改变Bootstrap ClassLoader的加载目录。

2. **Extention ClassLoader**: 扩展类的加载器，加载目录**<u>%JRE_HOME%\lib\ext</u>**目录下的jar包和class文件。另外还可以加载`-D java.ext.dirs`选项指定的目录。 

3. **Appclass Loader**: **也称为SystemAppClass** 加载当前应用的classpath的所有类。

    ![image-20180521151321140](/var/folders/zs/d6lr89wj1l3_v89_d8jg68r40000gn/T/abnerworks.Typora/image-20180521151321140.png)

#### 2）3个类加载器的加载顺序

1. Bootstrap Classloader
2. Extention Classloader
3. AppClassLoader

为了理解，我们可以看下Launcher的源码，有精简

```java

public class Launcher {
    private static URLStreamHandlerFactory factory = new Launcher.Factory();
    private static Launcher launcher = new Launcher();
    private static String bootClassPath = System.getProperty("sun.boot.class.path");
    private ClassLoader loader;
    private static URLStreamHandler fileHandler;

    public static Launcher getLauncher() {
        return launcher;
    }

    public Launcher() {
        Launcher.ExtClassLoader var1;
        try {
            // Create the extension class loader
            var1 = Launcher.ExtClassLoader.getExtClassLoader();
        } catch (IOException var10) {
            throw new InternalError("Could not create extension class loader", var10);
        }
        try {
            // Now create the class loader to use to launch the application
            this.loader = Launcher.AppClassLoader.getAppClassLoader(var1);
        } catch (IOException var9) {
            throw new InternalError("Could not create application class loader", var9);
        }
    }
	/*
     * Returns the class loader used to launch the main application.
     */
    public ClassLoader getClassLoader() {
        return this.loader;
    }

    public static URLClassPath getBootstrapClassPath() {
        return Launcher.BootClassPathHolder.bcp;
    }
     /**
     * The class loader used for loading from java.class.path.
     * runs in a restricted security context.
     */
    static class AppClassLoader extends URLClassLoader {}
    /*
     * The class loader used for loading installed extensions.
     */
    static class ExtClassLoader extends URLClassLoader {}
```

我们从源码中可以看到：

1. Launcher初始化了ExtClassLoader和AppClassLoader
2. Launcher中并没有看见BootstrapClassLoader，但通过`System.getProperty("sun.boot.class.path")`得到了字符串`bootClassPath`,这个应该就是BootstrapClassLoader加载的jar包路径。

而 `sun.boot.class.path`是JRE目录下的jar包或者class文件的目录。

#### 3）ExtentionClassLoader源码

```Java
  static class ExtClassLoader extends URLClassLoader {
      
      /**
         * create an ExtClassLoader. The ExtClassLoader is created
         * within a context that limits which files it can read
         */
      public static Launcher.ExtClassLoader getExtClassLoader() throws IOException {
            final File[] var0 = getExtDirs();

            try {
                return (Launcher.ExtClassLoader)AccessController.doPrivileged
                  (new PrivilegedExceptionAction<Launcher.ExtClassLoader>() {
                    public Launcher.ExtClassLoader run() throws IOException {
                        int var1 = var0.length;
                        for(int var2 = 0; var2 < var1; ++var2) {
                            MetaIndex.registerDirectory(var0[var2]);
                        }
                        return new Launcher.ExtClassLoader(var0);
                    }
                });
            } catch (PrivilegedActionException var2) {
                throw (IOException)var2.getException();
            }
        }

        void addExtURL(URL var1) {
            super.addURL(var1);
        }

        public ExtClassLoader(File[] var1) throws IOException {
            super(getExtURLs(var1), (ClassLoader)null, Launcher.factory);
            SharedSecrets.getJavaNetAccess().getURLClassPath(this).initLookupCache(this);
        }

        private static File[] getExtDirs() {
            String var0 = System.getProperty("java.ext.dirs");
            File[] var1;
            if (var0 != null) {
                StringTokenizer var2 = new StringTokenizer(var0, File.pathSeparator);
                int var3 = var2.countTokens();
                var1 = new File[var3];

                for(int var4 = 0; var4 < var3; ++var4) {
                    var1[var4] = new File(var2.nextToken());
                }
            } else {
                var1 = new File[0];
            }

            return var1;
        }
      .............
}
```

可以指定`-D java.ext.dirs`参数来添加和改变ExtClassLoader的加载路径。

#### 4）AppClassLoader源码

```java
static class AppClassLoader extends URLClassLoader {
        final URLClassPath ucp = SharedSecrets
          .getJavaNetAccess().getURLClassPath(this);

        public static ClassLoader getAppClassLoader(final ClassLoader var0) throws IOException {
            final String var1 = System.getProperty("java.class.path");
            final File[] var2 
                = var1 == null ? new File[0] : Launcher.getClassPath(var1);
            return (ClassLoader)AccessController
                .doPrivileged(new PrivilegedAction<Launcher.AppClassLoader>() {
                public Launcher.AppClassLoader run() {
                    URL[] var1x 
                        = var1 == null ? new URL[0] : Launcher.pathToURLs(var2);
                    return new Launcher.AppClassLoader(var1x, var0);
                }
            });
        }

        AppClassLoader(URL[] var1, ClassLoader var2) {
            super(var1, var2, Launcher.factory);
            this.ucp.initLookupCache(this);
        }
}
```

AppClassLoader加载的就是`java.class.path`下的路径。

### 二、每个类加载器都有一个父加载器

通过编写测试代码可知：

- 任何一个类的加载器是AppClassLoader
- AppClassLoader的父加载器是ExtClassLoader
- ExtClassLoader 的父加载器为null

#### 1）父加载器不是父类，ClassLoader源码

上面ExtClassLoader和AppClassLoader的代码可以看出，他们都继承了URLClassLoader{}

```java
static class ExtClassLoader extends URLClassLoader{}
static class AppClassLoader extends URLClassLoader{}
```

URLClassLoader 继承 ClassLoader，ClassLoader有getParent()方法，可以获取父加载器

```java
/**
 * Returns the parent class loader for delegation. Some implementations may
 * use <tt>null</tt> to represent the bootstrap class loader. This method
 * will return <tt>null</tt> in such implementations if this class loader's
 * parent is the bootstrap class loader.
 *
 * <p> If a security manager is present, and the invoker's class loader is
 * not <tt>null</tt> and is not an ancestor of this class loader, then this
 * method invokes the security manager's {@link
 * SecurityManager#checkPermission(java.security.Permission)
 * <tt>checkPermission</tt>} method with a {@link
 * RuntimePermission#RuntimePermission(String)
 * <tt>RuntimePermission("getClassLoader")</tt>} permission to verify
 * access to the parent class loader is permitted.  If not, a
 * <tt>SecurityException</tt> will be thrown.  </p>
 *
 * @return  The parent <tt>ClassLoader</tt>
 *
 * @throws  SecurityException
 *          If a security manager exists and its <tt>checkPermission</tt>
 *          method doesn't allow access to this class loader's parent class
 *          loader.
 *
 * @since  1.2
 */
@CallerSensitive
public final ClassLoader getParent() {
    if (parent == null)
        return null;
    SecurityManager sm = System.getSecurityManager();
    if (sm != null) {
        // Check access to the parent class loader
        // If the caller's class loader is same as this class loader,
        // permission check is performed.
        checkClassLoaderPermission(parent, Reflection.getCallerClass());
    }
    return parent;
}
```

通过ClassLoader的源码可以看出

我们可以看到`getParent()`实际上返回的就是一个ClassLoader对象parent，parent的赋值是在ClassLoader对象的构造方法中，它有两个情况： 

1. 由外部类创建ClassLoader时直接指定一个ClassLoader为parent。 
2. 由`getSystemClassLoader()`方法生成，也就是在sun.misc.Laucher通过`getClassLoader()`获取，也就是AppClassLoader。直白的说，一个ClassLoader创建时如果没有指定parent，那么它的parent默认就是AppClassLoader。

- 另外，一些实现类会用null去表示Bootstrap ClassLoader。

#### 2）ExtClassLoader和AppClassLoader的关系

Launcher类正好又初始化了ExtClassLoader和AppClassLoader，通过他的源码可以看出

```java
 Launcher.ExtClassLoader var1;
        try {
            // Create the extension class loader
            var1 = Launcher.ExtClassLoader.getExtClassLoader();
        } catch (IOException var10) {
            throw new InternalError("Could not create extension class loader", var10);
        }
        try {
            // Now create the class loader to use to launch the application
            this.loader = Launcher.AppClassLoader.getAppClassLoader(var1);
        } catch (IOException var9) {
            throw new InternalError("Could not create application class loader", var9);
        }
    }
```

AppClassLoader的parent是一个ExtClassLoader的实例。

ExtClassLoader并没有直接找到对parent的赋值。它调用了它的父类也就是URLClassLoder的构造方法并传递了3个参数。

```Java
   public ExtClassLoader(File[] var1) throws IOException {
            super(getExtURLs(var1), (ClassLoader)null, Launcher.factory);
    }
```

而URLClassLoader中的方法：

```java
  /**
     * Constructs a new URLClassLoader for the given URLs. The URLs will be
     * searched in the order specified for classes and resources after first
     * searching in the specified parent class loader. Any URL that ends with
     * a '/' is assumed to refer to a directory. Otherwise, the URL is assumed
     * to refer to a JAR file which will be downloaded and opened as needed.
     *
     * <p>If there is a security manager, this method first
     * calls the security manager's {@code checkCreateClassLoader} method
     * to ensure creation of a class loader is allowed.
     *
     * @param urls the URLs from which to load classes and resources
     * @param parent the parent class loader for delegation
     * @exception  SecurityException  if a security manager exists and its
     *             {@code checkCreateClassLoader} method doesn't allow
     *             creation of a class loader.
     * @exception  NullPointerException if {@code urls} is {@code null}.
     * @see SecurityManager#checkCreateClassLoader
     */
    public URLClassLoader(URL[] urls, ClassLoader parent) {
        super(parent);
        // this is to make the stack depth consistent with 1.1
        SecurityManager security = System.getSecurityManager();
        if (security != null) {
            security.checkCreateClassLoader();
        }
        this.acc = AccessController.getContext();
        ucp = new URLClassPath(urls, acc);
    }
```

第二个parent参数，在构造ExtClassLoader时，传入的null，因此测试代ExtClassLoader.getParent()返回null是真孤儿的。

但是Bootstrap ClassLoader可以当成他的父加载器。

#### 3）双亲委托模式

- Bootstrap ClassLoader是由C/C++编写的，它本身是虚拟机的一部分，所以它并不是一个JAVA类，也就是无法在java代码中获取它的引用，
- JVM启动时通过Bootstrap类加载器加载rt.jar等核心jar包中的class文件，之前的int.class,String.class都是由它加载。
- 然后呢，我们前面已经分析了，JVM初始化sun.misc.Launcher并创建Extension ClassLoader和AppClassLoader实例。并将ExtClassLoader设置为AppClassLoader的父加载器。
- Bootstrap没有父加载器，但是它却可以作用一个ClassLoader的父加载器。比如ExtClassLoader。这也可以解释之前通过ExtClassLoader的getParent方法获取为Null的现象。

一个类加载器查找class和resource时，是通过**<u>委托模式</u>** 进行的。它首先判断这个class是不是已经加载成功，如果没有的话它并不是自己进行查找，而是先通过父加载器，然后递归下去，直到Bootstrap ClassLoader，如果Bootstrap classloader找到了，直接返回，如果没有找到，则一级一级返回，最后到达自身去查找这些对象。这种机制就叫做双亲委托。 

![image-20180521212705682](/var/folders/zs/d6lr89wj1l3_v89_d8jg68r40000gn/T/abnerworks.Typora/image-20180521212705682.png)

### 三、重要方法

上面已经详细介绍了加载过程，但具体为什么是这样加载，我们还需要了解几个个重要的方法loadClass()、findLoadedClass()、findClass()、defineClass()。

####  1）loadClass()

java通过指定的全限定类名加载class：

```Java
 protected Class<?> loadClass(String name, boolean resolve)
        throws ClassNotFoundException
    {
        synchronized (getClassLoadingLock(name)) {
            // First, check if the class has already been loaded
            Class<?> c = findLoadedClass(name);
            if (c == null) {
                long t0 = System.nanoTime();
                try {
                    if (parent != null) {
                        c = parent.loadClass(name, false);
                    } else {
                        c = findBootstrapClassOrNull(name);
                    }
                } catch (ClassNotFoundException e) {
                    // ClassNotFoundException thrown if class not found
                    // from the non-null parent class loader
                }

                if (c == null) {
                    // If still not found, then invoke findClass in order
                    // to find the class.
                    long t1 = System.nanoTime();
                    c = findClass(name);

                    // this is the defining class loader; record the stats
                    sun.misc.PerfCounter.getParentDelegationTime().addTime(t1 - t0);
                    sun.misc.PerfCounter.getFindClassTime().addElapsedTimeFrom(t1);
                    sun.misc.PerfCounter.getFindClasses().increment();
                }
            }
            if (resolve) {
                resolveClass(c);
            }
            return c;
        }
    }
```

上面是方法原型，一般实现这个方法的步骤是  

1. 执行`findLoadedClass(String)`去检测这个class是不是已经加载过了。 
2. 执行父加载器的`loadClass`方法。如果父加载器为null，则jvm内置的加载器去替代，也就是Bootstrap ClassLoader。这也解释了ExtClassLoader的parent为null,但仍然说Bootstrap ClassLoader是它的父加载器。
3. 如果向上委托父加载器没有加载成功，则通过`findClass(String)`查找。

如果class在上面的步骤中找到了，参数resolve又是true的话，那么`loadClass()`又会调用`resolveClass(Class)`这个方法来生成最终的Class对象。java建议，ClassLoader的子类直接覆盖findClass()这个方法，而不要直接改写loadClass()方法。

```Java
 if (parent != null) {
       c = parent.loadClass(name, false);
 } else {
        c = findBootstrapClassOrNull(name);
 }
```

这里解释了当ExtClassLoader的parent为null的时候，java会选择BootstrapClassLoader作为他的向上委托。

#### 2）defineClass()

这个方法在编写自定义classloader的时候非常重要，它能将class二进制内容转换成Class对象，如果不符合要求的会抛出各种异常。

#### 3）自定义loader步骤

1. 编写一个类继承自ClassLoader抽象类。
2. 复写它的`findClass()`方法。
3. 在`findClass()`方法中调用`defineClass()`。

#### 4）ContextClassLoader 线程上下文类加载器

### 总结

1. ClassLoader用来加载class文件的。
2. 系统内置的ClassLoader通过双亲委托来加载指定路径下的class和资源。
3. 可以自定义ClassLoader一般覆盖findClass()方法。
4. ContextClassLoader与线程相关，可以获取和设置，可以绕过双亲委托的机制。