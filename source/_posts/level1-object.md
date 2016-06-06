title: 初级Java：基础概念，集合等
date: 2015-12-01 12:00:13
tags:
categories: [初级]
---

目的：了解常用的类和方法，了解Java语言的语法和意义。

### 抽象类与接口有什么区别？
>抽象类与接口都用于抽象，但是抽象类可以有自己的部分实现，而接口则完全是一个标识(同时有多重继承的功能)。

### 什么是自动拆装箱？
>自动装箱是Java编译器在基本数据类型和对应的对象包装类型之间做的一个转化。比如：把`int`转化成`Integer`，`double`转化成`Double`，等等。反之就是自动拆箱。

### switch语句可以用String做条件么？
>考察对版本的了解，JDK7后是可以的，以前的版本不可以。

### 能在一个静态代码块里里调用非静态方法么?
>static变量在Java中是属于类的，它在所有的实例中的值是一样的。当类被Java虚拟机载入的时候，会对static变量进行初始化。如果你的代码尝试不用实例来访问非static的变量，编译器会报错，因为这些变量还没有被创建出来，还没有跟任何实例关联上。

### Java为什么要用范型，有什么好处？
>通过编译期的报错，范型可以避免运行时的`ClassCastException`，也不用每次都用`instanceof`判断是否是此类型的对象。

### final、finally代码块和finalize()方法有什么区别？
>final—修饰符（关键字）如果一个类被声明为final，意味着它不能再派生出新的子类，不能作为父类被继承。 被声明为final的方法也同样只能使用，不能重载。 
无论是否抛出异常，finally代码块都会执行，它主要是用来释放应用占用的资源。
finalize()方法是Object类的一个protected方法，它是在对象被垃圾回收之前由Java虚拟机来调用的。

### abstract的method是否可同时是static,是否可同时是native，是否可同时是synchronized?
>都不能

### try {}里有一个return语句，那么紧跟在这个try后的finally {}里的code会不会被执行，什么时候被执行，在return前还是后?
>会执行，在return前执行

### Java中Exception和Error有什么区别？
>Exception和Error都是Throwable的子类。Exception用于用户程序可以捕获的异常情况。Error定义了不期望被用户程序捕获的异常。

### GC 是什么? 为什么要有 GC?
>GC 是垃圾收集的意思(Gabage Collection),内存处理是编程人员容易出现问题的地
方,忘记或者错误的内存回收会导致程序或系统的不稳定甚至崩溃,Java 提供的 GC 功能可 以自动监测对象是否超过作用域从而达到自动回收内存的目的,Java 语言没有提供释放已 分配内存的显示操作方法。

### 用最有效率的方法算出2乘以8等於几
>```2 << 3```

### float型`float f=3.4`是否正确?
>不正确。精度不准确,应该用强制类型转换，如下所示：`float f=(float)3.4`

### String是最基本的数据类型吗，基本类型都有哪些？
>基本数据类型包括byte、int、char、long、float、double、boolean和short。String不是。

### 一个".java"源文件中是否可以包括多个类（不是内部类）？有什么限制？
>可以。必须只有一个类名与文件名相同。

### 给我一个你最常见到的runtime exception
>说几个就成，常见的运行时异常有如下这些`ArithmeticException`, `ArrayStoreException`, `BufferOverflowException`, `BufferUnderflowException`, `CannotRedoException`, `CannotUndoException`, `ClassCastException`, `CMMException`, `ConcurrentModificationException`, `DOMException`, `EmptyStackException`, `IllegalArgumentException`, `IllegalMonitorStateException`, `IllegalPathStateException`, `IllegalStateException`, `ImagingOpException`, `IndexOutOfBoundsException`, `MissingResourceException`, `NegativeArraySizeException`, `NoSuchElementException`, `NullPointerException`, `ProfileDataException`, `ProviderException`, `RasterFormatException`, `SecurityException`, `SystemException`, `UndeclaredThrowableException`, `UnmodifiableSetException`, `UnsupportedOperationException`

### throw和throws有什么区别？
>throw关键字用来在程序中明确的抛出异常，相反，throws语句用来表明方法不能处理的异常。每一个方法都必须要指定哪些异常不能处理，所以方法的调用者才能够确保处理可能发生的异常，多个异常是用逗号分隔的。

### OverLoad 与 Override 的区别
>Overload：重载，表现类的多态性，可以有相同的函数名但是参数名、返回值、类型不能相同。
Override：重写，在子类继承父类的时候子类中可以定义某方法与其父类有相同的名称和参数，当子类在调用这一函数时自动调用子类的方法，而父类相当于被覆盖了。

### 在JAVA中，如何跳出当前的多重嵌套循环？
>使用多个break，或者使用return

### &和&&的区别
>&是位运算符，表示按位与运算，&&是逻辑运算符，表示逻辑与（and）

### Math.round(11.5)等於多少? Math.round(-11.5)等於多少
>Math.round(11.5)==12;Math.round(-11.5)==-11;round方法返回与参数最接近。同理还有Math.floor()

### Java有没有goto?
>java中的保留字，现在没有在java中使用

### 数组有没有length()这个方法? String有没有length()这个方法
>数组没有length()这个方法，有length的属性。String有有length()这个方法

### 你都知道哪些Java创建对象的方式
>- 通过new关键字
>- 运用反射手段,调用java.lang.Class 或者 java.lang.reflect.Constructor 类的newInstance()实例方法
>- clone() 通过克隆
>- 通过反序列化

