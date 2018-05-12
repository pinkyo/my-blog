# Object类的分析

[TOC]

在Java中，Object类是所有类的根类，对于这个类的了解的比较多的只有hashCode, equals和toString这几个方法。

从这个类的定义来看：

``` java
/**
 * Class {@code Object} is the root of the class hierarchy.
 * Every class has {@code Object} as a superclass. All objects,
 * including arrays, implement the methods of this class.
 *
 * @author  unascribed
 * @see     java.lang.Class
 * @since   1.0
 */
public class Object {
```

字面理解：Object是所有类的超类。所有对象包括数组都实现Object的方法。

这里有个问题：为什么数组也实现Obect的方法呢？

我们从Java Language Specification的Arrays章开头可以看到：

``` java
IN the Java programming language, arrays are objects (§4.3.1), are dynamically
created, and may be assigned to variables of type Object
```

所以array的实例也是对象，这样就说得通了。

再来看看Object的几个方法:

1. getClass

```java
    /**
     * Returns the runtime class of this {@code Object}. The returned
     * {@code Class} object is the object that is locked by {@code
     * static synchronized} methods of the represented class.
     *
     * <p><b>The actual result type is {@code Class<? extends |X|>}
     * where {@code |X|} is the erasure of the static type of the
     * expression on which {@code getClass} is called.</b> For
     * example, no cast is required in this code fragment:</p>
     *
     * <p>
     * {@code Number n = 0;                             }<br>
     * {@code Class<? extends Number> c = n.getClass(); }
     * </p>
     *
     * @return The {@code Class} object that represents the runtime
     *         class of this object.
     * @jls 15.8.2 Class Literals
     */
    @HotSpotIntrinsicCandidate
    public final native Class<?> getClass();
```

这个方法没有什么说的，但是`The actual result type is {@code Class<? extends |X|>} where {@code |X|} is the erasure of the static type of the expression on which {@code getClass} is called.`。所以在使用的时候要注意，不要直接使用Raw type也就是Class作为返回值，这样做虽然不会有大的问题，但不是好的编程习惯。

2. hashCode

```java
    /**
     * Returns a hash code value for the object. This method is
     * supported for the benefit of hash tables such as those provided by
     * {@link java.util.HashMap}.
     * <p>
     * The general contract of {@code hashCode} is:
     * <ul>
     * <li>Whenever it is invoked on the same object more than once during
     *     an execution of a Java application, the {@code hashCode} method
     *     must consistently return the same integer, provided no information
     *     used in {@code equals} comparisons on the object is modified.
     *     This integer need not remain consistent from one execution of an
     *     application to another execution of the same application.
     * <li>If two objects are equal according to the {@code equals(Object)}
     *     method, then calling the {@code hashCode} method on each of
     *     the two objects must produce the same integer result.
     * <li>It is <em>not</em> required that if two objects are unequal
     *     according to the {@link java.lang.Object#equals(java.lang.Object)}
     *     method, then calling the {@code hashCode} method on each of the
     *     two objects must produce distinct integer results.  However, the
     *     programmer should be aware that producing distinct integer results
     *     for unequal objects may improve the performance of hash tables.
     * </ul>
     * <p>
     * As much as is reasonably practical, the hashCode method defined
     * by class {@code Object} does return distinct integers for
     * distinct objects. (The hashCode may or may not be implemented
     * as some function of an object's memory address at some point
     * in time.)
     *
     * @return  a hash code value for this object.
     * @see     java.lang.Object#equals(java.lang.Object)
     * @see     java.lang.System#identityHashCode
     */
    @HotSpotIntrinsicCandidate
    public native int hashCode();
```

我们注意看注释，可以看到说的：两个对象equals，则两个对象的hashCode值一定相等；但两个对象的hashCode值相等，两个对象不要求一定要equals。但是不同的对象产生不同的hashCode可以提高hash tables的性能。

3. equals

