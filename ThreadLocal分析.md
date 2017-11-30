# ThreadLocal技术分析

[TOC]

## 前言

`java.lang.ThreadLocal`是Java中并行开发的一个实用工具，在我们的功能中如果参数很多，有很多状态要维护的时候可以使用它来简化代码的复杂度。比如，我曾经参与过的一个有Multi Tenancy需求的组件，在功能的实现过程中经常要涉及Tenant信息的处理，但是将Tenant的这个参数在整个调用链上不断地向下传递，会使得整个代码很乱，而且也并不调用链中的每个环节都要使用这个参数，这样在有些地方就尤为多余，甚至会使用方法有迷惑性。引入ThreadLocal很好的解决了这些问题，使得代码的可维护性得到提升。接下来我会分析一下ThreadLocal的实现细节，并探讨一个怎样正确地使用这个工具。

## 实现细节

ThreadLocal的实现核心是对`java.lang.ThreadLocal.ThreadLocalMap`的维护。没有仔细看过源代码的人可能会认为是所有线程共享一个ThreadLocalMap，想想也是，这样的实现并不是不可以，要创建的对象也少，只要ThreadLocalMap是线程安全并且在Key上做下手脚实现线程隔离就可以了。然而事实与我们想的不同，这个ThreadLocalMap实际是与Thread绑定的，即一个Thread单独维护一个ThreadLocalMap，ThreadLocal只是从Thread中拿至Map再做相应的set、get操作。这样做虽然要创建的Map多了，但是有多个好处：

1. 不再需要ThreadLocalMap线程安全，可以减少锁之类的性能开销；
2. 虽然要创建的ThreadLocalMap的实例多了，但是在存内容的时候不再需要使用诸如ThreadId + key这类手动隔离的方法，内存的使用并不会变大；
3. 在线程的信息发生改变的时候，不需要重新处理key值来处理同步问题，扩展性得到提高。

实际的Thread的源代码如下：

```java
public
class Thread implements Runnable {
    /* Make sure registerNatives is the first thing <clinit> does. */
    private static native void registerNatives();
    static {
        registerNatives();
    }

    private volatile String name;
    private int            priority;
    private Thread         threadQ;
    private long           eetop;

    /* Whether or not to single_step this thread. */
    private boolean     single_step;

    /* Whether or not the thread is a daemon thread. */
    private boolean     daemon = false;

    /* JVM state */
    private boolean     stillborn = false;

    /* What will be run. */
    private Runnable target;

    /* The group of this thread */
    private ThreadGroup group;

    /* The context ClassLoader for this thread */
    private ClassLoader contextClassLoader;

    /* The inherited AccessControlContext of this thread */
    private AccessControlContext inheritedAccessControlContext;

    /* For autonumbering anonymous threads. */
    private static int threadInitNumber;
    private static synchronized int nextThreadNum() {
        return threadInitNumber++;
    }

    /* ThreadLocal values pertaining to this thread. This map is maintained
     * by the ThreadLocal class. */
    ThreadLocal.ThreadLocalMap threadLocals = null;

    /*
     * InheritableThreadLocal values pertaining to this thread. This map is
     * maintained by the InheritableThreadLocal class.
     */
    ThreadLocal.ThreadLocalMap inheritableThreadLocals = null;

    ...
}
```

下面我们分析下ThreadLocalMap，先看ThreadLocalMap的一段源代码：

``` java
static class ThreadLocalMap {

    /**
    * The entries in this hash map extend WeakReference, using
    * its main ref field as the key (which is always a
    * ThreadLocal object).  Note that null keys (i.e. entry.get()
    * == null) mean that the key is no longer referenced, so the
    * entry can be expunged from table.  Such entries are referred to
    * as "stale entries" in the code that follows.
    */
    static class Entry extends WeakReference<ThreadLocal<?>> {
        /** The value associated with this ThreadLocal. */
        Object value;

        Entry(ThreadLocal<?> k, Object v) {
            super(k);
            value = v;
        }
    }

    /**
    * The initial capacity -- MUST be a power of two.
    */
    private static final int INITIAL_CAPACITY = 16;

    /**
    * The table, resized as necessary.
    * table.length MUST always be a power of two.
    */
    private Entry[] table;

    /**
    * The number of entries in the table.
    */
    private int size = 0;
    ...
}
```

从ThreadLocalMap的这一小段代码中，我们可以得到几个内容：

1. Entry继承WeakReference类，实际是Entry的key是WeakReference，并且从注释我们也可以知道，ThreadLocal变量只要没有外部强引用，就可以被GC，但是ThreadLocal被GC并不代表对应的value也会被马上GC，这个我们可以从后面的分析知道；
2. Entry的Key是ThreadLocal，这个的意思是无论在我们定义多少ThreadLocal变量，实际上这些ThreadLocal变量的值都会在存储在这个Thread中的ThreadLocalMap中；
3. ThreadLocalMap中Entry的Size是从16开始，每次不足就*2这样增长的，并不像ArrayList那么聪明。

