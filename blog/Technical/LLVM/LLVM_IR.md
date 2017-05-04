# LLVM IR

>clang: 3.9.0

### What is LLVM IR

LLVM IR is  short for LLVM intermediate representation. LLVM is a **Static Single Assignment (SSA)** based representation that provides type safety, low-level operations, flexibility, and the capability of representing ‘all’ high-level languages cleanly. It is the common code representation used throughout all phases of the LLVM compilation strategy.

The LLVM code representation is designed to be used in three different forms: as an in-memory compiler IR, as an on-disk bitcode representation (suitable for fast loading by a Just-In-Time compiler), and **as a human readable assembly language representation.** This allows LLVM to provide a powerful intermediate representation for efficient compiler transformations and analysis, while providing a natural means to debug and visualize the transformations. The three different forms of LLVM are all equivalent. This document describes the human readable representation and notation.

The LLVM representation aims to be light-weight and low-level while being expressive, typed, and extensible at the same time. It aims to be a “universal IR” of sorts, by being at a low enough level that high-level ideas may be cleanly mapped to it (similar to how microprocessors are “universal IR’s”, allowing many source languages to be mapped to them). By providing type information, LLVM can be used as the target of optimizations: for example, through pointer analysis, it can be proven that a C automatic variable is never accessed outside of the current function, allowing it to be promoted to a simple SSA value instead of a memory location.

### An example

我们使用的clang3.9.0, Ubuntu 16.04编译如下文件

```c
// file test.c
int getNextInt(int n)
{
	return n + 1;
}

int main(int argc, char const *argv[])
{
	int i;

	i = getNextInt(5);

	i = i * i;

	i = getNextInt(i);

	return 0;
}
```

产生的ll文件如下

```
; ModuleID = 'test.c'
source_filename = "test.c"
target datalayout = "e-m:e-i64:64-f80:128-n8:16:32:64-S128"
target triple = "x86_64-unknown-linux-gnu"

; Function Attrs: nounwind uwtable
define i32 @getNextInt(i32 %n) #0 {
entry:
  %n.addr = alloca i32, align 4
  store i32 %n, i32* %n.addr, align 4
  %0 = load i32, i32* %n.addr, align 4
  %add = add nsw i32 %0, 1
  ret i32 %add
}

; Function Attrs: nounwind uwtable
define i32 @main(i32 %argc, i8** %argv) #0 {
entry:
  %retval = alloca i32, align 4
  %argc.addr = alloca i32, align 4
  %argv.addr = alloca i8**, align 8
  %i = alloca i32, align 4
  store i32 0, i32* %retval, align 4
  store i32 %argc, i32* %argc.addr, align 4
  store i8** %argv, i8*** %argv.addr, align 8
  %call = call i32 @getNextInt(i32 5)
  store i32 %call, i32* %i, align 4
  %0 = load i32, i32* %i, align 4
  %1 = load i32, i32* %i, align 4
  %mul = mul nsw i32 %0, %1
  store i32 %mul, i32* %i, align 4
  %2 = load i32, i32* %i, align 4
  %call1 = call i32 @getNextInt(i32 %2)
  store i32 %call1, i32* %i, align 4
  ret i32 0
}

attributes #0 = { nounwind uwtable "disable-tail-calls"="false" "less-precise-fpmad"="false" "no-frame-pointer-elim"="true" "no-frame-pointer-elim-non-leaf" "no-infs-fp-math"="false" "no-jump-tables"="false" "no-nans-fp-math"="false" "no-signed-zeros-fp-math"="false" "stack-protector-buffer-size"="8" "target-cpu"="x86-64" "target-features"="+fxsr,+mmx,+sse,+sse2,+x87" "unsafe-fp-math"="false" "use-soft-float"="false" }

!llvm.ident = !{!0}

!0 = !{!"clang version 3.9.0 (tags/RELEASE_390/final)"}
```

### Deep in IR

我们先从这里例子来分析IR的语义，超过这个例子中的内容，我们再给出相应的源代码和IR。

#### Identifier

其实：IR的内部都是一个一个指令组成，并没有标识符，这些表示符是后生成给人看的。特别是以数字命名的标识符，是中间结果。可以认为所有的指令等号左边是一个引用。后面用到的地方都是通过这个引用直接用指令替换。

