
Android 所使用的虚拟机（Dalvik 和 ART）与 JVM 有所不同;

#### Dalvik 虚拟机
Dalvik是Google公司自己设计用于Android平台的Java虚拟机，它是Android平台的重要组成部分，支持dex格式（Dalvik Executable）的Java应用程序的运行。dex格式是专门为Dalvik设计的一种压缩格式，适合内存和处理器速度有限的系统。Google对其进行了特定的优化，使得Dalvik具有高效、简洁、节省资源的特点。从Android系统架构图知，Dalvik虚拟机运行在Android的运行时库层。

Dalvik作为面向Linux、为嵌入式操作系统设计的虚拟机，主要负责完成对象生命周期管理、堆栈管理、线程管理、安全和异常管理，以及垃圾回收等。另外，Dalvik早期并没有JIT编译器，直到Android2.2才加入了对JIT的技术支持。

##### Dalvik 虚拟机特点
> - 体积小，占用内存空间小；
> - 专有的DEX可执行文件格式，体积更小，执行速度更快；
> - 常量池采用32位索引值，寻址类方法名，字段名，常量更快；
> - 基于寄存器架构，并拥有一套完整的指令系统；
> - 提供了对象生命周期管理，堆栈管理，线程管理，安全和异常管理以及垃圾回收等重要功能；
> - 所有的Android程序都运行在Android系统进程里，每个进程对应着一个Dalvik虚拟机实例。

##### Dalvik 虚拟机 和 Java 虚拟机 的区别
JVM | DVM
---|---
java虚拟机基于栈，基于栈的机器必须使用指令来载入和操作栈上数据 | Dalvik虚拟机基于寄存器
java虚拟机运行的是java字节码（java类会被编译成一个或多个字节码.class文件，打包到.jar文件中，java虚拟机从相应的.class文件和.jar获取相应的字节码） | Dalvik虚拟机运行的是Dalvik字节码（java类被编译成.class文件后，会通过一个dx工具将所有的.class文件转换成一个.dex文件，然后dalvik虚拟机会从其中读取指令和数据）
- | 一个应用对应一个Diavik虚拟机实例，独立运行
JVM在运行的时候为每一个类装载字节码 | Dalvik程序只包含一个.dex文件，这个文件包含了程序中所有的类

---

#### ART 虚拟机
即Android Runtime，Android 4.4发布了一个ART运行时，准备用来替换掉之前一直使用的Dalvik虚拟机。
ART 的机制与 Dalvik 不同。在Dalvik下，应用每次运行的时候，字节码都需要通过即时编译器（just in time ，JIT）转换为机器码，这会拖慢应用的运行效率，而在ART 环境中，应用在第一次安装的时候，字节码就会预先编译成机器码，使其成为真正的本地应用。这个过程叫做预编译（AOT,Ahead-Of-Time）。这样的话，应用的启动(首次)和执行都会变得更加快速。

#### Dalvik与Art的区别
> - Dalvik每次都要编译再运行，Art只会首次启动编译
> - Art占用空间比Dalvik大（原生代码占用的存储空间更大），就是用“空间换时间”
> - Art减少编译，减少了CPU使用频率，使用明显改善电池续航
> - Art应用启动更快、运行更快、体验更流畅、触感反馈更及时

---

#### Android 虚拟机发展史

###### Android 诞生之初
自2008年9月23日Android 1.0以来，初期不同于Java平台使用不同于 Java 平台使用 JVM 加载字节码文件(.class)，Android 系统由 Dalvik 担任虚拟机的角色，每次运行程序的时候，Dalvik 负责加载 dex/odex 文件并解析成机器码交由系统调用。
###### JIT 首次登场（Android 2.2）
为了适应硬件速度的提升，Android 系统系统也在不断更新，单一的 Dalvik 虚拟机已经渐渐地满足系统的要求了，2010年5月20日，Google 发布 Android 2.2（Froyo冻酸奶），在这个版本中，Google 在 Android 虚拟中加入了 JIT 编译器（Just-In-Time Compiler）。和其他大多数 JVM 一样，Dalvik 使用 JIT 进行即时编译，借助 Java HotSpot VM，JIT 编译器可以对执行次数频繁的 dex/odex 代码进行编译与优化，将 dex/odex 中的 Dalvik Code（Smali 指令集）翻译成相当精简的 Native Code 去执行，JIT 的引入使得 Dalvik 的性能提升了 3~6 倍。
> 但是 JIT 模式的缺点也不容忽视：
> - 每次启动应用都需要重新编译
> - 运行时比较耗电，造成电池额外的开销

###### ART 和 AOT（Android 4.4）
2013年10月31日，Google 发布 Android 4.4（奇巧Kitkat）,带来了全新的虚拟机运行环境 ART（Android RunTime）的预览版和全新的编译策略 AOT（Ahead-of-time），需要注意的是，彼时 ART 是和 Dalvik 共存的，用户可以在两者之间进行选择。

###### ART 全面取代 Dalvik（Android 5.0）
2014年10月16日，Google 发布 Android 5.0（棒棒糖Lollipop），ART 全面取代 Dalvik 成为 Android 虚拟机运行环境，至此，Dalvik 退出历史舞台，AOT 也成为唯一的编译模式。

AOT 和 JIT 的不同之处在于：JIT 是在运行时进行编译，是动态编译，并且每次运行程序的时候都需要对 odex 重新进行编译；而 AOT 是静态编译，应用在安装的时候会启动 dex2oat 过程把 dex 预编译成 ELF 文件，每次运行程序的时候不用重新编译，是真正意义上的本地应用。

另外，相比于 Dalvik，ART 对 Garbage Collection（GC）过程的也进行了改进：

只有一次 GC 暂停（Dalvik 需要两次）
在 GC 保持暂停状态期间并行处理
在清理最近分配的短时对象这种特殊情况中，回收器的总 GC 时间更短
优化了垃圾回收的工效，能够更加及时地进行并行垃圾回收，这使得 GC_FOR_ALLOC 事件在典型用例中极为罕见
压缩 GC 以减少后台内存使用和碎片
AOT 模式解决了应用启动和运行速度和耗电问题的同时也带来了另外两个问题：

应用安装和系统升级之后的应用优化比较耗时
优化后的文件会占用额外的存储空间

###### JIT 回归（Android 7.0）
在 Android 5.x 和 6.x 的机器上，系统每次 OTA 升级完成重启的时候都会有个应用优化的过程，这个过程就是刚才所说的 dex2oat 过程，这个过程比较耗时并且会占用额外的存储空间。

2016年8月22日，Google 发布 Android 7.0（牛轧糖Nougat），JIT 编译器回归，形成 AOT/JIT 混合编译模式，这种混合编译模式的特点是：

应用在安装的时候 dex 不会被编译
应用在运行时 dex 文件先通过解析器（Interpreter）后会被直接执行（这一步骤跟 Android 2.2 - Android 4.4之前的行为一致），与此同时，热点函数（Hot Code）会被识别并被 JIT 编译后存储在 jit code cache 中并生成 profile 文件以记录热点函数的信息。
手机进入 IDLE（空闲） 或者 Charging（充电） 状态的时候，系统会扫描 App 目录下的 profile 文件并执行 AOT 过程进行编译。
可以看出，混合编译模式综合了 AOT 和 JIT 的各种优点，使得应用在安装速度加快的同时，运行速度、存储空间和耗电量等指标都得到了优化。