好了，到此基本ThreadLocal的设计已经基本理清了，我们可以再看看ThreadLocal的几个函数印证一下我们的想法。

看看`java.lang.ThreadLocal#set`:

```java
/**
* Sets the current thread's copy of this thread-local variable
* to the specified value.  Most subclasses will have no need to
* override this method, relying solely on the {@link #initialValue}
* method to set the values of thread-locals.
*
* @param value the value to be stored in the current thread's copy of
*        this thread-local.
*/
public void set(T value) {
    Thread t = Thread.currentThread();
    ThreadLocalMap map = getMap(t);
    if (map != null)
        map.set(this, value);
    else
        createMap(t, value);
}
/**
* Get the map associated with a ThreadLocal. Overridden in
* InheritableThreadLocal.
*
* @param  t the current thread
* @return the map
*/
ThreadLocalMap getMap(Thread t) {
    return t.threadLocals;
}
```

可以清楚的看到是从Thread中先拿到ThreadLocalMap，然后再将值以ThreadLocal作为key存进Map.

java.lang.ThreadLocal#get的实现类似：

```java
/**
* Returns the value in the current thread's copy of this
* thread-local variable.  If the variable has no value for the
* current thread, it is first initialized to the value returned
* by an invocation of the {@link #initialValue} method.
*
* @return the current thread's value of this thread-local
*/
public T get() {
    Thread t = Thread.currentThread();
    ThreadLocalMap map = getMap(t);
    if (map != null) {
        ThreadLocalMap.Entry e = map.getEntry(this);
        if (e != null) {
            @SuppressWarnings("unchecked")
            T result = (T)e.value;
            return result;
        }
    }
    return setInitialValue();
}
```

好了，到现在ThreadLocal实现细节已经很清楚了，我们接下来探讨一下如何正确的使用ThreadLocal。

## 使用探讨

在使用的时候，我们要根据ThreadLocal的实现来做相应的调整，不能让ThreadLocal变量影响内存管理和CPU时间的正常使用。我们先从ThreadLocalMap开始：

1. `java.lang.ThreadLocal.ThreadLocalMap#INITIAL_CAPACITY`=16；
2. `threshold` is max entry list size；
3. 从`java.lang.ThreadLocal.ThreadLocalMap#set`，我们可以知道在size>threshold的时候调用`java.lang.ThreadLocal.ThreadLocalMap#rehash`，在rehash函数中的先`java.lang.ThreadLocal.ThreadLocalMap#expungeStaleEntries`。然后如果expunge后的entry list的size > 3/4 * threshold，会resize entry list size变成原来的2倍；

到此我们可以得出一点简单的结论——如果我们会一个作用域中宝比较多的ThreadLocal变量，对于其中的value不再有用的ThreadLocal变量，虽然在作用域结束的时候会自动GC它们，但要尽快的使用`java.lang.ThreadLocal#remove`去掉它，防止ThreadLocalMap的entry list size被撑大，多占用内存。还有如果ThreadLocal中保存是大对象，因为value被ThreadLocalMap中entry的引用，只有在对应entry被GC以后value才会被GC，而entry要在rehash中才会被释放后才能被GC, rehash只有在size>=threshold时才会触发，整个过程会十分漫长，为了更好的使用内存资源，也应该尽快的调用remove，使用entry尽快被GC。

再来说说一个关注比较多的点，那是因为使用ThreadLocal不当会造成的内存
泄漏的问题。对于这一块，我并没有遇到什么具体的实例。在实际的使用中，如果我们在Singleton对象中定义了ThreadLocal对象并不会出现什么问题，d在singleton中ThreadLocal变量也不大可能会被大量定义，比如在Spring中的singleton bean中。不过如果是在prototype这样会大量实例化的类中，就要注意及时的解引用甚至remove无用的ThreadLocal对象，一般来说，小心使用也不会有什么大的问题。

我原来看到有一些博文说在一些Web应用中在线程池中的Thread定义ThredLocal变量，因为Thread被复用，Thread没有被销毁和重新创建，使得ThreadLocalMap中的Entry因为没有达到回收的条件一直没有被回收，大量的线程数量导致存在的ThreadLocal中的value对象也就没有被回收，最后OOM。在我看来，这是对ThreadLocal理解不深导致的，在高并发的应用中对于大对象我们本来就应该重点关注，做到及时回收内存，这并不是ThreadLocal的设计问题，对于ThreadLocal的使用也不要因为有这个问题的存在避而远之。

## 总结

通过分析，我们可以知道，迟早的调用`java.lang.ThreadLocal#remove`可以避免很多问题，所以不要把释放对象占用内存的事情完全交给ThreadLocal来处理。