```java
    /**
     * Indicates whether some other object is "equal to" this one.
     * <p>
     * The {@code equals} method implements an equivalence relation
     * on non-null object references:
     * <ul>
     * <li>It is <i>reflexive</i>: for any non-null reference value
     *     {@code x}, {@code x.equals(x)} should return
     *     {@code true}.
     * <li>It is <i>symmetric</i>: for any non-null reference values
     *     {@code x} and {@code y}, {@code x.equals(y)}
     *     should return {@code true} if and only if
     *     {@code y.equals(x)} returns {@code true}.
     * <li>It is <i>transitive</i>: for any non-null reference values
     *     {@code x}, {@code y}, and {@code z}, if
     *     {@code x.equals(y)} returns {@code true} and
     *     {@code y.equals(z)} returns {@code true}, then
     *     {@code x.equals(z)} should return {@code true}.
     * <li>It is <i>consistent</i>: for any non-null reference values
     *     {@code x} and {@code y}, multiple invocations of
     *     {@code x.equals(y)} consistently return {@code true}
     *     or consistently return {@code false}, provided no
     *     information used in {@code equals} comparisons on the
     *     objects is modified.
     * <li>For any non-null reference value {@code x},
     *     {@code x.equals(null)} should return {@code false}.
     * </ul>
     * <p>
     * The {@code equals} method for class {@code Object} implements
     * the most discriminating possible equivalence relation on objects;
     * that is, for any non-null reference values {@code x} and
     * {@code y}, this method returns {@code true} if and only
     * if {@code x} and {@code y} refer to the same object
     * ({@code x == y} has the value {@code true}).
     * <p>
     * Note that it is generally necessary to override the {@code hashCode}
     * method whenever this method is overridden, so as to maintain the
     * general contract for the {@code hashCode} method, which states
     * that equal objects must have equal hash codes.
     *
     * @param   obj   the reference object with which to compare.
     * @return  {@code true} if this object is the same as the obj
     *          argument; {@code false} otherwise.
     * @see     #hashCode()
     * @see     java.util.HashMap
     */
    public boolean equals(Object obj) {
        return (this == obj);
    }
```

这个函数的默认实现是直接比较两个引用是不是指向同一个对象，实际的作用与==没有什么两样。

覆写equals函数的要求。设a、b、c是三个非null对象,:

- 自反性。a.equals(a)恒为true;
- 对称性。a.equals(b)和b.equals(a)的结果恒相等；
- 传递性。a.equals(b)为true并且b.equals(a)为true,则a.equals(c)为true;
- 一致性。a.equals(b)只能恒为true或false,不能一会儿为true，一会儿为false;

对于所有非null对象a，a.equals(null)恒为false。

有这些限制的存在，我们就不会特别去处理超类和子类的相等关系，只要类型不同，就返回false。在《Effectiva Java》里建议如果覆写equals，要写足够的测试。并且覆写equals函数的同时，也要将hashCode一起覆写。

4. toString

```java
    /**
     * Returns a string representation of the object. In general, the
     * {@code toString} method returns a string that
     * "textually represents" this object. The result should
     * be a concise but informative representation that is easy for a
     * person to read.
     * It is recommended that all subclasses override this method.
     * <p>
     * The {@code toString} method for class {@code Object}
     * returns a string consisting of the name of the class of which the
     * object is an instance, the at-sign character `{@code @}', and
     * the unsigned hexadecimal representation of the hash code of the
     * object. In other words, this method returns a string equal to the
     * value of:
     * <blockquote>
     * <pre>
     * getClass().getName() + '@' + Integer.toHexString(hashCode())
     * </pre></blockquote>
     *
     * @return  a string representation of the object.
     */
    public String toString() {
        return getClass().getName() + "@" + Integer.toHexString(hashCode());
    }
```

默认返回的是“类名@hashCode”(PS, 我原来以为是内存的地址)。一般建议我们自己的类都要覆写这个方法，也是为也方便自己开发的时候调试。

4. notify

```java
    /**
     * Wakes up a single thread that is waiting on this object's
     * monitor. If any threads are waiting on this object, one of them
     * is chosen to be awakened. The choice is arbitrary and occurs at
     * the discretion of the implementation. A thread waits on an object's
     * monitor by calling one of the {@code wait} methods.
     * <p>
     * The awakened thread will not be able to proceed until the current
     * thread relinquishes the lock on this object. The awakened thread will
     * compete in the usual manner with any other threads that might be
     * actively competing to synchronize on this object; for example, the
     * awakened thread enjoys no reliable privilege or disadvantage in being
     * the next thread to lock this object.
     * <p>
     * This method should only be called by a thread that is the owner
     * of this object's monitor. A thread becomes the owner of the
     * object's monitor in one of three ways:
     * <ul>
     * <li>By executing a synchronized instance method of that object.
     * <li>By executing the body of a {@code synchronized} statement
     *     that synchronizes on the object.
     * <li>For objects of type {@code Class,} by executing a
     *     synchronized static method of that class.
     * </ul>
     * <p>
     * Only one thread at a time can own an object's monitor.
     *
     * @throws  IllegalMonitorStateException  if the current thread is not
     *               the owner of this object's monitor.
     * @see        java.lang.Object#notifyAll()
     * @see        java.lang.Object#wait()
     */
    @HotSpotIntrinsicCandidate
    public final native void notify();
