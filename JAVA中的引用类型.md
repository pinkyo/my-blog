# JAVA中的引用类型

[TOC]

如果你熟悉面向对象编程语言，必然会了解`引用（Reference）`的概念。在面向对象编程语言中，不得不说的两种数据类型：原始数据类型和复合数据类型，相应的就是`值（Value）`和`引用（Reference）`。实际上来说，引用是一种特殊的Value，只不过它是指向一个复合数据对象的逻辑内存地址，我们使用的时候通过这个内存地址来操作地址对应的数据。在JAVA中，JAVA虚拟机机制的存在，为了合理地使用内存空间，方便`内存的回收（Garbage Collection）`等原因，JAVA语言中有几种引用类型：

1. `强引用（Strong Reference）`,一般`new`出来的对象都是强引用，对象只要有强引用就不会被GC回收；
2. `弱引用（Weak Reference）`，在GC的时候会被回收，实现类为`java.lang.ref.WeakReference`；
3. `软引用（Soft Reference）`，在Full GC也就是内存不足的时候被回收，实现类为`java.lang.ref.SoftReference`；
4. `虚引用（Phantom Reference）`，最弱的引用，甚至无法通过虚引用取到对象，通常只用于收取被GC通知，实现类为`java.lang.ref.PhantomReference`。


这几类引用的Java代码很少，感觉就是一个简单的Tag Class，具体的处理都交给了HotSpot JVM。