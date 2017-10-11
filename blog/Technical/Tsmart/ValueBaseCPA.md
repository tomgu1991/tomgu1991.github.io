# Notes on ValueBaseCPA

对于ValueBaseCPA, 和CompositeCPA很像，是一个wrapper cpa，可以包含其他的若干个子模块(plugin)。实际上，value base 是一个框架。最顶层的value base cpa只从全局的角度去关注内存结构的变化，注意只关注结构的变化，不管里面的内容是什么。下面的子plugin则只关注自己的领域属性，比如：pointer只关注所有指针的操作。

---

### MemoryStructure

framework.state.memory这部分是memory结构。

##### CallStack

- List: LlvmFunction+
- fromMemoryModel: stack#bottomup#map(frame#getFunction)



##### size

* MemorySize
  * ExplicitSize

    long

  * ImplicitSize

    llvm.Value

  * UnknownSize

  ​



##### loc

- MemoryOffsetInterval

  long start

  long end

- MemoryLocation

  MemoryObject base

  long offset

- MemoryRange

  MemoryLocation location

  long size





##### object

* ObjectSource: register, stack, global, heap, unknown
* AllocationEdge: for global/local variables

  CFAEdge
* AllocationSite: for heap

  CFAEdge

  sequence


* AbstractObject

  ObjectSource

  Type

  MemorySize

  * StackObject/GlobalObject

    CallStack callStack

    CFAEdge site

  * RegisterObject

    CallStack callStack

    String name

  * HeapObject

    CFAEdge site

    int sequence

  * UnknownObject

    int ID





##### model

* MemoryFrame

  CallStack

  LlvmFunction

  Map: String -> RegisterObject

  Stack: StackObjects+

  ofFunction: 构造新的frame

  allocaNamedObject: 处理alloca指令

* MemoryHeap

  Set: HeapObject+

* MemoryModel =====> this is the struture of ***memory*** without content

  MemoryFrame global

  Stack: MemoryFrame+

  MemoryHeap heap




##### succ_monad

这部分相当于记录每一次的更改。相当于Old 每次transform， 产生一个New + ChangeLog

* MemoryOperation 记录每一次操作的old 以及操作的类型

  MemoryObject object

  OperationType type : CREATE, REMOVE

* MemoryChange 记录所有的operations，这个只针对

  List\<MemoryOperation\> history 

  nochange(), concate(x, y), create(), remove(), history relevant operations 

* OperationSuccessor\<R\> 某一个操作作用于某个状态的后记状态，所有具体的状态改变通过transform来执行。

  R successor

  MemoryChange change

  * FrameSuccessor
  * MemorySuccessor


```java
这里比较绕。
所有的操作都是通过transform进行，就是把一个函数当作参数，后面会调用这个函数进行修改。

比如initial的时候，
succ = succ.transform(x -> x.pushStackAndCopy(function))
后面会讲这个succ作为参数并且调用pushStackAndCopy函数，结果存在successor中，结果由两部分组成（1）更改后的状态 （2）更改的内容的历史记录

所以我们可以这让理解：allocate一个变量
1. create object
2. MemoryFrame#addObject -> newFrame
3. create change(object)
3. create FrameSuccessor(newFrame, change)

要想让这个工作：
定义基本的操作：增、改、删
定义Change：记录这些基本的操作
定义一个successor的结构：存放新的state和Change
```



```
在后面的构成中，我们可以根据上述的内存结构的变化，来在具体的plugin中作相应的处理。
比如：处理global declaration，会有两个object生产：registerObject 和 stackObject
所以可以根据这两个分别处理
```




##### MemorySkeletonTransferRelation

处理memory structure的转移函数，只关注结构变化的操作。

* function call edge
  * actual parameters -> implement separately 
  * push stack and create register object
  * assign formal parameters? -> implement separately
* function return edge
* global decl edge
* instruction edge

---

### Value

这部分存储的是真实值的接口。

##### single

* IValue

  * TypedValue ===> base type for values related to llvm types

    getType(), getBitWidth(), getByteWidth(), getIsPoision()

    * SingleValue ===> Integer, Float, Pointer, Vector

      isExplicit(), evalExplicitly()

      * IntValue
      * FloatValue

    * AggregateValue ===> Array, Struct

      isExplicit(), evalExplicitly()

  * IConstValue





