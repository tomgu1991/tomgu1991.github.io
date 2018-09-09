# 面试问题整理-Java

[TOC]

### 什么是面向对象

将问题分解成一个一个步骤，每个步骤进行抽象。通过不同对象之间的调用，组合解决问题。注意封装属性、行为等。基于对象进行业务逻辑实现。

### 面向对象三大基本特征

封装（Encapsulation）、继承（Inheritance）和多态（Polymorphism）

### 面向对象五大基本原则 SOLID

* 单一职责原则SRP(Single Responsibility Principle)
  是指一个类的功能要单一，不能包罗万象。如同一个人一样，分配的工作不能太多，否则一天到晚虽然忙忙碌碌的，但效率却高不起来。

* 开放封闭原则OCP(Open－Close Principle) 

  一个模块在扩展性方面应该是开放的而在更改性方面应该是封闭的。比如：一个网络模块，原来只服务端功能，而现在要加入客户端功能，
  那么应当在不用修改服务端功能代码的前提下，就能够增加客户端功能的实现代码，这要求在设计之初，就应当将服务端和客户端分开，公共部分抽象出来。

* 替换原则(the Liskov Substitution Principle LSP) 
  子类应当可以替换父类并出现在父类能够出现的任何地方。比如：公司搞年度晚会，所有员工可以参加抽奖，那么不管是老员工还是新员工，
  也不管是总部员工还是外派员工，都应当可以参加抽奖，否则这公司就不和谐了。

* 依赖原则(the Dependency Inversion Principle DIP) 具体依赖抽象，上层依赖下层。假设B是较A低的模块，但B需要使用到A的功能，这个时候，B不应当直接使用A中的具体类： 而应当由B定义一抽象接口，并由A来实现这个抽象接口，B只使用这个抽象接口：这样就达到
  了依赖倒置的目的，B也解除了对A的依赖，反过来是A依赖于B定义的抽象接口。通过上层模块难以避免依赖下层模块，假如B也直接依赖A的实现，那么就可能造成循环依赖。一个常见的问题就是编译A模块时需要直接包含到B模块的cpp文件，而编译B时同样要直接包含到A的cpp文件。

* 接口分离原则(the Interface Segregation Principle ISP) 
  模块间要通过抽象接口隔离开，而不是通过具体的类强耦合起来

### **String类能不能被继承，为什么？这种设计有什么好处？**

Final 类型

* 字符串hashcode，高效
* 安全性，作为参数不可以改变，比如通过引用修改内容

### String的'+'如何实现

* 常量，编译时期拼接
* 优化方法，使用StringBuilder的append方法，最后toString

### **通过Array.asList获得的List有什么特点，使用时应该注意什么？**

一个Arrays的内部类，不可以修改

### **Serializable和Externalizable接口有何不同？**

前者是一个标记，用于序列化。后者继承了前者，需要实现两个接口writeExternal 和 readExternal，且类需要提供一个public无参数构造器。

### **try()里面有一个return语句，那么后面的finally{}里面的代码会不会执行，什么时候执行？**

会。return表示的是要整体方法返回，所以finally会执行。

### **serialVersionUID有何用途？如果没定义会有什么问题？**

反序列化时需从文件读取时，进行比对。

### **Java枚举类的比较用 == 还是 equals，有哪些区别？**

一样，equals用==实现。

### **Java集合框架是什么？说出一些集合框架的优点？**

每种编程语言中都有集合，最初的Java版本包含几种集合类：Vector、Stack、HashTable和Array。

随着集合的广泛使用，Java1.2提出了囊括所有集合接口、实现和算法的集合框架。在保证线程安全的情况下使用泛型和并发集合类，Java已经经历了很久。它还包括在Java并发包中，阻塞接口以及它们的实现。

集合框架的部分优点如下：

（1）使用核心集合类降低开发成本，而非实现我们自己的集合类。

（2）随着使用经过严格测试的集合框架类，代码质量会得到提高。

