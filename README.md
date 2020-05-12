# Android 学习进阶路线
理解的才是懂的

## Java基础

1. **泛型**
    - [作用与定义](https://github.com/XiaoArea/Learning-route/blob/master/document/Java%E5%9F%BA%E7%A1%80/%E6%B3%9B%E5%9E%8B/%E6%B3%9B%E5%9E%8B%E7%9A%84%E4%BD%9C%E7%94%A8%E4%B8%8E%E5%AE%9A%E4%B9%89.md)
    - [通配符与嵌套](https://github.com/XiaoArea/Learning-route/blob/master/document/Java%E5%9F%BA%E7%A1%80/%E6%B3%9B%E5%9E%8B/%E6%B3%9B%E5%9E%8B%E7%9A%84%E9%80%9A%E9%85%8D%E7%AC%A6%E4%B8%8E%E5%B5%8C%E5%A5%97.md)
    - [上下边界](#)
    - [RxJava中的使用](#)
	
2. **注解**
    - [内置注解、元注解、自定义注解、参数与默认值](#)
    - [APT，编译时注解处理器](#)
    - [插桩，编译后处理筛选](#)
    - [反射，运行时动态获取](#)
    - [自定义Gradle插件，Transform](#)
    - [Retofit中的使用](#)
    
3. **并发**
    - [CPU核心数、线程数与时间片轮机制](#)
    - [synchronized、Lock、volatile与ThreadLocal 实现线程共享](#)
    - [Wait、Notify、NotifyAll与Join方法实现线程间协作](#)
    - [CAS原理及ABA问题](#)
    - [Callbale、Future与FutureTask](#)
    - [线程池底层实现](#)
    - [线程池排队机制](#)
    - [线程池使用及手写线程池](#)
    - [Executor实战](#)
    - [AsyncTask](#)
    
4. **序列化**
    - [Serializable](#)
    - [Parcelable](#)
    - [Json与XML](#)
    
5. **JVM**
    - [JVM原理](#)
    - [强引用、软引用、弱引用、虚引用](#)
    - [GC日志解读](#)
    - [JVN栈帧及方法调用](#)
    - [Dalvik虚拟机](#)
    
6. **反射**
    - [获取类信息、方法、属性、实例化](#)
    - [动态代理](#)
    - [Hook原理](#)
    - [Davilk与ART](#)
    - [PathClassLoader、DexClassLoader、BootClassLoader](#)
    - [双亲委托机制](#)
    
7. **IO**
    - [File基础](#)
    - [OutputStream与InputStream](#)
    - [Writer与Reader](#)
    - [FileChannel](#)
    - [Mmap内存映射](#)
    - [IO操作Dex加密](#)

## NDK

1. **C/C++**
	- [数据类型](#)
	- [内存结构与管理](#)
	- [预处理指令、Typedef别名](#)
	- [结构体与共用体](#)
	- [指针、智能指针、方法指针](#)
	- [线程](#)
	- [类](#)
	    - [函数、虚函数、纯函数与析构函数](#)
	    - [初始化列表](#)
	    
2. **JNI开发**
	- [JNI基础](#)
	- [静态与动态注册](#)
	- [方法签名、与Java通信](#)
	- [本地引用与全局引用](#)
	
3. **Native开发工具**
	- [编译器、打包工具与分析器](#)
	- [静态库与动态库](#)
	- [CPU架构与注意事项](#)
	- [构建脚本与构建工具](#)
		- [Cmake](#)
		- [Makefile](#)
	- [交叉编译移植](#)
		- [FFmpeg交叉编译](#)
		- [X264、FAAC交叉编译](#)
		- [移植问题解决](#)
	- [AS构建NDK项目](#)
4. **Linux编程**
	- [Linux环境搭建、系统管理、权限系统和工具使用（vim等）](#)
	- [Shell脚本编程](#)

## 其他
1. [Kotlin](#)
2. [Flutter](#)
3. [React Native](#)
4. [IOS](#)
5. [H5](#)
6. [小程序](#)
7. [Java 后台](#)