* AbstractValue

  * AbstractTypedValue

    Type type

    long bitWidth

    long byteSize

    Ternary isPoison

    * UnknownValue

      Kind kind {UNINIT, ASSUMED, MERGED}

    * PointerValue

      Type pointerType

      MemoryLocation location

    * AbStractAggregateValue

      * ArrayValue
      * StructValue


##### multiple

Objects that stores multiple values, can be considered as container.

* MultiValue

  size(), isEmpty(), hasUniqueValue(), getOnlyValue(), getValues(), getType(), merge()

  * AbstractMultiValue

    * SetMultiValue

      Set\<IValue\> values

##### tuple

Like a list, treat multiple values as its element. 所以看起来这个像是一个笛卡尔积，第一个元素的多个值 X 第二个元素的多个值 X ...

* CartesianValue

* TupleValue

  MultiValue[] multiValues

  * EmptyValue

    int size

##### evaluator

visitor to evaluate an instruction

* InsEvaluator
  * AbstractInstInsEvaluator

---

### Plugin

这部分是plugin，也就是特定领域的CPA的框架

* PluginRegistry

  index : Class -> Integer

  classMap : Class -> ValueBasePlugin

  plugins: ValueBasePlugin+

* ValueBasePlugin===> like cpa, (a)state of plugin must be stored as memory model (b) must work with valuebase

  String name

  PluginTransferRelation trans

  getInitialState

  join

  isLessOrEqual

  MergeOperator

  StopOperator

* SubState

  Typically, it is a map from MemoryRange to Value. 这里就是精华所在，这个value是一个抽象概念，里面存什么值可以由具体的plugin决定

* PluginTransferRelation

  PluginRegistry registry

  getSuccessorsForEdge

* PluginStopOperator

  stop(ValueBasePlugin, SubState, Collection reached)

* PluginMergeOperator

  merge(ValueBasePlugin, SubState, SubState reached)

  ​




---

### ValueBaseCPA

这个CPA是顶层cpa，只关注内存模型的修改相关的操作。

##### ValueBaseCPA

* PluginRegistry: 注册器，我们维护所有的子plugin。
* ValueBaseDomain:
  * join:  MemoryModel#join，SubState#join
  * isLessOrEqual：MemoryModel#isLessOrEqual, SubState#isLessOrEqual
* StopOperator: Domain#isLessOrEqual

##### ValueBaseState

* MemoryModel
* SubState+
* initialFor: MemoryModel, SubState#getinitialState


##### ValueBaseTransferRelation

其实是一个傀儡，先让SkeletonTransferRelation做内存上的事情，然后调用后边的registry处理SubState

* PluginRegistry registry
* MemorySkeletonTransferRelation mstr


---

### CheckerCPA

通过配置来执行各种checker实例，每一种checker要实现AbstractChecker的实例，即写一个自己的transferRelation。

* CheckerRegistry

  List\<Checker\> checkers


* CheckerCPA

  CheckerRegistry registry

  stop: True; 这是一个检查checker，所以不对运行结果进行关注

* CheckerState

  private static CheckerState INSTANCE

* CheckerTransferRelation

  调用每一个checker#onEdge方法去单独判断

* Checker

  onEdge() 判断是否发现缺陷

* AbstractChecker

  reportHere(Weakness, message, Traceable) -> add new CheckerSite into ReportCollector

* CheckerSite 记录出错的位置

  Weakness, message, Traceable, argState, node, edge, CheckerTraceGenerator




---

### PluginCPA Instance - Pointer

##### cpa

* PointerAnalysisPlugin

  initState() 处理函数的参数

* PointerAnalysisTransferRelation 作用于successor，current，change(Memory Model的change)

  这里比较有意思，因为memory处理后change是有顺序关系的，所以这里面需要根据顺序关系，拿出相应object进行处理。

* PointerState

  Multimap\<MemoryRange, PointerValue\> pointTo 可能的point2

  Multimap\<MemoryRange, MemoryObject\> loosePointTo 指针可能指向object的任何位置

  Multimap\<MemoryObject, MemoryRange\> pointFrom 一个对象可能被指像的pointer

##### value

* PointToInfo

  Set\<PointerValue\> pointTo

  Set\<MemoryObject\> loosePointTo

##### successor

* EvaluationResult\<R, S\> result, successor

* PointerResult\<Set\<PointerValue\>, PointerState\>

  create()

##### checker

* PointerBasedChecker

  onEdge() -> checkEdge() -> check defects

  * NPD
  * UAF
  * DF
  * Memory Leak