Global identifiers (functions, global variables) begin with the `'@'` character. Local identifiers (register names, types) begin with the `'%'` character. Additionally, there are three different formats for identifiers, for different purposes:

* Named values are represented as a string of characters with their prefix. 对应于源程序中有名字的变量，或者有特定含义的变量。

  ```c
  int getNextInt(int n)
  	@getNextInt
  	%n
  ```

* Unnamed values are represented as an unsigned numeric value with their prefix. Unnamed temporaries are created when the result of a computation is not assigned to a named value.
  ```c
  i = i * i;
  	%i = alloca i32, align 4
    	%0 = load i32, i32* %i, align 4
    	%1 = load i32, i32* %i, align 4
    	%mul = mul nsw i32 %0, %1
    	store i32 %mul, i32* %i, align 4
  ```
* Constants



#### Comments

Comments are delimited with a ‘`;`‘ and go until the end of line.

```
; ModuleID = 'test.c' => 这个module是test.c文件的
```



#### Module

LLVM programs are composed of `Module`‘s, each of which is a translation unit of the input programs. Each module consists of functions, global variables, and symbol table entries. Modules may be combined together with the LLVM linker, which merges function (and global variable) definitions, resolves forward declarations, and merges symbol table entries.

```
; ModuleID = 'test.c'
; other constant

; External declaration of functions

; Definition of function 

; attributs

; Named metadata
```



#### Function

