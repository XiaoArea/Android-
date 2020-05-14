## 特性说明
==**可见性**==：**指线程之间的可见性，一个线程修改的状态对另一个线程是可见的**。也就是一个线程修改的结果。另一个线程马上就能看到。比如：用volatile修饰的变量，就会具有可见性。volatile修饰的变量不允许线程内部缓存和重排序，即直接修改内存。所以对其他线程是可见的。但是这里需要注意一个问题，volatile只能让被他修饰内容具有可见性，但不能保证它具有原子性。比如 volatile int a =0；之后有一个操作 a++；这个变量a具有可见性，但是a++依然是一个非原子操作，也就是这个操作同样存在线程安全问题。在 Java 中 volatile、synchronized 和 final 实现可见性。

==**原子性**==：**原子是世界上的最小单位，具有不可分割性**。比如 a=0；（a非long和double类型） 这个操作是不可分割的，那么我们说这个操作时原子操作。再比如：a++； 这个操作实际是a = a + 1；是可分割的，所以他不是一个原子操作。非原子操作都会存在线程安全问题，需要使用同步技术（sychronized）来让它变成一个原子操作。一个操作是原子操作，那么称它具有原子性。java的concurrent包下提供了一些原子类，可以通过阅读API来了解这些原子类的用法。比如：AtomicInteger、AtomicLong、AtomicReference等。
在 Java 中 synchronized 和在 lock、unlock 中操作保证原子性。

==**有序性**==：Java 语言提供了 volatile 和 synchronized 两个关键字来保证线程之间操作的有序性，volatile 是因为其本身包含“禁止指令重排序”的语义，synchronized 是由“一个变量在同一个时刻只允许一条线程对其进行 lock 操作”这条规则获得的，此规则决定了持有同一个对象锁的两个同步块只能串行执行。

---

## synchronized
##### synchronized 作用
> 1. 确保线程互斥地访问同步代码
> 2. 保证共享变量的线程可见性
> 3. 禁止指令重排

其中1和3相当于volatile关键字的作用

##### sychronized 使用
> 1. **静态方法**：将类对象自身作为monitor对象，对该类所有使用了sychronized修饰的静态方法进行同步，即任何时候只能存在一个线程在调用该类的使用了synchronized修饰的静态方法，其他调用了该类的使用了synchronized修饰的静态方法的线程需要阻塞；（锁对象是该类，即XXX.class）

> 2. **普通成员方法**：使用类的对象实例作为monitor对象，该类所有使用了synchronized修饰的成员方法，在任何时刻只能被一个线程访问，其他线程需要阻塞；（锁对象是this，即该类实例本身）

> 3. **代码块**：使用某个对象作为monitor对象，通常为一个普通的private成员变量，如private Object object = new Object();，这样所有使用了该object对象的同步块，在任何时候只能存在一个线程访问。（锁对象可指定，可为this、XXX.class、全局变量）

**优点**：线程竞争不自旋，不消耗CPU

**缺点**：

- 效率低：
    - 锁的释放情况少，只在程序正常执行完成和抛出异常时释放锁；
    - 试图获得锁是不能设置超时；
    - 不能中断一个正在试图获得锁的线程；

- 无法知道是否成功获取到锁；

**适用场景**：追求吞吐量、同步块执行时间较长

参考：https://www.cnblogs.com/shoshana-kong/p/10877555.html

---

## Lock

##### 定义
Lock接口的功能跟synchronzied一样的，只不过synchronized同步块执行结束后锁会自动释放，而lock接口必须要调用unlock()方法释放锁。Lock接口的其他方面会比synchronized更为灵活，提供了一些方法和多方式控制加锁 / 解锁，可以选择线程进行一些操作。

##### 锁Lock接口区分：
> 1. 公平锁：意思是线程获取锁的顺序是按先来先得顺序的，就是线程加锁的顺序来分配的。
> 2. 非公平锁：就是随机性获得锁，没有顺序的

##### Lock接口的主要三个实现类：
1. ReentrantLock 
2. ReentrantReadWriteLock.ReadLock
3. ReentrantReadWriteLock.WriteLock

> 因为Lock是接口所以使用时要结合它的实现类，另外在finall语句块中释放锁的目的是保证获取到锁之后，最终能够被释放。

##### 写法区分：
###### 公平锁
```
// 常规锁
Lock lock = ReentrantLock();
Lock lock = ReentrantLock(true);
// 读写锁
ReentrantReadWriteLock lock = new ReentrantReadWriteLock();
ReentrantReadWriteLock lock = new ReentrantReadWriteLock(true);
```
###### 非公平锁
```
// 常规锁
Lock lock = ReentrantLock(false);
// 读写锁
ReentrantReadWriteLock lock = new ReentrantReadWriteLock(false);
```

ReentrantReadWriteLock 是可重复进入读写锁，读取锁和写入锁，

> 1. 读取锁含义：在同一时刻可以被多个读线程访问，线程之间不存在阻塞；

> 2. 写入锁含义：在多个写入线程同一时刻进行访问写入时，只有一个线程可以写入，其他写入线程和所有读取线程都会被阻塞。

