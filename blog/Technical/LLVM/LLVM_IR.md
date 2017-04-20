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



#### Instruction

###### alloca

Syntax

```
<result> = alloca [inalloca] <type> [, <ty> <NumElements>] [, align <alignment>]     ; yields type*:result
```

C

```c
return 0;
```
llvm
	%retval = alloca i32, align 4
	store i32 0, i32* %retval, align 4
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



###### store

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



###### load

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



###### mul

Syntax

```
<result> = mul <ty> <op1>, <op2>          ; yields ty:result
<result> = mul nuw <ty> <op1>, <op2>      ; yields ty:result
<result> = mul nsw <ty> <op1>, <op2>      ; yields ty:result
<result> = mul nuw nsw <ty> <op1>, <op2>  ; yields ty:result
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
The value produced is the integer product of the two operands.
nuw and nsw stand for “No Unsigned Wrap” and “No Signed Wrap”, respectively. If the nuw and/or nsw keywords are present, the result value of the mul is a poison value if unsigned and/or signed overflow, respectively, occurs.
```

Overview

```
The ‘mul‘ instruction returns the product of its two operands.
```

Arguments

```
The two arguments to the ‘mul‘ instruction must be integer or vector of integer values. Both arguments must have identical types.
```



###### ret

Syntax

```
ret <type> <value>       ; Return a value from a non-void function

ret void                 ; Return from void function
```

C

```c

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



###### call

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





###### Instruction-Name

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



