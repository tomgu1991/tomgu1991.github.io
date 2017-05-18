# Introduction to LLVM

>This is a note on LLVM.
>
>Currently, we are doing some research based on program analysis. LLVM as one of them most powerful C compiler, it provides a promising intermediate representation named LLVM IR. This is my first time to work with it, and I will markdown some basic steps to start up.

[TOC]



### What is LLVM

The [LLVM](http://llvm.org/) Project is a collection of modular and reusable compiler and toolchain technologies. 

The name "LLVM" itself is not an acronym; it is the full name of the project.

In short, it is a project.

The primary sub-projects of LLVM are:

1. The LLVM Core libraries provide a modern source- and target-independent [optimizer](http://llvm.org/docs/Passes.html), along with [code generation support](http://llvm.org/docs/CodeGenerator.html) for many popular CPUs (as well as some less common ones!) These libraries are built around a [well specified](http://llvm.org/docs/LangRef.html) code representation known as the LLVM intermediate representation ("LLVM IR”).
2. **Clang** is an "LLVM native" C/C++/Objective-C compiler, which aims to deliver amazingly fast compiles (e.g. about [3x faster than GCC](http://clang.llvm.org/features.html#performance) when compiling Objective-C code in a debug configuration), extremely useful [error and warning messages](http://clang.llvm.org/diagnostics.html) and to provide a platform for building great source level tools.

So, LLVM is a project.

Clang is part of it as a compiler just as gcc. 

LLVM sub-projects communicate with a code representation known as LLVM IR.

![LLVM Structure](./LLVMStructure.png)

### Installation

Basically, we install llvm-core and clang is enough for current practice.

We focus on version 3.9.0 and build from the source code on ubuntu 16.04.

1. down load source code from here: [http://releases.llvm.org/download.html#3.9.0](http://releases.llvm.org/download.html#3.9.0) , 

LLVM source code

Clang source code

2. extract LLVM source code into path/llvm
3. extract Clang to **path/llvm/tools**
4. **move to the path, mkdir build, so there should be a folder as path/build**

   ```
    - build
    - llvm
    	- tools/clang***
   ```

5. open **build** folder

   ```shell
   cd build
   ```

6. then we will build source code by cmake(how to install cmake see:[https://cmake.org/install/](https://cmake.org/install/)

 ```shell
 // generate makefile, currently we are in folder named build
 cmake ../llvm
 // build by 2 cores
 make -j2
 // install
 make install
 ```

7. delete the build and source folder -> it will be huge
8. check it by : clang -v

### Use LLVM

1. compiler a file by clang

   ```shell
   clang test.c
   ```

2. using -emit-llvm can generate an llvm file with -S or -c

   ```
   clang test.c -S -emit-llvm3.
   ```

```c
Source: test.c
int main(int argc, char const *argv[])
{
     int x = 2;
     return 0;
}
```
```
IR:test.ll
; ModuleID = 't.c'
source_filename = "t.c"
target datalayout = "e-m:e-i64:64-f80:128-n8:16:32:64-S128"
target triple = "x86_64-unknown-linux-gnu"

; Function Attrs: nounwind uwtable
define i32 @main(i32 %argc, i8** %argv) #0 {
entry:
  %retval = alloca i32, align 4
  %argc.addr = alloca i32, align 4
  %argv.addr = alloca i8**, align 8
  %x = alloca i32, align 4
  store i32 0, i32* %retval, align 4
  store i32 %argc, i32* %argc.addr, align 4
  store i8** %argv, i8*** %argv.addr, align 8
  store i32 2, i32* %x, align 4
  ret i32 0
}

attributes #0 = { nounwind uwtable "disable-tail-calls"="false" "less-precise-fpmad"="false" "no-frame-pointer-elim"="true" "no-frame-pointer-elim-non-leaf" "no-infs-fp-math"="false" "no-jump-tables"="false" "no-nans-fp-math"="false" "no-signed-zeros-fp-math"="false" "stack-protector-buffer-size"="8" "target-cpu"="x86-64" "target-features"="+fxsr,+mmx,+sse,+sse2,+x87" "unsafe-fp-math"="false" "use-soft-float"="false" }

!llvm.ident = !{!0}

!0 = !{!"clang version 3.9.0 (tags/RELEASE_390/final)"}
```

The IR reference: [http://llvm.org/docs/LangRef.html](http://llvm.org/docs/LangRef.html)

Further references:

[http://llvm.org/pubs/2008-10-04-ACAT-LLVM-Intro.pdf](http://llvm.org/pubs/2008-10-04-ACAT-LLVM-Intro.pdf)

[http://www.aosabook.org/en/llvm.html](http://www.aosabook.org/en/llvm.html)