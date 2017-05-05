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

%ptr = alloca i32, i32 4, align 1024
```

Semantic

```
Memory is allocated; a pointer is returned.

 ‘alloca‘d memory is automatically released when the function returns. 
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
fence [singlethread] <ordering>                   ; yields void
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
The ‘fence‘ instruction is used to introduce happens-before edges between operations.
```

Arguments

```

```



##### cmpxchg

Syntax

```
cmpxchg [weak] [volatile] <ty>* <pointer>, <ty> <cmp>, <ty> <new> [singlethread] <success ordering> <failure ordering> ; yields  { ty, i1 }
```

C

```c

```

llvm

```
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
```

Semantic

```
The contents of memory at the location specified by the ‘<pointer>‘ operand is read
compared to ‘<cmp>‘
if the read value is the equal, the ‘<new>‘ is written
The original value at the location is returned, together with a flag indicating success (true) or failure (false).
```

Overview

```
The ‘cmpxchg‘ instruction is used to atomically modify memory. It loads a value in memory and compares it to a given value. If they are equal, it tries to store a new value into the memory.
```

Arguments

```

```



##### atomicrmw

Syntax

```
atomicrmw [volatile] <operation> <ty>* <pointer>, <ty> <value> [singlethread] <ordering>          ; yields ty
```

C

```c

```

llvm

```
%old = atomicrmw add i32* %ptr, i32 1 acquire                        ; yields i32
```

Semantic

```
The contents of memory at the location specified by the ‘<pointer>‘ operand are atomically read, modified, and written back. The original value at the location is returned. The modification is specified by the operation argument:
xchg: *ptr = val
add: *ptr = *ptr + val
sub: *ptr = *ptr - val
and: *ptr = *ptr & val
nand: *ptr = ~(*ptr & val)
or: *ptr = *ptr | val
xor: *ptr = *ptr ^ val
max: *ptr = *ptr > val ? *ptr : val (using a signed comparison)
min: *ptr = *ptr < val ? *ptr : val (using a signed comparison)
umax: *ptr = *ptr > val ? *ptr : val (using an unsigned comparison)
umin: *ptr = *ptr < val ? *ptr : val (using an unsigned comparison)
```

Overview

```
The ‘atomicrmw‘ instruction is used to atomically modify memory.
```

Arguments

```
The type of ‘<value>’ must be an integer type whose bit width is a power of two greater than or equal to eight and less than or equal to a target-specific size limit.
```



##### getelementptr

Syntax

```
<result> = getelementptr <ty>, <ty>* <ptrval>{, <ty> <idx>}*
<result> = getelementptr inbounds <ty>, <ty>* <ptrval>{, <ty> <idx>}*
<result> = getelementptr <ty>, <ptr vector> <ptrval>, <vector index type> <idx>
```

C

```c
struct RT {
  char A;
  int B[10][20];
  char C;
};
struct ST {
  int X;
  double Y;
  struct RT Z;
};

int *foo(struct ST *s) {
  return &s[1].Z.B[5][13];
}
```

llvm

```
%struct.RT = type { i8, [10 x [20 x i32]], i8 }
%struct.ST = type { i32, double, %struct.RT }

define i32* @foo(%struct.ST* %s) nounwind uwtable readnone optsize ssp {
entry:
  %arrayidx = getelementptr inbounds %struct.ST, %struct.ST* %s, i64 1, i32 2, i32 1, i64 5, i64 13
  ret i32* %arrayidx
}

define i32* @foo(%struct.ST* %s) {
  %t1 = getelementptr %struct.ST, %struct.ST* %s, i32 1                        ; yields %struct.ST*:%t1
  %t2 = getelementptr %struct.ST, %struct.ST* %t1, i32 0, i32 2                ; yields %struct.RT*:%t2
  %t3 = getelementptr %struct.RT, %struct.RT* %t2, i32 0, i32 1                ; yields [10 x [20 x i32]]*:%t3
  %t4 = getelementptr [10 x [20 x i32]], [10 x [20 x i32]]* %t3, i32 0, i32 5  ; yields [20 x i32]*:%t4
  %t5 = getelementptr [20 x i32], [20 x i32]* %t4, i32 0, i32 13               ; yields i32*:%t5
  ret i32* %t5
}
```

Semantic

```

```

Overview

```
The ‘getelementptr‘ instruction is used to get the address of a subelement of an aggregate data structure. It performs address calculation only and does not access memory. The instruction can also be used to calculate a vector of such addresses.
```

Arguments

```
The first index always indexes the pointer value given as the first argument, the second index indexes a value of the type pointed to (not necessarily the value directly pointed to, since the first index can be non-zero), etc. 
The first type indexed into must be a pointer value, subsequent types can be arrays, vectors, and structs. Note that subsequent types being indexed into can never be pointers, since that would require loading the pointer before continuing calculation.
```



#### Conversion Operations

##### trunt

Syntax

```
<result> = trunc <ty> <value> to <ty2>             ; yields ty2
```

C

```c

