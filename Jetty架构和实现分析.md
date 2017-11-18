# Jetty架构和实现分析

[TOC]

## 架构分析

Jetty的架构，可以参见[Jetty 7 Architecture](https://wiki.eclipse.org/Jetty/Reference/Jetty_Architecture#Jetty_7_Architecture)，里面的内容包括了Jetty总体设计以及各个部分的实现方式。文档虽然有点儿旧，但是Jetty的架构设计基本没有什么大的变化，所以参考价值很高。

## 实现分析

### 代码组织方式

[eclipse/jetty.project](https://github.com/eclipse/jetty.project)使用Maven管理项目，一个主要功能都是项目的一个子模块。

```xml
<modules>
    <module>jetty-ant</module>
    <module>jetty-util</module>
    <module>jetty-jmx</module>
    <module>jetty-io</module>
    <module>jetty-http</module>
    <module>jetty-http2</module>
    <module>jetty-continuation</module>
    <module>jetty-server</module>
    ...
</modules>
```

我们分析Jetty的实现，从jetty-server这个模块入手。

## jetty-server分析

从Jetty的架构可知，`Server`主要包括几个部分：

1. **Connector**
2. **Handler**
3. **ThreadPool**

同时由下面的Server的一部分成员变量也可以清楚地知晓。

```java
@ManagedObject(value="Jetty HTTP Servlet server")
public class Server extends HandlerWrapper implements Attributes
{
    private static final Logger LOG = Log.getLogger(Server.class);

    private final AttributesMap _attributes = new AttributesMap();
    private final ThreadPool _threadPool;
    private final List<Connector> _connectors = new CopyOnWriteArrayList<>();
    private SessionIdManager _sessionIdManager;
    private boolean _stopAtShutdown;
    private boolean _dumpAfterStart=false;
    private boolean _dumpBeforeStop=false;
    private ErrorHandler _errorHandler;
    private RequestLog _requestLog;

    private final Locker _dateLocker = new Locker();
    private volatile DateField _dateField;
    ...
}
```

`org.eclipse.jetty.server.Server`就继承了`org.eclipse.jetty.server.handler.HandlerWrapper`，也是这个由Handler组成的Responsibility Chain的第一个环节。

在`org.eclipse.jetty.util.component.ContainerLifeCycle`中

```java
@ManagedObject("Implementation of Container and LifeCycle")
public class ContainerLifeCycle extends AbstractLifeCycle implements Container, Destroyable, Dumpable
{
    private static final Logger LOG = Log.getLogger(ContainerLifeCycle.class);
    private final List<Bean> _beans = new CopyOnWriteArrayList<>();
    private final List<Container.Listener> _listeners = new CopyOnWriteArrayList<>();
    private boolean _doStarted;
    private boolean _destroyed;
    ...
}
```

`_beans`可以统一保存和管理与Server相关的对象，包括Connector, Handler等等。

以Server开始的Handler Chain，传递给Connector，用于处理请求。其实这个程序的核心并在Server类，而在于Connector里面。

```java
/* ------------------------------------------------------------ */
/**
* Convenience constructor
* <p>
* Creates server and a {@link ServerConnector} at the passed address.
* @param addr the inet socket address to create the connector from
*/
public Server(@Name("address")InetSocketAddress addr)
{
  this((ThreadPool)null);
  
  // server is pass to connector when create.
  ServerConnector connector=new ServerConnector(this);
  connector.setHost(addr.getHostName());
  connector.setPort(addr.getPort());
  setConnectors(new Connector[]{connector});
}
```

在ServerConnector中会创建一个java.nio.channels.ServerSocketChannel，并监听一个端口。org.eclipse.jetty.server.ServerConnector.openAcceptChannel()代码如下：

```java
/**
* Called by {@link #open()} to obtain the accepting channel.
* @return ServerSocketChannel used to accept connections.
* @throws IOException
*/
protected ServerSocketChannel openAcceptChannel() throws IOException
{
    ServerSocketChannel serverChannel = null;
    if (isInheritChannel())
    {
        Channel channel = System.inheritedChannel();
        if (channel instanceof ServerSocketChannel)
            serverChannel = (ServerSocketChannel)channel;
        else
            LOG.warn("Unable to use System.inheritedChannel() [{}]. Trying a new ServerSocketChannel at {}:{}", channel, getHost(), getPort());
    }

    if (serverChannel == null)
    {
        serverChannel = ServerSocketChannel.open();

        InetSocketAddress bindAddress = getHost() == null ? new InetSocketAddress(getPort()) : new InetSocketAddress(getHost(), getPort());
        serverChannel.socket().setReuseAddress(getReuseAddress());
        serverChannel.socket().bind(bindAddress, getAcceptQueueSize());
    }

    return serverChannel;
}
```

在请求进入的时候，会把请求的SocketChannel交给SelectorManager，等待处理，`org.eclipse.jetty.server.ServerConnector.accepted(SocketChannel)`代码：

```java
@Override
public void accept(int acceptorID) throws IOException
{
    ServerSocketChannel serverChannel = _acceptChannel;
    if (serverChannel != null && serverChannel.isOpen())
    {
        SocketChannel channel = serverChannel.accept();
        accepted(channel);
    }
}

private void accepted(SocketChannel channel) throws IOException
{
    channel.configureBlocking(false);
    Socket socket = channel.socket();
    configure(socket);
    _manager.accept(channel);
}
```

在SelectorManager接到SocketChannel，会选择一个`org.eclipse.jetty.io.ManagedSelector`给它，`org.eclipse.jetty.io.SelectorManager.accept(SelectableChannel, Object)`代码如下：

```java
/**
* <p>Registers a channel to perform non-blocking read/write operations.</p>
* <p>This method is called just after a channel has been accepted by {@link ServerSocketChannel#accept()},
* or just after having performed a blocking connect via {@link Socket#connect(SocketAddress, int)}, or
* just after a non-blocking connect via {@link SocketChannel#connect(SocketAddress)} that completed
* successfully.</p>
*
* @param channel    the channel to register
* @param attachment the attachment object
*/
public void accept(SelectableChannel channel, Object attachment)
{
    final ManagedSelector selector = chooseSelector(channel);
    selector.submit(selector.new Accept(channel, attachment));
}
```

在ManagedSelector中，会创建一个Action，并放进一个Queue中。`org.eclipse.jetty.io.ManagedSelector.submit(Runnable)`代码如下

```java
public void submit(Runnable change)
{
    if (LOG.isDebugEnabled())
        LOG.debug("Queued change {} on {}", change, this);

    Selector selector = null;
    try (Locker.Lock lock = _locker.lock())
    {
        _actions.offer(change);
        
        if (_selecting)
        {
            selector = _selector;
            // To avoid the extra select wakeup.
            _selecting = false;
        }
    }
    if (selector != null)
        selector.wakeup();
}
```

到目前为止，请求处理的准备工作已经做完了，现在就只是等待处理了。处理过程的核心代码如下：

`org.eclipse.jetty.io.ManagedSelector.doStart()`启动了一个ManagedSelector的请求调度线程。

```java
@Override
protected void doStart() throws Exception
{
    super.doStart();

    _selector = _selectorManager.newSelector();

    // The producer used by the strategies will never
    // be idle (either produces a task or blocks).

    // The normal strategy obtains the produced task, schedules
    // a new thread to produce more, runs the task and then exits.
    _selectorManager.execute(_strategy::produce);
}
```

这个调度线程使用`org.eclipse.jetty.io.ManagedSelector.SelectorProducer`取得一个请求处理的Task，在ThreadPool中处理。`org.eclipse.jetty.util.thread.strategy.EatWhatYouKill.doProduce()`代码如下：

```java
public boolean doProduce()
{
    boolean producing = true;
    while (isRunning() && producing)
    {
        // If we got here, then we are the thread that is producing.
        Runnable task = null;
        try
        {
            // get the task from ManagedSelector
            task = _producer.produce();
        }
        catch(Throwable e)
        {
            LOG.warn(e);
        }

        if (LOG.isDebugEnabled())
            LOG.debug("{} t={}/{}",this,task,Invocable.getInvocationType(task));

        if (task==null)
        {
            try (Lock locked = _locker.lock())
            {
                // Could another one just have been queued with a produce call?
                if (_state==State.REPRODUCING)
                {
                    _state = State.PRODUCING;
                }
                else
                {
                    if (LOG.isDebugEnabled())
                        LOG.debug("{} IDLE",toStringLocked());
                    _state = State.IDLE;
                    producing = false;
                }
            }
        }
        else
        {
            boolean consume;
            if (Invocable.getInvocationType(task) == InvocationType.NON_BLOCKING)
            {
                // PRODUCE CONSUME (EWYK!)
                if (LOG.isDebugEnabled())
                    LOG.debug("{} PC t={}", this, task);
                consume = true;
                _nonBlocking.increment();
            }
            else
            {
                try (Lock locked = _locker.lock())
                {
                    if (_producers.tryExecute(this))
                    {
                        // EXECUTE PRODUCE CONSUME!
                        // We have executed a new Producer, so we can EWYK consume
                        _state = State.IDLE;
                        producing = false;
                        consume = true;
                        _blocking.increment();
                    }
                    else
                    {
                        // PRODUCE EXECUTE CONSUME!
                        consume = false;
                        _executed.increment();
                    }
                }

                if (LOG.isDebugEnabled())
                    LOG.debug("{} {} t={}", this, consume ? "EPC" : "PEC", task);
            }

            // Consume or execute task
            try
            {
                if (consume)
                    task.run();
                else
                    _executor.execute(task);
            }
            catch (RejectedExecutionException e)
            {
                LOG.warn(e);
                if (task instanceof Closeable)
                {
                    try
                    {
                        ((Closeable)task).close();
                    }
                    catch (Throwable e2)
                    {
                        LOG.ignore(e2);
                    }
                }
            }
            catch (Throwable e)
            {
                LOG.warn(e);
            }
        }
    }

    return producing;
}
```

至此，jetty-server实现的主线已经基本理清了。