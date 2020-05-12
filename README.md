# Android 学习进阶路线
理解的才是懂的

## 大纲
> [Java基础](#1)

> [Android基础](#2)

> [性能优化](#3)

> [开源框架](#4)

> [NDK开发](#5)

> [其他](#6)

---

## 思维导图
![image](https://note.youdao.com/yws/api/personal/file/5FD4D5E083194785ACAA77F9462D8342?method=download&shareKey=65878145e2ed4a5b367d61039cb51d4a)

## <a name="#1">Java基础</a>

1. **泛型**
    - [作用与定义](#)
    - [通配符与嵌套](#)
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

## <a name="#2">Android基础</a>

1. **高级UI**
    - [常用View](#)
        1. [RecycleView](#)
            - [源码解析](#)
            - [布局管理器LayoutManager](#)
            - [条目装饰ItemDecoration](#)
            - [ViewHolder与回收复用机制](#)
            - [手写RecycleView](#)
            - [天猫VLayout](#)
        2. [CradView](#)
            - [源码解析](#)
            - [圆角与阴影](#)
            - [5.0以下阴影与边距适配](#)
        3. [ViewPager](#)
            - [加载机制与优化](#)
            - [与Fragment结合使用](#)
        4. [WebView](#)
            - [原理及使用](#)
            - [JS与Java交互](#)
            - [多进程使用WebView](#)
            - [WebView与Native通信](#)
    - [布局ViewGroup](#)
        - [ConstraintLayout](#)
        - [LinearLayout](#)
        - [RelativeLayout](#)
        - [FrameLayout](#)
        - [GridLayout](#)
    - [View渲染](#)
        - [onLayout与onMeasure](#)
        - [onDraw映射机制](#)
    - [触摸事件分发机制](#)
    - [自定义View](#)
    - [Material Design](#)
    - [ViewPager2](#)
    
2. **组件内核**
    - [Activity与调用栈](#)
    - [Fragment管理与内核](#)
    - [Service内核原理](#)
    - [组件间通信方案](#)
    
3. **IPC**
	- [Binder机制](#)
	    - [AIDL配置文件](#)
	    - [C/S架构binder原理](#)
	    - [Messager](#)
	    - [实战binder，进程通信原理与实现](#)
    - [其他IPC方式](#)
	    - [Broadcast](#)
	    - [ContentPreovider](#)
	    - [File文件](#)
	    - [Socket](#)
	    - [共享内存与管道](#)
	
4. **数据持久化**
    - [文件系统：sdcard与内部存储](#)
    - [轻量级KV持久化](#)
	    1. [SharedPreference](#)
	    2. [MMKV原理与实现](#)
		    - [Mmap内存映射](#)
		    - [文件数据结构](#)
		    - [增量更新与全量更新](#)
    - [Sqlite](#)
	    1. [SqliteOpenHelper](#)
	    2. [Sqlite升级、分库与数据迁移方案](#)
	    3. [ORM数据库框架](#)
		    - [GreenDao](#)
		    - [LitePal](#)
		    - [Afinal](#)
		    - [LiteOrm](#)
		    - [ORMLite](#)
		    - [SugarORM](#)
		    - [Realm](#)
		    - [DBFlow](#)

5. **Framework**
	- [XMS内核管理](#)
		1. [AMS](#)
			- [Activity管理](#)
			- [实战插件化核心启动未安装的Activity](#)
		2. [WMS](#)
			- [Windows体系](#)
			- [悬浮窗工具实现](#)
		3. [PMS](#)
		4. [实战插件化框架原理与实现](#)
	- [Handler消息机制](#)
		- [Looper](#)
		- [Message链表与对象池](#)
		- [MessageQueue消息队列与epoll机制](#)
	- [布局加载与资源系统](#)
		- [LayoutManager加载布局流程](#)
		- [Resource与AssetManager](#)
		- [实战网易云换肤、加载外部APK资源](#)

## <a name="#3">性能优化</a>

1. **设计思想**
	- [六大原则](#)
		- [单一职责原则](#)
		- [开闭原则](#)
		- [里氏替换原则](#)
		- [依赖倒置原则](#)
		- [接口隔离原则](#)
		- [迪米特法则](#)
	- [设计模式](#)
		1. [结构型](#)
			- [桥接模式](#)
			- [适配器模式](#)
			- [装饰器模式](#)
			- [代理模式](#)
			- [组合模式](#)
		2. [创建型](#)
			- [建造者模式](#)
			- [单例模式](#)
			- [抽象工厂模式](#)
			- [工厂方法模式](#)
			- [静态工厂模式](#)
		3. [行为型](#)
			- [模板方法模式](#)
			- [策略模式](#)
			- [观察者模式](#)
			- [责任链模式](#)
			- [命令模式](#)
			- [访问者模式](#)
		4. [实战设计模式解耦项目网络层框架](#)
	- [数据结构](#)
		1. [线性表ArrayList](#)
		2. [链表LinkedList](#)
		3. [栈Stack](#)
		4. [队列](#)
			- [Queue](#)
			- [Deque](#)
			- [阻塞队列](#)
		5. [Tree](#)
			- [平衡二叉树](#)
			- [红黑树](#)
		6. [映射表](#)
			- [HashTable](#)
			- [HashMap](#)
			- [SparseArray](#)
			- [ArrayMap](#)
	- [算法](#)
		1. [排序](#)
			- [冒泡排序](#)
			- [选择排序](#)
			- [插入排序](#)
			- [快速排序](#)
			- [堆排序](#)
			- [基数排序](#)
		2. [查找](#)
			- [折半查找](#)
			- [二分查找](#)
			- [树形查找](#)
			- [hash查找](#)


2. **性能优化**
	- [启动速度与执行效率优化](#)
		- [冷暖启动耗时检测与分析](#)
		- [启动黑白屏解决方案](#)
		- [卡顿分析](#)
		- [StickMode严苛模式](#)
		- [Systrace与TraceView工具](#)
	- [布局检测与优化](#)
		- [布局层级优化](#)
		- [过度渲染检测](#)
		- [Hierarchy Viewer与Layout Inspactor工具](#)
	- [内存优化](#)
		- [内存抖动和内存泄漏](#)
		- [内存大户，Bitmap内存优化](#)
		- [Profile内存监测工具](#)
		- [Mat大对象与泄漏检测](#)
	- [耗电优化](#)
		- [Doze&Standby](#)
		- [Battery Historian](#)
		- [JobScheduler、WorkManager](#)
	- [网络传输与数据存储优化](#)
		- [Google序列化工具Protobuf](#)
		- [7z极限压缩](#)
		- [webp图片](#)
	- [APK大小优化](#)
		- [APK瘦身](#)
		- [动态SO](#)
		- [微信资源混淆原理](#)
		
3. **开发效率**
	- [Git分布式版本控制](#)
	- [Gradle自动化构建](#)
		1. [Gradle与Android插件](#)
		2. [Transform API](#)
		3. [自定义插件开发](#)
		4. [插件实战](#)
			- [多渠道打包](#)
			- [发版自动钉钉/企微](#)

## <a name="#4">开源框架</a>

1. **热修复**
	- [AOT/JIT、dexopt与dex2oat](#)
	- [CLASS_ISPREVERIFIED问题与解决](#)
	- [即时生效与重启生效热修复原理](#)
	- [Gradle自动补丁包](#)

2. **插件化**
	- [Class文件加载Dex原理](#)
	- [Android资源加载与管理](#)
	- [四大组件的加载与管理](#)
	- [SO库的加载原理](#)
	- [Android系统服务运行原理](#)

3. **组件化**
	- [ARouter原理](#)
	- [APT自动生成代码与动态类加载](#)
	- [Java SPI机制实现组件服务调用](#)
	- [AOP切面、路由参数传递与IOC注入](#)
	- [手写组件化路由](#)

4. **图片加载**
	- [框架选型](#)
		- [ImageLoader、Glide、Picasso与Fresco](#)
		- [Glide](#)
		- [Picasso](#)
		- [Fresco](#)
	- [Glide原理](#)
		1. [Fragment感知生命周期原理](#)
		2. [自动图片大小计算](#)
		3. [图片解码](#)
		4. [优先级请求队列](#)
		5. [ModelLoader与Registry机制](#)
		6. [内存缓存原理](#)
			- [LRU内存缓存](#)
			- [引用计数与弱引用活跃缓存](#)
			- [Bitmap复用池](#)
			- [缓存大小配置](#)
		7. [磁盘文件缓存](#)
			- [原始图像文件缓存](#)
			- [解码图像文件缓存](#)
	- [手写图片加载框架](#)

5. **网络访问**
	- [通信基础](#)
		1. [HTTP协议与TCP/IP协议](#)
		2. [SSL握手与加密](#)
		3. [DNS解析](#)
		4. [Restful URL](#)
		5. [Socket通信](#)
			- [SOCKS代理](#)
			- [Http普通代理与隧道代理](#)
	- [OkHttp原理](#)
		- [Socket连接池复用机制](#)
		- [Http协议重定向与缓存处理](#)
		- [高并发请求队列：任务分发](#)
		- [责任链模式拦截器设计](#)
	- [Retrofit原理](#)

6. **响应式编程**
	- [链式调用](#)
	- [扩展的观察者模式](#)
	- [事件变化设计](#)
	- [Scheduler线程控制](#)

7. **IOC架构**
	- [依赖注入与控制反转](#)
	- [ButterKnife原理](#)
	- [Dagger原理](#)

8. **Jetpack**
	- [LiveData原理](#)
	- [Navigation解决tabLayout问题](#)
	- [ViewMode感知View生命周期及内核原理](#)
	- [Room架构方式](#)
	- [DataBinding怎么支持MVVM](#)
	- [WorkManager原理](#)
	- [Lifecycles生命周期](#)


## <a name="#5">NDK开发</a>

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

## <a name="#6">其他</a>
1. [Kotlin](#)
2. [Flutter](#)
3. [React Native](#)
4. [IOS](#)
5. [H5](#)
6. [小程序](#)
7. [Java 后台](#)