### Object类都有哪些常用的方法？
> clone(),equals(),hashCode(),notify(),notifyAll(), toString(),wait(),finalize() 
> 主要是为了考察对java的大体了解

### 一个对象要作为HashMap的key，需要重写什么方法?
>必须同时覆盖equals和hashCode方法。如果不这样做的话，就会违反`Object.hashCode`的通用约定，从而导致该类无法结合所有基于散列的集合一起正常运作，这样的集合包括`HashMap`、`HashSet`和`Hashtable`等。 

### Comparable和Comparator接口是干什么的？列出它们的区别。
>Java提供了只包含一个compareTo()方法的`Comparable接口`。这个方法可以个给两个对象排序。具体来说，它返回负数，0，正数来表明输入对象小于，等于，大于已经存在的对象。
Java提供了包含`compare()`和`equals()`两个方法的Comparator接口。compare()方法用来给两个输入参数排序，返回负数，0，正数表明第一个参数是小于，等于，大于第二个参数。equals()方法需要一个对象作为参数，它用来决定输入参数是否和comparator相等。只有当输入参数也是一个comparator并且输入参数和当前comparator的排序结果是相同的时候，这个方法才返回true。

### ==, equals()和 hashCode()作用
>java中的==是比较两个对象在JVM中的地址。equals是比较的值是否相等。举个例子，比较字符串值是否相等，用equals。
>hashCode和equals是一对协同方法。如果你的对象打算所谓散列表的key，一般先先判断hashcode再判断equals。它们之间有以下关系：
覆盖equals时总要覆盖hashCode。
如果一个对象equals另外一个对象，则它们的hashcode一定相等。
>有很大可能会追问HashMap的存储结构。如下:

### HashMap的存储结构
>`HashMap采用数组加链表的方式存储`。拿put方法来说，首先，通过key的hashcode定位到数组的某一位置，如果这个位置为空，则将值插入到此位置。假如此位置有其他数据，则证明有冲突，HashMap使用链表解决冲突，会使用equals方法来逐一判断列表中的值，有相等的则覆盖，否则加入到冲突列表的结尾。

### HashSet和TreeSet有什么区别？
>- HashSet是由一个hash表来实现的，因此，它的元素是无序的。add()，remove()，contains()方法的时间复杂度是O(1)。
- TreeSet是由一个树形的结构来实现的，它里面的元素是有序的。因此，add()，remove()，contains()方法的时间复杂度是O(logn)。

### HashMap和Hashtable有什么区别？
>- HashMap和Hashtable都实现了Map接口，因此很多特性非常相似。但是，他们有以下不同点：
- HashMap允许键和值是null，而Hashtable不允许键或者值是null。
- Hashtable是同步的，而HashMap不是。因此，HashMap更适合于单线程环境，而Hashtable适合于多线程环境。
- HashMap提供了可供应用迭代的键的集合，因此，HashMap是快速失败的。另一方面，Hashtable提供了对键的列举(Enumeration)。
- 一般认为Hashtable是一个遗留的类。

### ArrayList和LinkedList有什么区别？
>ArrayList和LinkedList都实现了List接口，他们有以下的不同点：
- ArrayList是基于索引的数据接口，它的底层是数组。它可以以O(1)时间复杂度对元素进行随机访问。与此对应，LinkedList是以元素列表的形式存储它的数据，每一个元素都和它的前一个和后一个元素链接在一起，在这种情况下，查找某个元素的时间复杂度是O(n)。
- 相对于ArrayList，LinkedList的插入，添加，删除操作速度更快，因为当元素被添加到集合任意位置的时候，不需要像数组那样重新计算大小或者是更新索引。
- LinkedList比ArrayList更占内存，因为LinkedList为每一个节点存储了两个引用，一个指向前一个元素，一个指向下一个元素。

### 有没有排序的map？
>可以使用TreeMap进行排序

### 怎样对一个List进行排序？
>`实现Comparable接口`,并调用`Collections`的`sort`方法。

### Java集合的基本接口都有哪些？
>`Collection`是`Set`和`List`的根类,`Set`是不可从福列表，`List`是可以包含重复数据的列表，`Map`是包含多个键值对的集合。

### String 与 StringBuffer , StringBuilder 有什么区别?
>String 是被 final 修饰的类,意味着 String 类不能被继承使用, 一旦声明是一个不可变的类。
>StringBuffer和StringBuilder是一个长度可变的,通过追加的方式扩充，当进行大量字符串操作时，会提高性能。
>StringBuffer是线程安全的，所以效率相对StringBuilder要低，线程安全的情况下建议使用StringBuilder。

### char型变量中能不能存贮一个中文汉字?
> 可以，因为java中以unicode编码，一个char占16个字节，所以放一个中文是没问题的。

### ISO-8859-1可以存储中文么？
> 可以，以ISO-8859-1编码的文本，都以bytes[]的形式保存，若要显示中文，只需以显示平台的默认编码格式进行解码即可。若仍然以ISO-8859-1格式解码，得到的中文字符肯定是乱码，因为ISO-8859-1自身不能显示中文。


### Enumeration和Iterator有什么关系?
>Enumeration是个接口，不是类，再次，这个东西就是为了实现遍历的，现在已经被迭代器Iterator取代了。
>Iterator支持fail-fast机制，而Enumeration不支持。

### 给一个字符串,找到第一个不重复到字符？
>可以用HashMap来搞，也可以用字符串函数如indexOf,lastIndexOf等 [讨论](http://www.iteye.com/topic/138206)

