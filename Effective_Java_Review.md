# 《Effective Java》Review

[TOC]

## 前言

说实话，我没有写读后感的习惯，一直都是书读完了，就没有下文有。有什么想法，也在读书的过程产生，在读完后逐渐遗忘。实际上，我也认可一本好的书是值得反复研读，细细琢磨的，这也是我现在为何想写点儿Review的原因。《Effective Java》是本好的技术类书籍，我读的第二版基于Jdk 1.5，虽然有点儿老但是很实用，我从中学到不少东西。

## Review

这个Review是以本书的组织方式来进行的。

### Creating and Destroying Objects

- Item 1: Consider static factory methods instead of constructors

使用静态工厂函数来创建新对象有以下的好处:

1. 静态工厂函数有名字，但是构造函数没有，名字可以增加代码的可读性；
2. 静态工厂函数不像构造函数一定会创建一个新的对象，比较灵活，比如可以使用在单例模式，Immutable对象的情况；
3. 静态工厂函数可以返回返回类型的任一子类实例；
4. 使用静态工厂函数可以使代码更简洁；

使用静态工厂函数来创建新对象有以下的坏处:

1. 只有静态工厂函数，但没有public或者protected的构造函数的类不能用于继承；
2. 静态工厂函数与其他的静态函数不能很好地区分，要区分只能依靠一些习惯的命名如valueOf、of、getInstance等来实现。

- Item 2: Consider a builder when faced with many constructor parameters

参数很多的情况可以考虑使用builder模式，在参数中，我们可以把必填的参数放在builder的构造函数的参数中，其他实现单独的setter函数。

- Item 3: Enforce the singleton property with a private constructor or an enum type

单例使用会使测试变得困难，因为单例对象很难mock出来，除非有实现其他的interface.
单元素的Enum类型是实现单例最好的函数。

- Item 4: Enforce noninstantiability with a private constructor

工具类只能有私有构造函数，不要让它可以实例化。

- Item 5: Avoid creating unnecessary objects

不能创建没用的对象，因为会影响性能。在一般情况下，优先使用primitives，而不是boxed primitives，小心不必要的autoboxing。
不能在使用过程中，我发现偶尔会出现参数optional的情况，这个时候感觉boxed primitives还是可以用的。

- Item 6: Eliminate obsolete object references

废弃的引用要及时置为null，不然会有内存泄漏的风险，比如数组对象，caches, listeners，callbacks等情况。

- Item 7: Avoid finalizers

Finalizer函数不安全，不可预见，一般来说也没有必要，还会影响性能。`不要使用！`

### Methods Common to All Objects

- Item 8: Obey the general contract when overriding equals

以下情况不要override equals函数：

1. 类的实例本来就是独一无二的；
2. 你不关心是否“逻辑相等”；
3. 超类已经override equals函数，而且仍然适用；
4. private类或package-private类，而且你确定equals函数不会被调用。

override equals函数有如下协定：

1. 自反性，X.equals(X)要恒为true;
2. 对称性，X.equals(Y)为true,当且仅当Y.equals(X)为true;
3. 传递性，如果X.equals(Y)为true并且Y.equals(X)为御前,则X.equals(Z)为true;
4. 一致性，X.equals(Y)只能一直为true或flase，不能一会儿为true，一会儿为false；
5. 对任意的非null引用X, X.equals(null)必须返回false。

尽量不要违反这些协定，不然你没法预测其行为。在override equals函数的时候也不要画蛇添足与超类的逻辑做比较，因为把超类作为逻辑比较的对象，肯定会违反协定，比如对称性。

总而言之，按如下方法可以override一个高质量的equals函数：

1. 使用==操作符来判断参数是指向本对象；
2. 使用instanceof来判断类型是不是相同；
3. 把参数从Object转成正确的类型;
4. 逻辑检查,看重要的字段是否相等；
5. 检查是否满足如上的协定。是否自反？是否对称？