（3）通过使用JDK附带的集合类，可以降低代码维护成本。

（4）复用性和可操作性。

### **集合框架中的泛型有什么优点？**

1.Java1.5引入了泛型，所有的集合接口和实现都大量地使用它。

2.泛型允许我们为集合提供一个可以容纳的对象类型，因此，如果你添加其它类型的任何元素，它会在编译时报错。

3.这避免了在运行时出现ClassCastException，因为你将会在编译时得到报错信息。

4.泛型也使得代码整洁，我们不需要使用显式转换和instanceOf操作符。

5.它也给运行时带来好处，因为不会产生类型检查的字节码指令

### **Java集合框架的基础接口有哪些？**

Collection为集合层级的根接口。一个集合代表一组对象，这些对象即为它的元素。Java平台不提供这个接口任何直接的实现。

Set是一个不能包含重复元素的集合。这个接口对数学集合抽象进行建模，被用来代表集合，就如一副牌。

List是一个有序集合，可以包含重复元素。你可以通过它的索引来访问任何元素。List更像长度动态变换的数组。

Map是一个将key映射到value的对象.一个Map不能包含重复的key：每个key最多只能映射一个value。

一些其它的接口有Queue、Dequeue、SortedSet、SortedMap和ListIterator。

### **为何Collection不从Cloneable和Serializable接口继承？**

Collection接口指定一组对象，对象即为它的元素。如何维护这些元素由Collection的具体实现决定。例如，一些如List的Collection实现允许重复的元素，而其它的如Set就不允许。

很多Collection实现有一个公有的clone方法。然而，把它放到集合的所有实现中也是没有意义的。这是因为Collection是一个抽象表现。重要的是实现。

