这应该让clang用Intel语法发出汇编代码：

 clang++ -S -mllvm --x86-asm-syntax=intel test.cpp 
您可以使用-mllvm <arg> 从clang命令行传入llvm选项。 可悲的是，这个选项似乎没有很好的logging，因此我只能通过浏览llvm邮件列表来find它。

如下面的@thakis所述 ，Clang（3.5+）的新版本不再需要它，因为它现在支持-masm=intel语法。

从r208683 （clang3.5+）开始，它理解-masm=intel 。 所以如果你的 clang 是新的，你可以使用它。

假设您可以让 Clang 发出正常的LLVM字节代码，然后可以使用llc编译为汇编语言，并使用其--x86-asm-syntax=intel选项以英特尔语法获得结果。

## C/C++ 编译流程

> 查看编译的步骤

```
# clang -ccc-print-phases hello.cpp

0: input, "hello.cpp", c++
1: preprocessor, {0}, c++-cpp-output
2: compiler, {1}, ir
3: backend, {2}, assembler
4: assembler, {3}, object
5: linker, {4}, image
6: bind-arch, "arm64", {5}, image
```

```
-> 1 预编译 --> 2. 编译 --> 3，4. 汇编 --> 5. 链接 --> 6. 封装为制定架构的可执行文件。
```

已下面的代码为例，因为 `iostream` 的内容太多，这里为了演示，不使用系统的函数，否则导入会导致代码太多而影响核心查看。

```c
// hello.h
#ifndef HELLO_CPP
#define HELLO_CPP

#define PI 3.1415926

#endif

// hello.cpp
#include "hello.h"

using namespace std;

int main() {
    int radius = 4;
    int air = PI * radius * radius;
}
```

分步执行

1. 预处理: 宏的替换，头文件的导入。

```c
clang -E hello.cpp
# 1 "hello.cpp"
# 1 "<built-in>" 1
# 1 "<built-in>" 3
# 395 "<built-in>" 3
# 1 "<command line>" 1
# 1 "<built-in>" 2
# 1 "hello.cpp" 2
# 1 "./hello.h" 1
# 2 "hello.cpp" 2

using namespace std;

int main() {
    int radius = 4;
    int air = 3.1415926 * radius * radius;
}
```

可以看到宏定义 `PI` 已经被替换成 `3.1415926`。


2. 编译: 将源代码编译成 LLVM 中间代码。

clang 是使用LLVM最为后端，所以支持编译成LLVM的字节码。gcc 不可以生成 LLVM 字节码，mac os 上 gcc 命令本质是 clang，也可以生成。


```
clang -emit-llvm -o hello.bc -c hello.cpp

# 或者
clang -O3 -emit-llvm hello.c -c -o hello.bc
```
将生成为 llvm 的中间机器码，之所以设计中间机器码一层，而不是直接编译为汇编代码，是为了隔离各种机器平台，方便优化语义分析、语法分析以及进行各种优化。而不必影响到各个平台相关代码。同时，对不同机器平台的编译进直接编译字节码和相应平台汇编程序，更加容易扩展。

`llvm-dis` 命令是LLVM反汇编。它可以一个LLVM bitcode文件并将其转换为人类可读的LLVM汇编语言。

反编译LLVM 字节码：

```
llvm-dis < hello.bc | less

llvm-dis hello.bc -o -
```

3. 汇编: 

```
clang++ -S -mllvm --x86-asm-syntax=intel hello.bc
# 或者
clang++ -S -masm=intel  hello.bc

# -S -masm=intel 参数是为了生成 intel 格式的汇编指令。如果不用指定，可以直接使用。

clang++ -S -masm=intel  hello.bc

```

会输出一个汇编文件 hello.s

```asm
	.section	__TEXT,__text,regular,pure_instructions
	.build_version macos, 11, 0	sdk_version 11, 1
	.globl	_main                   ; -- Begin function main
	.p2align	2
_main:                                  ; @main
	.cfi_startproc
; %bb.0:
	sub	sp, sp, #16             ; =16
	.cfi_def_cfa_offset 16
	mov	w8, #4
	str	w8, [sp, #12]
	ldr	w8, [sp, #12]
	scvtf	d0, w8
	mov	x9, #55370
	movk	x9, #19730, lsl #16
	movk	x9, #8699, lsl #32
	movk	x9, #16393, lsl #48
	fmov	d1, x9
	fmul	d0, d1, d0
	ldr	w8, [sp, #12]
	scvtf	d1, w8
	fmul	d0, d0, d1
	fcvtzs	w8, d0
	str	w8, [sp, #8]
	mov	w8, #0
	mov	x0, x8
	add	sp, sp, #16             ; =16
	ret
	.cfi_endproc
                                        ; -- End function
.subsections_via_symbols
```


4. 汇编: 将LLVM 中间代码编译为汇编






> 查看编译结果

clang -rewrite-objc hello.cpp


> 查看操作内部命令，可以使用 -### 命令

clang -### hello.cpp -o main

> 想看清clang的全部过程，可以先通过-E查看clang在预编译处理这步做了什么。

clang -E hello.cpp



