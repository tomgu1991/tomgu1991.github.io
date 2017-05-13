# Memory Graph Proposal

Memory Graph (MG) models the state of memory and registers, it is the core CPA of analysis.

### Memory Graph Definition

#### Terminology

$$
\begin{align}
	& \text{tuple}
	&& \langle ~~ \rangle \\
	& \text{set of instance of object}
	&& [ ~~ ]^+ \\
	& \text{an instance of } type \text{ named as } name
	&& name:type \\
	\\
	& \text{Function to access } version\_id:int \text{ by } name:String &  \mathsf{V_{ersion}} & : {String \rightarrow int}  \\
	& \text{Function to extract } value:MemoryValue \text{ by positions} &  \mathsf{M_{emoryContent}} & :  {int \times int \rightarrow MemoryValue}  \\
	\\
	& \text{Declaration of function} & function & : FunctionDeclaration \\
	& \text{Identifier } & id & : int \\
	& \text{Register label or name of a value } & label & : String \\
	& \text{Bit width size for } object \text{ self} & size & : long \\
	& \text{Specific field, such as } point\_to \text{ for } pointer \text{ type} & properties & : Key \rightarrow Value
\end{align}
$$

#### Definition

$$
\begin{align}

	MemoryGraphState & ::=  \langle{MemoryGraph, MemoryLayout}\rangle \\
	MemoryGraph & ::=  \langle{StackMemory, GlobalMemory, HeapMemory, RegisterMemory}\rangle \\
	StackMemory & ::=   [stackFrame]^+ \\
	StackFrame & ::= \langle{function, [MemoryRegion]^+}\rangle\\
	GlobalMemory & ::=  [MemoryRegion]^+\\
	HeapMemory & ::=  [MemoryRegion]^+\\
	RegisterMemory & ::=  [MemoryRegion]^+\\
	\\
	MemoryRegion & ::=  \langle{id, label, type, size, M_{emoryContent} }\rangle \\
	MemoryValue & ::= MemoryBitVector | MemoryInterpretedValue | MemorySymbolicValue \\
	MemoryBitVector & ::= [MemoryBitValue]^+ \\
	MemoryInterpretedValue & ::= \langle{type, size, value, properties}\rangle \\
	MemorySymbolicValue & ::=\langle{label, version\_id}\rangle \\
	MemoryBitValue & ::=0 | 1 | * \\
	\\
	MemoryLayout & ::=  {LLVM\_MachineModel}\\
	type & ::= { LLVM\_type}\\
	value & ::= {LLVM\_value} \\
	\\

\end{align}
$$

#### MemoryGraphState

An instance of memory graph (considered as a CPA state) is a tuple of `<graph, layout>` where `graph` is the state of memory/register and `layout` is the machine layout defined by LLVM IR, i.e.,

```haskell
type MemoryGraphState = <
	graph : MemoryGraph, 
	layout : MemoryLayout
	>
```

#### MemoryGraph

The core component is `MemoryGraph`, which is the graph that captures the logical relation between identities in the memory. 

```haskell
type MemoryGraph = <
	stack : StackMemory, 
	global : GlobalMemory, 
	heap : HeapMemory, 
	register : RegisterMemory>
```

#### StackMemory

Stack memory is organized as a stack of stack frames.

```haskell
type StackMemory = frames: [StackFrame]+
```

#### StackFrame

 Each stack frame is a segment of memory objects.

```haskell
type StackFrame = <
	del : Declaration, 
	objs : [MemoryRegion]+ -- array
	>
```

#### GlobalMemory

Global memory is like a stack frame, treated as $array$

```haskell
type GlobalMemory = objs :　[MemoryRegion]+
```

#### HeapMemory

Heap memory is a set of unrelated memory objects, treated as $set$

```haskell
type HeapMemory = objs :　[MemoryRegion]+
```

#### RegisterMemory

Register memory is a map to save all the registers,  $RegisterMemory : String \rightarrow MemoryRegion$

```haskell
type RegisterMemory = objs :　[MemoryRegion]+
```

#### MemoryRegion

A memory region is  in the stack / heap / global / register, which represents a region of memory

```haskell

type MemoryRegion = <
	id : int -- indentifier
	label : String -- name of register
	size : long, -- length in bits    
	t : Type, -- LLVM type of the region
    content : MemoryContent -- the content
    >
```

#### MemoryValue

Used to present values stored in `MemoryContent`