```

llvm

```
%X = trunc i32 257 to i8                        ; yields i8:1
%Y = trunc i32 123 to i1                        ; yields i1:true
%Z = trunc i32 122 to i1                        ; yields i1:false
%W = trunc <2 x i16> <i16 8, i16 7> to <2 x i8> ; yields <i8 8, i8 7>
```

Semantic

```
The ‘trunc‘ instruction truncates the high order bits in value and converts the remaining bits to ty2. Since the source size must be larger than the destination size, trunc cannot be a no-op cast. It will always truncate bits.
```

Overview

```

```

Arguments

```
The ‘trunc‘ instruction takes a value to trunc, and a type to trunc it to. Both types must be of integer types, or vectors of the same number of integers. The bit size of the value must be larger than the bit size of the destination type, ty2. Equal sized types are not allowed
```



##### zext

Syntax

```
<result> = zext <ty> <value> to <ty2>             ; yields ty2
```

C

```c

```

llvm

```
%X = zext i32 257 to i64              ; yields i64:257
%Y = zext i1 true to i32              ; yields i32:1
%Z = zext <2 x i16> <i16 8, i16 7> to <2 x i32> ; yields <i32 8, i32 7>
```

Semantic

```
The zext fills the high order bits of the value with zero bits until it reaches the size of the destination type, ty2.
When zero extending from i1, the result will always be either 0 or 1.
```

Overview

```
The ‘zext‘ instruction zero extends its operand to type ty2.
```

Arguments

```

```



##### sext

Syntax

```
<result> = sext <ty> <value> to <ty2>             ; yields ty2
```

C

```c

```

llvm

```
%X = sext i8  -1 to i16              ; yields i16   :65535
%Y = sext i1 true to i32             ; yields i32:-1
%Z = sext <2 x i16> <i16 8, i16 7> to <2 x i32> ; yields <i32 8, i32 7>
```

Semantic

```
The ‘sext‘ instruction performs a sign extension by copying the sign bit (highest order bit) of the value until it reaches the bit size of the type ty2.
When sign extending from i1, the extension always results in -1 or 0.
```

Overview

```

```

Arguments

```

```



##### fptrunc

```
<result> = fptrunc <ty> <value> to <ty2>             ; yields ty2

%X = fptrunc double 123.0 to float         ; yields float:123.0
%Y = fptrunc double 1.0E+300 to float      ; yields undefined

The ‘fptrunc‘ instruction casts a value from a larger floating point type to a smaller floating point type. If the value cannot fit (i.e. overflows) within the destination type, ty2, then the results are undefined. If the cast produces an inexact result, how rounding is performed (e.g. truncation, also known as round to zero) is undefined.
```



##### fpext

```
<result> = fpext <ty> <value> to <ty2>             ; yields ty2

%X = fpext float 3.125 to double         ; yields double:3.125000e+00
%Y = fpext double %X to fp128            ; yields fp128:0xL00000000000000004000900000000000

The ‘fpext‘ instruction extends the value from a smaller floating point type to a larger floating point type. The fpext cannot be used to make a no-op cast because it always changes bits. Use bitcast to make a no-op cast for a floating point cast.
```



##### fptoui

```
<result> = fptoui <ty> <value> to <ty2>             ; yields ty2

%X = fptoui double 123.0 to i32      ; yields i32:123
%Y = fptoui float 1.0E+300 to i1     ; yields undefined:1
%Z = fptoui float 1.04E+17 to i8     ; yields undefined:1

The ‘fptoui‘ instruction converts its floating point operand into the nearest (rounding towards zero) unsigned integer value. If the value cannot fit in ty2, the results are undefined.
```



##### fptosi

```
<result> = fptosi <ty> <value> to <ty2>             ; yields ty2

%X = fptosi double -123.0 to i32      ; yields i32:-123
%Y = fptosi float 1.0E-247 to i1      ; yields undefined:1
%Z = fptosi float 1.04E+17 to i8      ; yields undefined:1

The ‘fptosi‘ instruction converts its floating point operand into the nearest (rounding towards zero) signed integer value. If the value cannot fit in ty2, the results are undefined.
```



##### uitofp/sitofp/ptrtoint/inttoptr

```
<result> = uitofp <ty> <value> to <ty2>             ; yields ty2
%X = uitofp i32 257 to float         ; yields float:257.0
%Y = uitofp i8 -1 to double          ; yields double:255.0

The ‘uitofp‘ instruction interprets its operand as an unsigned integer quantity and converts it to the corresponding floating point value. If the value cannot fit in the floating point value, the results are undefined.


<result> = sitofp <ty> <value> to <ty2>             ; yields ty2
%X = sitofp i32 257 to float         ; yields float:257.0
%Y = sitofp i8 -1 to double          ; yields double:-1.0

The ‘sitofp‘ instruction interprets its operand as a signed integer quantity and converts it to the corresponding floating point value. If the value cannot fit in the floating point value, the results are undefined.


