# Notes on Memory Model

### Graphs

#### Node

SGObject

```java
// 明确的一块内存
ShapeExplicitValue size;
String label;
```

* SGRegion

  ```java
  // 明确的一块内存，同时包含类型信息
  CType type;
  boolean isDynamic;
  ```

* SGResource

  ```java
  // 表示文件信息
  String creator;
  ```



#### Edge

SGEdge

```java
long value; // 这个value对应于CShapeGraph中的values，相当于边的标记. 对于一个SGRegion r, 指向r的ptEdge和r的hvEdge，两者的value是同一个值 
SGObject object;
```

* SGPointToEdge

  ```java
  int offset; // 所指的内存块的便宜，因为SGObject是一块内存，而且label是这块内存存储的变量的名字
  ```

* SGHasValueEdge

  ```java
  // 这个值的类型以及相对于这个内存块的偏移, 这时候如果有确定的值，那么value对应的State中explicitValues的确定的值
  // 这个 explicitValues 是BiMap<KnownSymbolicValue, KnownExplicitValue> 
  // edge 中的value相当于symbolicValue
  CType type;
  int offset;
  ```

  ​

#### Model

StackUnit

```java
String label; // TODO functionName
```

CStackFrame

```java
CFunctionDeclaration function;
SGRegion returnObject;
Map<String, SGRegion> stack;
Set<SGRegion> VLASet;
```



#### CShapeGraph

```java
MachineModel machineModel;
Set<SGObject> objects;
Set<Long> values;
Set<SGHasValueEdge> HVEdges;
Map<Long, SGPointToEdge> PTEdges
Deque<CStackFrame> stackObjects;
Map<String, SGObject> heapObjects;
Map<String, SGRegion> globalObjects;
Map<MemoryLocation, Long> memoryValue; // 这个就是SGObject所拥有的hvEdge和指向这个SGObject的ptEdge
```





### Values

Address

```java
// 包含内存对象和偏移量，相当于确定的一块内存的地址
SGObject object;
ShapeExplicitValue offset;
```



ShapeKnownValue

```java
存所有shape中用的值，自包含
Number value;
NumberKind kind; // int float double Big_int
```


* KnownExplicitValue

* KnownSymbolicValue

  ```java
  // symbol or memory address
  NumberKind is BigInteger
  ```

  * KnownAddressValue

    ```java
    Address address;
    ```




### Examples

Step1.

```
int global = 1;// global

1. SGRegion named as sg_global
KnownExplicitValue size[value=4, kind=int]
String label[global]
CType type[CSimpleType->signed int]

2. SGHasValueEdge as hvE_global
long value [1]
CType type[CSimpleType-> singed int]
int offset [0]
SGObject [sg_global]

3. CShapeGraph
Set<Long> values 放入 hvE_global's value, 也就是1

4. ShapeState
explicitValues 放入边value作为symbolicValue对应的explicitValue
KnownSymbolicValue key [value=1 kind=BigInteger]
KnownExplicitValue value [value=1 kind=Int]
```



Step2.

```
int main(){}

1. CStackFrame as stackFrame_main
CFunctionDeclaration function;
SGRegion returnObject;
```



Step3.

```
int i;

1. SGRegion named as main_i
KnownExplicitValue size[value=4, kind=int]
String label[i]
CType type[CSimpleType->signed int]

2. main_i 放入 stackFrame_main
```



Step4.

```
i = getNextInt(5) // int getNextInt(int n)

1. SGRegion named as getNextInt::n
KnownExplicitValue size[value=4, kind=int]
String label[n]
CType type[CSimpleType->signed int]

2. SGHasValueEdge as hvE_getNextInt::n
long value [2]
CType type[CSimpleType-> singed int]
int offset [0]
SGObject [getNextInt::n]

3. CShapeGraph
Set<Long> values 放入 hvE_getNextInt::n's value, 也就是2

4. ShapeState
explicitValues 放入边value作为symbolicValue对应的explicitValue
KnownSymbolicValue key [value=2 kind=BigInteger]
KnownExplicitValue value [value=5 kind=Int]

do in getNextInt
return from getNextInt

5. 释放getNextInt 的stackFrame
6. 更改main_i的hvEdge
```



Step5.

```
int *p = &global;

1. SGRegion named as main::p
KnownExplicitValue size[value=4, kind=int] 对应于指针长度
String label[p]
CType type[CPointerType->signed int]

2. SGPointToEdge as ptE_main::p
long value [4]
int offset [0]
SGObject [sg_global]

3. SGHasValueEdge as hvE_main::p
long value [4]
CType type[CPointerType->signed int]
int offset [0]
SGObject [main::p]

4. CShapeGraph
Set<Long> values 放入 ptE_main::p's value, 也就是4
```

至此

有一条HV边 hvE_main::P(SGHasValueEdge) 指向 main::p(SGRegion)  拥有一条PT边 ptE_main::P(SGPointToEdge) 指向 sg_global(SGRegion) 拥有一条HV边 hvE_global(SGHasValueEdge) 指向具体的值



Step6.

```
float f = 0.0;

1. SGRegion named as main::f
KnownExplicitValue size[value=4, kind=int] 对应于指针长度
String label[f]
CType type[CSimpleType->float]

2. SGHasValueEdge as hvE_main::f
long value [0]
CType type[CSimpleType->float]
int offset [0]
SGObject [main::f]

3. ShapeState
explicitValues 放入边value作为symbolicValue对应的explicitValue
KnownSymbolicValue key [value=0 kind=BigInteger]
KnownExplicitValue value [value=0 kind=Int]
```