```haskell
data MemoryValue = 
	MemoryBitVector | -- in format of bit vectors for explicit values
	MemoryInterpretedValue | -- in format of readability for explicit values
	MemorySymbolicValue -- for symbolic value
	
type MemoryBitVector = [MemoryBitValue]+ -- bit-width is a property of MemoryBitVector
data MemoryBitValue = '0' | '1' | '*' -- 0, 1, or uknown

-- interpreted value that may occur in the memory graph
-- notice that not all types in the LLVM type system are included (e.g., Token)
-- TODO: (1) the value of PointerValue and all AggregateValue needs discussion. (2) whether we need Opaque structure ?
type MemoryInterpretedValue (MemoryValue) =		<		-- abstract super class of interpreted values
	t : Type,		-- LLVM type of the value
	size : long 	-- size of value 
	>
type IntegerValue (MemoryInterpretedValue) = <
	value : long	-- to be determined, whether we use BigDecimal or long
	>
type FloatValue (MemoryInterpretedValue) = <
	value : float
	>
type PointerValue (MemoryInterpretedValue) = <
	value : ??? -- should be point-to address
	>
type AggregateValue (MemoryInterpretedValue)  			-- abstract super class of aggregate values
type ArrayValue (AggregateValue) = <
	value : ???
	>
type StructureValue (AggregateValue) = <
	value : ???
	>

type MemorySymbolicValue = <
	name : String
	version : int
	>
```



#### Memory Content

```haskell
MemoryContent:: int -> int -> MemoryValue
```

*Memory Content* is the value which is stored at the given memory region. It may be interpreted or uninterpreted:

- Uninterpreted Memory Content: the content is recorded as a bit-vector, 
  - e.g., (0000 0101): the binary encoding of a 8-bit integer with value 5.
- Interpreted Memory Content: the interpreted value of the bit-vector. 
  - e.g., i32 value 5.
  - e.g., float value 1.0

Interpreted / uninterpreted memory content can convert from each other with the help of their **type** and **bit-width**.

*Why there are both interpreted / uninterpreted memory contents?* The reason is that they are both used in programming. For instance, we may store the value of `x` by `x = 5` and use its value via `(x & 1) == 0`. The format of writing and reading may not be always consistent. To avoid frequent conversion between the two forms, we store the value of certain memory region by the format in which it is defined, and perform the conversion on demand.

#### **Example**

Assume `int` is 32-bit and `float` is 64-bit. A memory region may store content of the following:

| Definition                           | Initial values  | Memory region                     | Note                   |
| ------------------------------------ | --------------- | --------------------------------- | ---------------------- |
| `x : int`                            | 100             | (32, i32, 100)                    |                        |
| `x : structure {a : int, b : float}` | not initialized | (96, {i32, float}, {?, ?})        | initial value unknown. |
| `x : int[10]={1,0}`                  | {1, 0}          | (320, [10 x i32], [1, 0, ..., 0]) |                        |



### Translation Semantics

#### Terminology

$$
\begin{align}
&\text{execution environment, including top of stack, global, heap and register} & &\Psi\\

&\text{operator to update $MemoryContent$, we leave details in implementation} & &\Delta_{content}\\

&\text{auto generate a id} & &\mathsf{ID}\\

& \text{tuple }  & \langle& ~~~\rangle \\

& \text{extract an element from $s$ by type $etp$ including, index, name, position, type and so on} & s [& ~~~]_{ety} \\

& \text{a collection of } T \text{ named as } C & C& : T \\ 

& \text{an instance of } T \text{ named as } v & v& : T \\

& \text{a pair of key and value}   & k & \rightarrow v \\

& \text{map of pairs of key and value }   & \{k & \rightarrow v\} \\

& \text{size of } T  & \mathsf{sizeof} (T) & \mapsto size:int \\

& \text{type of } T  & \mathsf{type} (T) & \mapsto ty:String \\