```

看注释的意思是在wait的时候会暂时将对象的monitor释放掉，notify方法会唤醒线程，让它重新去获取monitor,在获取成功之前不会继续执行。

5. notifyAll

```java
    /**
     * Wakes up all threads that are waiting on this object's monitor. A
     * thread waits on an object's monitor by calling one of the
     * {@code wait} methods.
     * <p>
     * The awakened threads will not be able to proceed until the current
     * thread relinquishes the lock on this object. The awakened threads
     * will compete in the usual manner with any other threads that might
     * be actively competing to synchronize on this object; for example,
     * the awakened threads enjoy no reliable privilege or disadvantage in
     * being the next thread to lock this object.
     * <p>
     * This method should only be called by a thread that is the owner
     * of this object's monitor. See the {@code notify} method for a
     * description of the ways in which a thread can become the owner of
     * a monitor.
     *
     * @throws  IllegalMonitorStateException  if the current thread is not
     *               the owner of this object's monitor.
     * @see        java.lang.Object#notify()
     * @see        java.lang.Object#wait()
     */
    @HotSpotIntrinsicCandidate
    public final native void notifyAll();
```

这个方法与notify的作用一样，只是相对于notify只唤醒一个线程，notifyAll会唤醒所有的线程。

6. wait

```java
    /**
     * Causes the current thread to wait until either another thread invokes the
     * {@link java.lang.Object#notify()} method or the
     * {@link java.lang.Object#notifyAll()} method for this object, or a
     * specified amount of time has elapsed.
     * <p>
     * The current thread must own this object's monitor.
     * <p>
     * This method causes the current thread (call it <var>T</var>) to
     * place itself in the wait set for this object and then to relinquish
     * any and all synchronization claims on this object. Thread <var>T</var>
     * becomes disabled for thread scheduling purposes and lies dormant
     * until one of four things happens:
     * <ul>
     * <li>Some other thread invokes the {@code notify} method for this
     * object and thread <var>T</var> happens to be arbitrarily chosen as
     * the thread to be awakened.
     * <li>Some other thread invokes the {@code notifyAll} method for this
     * object.
     * <li>Some other thread {@linkplain Thread#interrupt() interrupts}
     * thread <var>T</var>.
     * <li>The specified amount of real time has elapsed, more or less.  If
     * {@code timeout} is zero, however, then real time is not taken into
     * consideration and the thread simply waits until notified.
     * </ul>
     * The thread <var>T</var> is then removed from the wait set for this
     * object and re-enabled for thread scheduling. It then competes in the
     * usual manner with other threads for the right to synchronize on the
     * object; once it has gained control of the object, all its
     * synchronization claims on the object are restored to the status quo
     * ante - that is, to the situation as of the time that the {@code wait}
     * method was invoked. Thread <var>T</var> then returns from the
     * invocation of the {@code wait} method. Thus, on return from the
     * {@code wait} method, the synchronization state of the object and of
     * thread {@code T} is exactly as it was when the {@code wait} method
     * was invoked.
     * <p>
     * A thread can also wake up without being notified, interrupted, or
     * timing out, a so-called <i>spurious wakeup</i>.  While this will rarely
     * occur in practice, applications must guard against it by testing for
     * the condition that should have caused the thread to be awakened, and
     * continuing to wait if the condition is not satisfied.  In other words,
     * waits should always occur in loops, like this one:
     * <pre>
     *     synchronized (obj) {
     *         while (&lt;condition does not hold&gt;)
     *             obj.wait(timeout);
     *         ... // Perform action appropriate to condition
     *     }
     * </pre>
     *
     * (For more information on this topic, see section 14.2,
     * Condition Queues, in Brian Goetz and others' "Java Concurrency
     * in Practice" (Addison-Wesley, 2006) or Item 69 in Joshua
     * Bloch's "Effective Java (Second Edition)" (Addison-Wesley,
     * 2008).
     *
     * <p>If the current thread is {@linkplain java.lang.Thread#interrupt()
     * interrupted} by any thread before or while it is waiting, then an
     * {@code InterruptedException} is thrown.  This exception is not
     * thrown until the lock status of this object has been restored as
     * described above.
     *
     * <p>
     * Note that the {@code wait} method, as it places the current thread
     * into the wait set for this object, unlocks only this object; any
     * other objects on which the current thread may be synchronized remain
     * locked while the thread waits.
     * <p>
     * This method should only be called by a thread that is the owner
     * of this object's monitor. See the {@code notify} method for a
     * description of the ways in which a thread can become the owner of
     * a monitor.
     *
     * @param      timeout   the maximum time to wait in milliseconds.
     * @throws  IllegalArgumentException      if the value of timeout is
     *               negative.
     * @throws  IllegalMonitorStateException  if the current thread is not
     *               the owner of the object's monitor.
     * @throws  InterruptedException if any thread interrupted the
     *             current thread before or while the current thread
     *             was waiting for a notification.  The <i>interrupted
     *             status</i> of the current thread is cleared when
     *             this exception is thrown.
     * @see        java.lang.Object#notify()
     * @see        java.lang.Object#notifyAll()
     */
    public final native void wait(long timeout) throws InterruptedException;

    /**
     * Causes the current thread to wait until another thread invokes the
     * {@link java.lang.Object#notify()} method or the
     * {@link java.lang.Object#notifyAll()} method for this object, or
     * some other thread interrupts the current thread, or a certain
     * amount of real time has elapsed.
     * <p>
     * This method is similar to the {@code wait} method of one
     * argument, but it allows finer control over the amount of time to
     * wait for a notification before giving up. The amount of real time,
     * measured in nanoseconds, is given by:
     * <blockquote>
     * <pre>
     * 1000000*timeout+nanos</pre></blockquote>
     * <p>
     * In all other respects, this method does the same thing as the
     * method {@link #wait(long)} of one argument. In particular,
     * {@code wait(0, 0)} means the same thing as {@code wait(0)}.
     * <p>
     * The current thread must own this object's monitor. The thread
     * releases ownership of this monitor and waits until either of the
     * following two conditions has occurred:
     * <ul>
     * <li>Another thread notifies threads waiting on this object's monitor
     *     to wake up either through a call to the {@code notify} method
     *     or the {@code notifyAll} method.
     * <li>The timeout period, specified by {@code timeout}
     *     milliseconds plus {@code nanos} nanoseconds arguments, has
     *     elapsed.
     * </ul>
     * <p>
     * The thread then waits until it can re-obtain ownership of the
     * monitor and resumes execution.
     * <p>
     * As in the one argument version, interrupts and spurious wakeups are
     * possible, and this method should always be used in a loop:
     * <pre>
     *     synchronized (obj) {
     *         while (&lt;condition does not hold&gt;)
     *             obj.wait(timeout, nanos);
     *         ... // Perform action appropriate to condition
     *     }
     * </pre>
     * This method should only be called by a thread that is the owner
     * of this object's monitor. See the {@code notify} method for a
     * description of the ways in which a thread can become the owner of
     * a monitor.
     *
     * @param      timeout   the maximum time to wait in milliseconds.
     * @param      nanos      additional time, in nanoseconds range
     *                       0-999999.
     * @throws  IllegalArgumentException      if the value of timeout is
     *                      negative or the value of nanos is
     *                      not in the range 0-999999.
     * @throws  IllegalMonitorStateException  if the current thread is not
     *               the owner of this object's monitor.
     * @throws  InterruptedException if any thread interrupted the
     *             current thread before or while the current thread
     *             was waiting for a notification.  The <i>interrupted
     *             status</i> of the current thread is cleared when
     *             this exception is thrown.
     */
    public final void wait(long timeout, int nanos) throws InterruptedException {
        if (timeout < 0) {
            throw new IllegalArgumentException("timeout value is negative");
        }

        if (nanos < 0 || nanos > 999999) {
            throw new IllegalArgumentException(
                                "nanosecond timeout value out of range");
        }

        if (nanos > 0) {
            timeout++;
        }

        wait(timeout);
    }

    /**
     * Causes the current thread to wait until another thread invokes the
     * {@link java.lang.Object#notify(Causes the current thread to wait)} method or the
     * {@link java.lang.Object#notifyAll()} method for this object.
     * In other words, this method behaves exactly as if it simply
     * performs the call {@code wait(0)}.
     * <p>
     * The current thread must own this object's monitor. The thread
     * releases ownership of this monitor and waits until another thread
     * notifies threads waiting on this object's monitor to wake up
     * either through a call to the {@code notify} method or the
     * {@code notifyAll} method. The thread then waits until it can
     * re-obtain ownership of the monitor and resumes execution.
     * <p>
     * As in the one argument version, interrupts and spurious wakeups are
     * possible, and this method should always be used in a loop:
     * <pre>
     *     synchronized (obj) {
     *         while (&lt;condition does not hold&gt;)
     *             obj.wait();
     *         ... // Perform action appropriate to condition
     *     }
     * </pre>
     * This method should only be called by a thread that is the owner
     * of this object's monitor. See the {@code notify} method for a
     * description of the ways in which a thread can become the owner of
     * a monitor.
     *
     * @throws  IllegalMonitorStateException  if the current thread is not
     *               the owner of the object's monitor.
     * @throws  InterruptedException if any thread interrupted the
     *             current thread before or while the current thread
     *             was waiting for a notification.  The <i>interrupted
     *             status</i> of the current thread is cleared when
     *             this exception is thrown.
     * @see        java.lang.Object#notify()
     * @see        java.lang.Object#notifyAll()
     */
    public final void wait() throws InterruptedException {
        wait(0);
    }