A function definition contains a list of basic blocks, forming the CFG (Control Flow Graph) for the function. Each basic block may optionally start with a label (giving the basic block a symbol table entry), contains a list of instructions, and ends with a [*terminator*](http://releases.llvm.org/3.9.0/docs/LangRef.html#terminators) instruction (such as a branch or function return). If an explicit label is not provided, a block is assigned an implicit numbered label, using the next value from the same counter as used for unnamed temporaries ([*see above*](http://releases.llvm.org/3.9.0/docs/LangRef.html#identifiers)). For example, if a function entry block does not have an explicit label, it will be assigned label “%0”, then the first unnamed temporary in that block will be “%1”, etc.

```c
define [linkage] [visibility] [DLLStorageClass]
       [cconv] [ret attrs]
       <ResultType> @<FunctionName> ([argument list])
       [(unnamed_addr|local_unnamed_addr)] [fn Attrs] [section "name"]
       [comdat [($name)]] [align N] [gc] [prefix Constant]
       [prologue Constant] [personality Constant] (!name !N)* { ... }
<type> [parameter Attrs] [name]

int getNextInt(int n){}

define i32 @getNextInt(i32 %n) #0 {}; # 0 is attributes group
```



#### Attribute

##### Attributes Group

Attribute groups are groups of attributes that are referenced by objects within the IR. They are important for keeping `.ll` files readable, because a lot of functions will use the same set of attributes.

```
attributes #0 = { nounwind uwtable "disable-tail-calls"="false" "less-precise-fpmad"="false" "no-frame-pointer-elim"="true" "no-frame-pointer-elim-non-leaf" "no-infs-fp-math"="false" "no-jump-tables"="false" "no-nans-fp-math"="false" "no-signed-zeros-fp-math"="false" "stack-protector-buffer-size"="8" "target-cpu"="x86-64" "target-features"="+fxsr,+mmx,+sse,+sse2,+x87" "unsafe-fp-math"="false" "use-soft-float"="false" }
```

##### Function Attributes

* nounwind

  ```
  This function attribute indicates that the function never raises an exception.
  ```

* uwtable - TODO

  ```
  This attribute indicates that the ABI being targeted requires that an unwind table entry be produced for this function even if we can show that no exceptions passes by it. This is normally the case for the ELF x86-64 abi, but it can be disabled for some compilation units.
  ```

* ​



#### Variable

##### Global Variable

As SSA values, global variables define pointer values that are in scope (i.e. they dominate) all basic blocks in the program. Global variables always define a pointer to their “content” type because they describe a region of memory, and all memory objects in LLVM are accessed through pointers. Global variables can be marked with `unnamed_addr` which indicates that the address is not significant, only the content. f the `local_unnamed_addr` attribute is given, the address is known to not be significant within the module. 



#### Label

```
The label type represents code labels.

Syntax:	
label

通常在每个block前边又一个label，例如
entry:

一个for循环
for.cond
for.body
for.inc
for.end
```



### Instruction

#### Terminator

##### ret

Syntax

```
ret <type> <value>       ; Return a value from a non-void function

ret void                 ; Return from void function
```

C

```c
return 0;
```

llvm

```
ret i32 5                       ; Return an integer value of 5
ret void                        ; Return from a void function
ret { i32, i8 } { i32 4, i8 2 } ; Return a struct of values 4 and 2
```

Semantic

```
When the ‘ret‘ instruction is executed, control flow returns back to the calling function’s context. If the caller is a “call” instruction, execution continues at the instruction after the call. If the caller was an “invoke” instruction, execution continues at the beginning of the “normal” destination block. If the instruction returns a value, that value shall set the call or invoke instruction’s return value.
```

Overview

```
The ‘ret‘ instruction is used to return control flow (and optionally a value) from a function back to the caller.

There are two forms of the ‘ret‘ instruction: one that returns a value and then causes control flow, and one that just causes control flow to occur.
```

Arguments

```
The ‘ret‘ instruction optionally accepts a single argument, the return value. The type of the return value must be a ‘first class‘ type.
```



##### br

Syntax

```
br i1 <cond>, label <iftrue>, label <iffalse>
br label <dest>          ; Unconditional branch
```

C

```c
	if (result > 10)
	{
		result ++;
	}else{
		return result;
	}
```

llvm

```
  %cmp = icmp sgt i32 %1, 10
  br i1 %cmp, label %if.then, label %if.else

if.then:                                          ; preds = %entry
  %2 = load i32, i32* %result, align 4
  %inc = add nsw i32 %2, 1
  store i32 %inc, i32* %result, align 4
  br label %if.end

if.else:                                          ; preds = %entry
  %3 = load i32, i32* %result, align 4
  store i32 %3, i32* %retval, align 4
  br label %return

if.end:                                           ; preds = %if.then
  %4 = load i32, i32* %result, align 4
  store i32 %4, i32* %retval, align 4
  br label %return
```

Semantic

```
Upon execution of a conditional ‘br‘ instruction, the ‘i1‘ argument is evaluated. If the value is true, control flows to the ‘iftrue‘ label argument. If “cond” is false, control flows to the ‘iffalse‘ label argument.
```

Overview

```
cause control flow to transfer to a different basic block in the current function
```

Arguments

```
a single ‘i1‘ value and two ‘label‘ values
a single ‘label‘ value
```

##### switch

Syntax

```
switch <intty> <value>, label <defaultdest> [ <intty> <val>, label <dest> ... ]
```

C

```c
	switch(n){
		case 1:
			result = 1;
			break;
		default:
			result = n;
	}
```

llvm

```
  %1 = load i32, i32* %n.addr, align 4
  switch i32 %1, label %sw.default [
    i32 1, label %sw.bb
  ]

sw.bb:                                            ; preds = %entry
  store i32 1, i32* %result, align 4
  br label %sw.epilog

sw.default:                                       ; preds = %entry
  %2 = load i32, i32* %n.addr, align 4
  store i32 %2, i32* %result, align 4
  br label %sw.epilog

sw.epilog:                                        ; preds = %sw.default, %sw.bb
```

Semantic

```
specifies a table of values and destinations
```

Overview

```
 transfer control flow to one of several different places
```

Arguments

```
an integer comparison value ‘value‘, a default ‘label‘ destination, and an array of pairs of comparison value constants and ‘label‘s.
```



##### indirectbr - TODO

Syntax

```
indirectbr <somety>* <address>, [ label <dest1>, label <dest2>, ... ]
```

C

```c

```

llvm

```

```

Semantic

```
Control transfers to the block specified in the address argument. All possible destination blocks must be listed in the label list, otherwise this instruction has undefined behavior.
```

Overview

```
The ‘indirectbr‘ instruction implements an indirect branch to a label within the current function, whose address is specified by “address”. Address must be derived from a blockaddress constant.
```

Arguments

```

```



##### invoke - TODO

Syntax

```
<result> = invoke [cconv] [ret attrs] <ty>|<fnty> <fnptrval>(<function args>) [fn attrs]
              [operand bundles] to label <normal label> unwind label <exception label>
```

C

```c

```

llvm

```

```

Semantic

```
This instruction is designed to operate as a standard ‘call‘ instruction in most regards. The primary difference is that it establishes an association with a label, which is used by the runtime library to unwind the stack.

This instruction is used in languages with destructors to ensure that proper cleanup is performed in the case of either a longjmp or a thrown exception. Additionally, this is important for implementation of ‘catch‘ clauses in high-level languages that support them.
```

Overview

```
The ‘invoke‘ instruction causes control to transfer to a specified function, with the possibility of control flow transfer to either the ‘normal‘ label or the ‘exception‘ label. If the callee function returns with the “ret” instruction, control flow will return to the “normal” label. If the callee (or any indirect callees) returns via the “resume” instruction or other exception handling mechanism, control is interrupted and continued at the dynamically nearest “exception” label.
```

Arguments

```

```



##### resume - TODO

Syntax

```

```

C

```c

```

llvm

```

```

Semantic

```
The ‘resume‘ instruction resumes propagation of an existing (in-flight) exception whose unwinding was interrupted with a landingpad instruction.
```

Overview

```
The ‘resume‘ instruction is a terminator instruction that has no successors.
```

Arguments

```

```



##### catchswitch - TODO

Syntax

```
<resultval> = catchswitch within <parent> [ label <handler1>, label <handler2>, ... ] unwind to caller

<resultval> = catchswitch within <parent> [ label <handler1>, label <handler2>, ... ] unwind label <default>
```

C

```c

```

llvm

```

```

Semantic

```
The catchswitch is both a terminator and a “pad” instruction, meaning that it must be both the first non-phi instruction and last instruction in the basic block. Therefore, it must be the only non-phi instruction in the block.
```

Overview

```
The ‘catchswitch‘ instruction is used by LLVM’s exception handling system to describe the set of possible catch handlers that may be executed by the EH personality routine.
```

Arguments

```

```



##### catchret - TODO

Syntax

```
catchret from <token> to label <normal>
```

C

```c

```

llvm

```

```

Semantic

```
The ‘catchret‘ instruction ends an existing (in-flight) exception whose unwinding was interrupted with a catchpad instruction. The personality function gets a chance to execute arbitrary code to, for example, destroy the active exception. Control then transfers to normal.
```

Overview

```
The ‘catchret‘ instruction is a terminator instruction that has a single successor.
```

Arguments

```

```



##### cleanupret - TODO

Syntax

```

```

C

```c

```

llvm

```

```

Semantic

```
The ‘cleanupret‘ instruction indicates to the personality function that one cleanuppad it transferred control to has ended. It transfers control to continue or unwinds out of the function.
```

Overview

```

```

Arguments

```

```



##### unreachable

Syntax

```

```

C

```c

```

llvm

```

```

Semantic

```
The ‘unreachable‘ instruction has no defined semantics.
```

Overview

```
The ‘unreachable‘ instruction has no defined semantics. This instruction is used to inform the optimizer that a particular portion of the code is not reachable. 
```

Arguments

```

```



#### Binary Operations

##### add/sub/mul

```
<result> = add <ty> <op1>, <op2>          ; yields ty:result
<result> = add nuw <ty> <op1>, <op2>      ; yields ty:result
<result> = add nsw <ty> <op1>, <op2>      ; yields ty:result
<result> = add nuw nsw <ty> <op1>, <op2>  ; yields ty:result

<result> = add i32 4, %var          ; yields i32:result = 4 + %var
```

##### fadd/fsub/fmul

```
<result> = fadd [fast-math flags]* <ty> <op1>, <op2>   ; yields ty:result

<result> = fadd float 4.0, %var          ; yields float:result = 4.0 + %var
```

##### udiv/sdiv

```
<result> = udiv <ty> <op1>, <op2>         ; yields ty:result
<result> = udiv exact <ty> <op1>, <op2>   ; yields ty:result

If the exact keyword is present, the result value of the udiv is a poison value if %op1 is not a multiple of %op2 (as such, “((a udiv exact b) mul b) == a”).

```

##### fdiv

```
<result> = fdiv [fast-math flags]* <ty> <op1>, <op2>   ; yields ty:result
```

##### urem/srem/frem

```
<result> = urem <ty> <op1>, <op2>   ; yields ty:result
The ‘urem‘ instruction returns the remainder from the unsigned division of its two arguments.

<result> = frem [fast-math flags]* <ty> <op1>, <op2>   ; yields ty:result
```

##### shl/lshr/ashr

```
<result> = shl <ty> <op1>, <op2>           ; yields ty:result
<result> = shl nuw <ty> <op1>, <op2>       ; yields ty:result
<result> = shl nsw <ty> <op1>, <op2>       ; yields ty:result
<result> = shl nuw nsw <ty> <op1>, <op2>   ; yields ty:result

<result> = lshr <ty> <op1>, <op2>         ; yields ty:result
<result> = lshr exact <ty> <op1>, <op2>   ; yields ty:result
This instruction always performs a logical shift right operation. The most significant bits of the result will be filled with zero bits after the shift. If op2 is (statically or dynamically) equal to or larger than the number of bits in op1, the result is undefined. 
```

add/or/xor

```
<result> = and <ty> <op1>, <op2>   ; yields ty:result
```



#### Vector Operations

##### extractelement

Syntax

```
<result> = extractelement <n x <ty>> <val>, <ty2> <idx>  ; yields <ty>
```

C

```c

```

llvm

```
<result> = extractelement <4 x i32> %vec, i32 0    ; yields i32
```

Semantic

```
The result is a scalar of the same type as the element type of val. Its value is the value at position idx of val. If idx exceeds the length of val, the results are undefined.
```

Overview

```

```

Arguments

```

```



##### insertelement

Syntax

```
<result> = insertelement <n x <ty>> <val>, <ty> <elt>, <ty2> <idx>    ; yields <n x <ty>>
```

C

```c

```

llvm

```
<result> = insertelement <4 x i32> %vec, i32 1, i32 0    ; yields <4 x i32>
```

Semantic

```
The result is a vector of the same type as val. Its element values are those of val except at position idx, where it gets the value elt. If idx exceeds the length of val, the results are undefined.
```

Overview

```

```

Arguments

```

```



##### shufflevector

Syntax

```
<result> = shufflevector <n x <ty>> <v1>, <n x <ty>> <v2>, <m x i32> <mask>    ; yields <m x <ty>>
```

C

```c

```

llvm

```
<result> = shufflevector <4 x i32> %v1, <4 x i32> %v2,
                        <4 x i32> <i32 0, i32 4, i32 1, i32 5>  ; yields <4 x i32>
<result> = shufflevector <4 x i32> %v1, <4 x i32> undef,
                        <4 x i32> <i32 0, i32 1, i32 2, i32 3>  ; yields <4 x i32> - Identity shuffle.
<result> = shufflevector <8 x i32> %v1, <8 x i32> undef,
                        <4 x i32> <i32 0, i32 1, i32 2, i32 3>  ; yields <4 x i32>
<result> = shufflevector <4 x i32> %v1, <4 x i32> %v2,
                        <8 x i32> <i32 0, i32 1, i32 2, i32 3, i32 4, i32 5, i32 6, i32 7 >  ; yields <8 x i32>
```

Semantic

```
The elements of the two input vectors are numbered from left to right across both of the vectors. The shuffle mask operand specifies, for each element of the result vector, which element of the two input vectors the result element gets. The element selector may be undef (meaning “don’t care”) and the second operand may be undef if performing a shuffle from only one vector.
```

Overview

```
The ‘shufflevector‘ instruction constructs a permutation of elements from two input vectors, returning a vector with the same element type as the input and length that is the same as the shuffle mask.
```

Arguments

```

```



#### Aggregate Operations

##### extractvalue

Syntax

```
<result> = extractvalue <aggregate type> <val>, <idx>{, <idx>}*
```

C

```c

```

llvm

```
<result> = extractvalue {i32, float} %agg, 0    ; yields i32
```

Semantic

```
The result is the value at the position in the aggregate specified by the index operands.
```

Overview

```
The ‘extractvalue‘ instruction extracts the value of a member field from an aggregate value.
```

Arguments

```

```



##### insertvalue

Syntax

```
<result> = insertvalue <aggregate type> <val>, <ty> <elt>, <idx>{, <idx>}*    ; yields <aggregate type>
```

C

```c

```

llvm

```
%agg1 = insertvalue {i32, float} undef, i32 1, 0              ; yields {i32 1, float undef}
%agg2 = insertvalue {i32, float} %agg1, float %val, 1         ; yields {i32 1, float %val}
%agg3 = insertvalue {i32, {float}} undef, float %val, 1, 0    ; yields {i32 undef, {float %val}}
```

Semantic

```
The result is an aggregate of the same type as val. Its value is that of val except that the value at the position specified by the indices is that of elt.
```

Overview

```

```

Arguments

```

```



#### Memory Operations

##### alloca

Syntax

```
<result> = alloca [inalloca] <type> [, <ty> <NumElements>] [, align <alignment>]     ; yields type*:result
```

C

```c
return 0;
```

llvm

```
%retval = alloca i32, align 4
store i32 0, i32* %retval, align 4
```

Semantic

```
Memory is allocated; a pointer is returned.
```

Overview:

```
The alloca instruction allocates memory on the stack frame of the currently executing function, to be automatically released when this function returns to its caller. The object is always allocated in the generic address space (address space zero).
```

Arguments:

```
If “NumElements” is specified, it is the number of elements allocated, otherwise “NumElements” is defaulted to be one. If a constant alignment is specified, the value result of the allocation is guaranteed to be aligned to at least that boundary. 
```



##### load

Syntax

```
<result> = load [volatile] <ty>, <ty>* <pointer>[, align <alignment>][, !nontemporal !<index>][, !invariant.load !<index>][, !invariant.group !<index>][, !nonnull !<index>][, !dereferenceable !<deref_bytes_node>][, !dereferenceable_or_null !<deref_bytes_node>][, !align !<align_node>]

<result> = load atomic [volatile] <ty>, <ty>* <pointer> [singlethread] <ordering>, align <alignment> [, !invariant.group !<index>]

!<index> = !{ i32 1 }
!<deref_bytes_node> = !{i64 <dereferenceable_bytes>}
!<align_node> = !{ i64 <value_alignment> }
```

C

```c
i = i * i;
```

llvm

```
  %0 = load i32, i32* %i, align 4
  %1 = load i32, i32* %i, align 4
  %mul = mul nsw i32 %0, %1
  store i32 %mul, i32* %i, align 4
```

Semantic

```
The location of memory pointed to is loaded. 
```

Overview

```
The ‘load‘ instruction is used to read from memory.
```

Arguments

```
The argument to the load instruction specifies the memory address from which to load. The type specified must be a first class type of known size (i.e. not containing an opaque structural type). 
```



##### store

syntax

```
store [volatile] <ty> <value>, <ty>* <pointer>[, align <alignment>][, !nontemporal !<index>][, !invariant.group !<index>]        ; yields void

store atomic [volatile] <ty> <value>, <ty>* <pointer> [singlethread] <ordering>, align <alignment> [, !invariant.group !<index>] ; yields void
```

C:

```c
return 0;

i = i * i;
```

llvm:

```
%retval = alloca i32, align 4
store i32 0, i32* %retval, align 4

%0 = load i32, i32* %i, align 4
%1 = load i32, i32* %i, align 4
%mul = mul nsw i32 %0, %1
store i32 %mul, i32* %i, align 4
```

Semantic

```
The contents of memory are updated to contain <value> at the location specified by the <pointer> operand.
```

Overview

```
The ‘store‘ instruction is used to write to memory.
```

Arguments:

```
There are two arguments to the store instruction: a value to store and an address at which to store it. The type of the <pointer> operand must be a pointer to the first class type of the <value> operand.
```



##### fence

Syntax

```

```

C

```c

```

llvm

```

```

Semantic

```

```

Overview

```

```

Arguments

```

```



##### cmpxchg

Syntax

```

```

C

```c

```

llvm

```

```

Semantic

```

```

Overview

```

```

Arguments

```

```



##### atomicrmw

Syntax

```

```

C

```c

```

llvm

```

```

Semantic

```

```

Overview

```

```

Arguments

```

```



##### getelementptr

Syntax

```

```

C

```c

```

llvm

```

```

Semantic

```

```

Overview

```

```

Arguments

```

```



#### Conversion Operations

##### trunt

Syntax

```

```

C

```c

```

llvm

```

```

Semantic

```

```

Overview

```

```

Arguments

```

```



##### zext

Syntax

```

```

C

```c

```

llvm

```

```

Semantic

```

```

Overview

```

```

Arguments

```

```



##### sext

Syntax

```

```

C

```c

```

llvm

```

```

Semantic

```

```

Overview

```

```

Arguments

```

```



##### bitcast

Syntax

```

```

C

```c

```

llvm

```

```

Semantic

```

```

Overview

```

```

Arguments

```

```



#### Other Operations

##### Icmp

Syntax

```
<result> = icmp <cond> <ty> <op1>, <op2>   ; yields i1 or <N x i1>:result
```

C

```c
if (global > 5)
```

llvm

```
%0 = load i32, i32* %global 
%cmp = icmp sgt i32 %0, 5
```

Semantic

```
The ‘icmp‘ compares op1 and op2 according to the condition code given as cond. 

If the operands are pointer typed, the pointer values are compared as if they were integers.

If the operands are integer vectors, then they are compared element by element. The result is an i1 vector with the same number of elements as the values being compared. Otherwise, the result is an i1.
```

Overview

```
The ‘icmp‘ instruction returns a boolean value or a vector of boolean values based on comparison of its two integer, integer vector, pointer, or pointer vector operands.
```

Arguments

```
cond
eq: equal
ne: not equal
ugt: unsigned greater than
uge: unsigned greater or equal
ult: unsigned less than
ule: unsigned less or equal
sgt: signed greater than
sge: signed greater or equal
slt: signed less than
sle: signed less or equal
```





##### fcmp

Syntax

```

```

C

```c

```

llvm

```

```

Semantic

```

```

Overview

```

```

Arguments

```

```



##### phi

Syntax

```

```

C

```c

```

llvm

```

```

Semantic

```

```

Overview

```

```

Arguments

```

```



##### select

Syntax

```

```

C

```c

```

llvm

```

```

Semantic

```

```

Overview

```

```

Arguments

```

```



##### call

Syntax

```
<result> = [tail | musttail | notail ] call [fast-math flags] [cconv] [ret attrs] <ty>|<fnty> <fnptrval>(<function args>) [fn attrs] [ operand bundles ]
```

C

```c
i = getNextInt(5);
```

llvm

```
%call = call i32 @getNextInt(i32 5)
```

Semantic

```
The ‘call‘ instruction is used to cause control flow to transfer to a specified function, with its incoming arguments bound to the specified values. Upon a ‘ret‘ instruction in the called function, control flow continues with the instruction after the function call, and the return value of the function is bound to the result argument.
```

Overview

```
The ‘call‘ instruction represents a simple function call.
```

Arguments

```
‘ty‘: the type of the call instruction itself which is also the type of the return value. Functions that return no value are marked void.

‘fnty‘: shall be the signature of the function being called. The argument types must match the types implied by this signature. This type can be omitted if the function is not varargs.

‘fnptrval‘: An LLVM value containing a pointer to a function to be called. In most cases, this is a direct function call, but indirect call‘s are just as possible, calling an arbitrary pointer to function value.

‘function args‘: argument list whose types match the function signature argument types and parameter attributes. All arguments must be of first class type. If the function signature indicates the function accepts a variable number of arguments, the extra arguments can be specified.
```



##### va_arg

Syntax

```

```

C

```c

```

llvm

```

```

Semantic

```

```

Overview

```

```

Arguments

```

```



##### landingpad

Syntax

```

```

C

```c

```

llvm

```

```

Semantic

```

```

Overview

```

```

Arguments

```

```





##### Instruction-Name

Syntax

```

```

C

```c

```

llvm

```

```

Semantic

```

```

Overview

```

```

Arguments

```

```


