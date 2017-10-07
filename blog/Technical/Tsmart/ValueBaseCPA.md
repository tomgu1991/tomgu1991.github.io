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

TODO



##### MemorySkeletonTransferRelation

TODO

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

### Checker



---

### PluginCPA Instance