```

这些wait的方法的作用都是让线程等待。

7. finalize

```java
    /**
     * Called by the garbage collector on an object when garbage collection
     * determines that there are no more references to the object.
     * A subclass overrides the {@code finalize} method to dispose of
     * system resources or to perform other cleanup.
     * <p>
     * The general contract of {@code finalize} is that it is invoked
     * if and when the Java&trade; virtual
     * machine has determined that there is no longer any
     * means by which this object can be accessed by any thread that has
     * not yet died, except as a result of an action taken by the
     * finalization of some other object or class which is ready to be
     * finalized. The {@code finalize} method may take any action, including
     * making this object available again to other threads; the usual purpose
     * of {@code finalize}, however, is to perform cleanup actions before
     * the object is irrevocably discarded. For example, the finalize method
     * for an object that represents an input/output connection might perform
     * explicit I/O transactions to break the connection before the object is
     * permanently discarded.
     * <p>
     * The {@code finalize} method of class {@code Object} performs no
     * special action; it simply returns normally. Subclasses of
     * {@code Object} may override this definition.
     * <p>
     * The Java programming language does not guarantee which thread will
     * invoke the {@code finalize} method for any given object. It is
     * guaranteed, however, that the thread that invokes finalize will not
     * be holding any user-visible synchronization locks when finalize is
     * invoked. If an uncaught exception is thrown by the finalize method,
     * the exception is ignored and finalization of that object terminates.
     * <p>
     * After the {@code finalize} method has been invoked for an object, no
     * further action is taken until the Java virtual machine has again
     * determined that there is no longer any means by which this object can
     * be accessed by any thread that has not yet died, including possible
     * actions by other objects or classes which are ready to be finalized,
     * at which point the object may be discarded.
     * <p>
     * The {@code finalize} method is never invoked more than once by a Java
     * virtual machine for any given object.
     * <p>
     * Any exception thrown by the {@code finalize} method causes
     * the finalization of this object to be halted, but is otherwise
     * ignored.
     *
     * @apiNote
     * Classes that embed non-heap resources have many options
     * for cleanup of those resources. The class must ensure that the
     * lifetime of each instance is longer than that of any resource it embeds.
     * {@link java.lang.ref.Reference#reachabilityFence} can be used to ensure that
     * objects remain reachable while resources embedded in the object are in use.
     * <p>
     * A subclass should avoid overriding the {@code finalize} method
     * unless the subclass embeds non-heap resources that must be cleaned up
     * before the instance is collected.
     * Finalizer invocations are not automatically chained, unlike constructors.
     * If a subclass overrides {@code finalize} it must invoke the superclass
     * finalizer explicitly.
     * To guard against exceptions prematurely terminating the finalize chain,
     * the subclass should use a {@code try-finally} block to ensure
     * {@code super.finalize()} is always invoked. For example,
     * <pre>{@code      @Override
     *     protected void finalize() throws Throwable {
     *         try {
     *             ... // cleanup subclass state
     *         } finally {
     *             super.finalize();
     *         }
     *     }
     * }</pre>
     *
     * @deprecated The finalization mechanism is inherently problematic.
     * Finalization can lead to performance issues, deadlocks, and hangs.
     * Errors in finalizers can lead to resource leaks; there is no way to cancel
     * finalization if it is no longer necessary; and no ordering is specified
     * among calls to {@code finalize} methods of different objects.
     * Furthermore, there are no guarantees regarding the timing of finalization.
     * The {@code finalize} method might be called on a finalizable object
     * only after an indefinite delay, if at all.
     *
     * Classes whose instances hold non-heap resources should provide a method
     * to enable explicit release of those resources, and they should also
     * implement {@link AutoCloseable} if appropriate.
     * The {@link java.lang.ref.Cleaner} and {@link java.lang.ref.PhantomReference}
     * provide more flexible and efficient ways to release resources when an object
     * becomes unreachable.
     *
     * @throws Throwable the {@code Exception} raised by this method
     * @see java.lang.ref.WeakReference
     * @see java.lang.ref.PhantomReference
     * @jls 12.6 Finalization of Class Instances
     */
    @Deprecated(since="9")
    protected void finalize() throws Throwable { }