还有几个要注意的东西：

1. 在override equals的时候记得override hashCode函数（Item 9）, 不然在使用hash的Collection会表现很奇怪，比如HashSet中；
2. 不要自作聪明，做在equals不应该做的事情;
3. 不要overloading equals函数。

在现代IDE,比如Intellij Idea中，一般都内置了equals的模板，可以快速的overide equals函数。

- Item 9: Always override hashCode when you override equals

在override equals的类中也要override hashCode函数，一般情况，如果两个equal对象的hashCode也要相等。
hashCode在现代IDE中一般也有模板可以使用。

- Item 10: Always override toString

提供一个好的toString实现可能使你的函数更好用，最佳的实践是在toString的返回值应该包括对象的所有重要信息，而且如果有特别的格式应该在toString函数的注释中说明。

- Item 11: Override clone judiciously

clone函数的坑太多，最好不要使用，可以使用copy constructor或者copy factory method来代替。

- Item 12: Consider implementing Comparable

如果要对象使用sort一类的函数，就要记得实现Comparable。

### Classes and Interfaces

- Item 13: Minimize the accessibility of classes and members

最小特权原则(Least Privilege)。
class的field应该都为private，因为有非私有field的class一定不是线程安全的，然而使用起来也有很多问题。
class不应该包含public的静态array，哪怕已经定义为final，因为array的对象可以被修改，使用起来会有安全风险。

- Item 14: In public classes, use accessor methods, not public fields

使用accessor函数包括getter和setter，不要直接访问field，因为这样不是线程安全的，而且很难保证field的值是有效的，两种情况都会造成state不可用。

- Item 15: Minimize mutability

如果可以的话，尽量使用Immutable对象，因为这样的话会避免很多问题，如线程安全等。不过在内存资源受限的应用中，要谨慎使用，因为Immutable不同的值一定不是同一个对象，会生成很多多余的对象。

- Item 16: Favor composition over inheritance

继承会破坏封装，不仅会继承不需要的函数。而且如果有公有函数之间的相互调用，如果被调用的函数被覆盖会使得原有函数的行为无法预测。组装相对安全，使用起来灵活。

- Item 17: Design and document for inheritance or else prohibit it

测试用于继承的类的方法是写子类。
构造函数不要调用可以重写的（overridable）函数，因为出现不可预测的行为。
最好的解决方案是禁止继承不是设计用于继承的类。

- Item 18: Prefer interfaces to abstract classes

因为可以同时实现多个interface，所以使用interface可以安全、灵活的扩展功能。
Abstract class比interface更容易演进，要增加一个函数，只要在abstract class中加一个默认函数实现就可以了，这是abstract class的一个优点。
小心定义公有interface，一旦interface发布出去，并且被大量实现，就几乎不可能更改。

- Item 19: Use interfaces only to define types

interface可以用来定义常量，但是这种使用方式不提倡。一般来说，interface只用来定义类型。

- Item 20: Prefer class hierarchies to tagged classes

有的时候，我们会用一个字段来区分不同类型的对象，这些对象对应的类就是tagged class。
tagged class在使用和扩展性上有很多缺点，而且效率相对低。在语义上，tagged class与类别继承体系相似。
但是相对而言，类别继承体系没有上面tagged class的缺陷。因此，我们应该使用类别继承体系来代替tagged class。

- Item 21: Use function objects to represent strategies

我们可以定义只有一个函数的interface代表策略，然后具体实现，把策略相关内容与通用的逻辑分离，实现代码重用，增加代码的可读性。

- Item 22: Favor static member classes over nonstatic

如果你定义一个成员类不需要访问它的包含类，一定记得把这个成员类置为static。

### Generics

- Item 23: Don’t use raw types in new code

不要使用raw types在新的代码中，因为这样做编译时不做进行类型检查，从而造成runtime error。

- Item 24: Eliminate unchecked warning