当与具体实现打交道的时候，克隆或序列化的语义和含义才发挥作用。所以，具体实现应该决定如何对它进行克隆或序列化，或它是否可以被克隆或序列化。点击[这里](http://mp.weixin.qq.com/s?__biz=MzI3ODcxMzQzMw==&mid=2247484372&idx=1&sn=381af66344b4b50d033c38e51bcb0e21&chksm=eb5386e2dc240ff4fbee79af445ffeb3856da484a8f5206bcaf4531f0d510564943b213f948c&scene=21#wechat_redirect)一文学会序列化。

在所有的实现中授权克隆和序列化，最终导致更少的灵活性和更多的限制。特定的实现应该决定它是否可以被克隆和序列化。点击[这里](http://mp.weixin.qq.com/s?__biz=MzI3ODcxMzQzMw==&mid=2247484372&idx=1&sn=381af66344b4b50d033c38e51bcb0e21&chksm=eb5386e2dc240ff4fbee79af445ffeb3856da484a8f5206bcaf4531f0d510564943b213f948c&scene=21#wechat_redirect)一文学会序列化。

### **为何Map接口不继承Collection接口？**

尽管Map接口和它的实现也是集合框架的一部分，但Map不是集合，集合也不是Map。因此，Map继承Collection毫无意义，反之亦然。

如果Map继承Collection接口，那么元素去哪儿？Map包含key-value对，它提供抽取key或value列表集合的方法，但是它不适合“一组对象”规范。

### **Iterator是什么？**

Iterator接口提供遍历任何Collection的接口。我们可以从一个Collection中使用迭代器方法来获取迭代器实例。迭代器取代了Java集合框架中的Enumeration。迭代器允许调用者在迭代过程中移除元素。

### **Enumeration和Iterator接口的区别？**

Enumeration的速度是Iterator的两倍，也使用更少的内存。Enumeration是非常基础的，也满足了基础的需要。但是，与Enumeration相比，Iterator更加安全，因为当一个集合正在被遍历的时候，它会阻止其它线程去修改集合。

迭代器取代了Java集合框架中的Enumeration。迭代器允许调用者从集合中移除元素，而Enumeration不能做到。为了使它的功能更加清晰，迭代器方法名已经经过改善。

### **为何迭代器没有一个方法可以直接获取下一个元素，而不需要移动游标？**

它可以在当前Iterator的顶层实现，但是它用得很少，如果将它加到接口中，每个继承都要去实现它，这没有意义。

### **fail-fast与fail-safe有什么区别？**

Iterator的fail-fast属性与当前的集合共同起作用，因此它不会受到集合中任何改动的影响。Java.util包中的所有集合类都被设计为fail-fast的，而java.util.concurrent中的集合类都为fail-safe的。

Fall—fast迭代器抛出ConcurrentModificationException，

fall—safe迭代器从不抛出ConcurrentModificationException。

### **UnsupportedOperationException是什么？**

UnsupportedOperationException是用于表明操作不支持的异常。在JDK类中已被大量运用，在集合框架java.util.Collections.UnmodifiableCollection将会在所有add和remove操作中抛出这个异常。

### **hashCode()和equals()方法有何重要性？**

HashMap使用Key对象的hashCode()和equals()方法去决定key-value对的索引。点击[这里](http://mp.weixin.qq.com/s?__biz=MzI3ODcxMzQzMw==&mid=2247486439&idx=1&sn=3afb389bca737a1799f7cdee92a7dc99&chksm=eb538ed1dc2407c7782cdf71344786c3db5c32469fdfb1ab97e58364c2902dbcde09ed98e16a&scene=21#wechat_redirect)一文搞懂它们之间的关系。

当我们试着从HashMap中获取值的时候，这些方法也会被用到。如果这些方法没有被正确地实现，在这种情况下，两个不同Key也许会产生相同的hashCode()和equals()输出，HashMap将会认为它们是相同的，然后覆盖它们，而非把它们存储到不同的地方。

同样的，所有不允许存储重复数据的集合类都使用hashCode()和equals()去查找重复，所以正确实现它们非常重要。equals()和hashCode()的实现应该遵循以下规则：

1.如果o1.equals(o2)，那么o1.hashCode() == o2.hashCode()总是为true的。

2.如果o1.hashCode() == o2.hashCode()，并不意味着o1.equals(o2)会为true。

### **Map接口提供了哪些不同的集合视图？**

Map接口提供三个集合视图：

1）Set keyset()：返回map中包含的所有key的一个Set视图。集合是受map支持的，map的变化会在集合中反映出来，反之亦然。当一个迭代器正在遍历一个集合时，若map被修改了（除迭代器自身的移除操作以外），迭代器的结果会变为未定义。集合支持通过Iterator的Remove、Set.remove、removeAll、retainAll和clear操作进行元素移除，从map中移除对应的映射。

它不支持add和addAll操作。

2）Collection values()：返回一个map中包含的所有value的一个Collection视图。这个collection受map支持的，map的变化会在collection中反映出来，反之亦然。当一个迭代器正在遍历一个collection时，若map被修改了（除迭代器自身的移除操作以外），迭代器的结果会变为未定义。集合支持通过Iterator的Remove、Set.remove、removeAll、retainAll和clear操作进行元素移除，从map中移除对应的映射。它不支持add和addAll操作。

3）Set<Map.Entry<K,V>> entrySet()：返回一个map钟包含的所有映射的一个集合视图。这个集合受map支持的，map的变化会在collection中反映出来，反之亦然。当一个迭代器正在遍历一个集合时，若map被修改了（除迭代器自身的移除操作，以及对迭代器返回的entry进行setValue外），迭代器的结果会变为未定义。集合支持通过Iterator的Remove、Set.remove、removeAll、retainAll和clear操作进行元素移除，从map中移除对应的映射。它不支持add和addAll操作。

### **HashMap和HashTable有何不同？**

（1）HashMap允许key和value为null，而HashTable不允许。

（2）HashTable是同步的，而HashMap不是。所以HashMap适合单线程环境，HashTable适合多线程环境。

（3）在Java1.4中引入了LinkedHashMap，HashMap的一个子类，假如你想要遍历顺序，你很容易从HashMap转向LinkedHashMap，但是HashTable不是这样的，它的顺序是不可预知的。

（4）HashMap提供对key的Set进行遍历，因此它是fail-fast的，但HashTable提供对key的Enumeration进行遍历，它不支持fail-fast。

（5）HashTable被认为是个遗留的类，如果你寻求在迭代的时候修改Map，你应该使用CocurrentHashMap。

### **ArrayList和Vector有何异同点？**

ArrayList和Vector在很多时候都很类似。

（1）两者都是基于索引的，内部由一个数组支持。

（2）两者维护插入的顺序，我们可以根据插入顺序来获取元素。

（3）ArrayList和Vector的迭代器实现都是fail-fast的。

（4）ArrayList和Vector两者允许null值，也可以使用索引值对元素进行随机访问。

以下是ArrayList和Vector的不同点。

（1）Vector是同步的，而ArrayList不是。然而，如果你寻求在迭代的时候对列表进行改变，你应该使用CopyOnWriteArrayList。

（2）ArrayList比Vector快，它因为有同步，不会过载。

（3）ArrayList更加通用，因为我们可以使用Collections工具类轻易地获取同步列表和只读列表。

### **Array和ArrayList有何区别？什么时候更适合用Array？**

Array可以容纳基本类型和对象，而ArrayList只能容纳对象。

Array是指定大小的，而ArrayList大小是固定的。

Array没有提供ArrayList那么多功能，比如addAll、removeAll和iterator等。尽管ArrayList明显是更好的选择，但也有些时候Array比较好用。

（1）如果列表的大小已经指定，大部分情况下是存储和遍历它们。

（2）对于遍历基本数据类型，尽管Collections使用自动装箱来减轻编码任务，在指定大小的基本类型的列表上工作也会变得很慢。

（3）如果你要使用多维数组，使用[][]比List<List<>>更容易。

### **ArrayList和LinkedList有何区别？**

ArrayList和LinkedList两者都实现了List接口，但是它们之间有些不同。

1）ArrayList是由Array所支持的基于一个索引的数据结构，所以它提供对元素的随机访问，复杂度为O(1)，但LinkedList存储一系列的节点数据，每个节点都与前一个和下一个节点相连接。所以，尽管有使用索引获取元素的方法，内部实现是从起始点开始遍历，遍历到索引的节点然后返回元素，时间复杂度为O(n)，比ArrayList要慢。

2）与ArrayList相比，在LinkedList中插入、添加和删除一个元素会更快，因为在一个元素被插入到中间的时候，不会涉及改变数组的大小，或更新索引。

3）LinkedList比ArrayList消耗更多的内存，因为LinkedList中的每个节点存储了前后节点的引用。

### **哪些集合类是线程安全的？**

Vector、HashTable、Properties和Stack是同步类，所以它们是线程安全的，可以在多线程环境下使用。Java1.5并发API包括一些集合类，允许迭代时修改，因为它们都工作在集合的克隆上，所以它们在多线程环境中是安全的。点击[这里](http://mp.weixin.qq.com/s?__biz=MzI3ODcxMzQzMw==&mid=2247486446&idx=2&sn=cb4f3aff0427c5ac3ffe5b61e150f506&chksm=eb538ed8dc2407ceb91fffe3c3bd559d9b15537446f84eb3bfb1a80e67f5efee176ca468a07b&scene=21#wechat_redirect)一文搞懂问什么线程不安全。

### 面向对象理论

* **避免重复，DRY(Don’t repeat yourself)**面相对设计理论的第一条就是避免重复，不要写重复的代码，尽量将共同的功能放在一个地方。如果你准备在不同地方写同一段代码，那么只写一个方法。如果你不止一次硬编码某个值，那么将其声明成public final常量。这么做的好处就是容易维护。但是不要滥用这一条，重复不是指代码的重复，而是指功能的重复。譬如你有一段相同的代码来验证OrderID和SSN，但它们代表的意义并不相同。如果你将两个不同的功能合并在一起，当OrderID更改了格式之后，那么检验SSN的代码就会失效。所以要警觉这种耦合，不要将任何相似但不相关的代码合并在一起。
* **将变化封装起来**在软件领域唯一不变的就是“变化”。所以最好将你觉得将来会有改变的代码封装起来。这样做的好处就是更容易测试和维护正确的被封装的代码。你应该先将变量声明成private，然后有需要的话再扩大访问权限，如将private变成protected。Java中很多设计模式都使用了封装，工厂设计模式就是封装的一个例子，它封装了对象的创建，如果要引入新的“产品”，也不必更改现有的代码。
* **开放且封闭的设计理论(Open Closed Design Principle)**类、方法以及功能应该对扩展开放(新的功能)，而对更改封闭。这是另一个优美的”SOLID”设计理论，这保证了有人更改已经经过测试了的代码。如果你要加入新的功能，你必须要先测试它，这正是开放且封闭的设计理论的目标。另外，Open Closed principle正是SOLID中的O的缩写。
* **单一责任原理(Single Responsibility Principle (SRP))**单一责任原理是另外一条”SOLID”设计理论，代表其中的“S”。每次一个类只有一个更改的原因，或者一个类只应该完成单一的功能。如果你将多过一个功能放在一个类中，它会将两个功能耦合在一起，如果你改变了其中的一个功能，可能会破坏另外一个功能，这样便需要更多的测试以确保上线时不出现什么岔子。
* **依赖注入或反转原理**容器会提供依赖注入，Spring非常好的实现了依赖注入。这条原理的美妙之处在于，每个被注入的类很容易的测试和维护，因为这些对象的创建代码都集中在容器中，有多种方法都可以进行依赖注射，譬如一些AOP框架如AspectJ使用的字节码注入(bytecode instrumentation)，以及Spring中使用的代理器(proxy)。来看看这个依赖注射的例子吧。这一条正是SOLID中的”D”。
* **多用组合，少用继承**如果可能的话，多用组合，少用继承。可能有的人会不同意，但我确实发现组合的灵活性高过继承。组合可以在运行时通过设置某个属性以及通过接口来组合某个类，我们可以使用多态，这样就能随时改变类的行为，大大提高了灵活性。Effective Java也更倾向于使用组合。
* **Liskov替代原理(Liskov Substitution Principle (LSP))**根据Liskov替代原理，子类必须可以替代父类，也就是使用父类的方法，也能够没有任何问题的和子类对象也兼容。LSP和单一责任原则以及接口分离原则的关系紧密。如果一个类比子类的功能要多，子类不能支持父类中的某些功能的话，就违反了LSP。为了遵循LSP原理，子类需要改进父类的功能，而不是减少功能。LSP代表SOLID中的”L”。
* **接口分离理论(Interface Segregation principle (ISP))**接口分离理论强调，如果客户端没有使用一个接口的话，就不要实现它。当一个接口包含两个以上的功能，如果客户端仅仅需要其中某个功能，而不需要另外一个，那么就不要实现它。接口的设计是件非常复杂的工作，因为一旦你发布了接口之后，就再也无法保证不破坏现有实现的情况下更改接口。分离接口的另一个好处就是，因为必须要实现方法才能使用接口，所以如果仅仅只有单一的功能，那么要实现的方法也会减少。
* **针对接口编程，而不是针对实现编程**尽量针对接口编程，这样如果要引入任何新的接口，也有足够的灵活性。在变量的类型、方法的返回类型以及参量类型中使用接口类型。很多程序员都建议这么做，包括Effective Java和head first design pattern等书。
* **代理理论(Delegation principle)**不要所有的事情都自己做，有时候要将任务代理给相应的类去做。运用代理模式最经典的例子就是equals()和hashCode()方法。为了比较两个对象的相等与否，我们没有用客户端代码去比较，而是让对象自己去比较。这么做的好处就是减少代码的重复，更容易更改行为。