```

Java Programming Language不保证线程一定会执行对象的finalize方法。但是只要执行了finalize方法，就可以保证线程不会持用户可见的有这个对象的同步锁。

Finalization会可能造成性能问题、死锁和挂起的问题，finalizer里而的错误可能会导致资源泄漏。

## 还有几点有意思的：

1.

```java
    /* Make sure registerNatives is the first thing <clinit> does. */
    private static native void registerNatives();
    static {
        registerNatives();
    }
```
有native函数存在的类都会先做这个操作。

2. 

```java
/**
 * The {@code @HotSpotIntrinsicCandidate} annotation is specific to the
 * HotSpot Virtual Machine. It indicates that an annotated method
 * may be (but is not guaranteed to be) intrinsified by the HotSpot VM. A method
 * is intrinsified if the HotSpot VM replaces the annotated method with hand-written
 * assembly and/or hand-written compiler IR -- a compiler intrinsic -- to improve
 * performance. The {@code @HotSpotIntrinsicCandidate} annotation is internal to the
 * Java libraries and is therefore not supposed to have any relevance for application
 * code.
 *
 * Maintainers of the Java libraries must consider the following when
 * modifying methods annotated with {@code @HotSpotIntrinsicCandidate}.
 *
 * <ul>
 * <li>When modifying a method annotated with {@code @HotSpotIntrinsicCandidate},
 * the corresponding intrinsic code in the HotSpot VM implementation must be
 * updated to match the semantics of the annotated method.</li>
 * <li>For some annotated methods, the corresponding intrinsic may omit some low-level
 * checks that would be performed as a matter of course if the intrinsic is implemented
 * using Java bytecodes. This is because individual Java bytecodes implicitly check
 * for exceptions like {@code NullPointerException} and {@code ArrayStoreException}.
 * If such a method is replaced by an intrinsic coded in assembly language, any
 * checks performed as a matter of normal bytecode operation must be performed
 * before entry into the assembly code. These checks must be performed, as
 * appropriate, on all arguments to the intrinsic, and on other values (if any) obtained
 * by the intrinsic through those arguments. The checks may be deduced by inspecting
 * the non-intrinsic Java code for the method, and determining exactly which exceptions
 * may be thrown by the code, including undeclared implicit {@code RuntimeException}s.
 * Therefore, depending on the data accesses performed by the intrinsic,
 * the checks may include:
 *
 *  <ul>
 *  <li>null checks on references</li>
 *  <li>range checks on primitive values used as array indexes</li>
 *  <li>other validity checks on primitive values (e.g., for divide-by-zero conditions)</li>
 *  <li>store checks on reference values stored into arrays</li>
 *  <li>array length checks on arrays indexed from within the intrinsic</li>
 *  <li>reference casts (when formal parameters are {@code Object} or some other weak type)</li>
 *  </ul>
 *
 * </li>
 *
 * <li>Note that the receiver value ({@code this}) is passed as a extra argument
 * to all non-static methods. If a non-static method is an intrinsic, the receiver
 * value does not need a null check, but (as stated above) any values loaded by the
 * intrinsic from object fields must also be checked. As a matter of clarity, it is
 * better to make intrinisics be static methods, to make the dependency on {@code this}
 * clear. Also, it is better to explicitly load all required values from object
 * fields before entering the intrinsic code, and pass those values as explicit arguments.
 * First, this may be necessary for null checks (or other checks). Second, if the
 * intrinsic reloads the values from fields and operates on those without checks,
 * race conditions may be able to introduce unchecked invalid values into the intrinsic.
 * If the intrinsic needs to store a value back to an object field, that value should be
 * returned explicitly from the intrinsic; if there are multiple return values, coders
 * should consider buffering them in an array. Removing field access from intrinsics
 * not only clarifies the interface with between the JVM and JDK; it also helps decouple
 * the HotSpot and JDK implementations, since if JDK code before and after the intrinsic
 * manages all field accesses, then intrinsics can be coded to be agnostic of object
 * layouts.</li>
 *
 * Maintainers of the HotSpot VM must consider the following when modifying
 * intrinsics.
 *
 * <ul>
 * <li>When adding a new intrinsic, make sure that the corresponding method
 * in the Java libraries is annotated with {@code @HotSpotIntrinsicCandidate}
 * and that all possible call sequences that result in calling the intrinsic contain
 * the checks omitted by the intrinsic (if any).</li>
 * <li>When modifying an existing intrinsic, the Java libraries must be updated
 * to match the semantics of the intrinsic and to execute all checks omitted
 * by the intrinsic (if any).</li>
 * </ul>
 *
 * Persons not directly involved with maintaining the Java libraries or the
 * HotSpot VM can safely ignore the fact that a method is annotated with
 * {@code @HotSpotIntrinsicCandidate}.
 *
 * The HotSpot VM defines (internally) a list of intrinsics. Not all intrinsic
 * are available on all platforms supported by the HotSpot VM. Furthermore,
 * the availability of an intrinsic on a given platform depends on the
 * configuration of the HotSpot VM (e.g., the set of VM flags enabled).
 * Therefore, annotating a method with {@code @HotSpotIntrinsicCandidate} does
 * not guarantee that the marked method is intrinsified by the HotSpot VM.
 *
 * If the {@code CheckIntrinsics} VM flag is enabled, the HotSpot VM checks
 * (when loading a class) that (1) all methods of that class that are also on
 * the VM's list of intrinsics are annotated with {@code @HotSpotIntrinsicCandidate}
 * and that (2) for all methods of that class annotated with
 * {@code @HotSpotIntrinsicCandidate} there is an intrinsic in the list.
 *
 * @since 9
 */
@Target({ElementType.METHOD, ElementType.CONSTRUCTOR})
@Retention(RetentionPolicy.RUNTIME)
public @interface HotSpotIntrinsicCandidate {
}
```

居然还可以这样:)

## Q&A

Thread.sleep()和wait有什么区别？

答案：

```java
    /**
     * Causes the currently executing thread to sleep (temporarily cease
     * execution) for the specified number of milliseconds, subject to
     * the precision and accuracy of system timers and schedulers. The thread
     * does not lose ownership of any monitors.
     *
     * @param  millis
     *         the length of time to sleep in milliseconds
     *
     * @throws  IllegalArgumentException
     *          if the value of {@code millis} is negative
     *
     * @throws  InterruptedException
     *          if any thread has interrupted the current thread. The
     *          <i>interrupted status</i> of the current thread is
     *          cleared when this exception is thrown.
     */
    public static native void sleep(long millis) throws InterruptedException;
```

从Thread.sleep的方法的注释知道:

- sleep不会释放任何的monitor,但前面我们已经知道wait会释放自己的monitor；
- sleep会根据系统的计时器和调试器来自动唤醒线程，而wait不会。

不过从上面，也可以看到，这两个方法都是调用native的库的来实现，对于线程调度这样的事情也只能这么做。