尽量消除每一个unchecked warning。
如果不能消除，在确认类型安全的前提下，使用@SuppressWarninggs来忽略waring，尽量在小的范围内使用这个annotation，并且在每个使用的地方都注释说明为什么是肉类安全的。

- Item 25: Prefer lists to arrays

array是covariant and reified，而范型Collection是invariant and erased，两者不可以混用，如果出现混用的话，优先考虑全部换成范型Collection。

- Item 26: Favor generic types

相对于Object，优先使用范型，这样可以更好地保证类型安全。

- Item 27: Favor generic methods

与Item 26相似。

- Item 28: Use bounded wildcards to increase API flexibility

为了最大的灵活性，只在入参中使用bounded wildcards来代表生产者或者是消费者。
PECS表示在生产者-extends，消费者-super。
不要使用wildcard类型作为返回值。
如果使用函数时需要考虑Wildcard入参的具体类型，那么wildcard使用一定有问题。
如果一个类型参数在函数定义中只出现一次，把这个类型参数换成wildcard。

- Item 29: Consider typesafe heterogeneous containers

### Enums and Annotations

- Item 30: Use enums instead of int constants

为了将enum常量和数据关联起来，可以定义成员变量并写一个构造器将数据存入成员变量。
将int常量转换到enum方便扩展常量相关的行为，因为在enum可以定义函数。

- Item 31: Use instance fields instead of ordinals

不要直接使用ordinal值，应该把相应的值存到成员变量，使用ordinal值会有代码可读性和维护性的问题。

- Item 32: Use EnumSet instead of bit fields

因为enum类型可以在Set中使用，不要使用bit fields来表示。使用bit fields是C/C++中常用的方式，但是可读性相对差。

- Item 33: Use EnumMap instead of ordinal indexing.

使用ordinal来index数组很少是合适的，应该使用EnumMap。

- Item 34: Emulate extensible enums with interfaces

虽然我们不能继承enum类型，但是我们可以让enum类型继承interface来实现扩展性。

- Item 35: Prefer annotations to naming patterns

不要再使用命名模式来指定特定的行为，而是应该使用annotation，这样代码更具灵活性和维护性。

- Item 36: Consistently use the Override annotation.

在每个你想覆盖的函数上加上@Override，这样可以避免错误。

- Item 37: Use marker interfaces to define types

相对marker annotations，marker interfaces定义了一种类型。
如果你正在写一个target是ElementType.TYPE的marker annotations，花点儿时间考虑一下是否合适，是不是应该换成marker interfaces。

### Methods

- Item 38: Check parameters for validity

在每个函数的开头检查参数的合法性，对不合法的参数抛出相应的异常，然后再进行逻辑操作和计算。

- Item 39: Make defensive copies when needed

在编写函数的时候要考虑全面，保证函数安全性。对于immutable object，要在contructor函数中对可变的参数做defensive copy以保证其不能修改。defensive copy在检查参数的合法性之前进行，合法性检查是对copy而不是原参数进行。
不要使用clone函数来进行defensive copy，特别是不可信Class的子类的参数。
对于不可修改类型的函数的返回值，如果是可以修改的，返回其defensive copy来维持不可修改性。

- Item 40: Design method signatures carefully

1. 小心选择函数名；
2. 不要写过多函数，只将经常使用的逻辑抽成函数。如果没想好，就保持原状；
3. 避免参数过多，过多类型相同的参数尤其有害。

减少方法参数的方法有：

1. 将方法拆成多个；
2. 写helper class将参数放在一个类里；
3. 使用Builder模式。

对于参数的类型，interface好过class，两个元素的enum类型好过boolean类型。

- Item 41: Use overloading judiciously

选择调用哪个重载的函数是在编译里确定的，对重载函数的选择是静态的，对覆盖函数的选择是动态的。
应该避免使用带有迷惑性的重载函数，最好不要编写参数数量相同的重载函数。

- Item 42: Use varargs judiciously

