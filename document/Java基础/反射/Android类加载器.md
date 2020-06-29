
### 概述
虚拟机加载类时，将二进制字节流读取到内存，可以从class文件、zip包、jar包、网络或动态代理生成二进制字节流。

某一个类，需要保证在虚拟机中的唯一性，由类加载器和他本身决定，不同的类加载器加载的类肯定不等，相同的类加载器保证某个类只加载一次。

在使用Java虚拟机时，经常需要自定义继承自ClassLoader的类加载器。然后通过defineClass方法来从一个二进制流中加载Class。而在Android中无法这么使用，Android中ClassLoader的defineClass方法具体是调用VMClassLoader的defineClass本地静态方法。而这个本地方法什么都没做，只是抛出了一个“UnsupportedOperationException”异常。 

Android官方为了解决这个问题，从ClassLoader中派生出了两个类：DexClassLoader和PathClassLoader;

##### Android类加载器                                                       
> PathClassLoader，DexClassLoader，BootClassLoader。

![image](https://note.youdao.com/yws/public/resource/bd20ef876cb61385c8c4cc42339590cf/xmlnote/06E6D2DB9E214123A48E5CFD9E41FFB5/6689)

##### ClassLoader
抽象类，定义了类加载的基础方法loadClass，findClass。

##### BootClassLoader
加载系统Sdk框架类，系统启动时创建，如Context，Activity类。
```
Activity.class.getClassLoader();
String.class.getClassLoader();
```

##### BaseDexClassLoader
类，父类，路径都在dalvik.system包中

##### PathClassLoader
应用启动时创建一个实例，加载系统内已安装apk中的类。
```
// 构造器说明
// path：文件或者目录的列表 
// libPath：包含lib库的目录列表 
// parent：父类加载器
public PathClassLoader (String path, ClassLoader parent)
public PathClassLoader (String path, String libPath, ClassLoader parent)

// 常规示例：
MainActivity.class.getClassLoader();
```

##### DexClassLoader
加载未安装apk中的类，dex文件，在热修复/插件化时可以使用，加载外部存储路径下的apk或dex补丁。
```
// dexPath：dex文件路径列表，多个路径使用":"分隔 
// dexOutputDir：经过优化的dex文件（odex）文件输出目录，API 26开始，此参数已弃用，并且无效
// libPath：动态库路径（将被添加到app动态库搜索路径列表中） 
// parent：这是一个ClassLoader，这个参数的主要作用是保留java中ClassLoader的委托机制（优先父类加载器加载classes，由上而下的加载机制，防止重复加载类字节码）
public DexClassLoader (String dexPath, String dexOutputDir, String libPath, ClassLoader parent)
```
它是一个可以从包含classes.dex实体的.jar或.apk文件中加载classes的类加载器。可以用于实现dex的动态加载、代码热更新等等。这个类加载器必须要有一个app私有、可写目录来缓存经过优化的classes（odex文件），使用Context.getDir(String, int)方法可以创建一个这样的目录，如下：
```
File dexOutputDir = context.getDir("path", 0);
```

##### 功能区分
> - DexClassLoader：能够加载未安装的jar/apk/dex 
> - PathClassLoader：只能加载系统中已经安装过的apk
