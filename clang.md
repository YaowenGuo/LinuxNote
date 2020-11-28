这应该让clang用Intel语法发出汇编代码：

 clang++ -S -mllvm --x86-asm-syntax=intel test.cpp 
您可以使用-mllvm <arg>从clang命令行传入llvm选项。 可悲的是，这个选项似乎没有很好的logging，因此我只能通过浏览llvm邮件列表来find它。

如下面的@thakis所述 ，Clang（3.5+）的新版本不再需要它，因为它现在支持-masm=intel语法。

从r208683 （clang3.5+）开始，它理解-masm=intel 。 所以如果你的 clang 是新的，你可以使用它。

假设您可以让 Clang 发出正常的LLVM字节代码，然后可以使用llc编译为汇编语言，并使用其--x86-asm-syntax=intel选项以英特尔语法获得结果。