<result> = ptrtoint <ty> <value> to <ty2>             ; yields ty2
%X = ptrtoint i32* %P to i8                         ; yields truncation on 32-bit architecture
%Y = ptrtoint i32* %P to i64                        ; yields zero extension on 32-bit architecture
%Z = ptrtoint <4 x i32*> %P to <4 x i64>; yields vector zero extension for a vector of addresses on 32-bit architecture

The ‘ptrtoint‘ instruction converts value to integer type ty2 by interpreting the pointer value as an integer and either truncating or zero extending that value to the size of the integer type. If value is smaller than ty2 then a zero extension is done. If value is larger than ty2 then a truncation is done. If they are the same size, then nothing is done (no-op cast) other than a type change.

<result> = inttoptr <ty> <value> to <ty2>             ; yields ty2
%X = inttoptr i32 255 to i32*          ; yields zero extension on 64-bit architecture
%Y = inttoptr i32 255 to i32*          ; yields no-op on 32-bit architecture
%Z = inttoptr i64 0 to i32*            ; yields truncation on 32-bit architecture
%Z = inttoptr <4 x i32> %G to <4 x i8*>; yields truncation of vector G to four pointers

The ‘inttoptr‘ instruction converts value to type ty2 by applying either a zero extension or a truncation depending on the size of the integer value. If value is larger than the size of a pointer then a truncation is done. If value is smaller than the size of a pointer then a zero extension is done. If they are the same size, nothing is done (no-op cast).
```



##### bitcast

Syntax

```
<result> = bitcast <ty> <value> to <ty2>             ; yields ty2
```

C

```c

```

llvm

```
%X = bitcast i8 255 to i8              ; yields i8 :-1
%Y = bitcast i32* %x to sint*          ; yields sint*:%x
%Z = bitcast <2 x int> %V to i64;        ; yields i64: %V
%Z = bitcast <2 x i32*> %V to <2 x i64*> ; yields <2 x i64*>
```

Semantic

```
The ‘bitcast‘ instruction converts value to type ty2. It is always a no-op cast because no bits change with this conversion. The conversion is done as if the value had been stored to memory and read back as type ty2. Pointer (or vector of pointers) types may only be converted to other pointer (or vector of pointers) types with the same address space through this instruction. To convert pointers to other types, use the inttoptr or ptrtoint instructions first.
```

Overview

```

```

Arguments

```

```



##### addrspacecast - TODO

```
<result> = addrspacecast <pty> <ptrval> to <pty2>       ; yields pty2

%X = addrspacecast i32* %x to i32 addrspace(1)*    ; yields i32 addrspace(1)*:%x
%Y = addrspacecast i32 addrspace(1)* %y to i64 addrspace(2)*    ; yields i64 addrspace(2)*:%y
%Z = addrspacecast <4 x i32*> %z to <4 x float addrspace(3)*>   ; yields <4 x float addrspace(3)*>:%z

The ‘addrspacecast‘ instruction converts the pointer value ptrval to type pty2. It can be a no-op cast or a complex value modification, depending on the target and the address space pair. Pointer conversions within the same address space must be performed with the bitcast instruction. Note that if the address space conversion is legal then both result and operand refer to the same memory location.
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
<result> = fcmp [fast-math flags]* <cond> <ty> <op1>, <op2>     ; yields i1 or <N x i1>:result
```

C

```c

```

llvm

```
<result> = fcmp oeq float 4.0, 5.0    ; yields: result=false
<result> = fcmp one float 4.0, 5.0    ; yields: result=true
<result> = fcmp olt float 4.0, 5.0    ; yields: result=true
<result> = fcmp ueq double 1.0, 2.0   ; yields: result=false
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
<result> = phi <ty> [ <val0>, <label0>], ...
```

C

```c

```

llvm

```
Loop:       ; Infinite loop that counts from 0 on up...
  %indvar = phi i32 [ 0, %LoopHeader ], [ %nextindvar, %Loop ]
  %nextindvar = add i32 %indvar, 1
  br label %Loop
```

Semantic

```
At runtime, the ‘phi‘ instruction logically takes on the value specified by the pair corresponding to the predecessor basic block that executed just prior to the current block.
```

Overview

```
The ‘phi‘ instruction is used to implement the φ node in the SSA graph representing the function.
```

Arguments

```

```



##### select

Syntax

```
<result> = select selty <cond>, <ty> <val1>, <ty> <val2>             ; yields ty 
selty is either i1 or {<N x i1>}
```

C

```c

```

llvm

```
%X = select i1 true, i8 17, i8 42          ; yields i8:17
```

Semantic

```
If the condition is an i1 and it evaluates to 1, the instruction returns the first value argument; otherwise, it returns the second value argument.
If the condition is a vector of i1, then the value arguments must be vectors of the same size, and the selection is done element by element.
If the condition is an i1 and the value arguments are vectors of the same size, then an entire vector is selected.
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



##### va_arg - TODO

Syntax

```
<resultval> = va_arg <va_list*> <arglist>, <argty>
```

C

```c

```

llvm

```

```

Semantic

```
The ‘va_arg‘ instruction loads an argument of the specified type from the specified va_list and causes the va_list to point to the next argument. For more information, see the variable argument handling Intrinsic Functions.
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