参考：https://www.jianshu.com/p/f45f20bbdeec

---

## volatile
Java语言提供了一种稍弱的同步机制，即volatile变量，用来确保将变量的更新操作通知到其他线程。当把变量声明为volatile类型后，编译器与运行时都会注意到这个变量是共享的，因此不会将该变量上的操作与其他内存操作一起重排序。volatile变量不会被缓存在寄存器或者对其他处理器不可见的地方，因此在读取volatile类型的变量时总会返回最新写入的值。

在访问volatile变量时不会执行加锁操作，因此也就不会使执行线程阻塞，因此volatile变量是一种比sychronized关键字更轻量级的同步机制。

当对非 volatile 变量进行读写的时候，每个线程先从内存拷贝变量到CPU缓存中。如果计算机有多个CPU，每个线程可能在不同的CPU上被处理，这意味着每个线程可以拷贝到不同的 CPU cache 中。

而声明变量是 volatile 的，JVM 保证了每次读变量都从内存中读，跳过 CPU cache 这一步。

##### 当一个变量定义为 volatile 之后，将具备两种特性：
1. 保证此变量对所有的线程的可见性，当一个线程修改了这个变量的值，volatile 保证了新值能立即同步到主内存，以及每次使用前立即从主内存刷新。但普通变量做不到这点，普通变量的值在线程间传递均需要通过主内存来完成。
2. 禁止指令重排序优化。有volatile修饰的变量，赋值后多执行了一个“load addl $0x0, (%esp)”操作，这个操作相当于一个内存屏障（指令重排序时不能把后面的指令重排序到内存屏障之前的位置），只有一个CPU访问内存时，并不需要内存屏障；（什么是指令重排序：是指CPU采用了允许将多条指令不按程序规定的顺序分开发送给各相应电路单元处理）。

##### volatile 性能：
volatile 的读性能消耗与普通变量几乎相同，但是写操作稍慢，因为它需要在本地代码中插入许多内存屏障指令来保证处理器不发生乱序执行。

参考：
1. https://blog.csdn.net/liudongdong19/article/details/86748072
2. https://blog.csdn.net/hb_csu/article/details/80431952
3. https://www.ibm.com/developerworks/cn/java/j-jtp06197.html


---

## ThreadLocal
ThreadLocal有3个方法，其中值得注意的是initialValue()，该方法是一个protected的方法，显然是为了子类重写而特意实现的。该方法返回当前线程在该线程局部变量的初始值，这个方法是一个延迟调用方法，在一个线程第1次调用get()或者set(Object)时才执行，并且仅执行1次。ThreadLocal中的确实实现直接返回一个null：

```
protected Object initialValue() { return null; }
```
ThreadLocal是如何做到为每一个线程维护变量的副本的呢？其实实现的思路很简单，在ThreadLocal类中有一个Map，用于存储每一个线程的变量的副本。

##### ThreadLocal与其它同步机制的比较

　　ThreadLocal和其它同步机制相比有什么优势呢？ThreadLocal和其它所有的同步机制都是为了解决多线程中的对同一变量的访问冲突，在普通的同步机制中，是通过对象加锁来实现多个线程对同一变量的安全访问的。这时该变量是多个线程共享的，使用这种同步机制需要很细致地分析在什么时候对变量进行读写，什么时候需要锁定某个对象，什么时候释放该对象的锁等等很多。所有这些都是因为多个线程共享了资源造成的。ThreadLocal就从另一个角度来解决多线程的并发访问，ThreadLocal会为每一个线程维护一个和该线程绑定的变量的副本，从而隔离了多个线程的数据，每一个线程都拥有自己的变量副本，从而也就没有必要对该变量进行同步了。ThreadLocal提供了线程安全的共享对象，在编写多线程代码时，可以把不安全的整个变量封装进ThreadLocal，或者把该对象的特定于线程的状态封装进ThreadLocal。
　　由于ThreadLocal中可以持有任何类型的对象，所以使用ThreadLocal get当前线程的值是需要进行强制类型转换。但随着新的Java版本（1.5）将模版的引入，新的支持模版参数的ThreadLocal<T>类将从中受益。也可以减少强制类型转换，并将一些错误检查提前到了编译期，将一定程度地简化ThreadLocal的使用。
　　
##### 总结

ThreadLocal并不能替代同步机制，两者面向的问题领域不同。同步机制是为了同步多个线程对相同资源的并发访问，是为了多个线程之间进行通信的有效方式；而ThreadLocal是隔离多个线程的数据共享，从根本上就不在多个线程之间共享资源（变量），这样当然不需要对多个线程进行同步了。所以，如果你需要进行多个线程之间进行通信， 则使用同步机制；如果需要隔离多个线程之间的共享冲突，可以使用ThreadLocal，这将极大地简化你的程序，使程序更加易读、简洁。

> 即：当前线程创建变量的副本

参考：
1. https://www.cnblogs.com/fsmly/p/11020641.html
2. https://www.cnblogs.com/studyLog-share/p/5295557.html
3. https://www.jb51.net/article/109191.htm
