### TODO

1. Type -> use llvm type?
2. align
3. build-in functions? intrinsic 
4. struct ? use object or value?

### Memory Graph Definition

#### Function

```l
Version = id:int -> version:int 

MemoryContent = start:int X end:int -> value:MemoryGraphValue

```



#### Type

```
TODO??
```



#### Value

```y

MemoryBitValue ::= <0>|<1>|<*>

MemoryGraphValue ::= <MemoryBitVector>|<MemoryInterpretedValue>|<MemorySymbolicValue>

MemoryBitVector ::= [MemoryBitValue]+

MemoryInterpretedValue ::= <type>, <value> [, other properties]
	i32, value
	i64, value
	float, value
	long, value
	double, value
	i32*, value, memoryObjectID
	
MemorySymbolicValue ::= <id>, Version(id), <label>
	
```


#### Object

```
MemoryObject ::= <MemoryRegisterObject>|<MemoryRegionObject>

MemoryRegisterObject ::= <bitsize>, <type>, <label>, MemoryContent
	label is name of register
	bitsize:MemoryGraphValue
		i32, 5
	MemoryContent = int X int -> MemoryGraphValue
		0 X 31 -> (i32, 5)
		24 X 31 -> (0000 0101)
		
MemoryRegionObject ::= <bitsize>, <type>, <memoryObjectID>, MemoryContent
	memoryObjectID is auto-generated
```



#### Graph

```
MemoryGraph ::= <MachineModel>, Version, <MemoryStack>, <MemoryGlobal>, <MemoryHeap>

MachineModel ::= [key -> value]+

MemoryStack ::= [MemoryStackFrame]+
MemoryStackFrame ::= <declaration>, [MemoryObject]+

MemoryGlobal ::= [MemoryObject]+

MemoryGlobal ::= [MemoryObject]+
```



### IR Translation Semantics

#### Terminator

##### ret

```
ret <type> <value>

ret i32 %1
%call = call i32 @getNextInt(i32 %1)

1. at MemoryStackFrame_N, check what needed
2. at MemoryStackFrame_N-1
	create a MemoryInterpretedValue v: i32, value
	create a MemoryRegisterObject ro: 32, i32, call, MemoryContent#(0,31,v)
	
```

##### br

```
br i1 <cond>, label <iftrue>, label <iffalse>
br label <dest>          ; Unconditional branch

do nothing
```

##### switch

```
switch <intty> <value>, label <defaultdest> [ <intty> <val>, label <dest> ... ]

do nothing
```

##### indirectbr - TODO



##### invoke - TODO



##### resume - TODO



##### catchswitch - TODO



##### catchret - TODO



##### cleanupret - TODO



##### unreachable - TODO



#### Binary Operations

##### add/fadd/sub/fsub/mul/fmul/udiv/sdiv/fdiv/urem/srem

##### shl/lshr/ashr/add/or/xor

```
<result> = add <ty> <op1>, <op2>          ; yields ty:result

1. at MemoryStackFrame_N
	create MemoryGraphValue
	create MemoryRegisterObject
```



#### Vector Operations

##### extractelement

```
<result> = extractelement <n x <ty>> <val>, <ty2> <idx>  ; yields <ty>
1. at MemoryStackFrame_N
	create MemoryGraphValue
	create MemoryRegisterObject
```



##### insertelement

```
<result> = insertelement <n x <ty>> <val>, <ty> <elt>, <ty2> <idx>    ; yields <n x <ty>>

1. access <val>
2. update MemoryContent, replace with <elt> at <idx>
```



##### shufflevector

```
<result> = shufflevector <n x <ty>> <v1>, <n x <ty>> <v2>, <m x i32> <mask>    ; yields <m x <ty>>
1. at MemoryStackFrame_N
	create m X MemoryGraphValue
	create MemoryRegisterObject, and update MemoryContent by each of <mask>

```



#### Aggregate Operations

##### extractvalue

```
<result> = extractvalue <aggregate type> <val>, <idx>{, <idx>}*

1. at MemoryStackFrame_N
	create MemoryGraphValue by <idx>+
	create MemoryRegisterObject, and update MemoryContent by each of <idx>
```



##### insertvalue

```
<result> = insertvalue <aggregate type> <val>, <ty> <elt>, <idx>{, <idx>}*    ; yields <aggregate type>

1. at MemoryStackFrame_N
	create MemoryGraphValue by <idx>+
	create MemoryRegisterObject, copy from <val> and update MemoryContent by each of <idx>
```



#### Memory Operations

##### alloca

##### load

##### store

##### fence

##### cmpxchg

##### atomicrmw

##### getelementptr

#### Conversion Operations

##### trunt

##### zext

##### sext

##### bitcast

#### Other Operations

##### icmp

##### fcmp

##### phi

##### select

##### call

##### va_arg

##### landingpad























