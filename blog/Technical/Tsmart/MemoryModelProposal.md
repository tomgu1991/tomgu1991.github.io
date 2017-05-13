1. 用latex，加文字梳理，写成数学形式
2. 先定义Memory Graph
3. 类和value对应
4. 实现细节可以先参考

### TODO

1. Type -> use llvm type?   %struct.RT = type { i8, [10 x [20 x i32]], i8 }  =》 用LLVM本身

2. align   =》 Graph的一个属性

3. build-in functions? intrinsic  =》 当作一个函数

4. struct ? use object or value?  => 在object拆开

5. MemoryRegisterObject: register size and point-to size? =》 register size

6. write i64 to i32, from left to right? right to left? =》 Model

7. need type to position

   ```
   %struct.RT = type { i8, [10 x [20 x i32]], i8 }
   %struct.ST = type { i32, double, %struct.RT }

   define i32* @foo(%struct.ST* %s) nounwind uwtable readnone optsize ssp {
   entry:
     %arrayidx = getelementptr inbounds %struct.ST, %struct.ST* %s, i64 1, i32 2, i32 1, i64 5, i64 13
     ret i32* %arrayidx
   }

   we store as bits, so we need to know, i32 2 is the third element, i32 1 is the seconde
   ```

8. float/double bit-version? =》 Chen guang

9. for return value? the same object or a copy?  =? TODO

10. aggregate value => 是放在object叉开  还是  value上

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



#### Value - TOOD - aggregate value

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
	aggregate value
	
MemorySymbolicValue ::= <type>, <id>, Version(id), <label>


	
```


#### Object 特殊的value

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

##### shl/lshr/ashr/and/or/xor

```
<result> = add <ty> <op1>, <op2>          ; yields ty:result

// register如何处理，写成推导

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

```
<result> = alloca [inalloca] <type> [, <ty> <NumElements>] [, align <alignment>]     ; yields type*:result

%ptr = alloca i32, i32 4, align 1024

1. at MemoryStackFrame_N
	create MemoryRegionObject: <NumElements>X<ty>, type, id, MemoryContent
    create MemoryRegisterObject: sizeTODO, <type>*, <result>, MemoryContent
    
2. ‘alloca‘d memory is automatically released when the function returns. 
	when do this when function return, we will drop the stackFrame_N
```



##### load

```
<result> = load [volatile] <ty>, <ty>* <pointer>[, align <alignment>][, !nontemporal !<index>][, !invariant.load !<index>][, !invariant.group !<index>][, !nonnull !<index>][, !dereferenceable !<deref_bytes_node>][, !dereferenceable_or_null !<deref_bytes_node>][, !align !<align_node>]

<result> = load atomic [volatile] <ty>, <ty>* <pointer> [singlethread] <ordering>, align <alignment> [, !invariant.group !<index>]

!<index> = !{ i32 1 }
!<deref_bytes_node> = !{i64 <dereferenceable_bytes>}
!<align_node> = !{ i64 <value_alignment> }

%ptr = alloca i32                               ; yields i32*:ptr
store i32 3, i32* %ptr                          ; yields void
%val = load i32, i32* %ptr                      ; yields i32:val = i32 3

1. at MemoryStackFrame_N
	create MemoryRegisterObject: sizeof(<ty>), <ty>, <result>, MemoryContent

```



##### store

```
store [volatile] <ty> <value>, <ty>* <pointer>[, align <alignment>][, !nontemporal !<index>][, !invariant.group !<index>]        ; yields void

store atomic [volatile] <ty> <value>, <ty>* <pointer> [singlethread] <ordering>, align <alignment> [, !invariant.group !<index>] ; yields void

%ptr = alloca i32                               ; yields i32*:ptr
store i32 3, i32* %ptr                          ; yields void
%val = load i32, i32* %ptr                      ; yields i32:val = i32 3

%0 = load i32, i32* %x, align 4
store i32 %0, i32* %i, align 8

1. at MemoryStackFrame_N
	(1) access <value>
	(2) access <pointer> must be a MemoryRegisterObject and type is pointer
	(3) access <pointer>#memoryObjectID
	(3)	update <value>#MemoryContent into <pointer>#memoryObjectID#MemoryContent
```



##### fence - TODO



##### cmpxchg

```
cmpxchg [weak] [volatile] <ty>* <pointer>, <ty> <cmp>, <ty> <new> [singlethread] <success ordering> <failure ordering> ; yields  { ty, i1 }

entry:
  %orig = load atomic i32, i32* %ptr unordered, align 4                      ; yields i32
  br label %loop

loop:
  %cmp = phi i32 [ %orig, %entry ], [%value_loaded, %loop]
  %squared = mul i32 %cmp, %cmp
  %val_success = cmpxchg i32* %ptr, i32 %cmp, i32 %squared acq_rel monotonic ; yields  { i32, i1 }
  %value_loaded = extractvalue { i32, i1 } %val_success, 0
  %success = extractvalue { i32, i1 } %val_success, 1
  br i1 %success, label %done, label %loop

done:
  ...
  

1. access <pointer>#memoryObjectID
2. access <cmp>
3. compare, 
	equal -> update <pointer>#memoryObjectID#MemoryContent by <new>
4. create MemoryRegisterObject: size, <{typ, i1}>, <result>, MemoryContent
```



##### atomicrmw

```
atomicrmw [volatile] <operation> <ty>* <pointer>, <ty> <value> [singlethread] <ordering>; yields ty

The type of ‘<value>’ must be an integer type whose bit width is a power of two greater than or equal to eight and less than or equal to a target-specific size limit. 

1. access <pointer>
2. create new MemoryRegisterObject by <pointer>
3. update <pointer>#MemoryContent by <value> according to <operation>
```



##### getelementptr - TODO more details needed

```
<result> = getelementptr <ty>, <ty>* <ptrval>{, <ty> <idx>}*
<result> = getelementptr inbounds <ty>, <ty>* <ptrval>{, <ty> <idx>}*
<result> = getelementptr <ty>, <ptr vector> <ptrval>, <vector index type> <idx>

1. access <ty> TODO
2. access <ptrval>*
3. create MemoryRegionObject by <ty> <idx>
4. create MemoryRegisterObject as <result>
```



#### Conversion Operations

##### trunt/ zext/ sext/ fptrunc/ fpext/fptoui/fptosi/uitofp/sitofp/ptrtoint/inttoptr/bitcast

```
<result> = trunc <ty> <value> to <ty2>             ; yields ty2

1. access <value>
2. create MemoryRegisterObject: sizeof(<ty2>), label, <ty2>, MemoryContent
```

##### addrspacecast - TODO

##### 

#### Other Operations

##### icmp/fcmp

```
<result> = icmp <cond> <ty> <op1>, <op2>   ; yields i1 or <N x i1>:result

1. access <op1> , <op2>
2. create MemoryRegisterObject: 1, <result>, i1, MemoryContent
```



##### phi

```
Do nothing
```



##### select - TODO

```
<result> = select selty <cond>, <ty> <val1>, <ty> <val2>             ; yields ty

selty is either i1 or {<N x i1>}

the same object or create a new object?
```



##### call

```
<result> = [tail | musttail | notail ] call [fast-math flags] [cconv] [ret attrs] <ty>|<fnty> <fnptrval>(<function args>) [fn attrs]
             [ operand bundles ]
             
             
1. create MemoryStackFrame
2. create parameters -> MemoryRegisterObject
3. access arguments
4. update values
```



##### va_arg - TODO

##### landingpad