不要把每个函数的参数都变成final array参数；
在对可变长参数真正进行调用的函数中使用varargs。

- Item 43: Return empty arrays or collections, not nulls

没理由在返回值是数组或集合的函数返回null值，应该使用空数组或空集合。这样做减少了不必要的判断条件，代码更简洁。

- Item 44: Write doc comments for all exposed API elements

对要发布的API要在代码中写好注释，然后使用工具自动生成注释文件方便使用。

### General Programming

- Item 45: Minimize the scope of local variables

在第一次使用的地方定义变量；
几乎每个局部变量的定义时都应该指定一个初始值，try catch等少数情况有特例；
相对于while循环，更应该使用for循环；
保持函数小而专注，每个函数只处理一种行为。

- Item 46: Prefer for-each loops to traditional for loops

一般来说，相对于for循环，优先使用for-each循环。
有三种情况不能使用for-each循环：

1. 过滤，会删除一些元素；
2. 变换，会替换元素；
3. 交行遍历。

- Item 47: Know and use the libraries
- Item 48: Avoid float and double if exact answers are required
- Item 49: Prefer primitive types to boxed primitives
- Item 50: Avoid strings where other types are more appropriate
- Item 51: Beware the performance of string concatenation
- Item 52: Refer to objects by their interfaces
- Item 53: Prefer interfaces to reflection
- Item 54: Use native methods judiciously.
- Item 55: Optimize judiciously
- Item 56: Adhere to generally accepted naming conventions

### Exceptions

- Item 57: Use exceptions only for exceptional conditions
- Item 58: Use checked exceptions for recoverable conditions and runtime exceptions for programming errors
- Item 59: Avoid unnecessary use of checked exceptions
- Item 60: Favor the use of standard exceptions.
- Item 61: Throw exceptions appropriate to the abstraction.
- Item 62: Document all exceptions thrown by each method.
- Item 63: Include failure-capture information in detail messages
- Item 64: Strive for failure atomicity
- Item 65: Don’t ignore exceptions

### Concurrency

- Item 66: Synchronize access to shared mutable data.
- Item 67: Avoid excessive synchronization
- Item 68: Prefer executors and tasks to threads.
- Item 69: Prefer concurrency utilities to wait and notify.
- Item 70: Document thread safety
- Item 71: Use lazy initialization judiciously
- Item 72: Don’t depend on the thread scheduler
- Item 73: Avoid thread groups

### Serialization

- Item 74: Implement Serializable judiciously.

实现Serializable的缺点

1. 减少了发布后改变类的实现灵活性，一个类实现Serializable，则这它有的子类也会实现Serializable，而实现Serializable可能会破坏封装。
2. 增加了Bug和安全漏洞的可能性。
3. 增加了测试的负担，因为必须保证新版本的类和旧的版本的兼容。

决定是否实现Serializable要谨慎，设计为可以继承的Class尽量不要实现Serializable，interface也不要继承Serializable。
Inner Class也不要实现Serializable，static member class可以。

- Item 75: Consider using a custom serialized form

不要在没有考虑是否合适的情况下直接使用默认的序列化格式，默认序列化格式只有在逻辑内容和物理内容一样的时候才合适。即使使用默认的序列化格式也要提供readObject函数保证非协变和安全性。

在逻辑内容和物理内容不相同的情况下使用默认的序列格式有以下四个缺陷:

1. 对外的API和内部的实现永远有绑在一起。
2. 浪费空间。
3. 浪费时间。
4. 会造成stackOverflow。

不管使用哪种序列化的方式，都要定义一个UID来进行版本的管理。

- Item 76: Write readObject methods defensively

当反序列化一个对象时，要保证做了defensive copy和包含的引用不被其他对象拥有。
不要使用readUnshared和writeUnshared函数。

- Item 77: For instance control, prefer enum types to readResolve
- Item 78: Consider serialization proxies instead of serialized instances
