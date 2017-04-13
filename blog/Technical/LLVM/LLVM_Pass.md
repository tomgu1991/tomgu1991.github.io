# Introduction to LLVM Pass

> 有一些工作要用LLVM IR，师兄们推荐LLVM Pass入手有一个感觉。所以这是一个Pass的配置，入门的介绍。
>
> Reference : 
>
> [Writing an LLVM Pass](http://llvm.org/docs/WritingAnLLVMPass.html)
>
> [LLVM for Grad Students](http://www.cs.cornell.edu/~asampson/blog/llvm.html)
>
> [Video](https://www.youtube.com/watch?v=2lfpMIl98Fw)

### What is a pass?

The LLVM Pass Framework is an important part of the LLVM system, because LLVM passes are where most of the interesting parts of the compiler exist. Passes perform the transformations and optimizations that make up the compiler, they build the analysis results that are used by these transformations, and they are, above all, a structuring technique for compiler code.

Actually, for us, it is much more convenient to write a llvm application. That is a independent cpp where contains a main function , reads in *.bc files and do the same as pass. In such way, we can generate a runnable application directly.

### Steps to build a pass

We use the examples in LLVM source code named as Hello in "llvm-3.9.0.src/lib/Transforms/Hello"

1. Configure and build LLVM. You can do as [this](http://tomgu1991.github.io/blog/Technical/LLVM/LLVM_Introduction)

2. Open your llvm src folder, my is "llvm-3.9.0.src". Create a build folder and generate makefile by CMake

   ```shell
   tomgu@ubuntu:~/Downloads$ cd llvm-3.9.0.src/
   tomgu@ubuntu:~/Downloads/llvm-3.9.0.src$ mkdir build
   tomgu@ubuntu:~/Downloads/llvm-3.9.0.src$ cd build/
   tomgu@ubuntu:~/Downloads/llvm-3.9.0.src/build$ cmake ..
   ```

   when you see these, you finish generate makefile

   ```shell
   -- Configuring done
   -- Generating done
   -- Build files have been written to: /home/tomgu/Downloads/llvm-3.9.0.src/build
   ```

3. build our Hello Pass by its makefile. It will take several minutes. 

   ```shell
   tomgu@ubuntu:~/Downloads/llvm-3.9.0.src/build$ cd lib/Transforms/Hello/
   tomgu@ubuntu:~/Downloads/llvm-3.9.0.src/build/lib/Transforms/Hello$ make
   Scanning dependencies of target LLVMHello_exports
   [  0%] Creating export file for LLVMHello
   [  0%] Built target LLVMHello_exports
   Scanning dependencies of target obj.llvm-tblgen
   [  0%] Building CXX object utils/TableGen/CMakeFiles/obj.llvm-tblgen.dir/AsmMatcherEmitter.cpp.o
   [  0%] Building CXX object utils/TableGen/CMakeFiles/obj.llvm-tblgen.dir/AsmWriterEmitter.cpp.o
   ```

   when you see these, you finish build, and it will generate a LLVMHello.so file in "build/lib/"

   ```shell
   [100%] Linking CXX shared module ../../LLVMHello.so
   [100%] Built target LLVMHello

   tomgu@ubuntu:~/Downloads/llvm-3.9.0.src/build/lib/Transforms/Hello$ cd ../../
   tomgu@ubuntu:~/Downloads/llvm-3.9.0.src/build/lib$ ls -l
   total 38164
   drwxrwxr-x  3 tomgu tomgu     4096 Apr 13 14:47 Analysis
   drwxrwxr-x  3 tomgu tomgu     4096 Apr 13 14:47 AsmParser
   drwxrwxr-x  5 tomgu tomgu     4096 Apr 13 14:47 Bitcode
   drwxrwxr-x  3 tomgu tomgu     4096 Apr 13 14:47 cmake
   drwxrwxr-x  2 tomgu tomgu     4096 Apr 13 14:47 CMakeFiles
   -rw-rw-r--  1 tomgu tomgu     2954 Apr 13 14:47 cmake_install.cmake
   drwxrwxr-x  7 tomgu tomgu     4096 Apr 13 14:47 CodeGen
   drwxrwxr-x  7 tomgu tomgu     4096 Apr 13 14:47 DebugInfo
   drwxrwxr-x  7 tomgu tomgu     4096 Apr 13 14:47 ExecutionEngine
   drwxrwxr-x  3 tomgu tomgu     4096 Apr 13 14:47 Fuzzer
   drwxrwxr-x  3 tomgu tomgu     4096 Apr 13 14:47 IR
   drwxrwxr-x  3 tomgu tomgu     4096 Apr 13 14:47 IRReader
   drwxrwxr-x  3 tomgu tomgu     4096 Apr 13 14:47 LibDriver
   -rw-rw-r--  1 tomgu tomgu 27240850 Apr 13 14:52 libLLVMSupport.a
   -rw-rw-r--  1 tomgu tomgu 11354178 Apr 13 14:53 libLLVMTableGen.a
   drwxrwxr-x  3 tomgu tomgu     4096 Apr 13 14:47 LineEditor
   drwxrwxr-x  3 tomgu tomgu     4096 Apr 13 14:47 Linker
   -rwxrwxr-x  1 tomgu tomgu   358008 Apr 13 14:53 LLVMHello.so
   drwxrwxr-x  3 tomgu tomgu     4096 Apr 13 14:47 LTO
   -rw-rw-r--  1 tomgu tomgu     9643 Apr 13 14:47 Makefile
   drwxrwxr-x  5 tomgu tomgu     4096 Apr 13 14:47 MC
   drwxrwxr-x  3 tomgu tomgu     4096 Apr 13 14:47 Object
   drwxrwxr-x  3 tomgu tomgu     4096 Apr 13 14:47 ObjectYAML
   drwxrwxr-x  3 tomgu tomgu     4096 Apr 13 14:47 Option
   drwxrwxr-x  3 tomgu tomgu     4096 Apr 13 14:47 Passes
   drwxrwxr-x  4 tomgu tomgu     4096 Apr 13 14:47 ProfileData
   drwxrwxr-x  3 tomgu tomgu     4096 Apr 13 14:47 Support
   drwxrwxr-x  3 tomgu tomgu     4096 Apr 13 14:47 TableGen
   drwxrwxr-x 16 tomgu tomgu     4096 Apr 13 14:47 Target
   drwxrwxr-x 11 tomgu tomgu     4096 Apr 13 14:47 Transforms
   tomgu@ubuntu:~/Downloads/llvm-3.9.0.src/build/lib$ 
   ```

4. use hello pass, which will output the functions in a c source file. (1) use clang to build a *.bc file (2) use this LLVMHello.so to analyze this file

   ```c
   hello.c
   tomgu@ubuntu:~/Downloads/llvm-3.9.0.src/build/lib$ gedit hello.c
   tomgu@ubuntu:~/Downloads/llvm-3.9.0.src/build/lib$ cat hello.c 
   int fC(){return 1;}

   void fA(){}
   int fB(){
   	return fC();
   }
   int main(){
   	fA();
   	return fB(); 
   }

   ```

   ```shell
   tomgu@ubuntu:~/Downloads/llvm-3.9.0.src/build/lib$ clang -c -emit-llvm hello.c -o hello.bc
   tomgu@ubuntu:~/Downloads/llvm-3.9.0.src/build/lib$ opt -load ./LLVMHello.so -hello < hello.bc
   WARNING: You're attempting to print out a bitcode file.
   This is inadvisable as it may cause display problems. If
   you REALLY want to taste LLVM bitcode first-hand, you
   can force output with the `-f' option.

   Hello: fC
   Hello: fA
   Hello: fB
   Hello: main
   tomgu@ubuntu:~/Downloads/llvm-3.9.0.src/build/lib$ opt -load ./LLVMHello.so -hello -time-passes  < hello.bc
   WARNING: You're attempting to print out a bitcode file.
   This is inadvisable as it may cause display problems. If
   you REALLY want to taste LLVM bitcode first-hand, you
   can force output with the `-f' option.

   Hello: fC
   Hello: fA
   Hello: fB
   Hello: main
   ===-------------------------------------------------------------------------===
                         ... Pass execution timing report ...
   ===-------------------------------------------------------------------------===
     Total Execution Time: 0.0040 seconds (0.0002 wall clock)

      ---User Time---   --User+System--   ---Wall Time---  --- Name ---
      0.0040 (100.0%)   0.0040 (100.0%)   0.0001 ( 53.9%)  Module Verifier
      0.0000 (  0.0%)   0.0000 (  0.0%)   0.0001 ( 46.1%)  Hello World Pass
      0.0040 (100.0%)   0.0040 (100.0%)   0.0002 (100.0%)  Total

   ===-------------------------------------------------------------------------===
                                   LLVM IR Parsing
   ===-------------------------------------------------------------------------===
     Total Execution Time: 0.0000 seconds (0.0006 wall clock)

      ---Wall Time---  --- Name ---
      0.0006 (100.0%)  Parse IR
      0.0006 (100.0%)  Total

   ```

### Deep in pass

So what this pass did?

```c++
//===- Hello.cpp - Example code from "Writing an LLVM Pass" ---------------===//
//
//                     The LLVM Compiler Infrastructure
//
// This file is distributed under the University of Illinois Open Source
// License. See LICENSE.TXT for details.
//
//===----------------------------------------------------------------------===//
//
// This file implements two versions of the LLVM "Hello World" pass described
// in docs/WritingAnLLVMPass.html
//
//===----------------------------------------------------------------------===//

#include "llvm/ADT/Statistic.h"
#include "llvm/IR/Function.h"
#include "llvm/Pass.h"
#include "llvm/Support/raw_ostream.h"
using namespace llvm;

#define DEBUG_TYPE "hello"

STATISTIC(HelloCounter, "Counts number of functions greeted");

namespace {
  // Hello - The first implementation, without getAnalysisUsage.
  // this is a pass and it is a function pass
  struct Hello : public FunctionPass {
    static char ID; // Pass identification, replacement for typeid
    Hello() : FunctionPass(ID) {}

    bool runOnFunction(Function &F) override { // we check functions print out it's name
      ++HelloCounter;
      errs() << "Hello: ";
      errs().write_escaped(F.getName()) << '\n';
      return false;
    }
  };
}

char Hello::ID = 0;
/* Lastly, we register our class Hello, giving it a command line argument “hello”, and a name “Hello World Pass”. The last two arguments describe its behavior: if a pass walks CFG without modifying it then the third argument is set to true; if a pass is an analysis pass, for example dominator tree pass, then true is supplied as the fourth argument. */
static RegisterPass<Hello> X("hello", "Hello World Pass");

namespace {
  // Hello2 - The second implementation with getAnalysisUsage implemented.
  struct Hello2 : public FunctionPass {
    static char ID; // Pass identification, replacement for typeid
    Hello2() : FunctionPass(ID) {}

    bool runOnFunction(Function &F) override {
      ++HelloCounter;
      errs() << "Hello: ";
      errs().write_escaped(F.getName()) << '\n';
      return false;
    }

    // We don't modify the program, so we preserve all analyses.
    void getAnalysisUsage(AnalysisUsage &AU) const override {
      AU.setPreservesAll();
    }
  };
}

char Hello2::ID = 0;
static RegisterPass<Hello2>
Y("hello2", "Hello World Pass (with getAnalysisUsage implemented)");
```

### Understanding LLVM IR (learn from [Adrian Sampson](http://www.cs.cornell.edu/~asampson/blog/llvm.html))

- A [Module](http://llvm.org/docs/doxygen/html/classllvm_1_1Module.html) represents a source file (roughly) or a *translation unit* (pedantically). Everything else is contained in a Module. 顶级容器
- Most notably, Modules house [Function](http://llvm.org/docs/doxygen/html/classllvm_1_1Function.html)s, which are exactly what they sound like: named chunks of executable code. (In C++, both functions and methods correspond to LLVM Functions.) 就是我们的函数
- Aside from declaring its name and arguments, a Function is mainly a container of [BasicBlock](http://llvm.org/docs/doxygen/html/classllvm_1_1BasicBlock.html)s. The [basic block](https://en.wikipedia.org/wiki/Basic_block) is a familiar concept from compilers, but for our purposes, it’s just a contiguous chunk of instructions. 是函数里面的一个片段，内部的语句不带分支，顺序执行
- An [Instruction](http://www.llvm.org/docs/doxygen/html/classllvm_1_1Instruction.html), in turn, is a single code operation. The level of abstraction is roughly the same as in [RISC](https://en.wikipedia.org/wiki/Reduced_instruction_set_computing)-like machine code: an instruction might be an integer addition, a floating-point divide, or a store to memory, for example. 原子操作

Most things in LLVM—including Function, BasicBlock, and Instruction—are C++ classes that inherit from an omnivorous base class called [Value](http://www.llvm.org/docs/doxygen/html/classllvm_1_1Value.html). A Value is any data that can be used in a computation—a number, for example, or the address of some code. Global variables and constants (a.k.a. literals or immediates, like 5) are also Values.

![Container](LLVM_Pass_1.png)

#### An Instruction

Here’s an example of an Instruction in the human-readable text form of LLVM IR:

```
; 加法操作
%5 = add i32 %4, 2 
```

This instruction adds two 32-bit integer values (indicated by the type `i32`). It adds the number in register 4 (written `%4`) and the literal number 2 (written `2`) and places its result in register 5. This is what I mean when I say LLVM IR looks like idealized RISC machine code: we even use the same terminology, like *register*, but there are infinitely many registers.

That same instruction is represented inside the compiler as an instance of the [Instruction](http://www.llvm.org/docs/doxygen/html/classllvm_1_1Instruction.html) C++ class. The object has an opcode indicating that it’s an addition, a type, and a list of operands that are pointers to other Value objects. In our case, it points to a [Constant](http://www.llvm.org/docs/doxygen/html/classllvm_1_1Constant.html) object representing the number 2 and another [Instruction](http://www.llvm.org/docs/doxygen/html/classllvm_1_1Instruction.html) corresponding to the register %4. (Since LLVM IR is in [static single assignment](https://en.wikipedia.org/wiki/Static_single_assignment_form) form, registers and Instructions are actually one and the same. Register numbers are an artifact of the text representation.)



#### A new pass

Now, let us do a interesting thing: we use '*' operator to replace '+'

```
a = b + c ;
we replace by 
a = b * c ;
```

New pass file - NOTE: **add header files**

```c++
#include "llvm/ADT/Statistic.h"
#include "llvm/IR/Function.h"
#include "llvm/Pass.h"
#include "llvm/Support/raw_ostream.h"
#include "llvm/IR/IRBuilder.h"
#include "llvm/IR/InstrTypes.h"
using namespace llvm;

#define DEBUG_TYPE "hello"

STATISTIC(HelloCounter, "Counts number of functions greeted");

namespace {
  // Hello - The first implementation, without getAnalysisUsage.
  struct Hello : public FunctionPass {
    static char ID; // Pass identification, replacement for typeid
    Hello() : FunctionPass(ID) {}

    bool runOnFunction(Function &F) override {
      ++HelloCounter;
      errs() << "Hello: ";
      errs().write_escaped(F.getName()) << '\n';
      for (auto& B : F) {
        for (auto& I : B) {
          if (auto* op = dyn_cast<BinaryOperator>(&I)) {
            // Insert at the point where the instruction `op` appears.
            IRBuilder<> builder(op);      

            // Make a multiply with the same operands as `op`.
            Value* lhs = op->getOperand(0);
            Value* rhs = op->getOperand(1);
            Value* mul = builder.CreateMul(lhs, rhs);     

            // Everywhere the old instruction was used as an operand, use our
            // new multiply instruction instead.
            for (auto& U : op->uses()) {
              User* user = U.getUser();  // A User is anything with operands.
              user->setOperand(U.getOperandNo(), mul);
            }     

            // We modified the code.
            return true;
          }
        }
      }      
      return false;
    }
  };
}
```

We have to rebuild hello pass

```shell
tomgu@ubuntu:~/Downloads/llvm-3.9.0.src/build/lib/Transforms/Hello$ make
[  0%] Built target LLVMHello_exports
[ 25%] Built target obj.llvm-tblgen
[100%] Built target LLVMSupport
[100%] Built target LLVMTableGen
[100%] Built target llvm-tblgen
[100%] Built target intrinsics_gen
Scanning dependencies of target LLVMHello
[100%] Building CXX object lib/Transforms/Hello/CMakeFiles/LLVMHello.dir/Hello.cpp.o
[100%] Linking CXX shared module ../../LLVMHello.so
[100%] Built target LLVMHello
```

We use the following file to test our pass

```c
#include <stdio.h>

int fC(){return 1;}

void fA(){
	int a = 1;
	int b = 2;
	printf("result in fA() = %d\n", a+b); // should be 3
}
int fB(){
	return fC();
}
int main(){
	fA();
	int x = fB() + 2;
	printf("x in main = %d\n", x); // should be 3
	return 0; 
}
```

Now we compile and output the result

```shell
tomgu@ubuntu:~/Downloads/llvm-3.9.0.src/build/lib$ clang hello.c 
tomgu@ubuntu:~/Downloads/llvm-3.9.0.src/build/lib$ ./a.out 
result in fA() = 3
x in main = 3

The result is right.
```

Now we use the new pass to rewrite the *.bc file and recompile the new *.bc file and check the result

```shell
# 1. first, generate llvm file
tomgu@ubuntu:~/Downloads/llvm-3.9.0.src/build/lib$ clang -c -emit-llvm hello.c -o hello.bc
# 2. second, use our new pass and out put the result in newHello.bc
tomgu@ubuntu:~/Downloads/llvm-3.9.0.src/build/lib$ opt -load ./LLVMHello.so -hello < hello.bc > newHello.bc
Hello: fC
Hello: fA
Hello: fB
Hello: main
NOTE: it will output the function names here, for we have 
      errs() << "Hello: ";
      errs().write_escaped(F.getName()) << '\n';
      
# 3. compile the newHello.bc and run
tomgu@ubuntu:~/Downloads/llvm-3.9.0.src/build/lib$ clang newHello.bc 
tomgu@ubuntu:~/Downloads/llvm-3.9.0.src/build/lib$ ./a.out 
result in fA() = 2
x in main = 2
Wow! Terrific! the result is expected!
```