& \text{create a new instance of } T \text{ with initializaiton} \{k \rightarrow v\}  \text{ in execution environment } \rho& \mathsf{new}_\rho (T\{k \rightarrow v\} & \mapsto obj:T \\

& \text{push a new instance  $ obj $ of } T \text{ into a stack of } T  & \mathsf{push} (Stack:T, obj:T) & \mapsto Stack:T\\

& \text{pop an instance  $ obj $ of } T \text{ from stack of } T  & \mathsf{pop} (Stack: T) & \mapsto \langle Stack:T , obj:T \rangle\\

& \text{peek an instance  $ obj $ of } T \text{ from stack of } T & \mathsf{peek} (stack: T) & \mapsto \langle Stack:T , obj:T \rangle\\

& \text{update function to update value of  } x \text{ by } val \text{ in a collection } s \mathsf{update} & \mathsf{update}(C:T, x:\_, e:T)  & \mapsto C:T\\

& \text{perform the operation  } op \text{ by } \text{ parameters } x_i &  \mathsf{perform}_{op}(x_1, x_2, ...)  & \mapsto obj:T\\

& \text{calculate the transition result of } \rho = (s, h, r, g) \text{on an edge $e$ labeled with the instrunction $inst$} & (s, h, r, g) &\overset{e, inst}{\mapsto} (s', h', r', g')  \\

\end{align}
$$



#### Convention

1. We omit type if there is no ambiguity.
2. We omit fields in $\rho = (s, h, r, g) $, if there are no changes.
3. We omit fields in initialization of $\mathsf{new}$ if it is not important and not ambiguous.



#### `PhiEdge` - TODO

#### `SwitchEdge`- TODO

#### `BlankEdge`- TODO



#### `FunctionCall` edge

```
<result> = [tail | musttail | notail ] call [fast-math flags] [cconv] [ret attrs] <ty>|<fnty> <fnptrval>(<function args>) [fn attrs]
             [ operand bundles ]
```


$$
(s, h, r, g) \overset{e, inst}{\mapsto} (s', h', r', g') , \text{where}\\
 
 e = FunctionCallEdge, \\
 
 inst = \text{%return = call type @f(ty1 arg1,...)}  \\
 
 frame = \mathsf{new}_{(s,h,r,g)} (StackFrame\{del \rightarrow f[Declaration]_{name}\}, objs \rightarrow NULL) \\
 
s' = \mathsf{push}(s, frame:StackFrame)
$$

#### `FunctionReturn` edge

$$
(s, h, r, g) \overset{e, inst}{\mapsto} (s', h', r', g') , \text{where}\\
 
 e = FunctionReturnEdge, \\
 
 inst = \text{%return = call type @f(ty1 arg1,...)}  \\
 
\langle s', frame \rangle = \mathsf{pop}(s) \\

r' = \mathsf{update} (r, \%return, frame[f.ret]_{name} )  \
$$



#### `Assume` edge

It will change memory model. Only extract memory models to do comparison.





#### `Instrunction`edge

#####  `ret`

$$
(s, h, r, g) \overset{e, inst}{\mapsto} (s', h', r', g') , \text{where}\\
 
 e = InstructionEdge, \\
 
 inst = \text{ret ty val}  \\
 
 region = \mathsf{new}_{(s,h,r,g)} (MemoryRegion \{  label \rightarrow f.ret,  type \rightarrow \text{ty}, size \rightarrow \mathsf{sizeof}( \text{val} ), content \rightarrow\Delta_{content}( \text{val} ) \}) \\

\langle s'', frame\rangle = \mathsf{pop}(s)\\

frame'' = \mathsf{update}(frame, f.ret, region) \\

s' = \mathsf{push} (s'', frame'')
$$

#### 

Binary Operations  / Conversion Operations

##### `add`

$$
(s, h, r, g) \overset{e, inst}{\mapsto} (s', h', r', g') , \text{where}\\
 
 e = InstructionEdge, \\
 
 inst = \text{result = add ty op1, op2}  \\
 
obj1 = \Psi[ \text{op1}]_{\_} \\

obj2 = \Psi[\text{op2}]_{\_} \\
 
val : MemoryValue= \mathsf{perform}_{add} (obj1, obj2) \\

region = \mathsf{new}_{(s,h,r,g)} (MemoryRegion \{  label \rightarrow \text{result},  type \rightarrow \text{ty}, size \rightarrow \mathsf{sizeof}(val),content \rightarrow\Delta_{content}(val) \}) \\

r' = \mathsf{update} (r, r[label]_{name}, region)
$$

Aggregate Operations

##### `extractvalue`

$$
(s, h, r, g) \overset{e, inst}{\mapsto} (s', h', r', g') , \text{where}\\
 
 e = InstructionEdge, \\
 
 inst = \text{result = extractvalue ty val [id]+}  \\
 
objs= \Psi[\text{val}]_{index} \\

region = \mathsf{new}_{(s,h,r,g)} (MemoryRegion \{  label \rightarrow \text{result},  type \rightarrow \mathsf{type}(objs), size \rightarrow \mathsf{sizeof}(objs),content \rightarrow\Delta_{content}(objs) \}) \\

r' = \mathsf{update} (r, r[\text{label}]_{name}, region)
$$



Memory Operations

##### `alloca`

$$
(s, h, r, g) \overset{e, inst}{\mapsto} (s', h', r', g') , \text{where}\\
 
 e = InstructionEdge, \\
 
 inst = \text{result = alloca type, ty num}  \\
 
val = \mathsf{new}_{(s,h,r,g)} ( ArrayValue\{ type \rightarrow \text{ty}, size \rightarrow \mathsf{sizeof}( \text{ty} )* \text{num}, value \rightarrow null \} ) \\

valRegion = \mathsf{new}_{(s,h,r,g)} (MemoryRegion \{ id \rightarrow \mathsf{ID},  label \rightarrow null,  type \rightarrow \mathsf{type}(val), size \rightarrow \mathsf{sizeof}(val),content \rightarrow\Delta_{content}(val) \}) \\

pt = \mathsf{new}_{(s,h,r,g)} ( PointerValue\{ type \rightarrow \text{type}, size \rightarrow \mathsf{sizeof}(\text{type}), value \rightarrow valRegion \} ) \\

ptRegion = \mathsf{new}_{(s,h,r,g)} (MemoryRegion \{  label \rightarrow result,  type \rightarrow \mathsf{type}(\text{type}), size \rightarrow \mathsf{sizeof}(pt),content \rightarrow\Delta_{content}(pt) \}) \\

s' = \mathsf{update} (s, s[id]_{id}, valRegion) \\

r' = \mathsf{update} (r, r[label]_{name}, ptRegion)
$$



##### `load`

$$
(s, h, r, g) \overset{e, inst}{\mapsto} (s', h', r', g') , \text{where}\\
 
 e = InstructionEdge, \\
 
 inst = \text{result = load ty, ty* pointer}  \\
 
val : MemoryValue= \mathsf{new}_{(s,h,r,g)} ( MemoryValue\{ type \rightarrow \text{ty}, size \rightarrow \mathsf{sizeof}(\text{ty})， value \rightarrow \Psi[\text{pointer}]_{name}[\text{ty}]_{type} \} ) \\

region = \mathsf{new}_{(s,h,r,g)} (MemoryRegion \{  label \rightarrow \text{result},  type \rightarrow \text{ty}, size \rightarrow \mathsf{sizeof}(\text{ty*}),content \rightarrow\Delta_{content}(val) \}) \\

r' = \mathsf{update} (r, r[\text{result}]_{name}, region)
$$

##### `store`

$$
(s, h, r, g) \overset{e, inst}{\mapsto} (s', h', r', g') , \text{where}\\
 
 e = InstructionEdge, \\
 
 inst = \text{store ty val, ty* pointer}  \\
 
val : MemoryValue= \mathsf{new}_{(s,h,r,g)} ( MemoryValue\{ type \rightarrow \text{ty}, size \rightarrow \mathsf{sizeof}(\text{ty})， value \rightarrow \text{val} ) \\

ptRegion = \Psi[pointer]_{name} \\

ptRegion' = \mathsf{update}(ptRegion, ptRegion[content]_{name}, \Delta_{content}(val)) \\

r' = \mathsf{update} (r, r[\text{pointer}]_{name}, ptRegion')
$$

##### `getelementptr` TODO

$$
(s, h, r, g) \overset{e, inst}{\mapsto} (s', h', r', g') , \text{where}\\
 
 e = InstructionEdge, \\
 
 inst = \text{result = getelementptr ty, ty* ptrval {, ty_i idx_i}}*  \\
 
 region : MemoryRegion = \Psi [\text{ptrval}]_{name}\\
 

val : MemoryValue=  region[\text{idx_0}]_{index}[...]_{index}[\text{idx_n}]_{index}\\


region' = \mathsf{new}_{(s,h,r,g)} (MemoryRegion \{  label \rightarrow \text{result},  type \rightarrow \mathsf{type}(val), size \rightarrow \mathsf{sizeof}(val),content \rightarrow\Delta_{content}(val) \}) \\

r' = \mathsf{update} (r, r[\text{result}]_{name}, region')
$$





#### Implementation:

1. For $size$, we treat  as bit-length.
2. All $id$ s are auto-generated.
3. For $build-in$ functions and $intrinsic$, we treat as general functions
4. For $align$, we treat as a property of $graph$
5. For $aggregate\_type$ , we treat as $atomatic\_type$, and decompose in $M_{emoryContent}$
6. For $size$, we store the size of $object$,  rather it's $M_{emoryContent}$
7. For $\Psi$ , we have to judge in implementation





### TODO

1. how to describe bit-version of float/double?
2. for return value, the reference or a copy?
3. Merge and Stop operator?
4. the value of PointerValue and all AggregateValue needs discussion
5. whether we need Opaque structure
6. BigDecimal or long for IntegerValue
7. global is stored continually? 
8. more examples
9. global/heap/stackFrame also need key->value???
10. where to store the result of `getelementptr`