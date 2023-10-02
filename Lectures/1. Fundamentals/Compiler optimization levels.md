[Video link](https://www.youtube.com/watch?v=Gwvn8ruzXT8)
### Code Generation Options[](https://clang.llvm.org/docs/CommandGuide/clang.html#code-generation-options "Permalink to this heading")

-O0, -O1, -O2, -O3, -Ofast, -Os, -Oz, -Og, -O, -O4[](https://clang.llvm.org/docs/CommandGuide/clang.html#cmdoption-O0 "Permalink to this definition")

Specify which optimization level to use:

> [`-O0`](https://clang.llvm.org/docs/CommandGuide/clang.html#cmdoption-O0) Means “no optimization”: this level compiles the fastest and generates the most debuggable code.
> 
> [`-O1`](https://clang.llvm.org/docs/CommandGuide/clang.html#cmdoption-O0) Somewhere between [`-O0`](https://clang.llvm.org/docs/CommandGuide/clang.html#cmdoption-O0) and [`-O2`](https://clang.llvm.org/docs/CommandGuide/clang.html#cmdoption-O0).
> 
> [`-O2`](https://clang.llvm.org/docs/CommandGuide/clang.html#cmdoption-O0) Moderate level of optimization which enables most optimizations.
> 
> [`-O3`](https://clang.llvm.org/docs/CommandGuide/clang.html#cmdoption-O0) Like [`-O2`](https://clang.llvm.org/docs/CommandGuide/clang.html#cmdoption-O0), except that it enables optimizations that take longer to perform or that may generate larger code (in an attempt to make the program run faster).
> 	This may not always make the program faster, though it usually does.
> 
> [`-Ofast`](https://clang.llvm.org/docs/CommandGuide/clang.html#cmdoption-O0) Enables all the optimizations from [`-O3`](https://clang.llvm.org/docs/CommandGuide/clang.html#cmdoption-O0) along with other aggressive optimizations that may violate strict compliance with language standards.
> 	This may invalidate the correctness of a program if incredibly precise computations are relevant to have correct
> 
> [`-Os`](https://clang.llvm.org/docs/CommandGuide/clang.html#cmdoption-O0) Like [`-O2`](https://clang.llvm.org/docs/CommandGuide/clang.html#cmdoption-O0) with extra optimizations to reduce code size.
> 
> [`-Oz`](https://clang.llvm.org/docs/CommandGuide/clang.html#cmdoption-O0) Like [`-Os`](https://clang.llvm.org/docs/CommandGuide/clang.html#cmdoption-O0) (and thus [`-O2`](https://clang.llvm.org/docs/CommandGuide/clang.html#cmdoption-O0)), but reduces code size further.
> 
> [`-Og`](https://clang.llvm.org/docs/CommandGuide/clang.html#cmdoption-O0) Like [`-O1`](https://clang.llvm.org/docs/CommandGuide/clang.html#cmdoption-O0). In future versions, this option might disable different optimizations in order to improve debuggability.
> 
> [`-O`](https://clang.llvm.org/docs/CommandGuide/clang.html#cmdoption-O0) Equivalent to [`-O1`](https://clang.llvm.org/docs/CommandGuide/clang.html#cmdoption-O0).
> 
> [`-O4`](https://clang.llvm.org/docs/CommandGuide/clang.html#cmdoption-O0) and higher
> 
> > Currently equivalent to [`-O3`](https://clang.llvm.org/docs/CommandGuide/clang.html#cmdoption-O0)

Also see [x86 options](https://gcc.gnu.org/onlinedocs/gcc/x86-Options.html) to possibly enable chip-specific optimizations (which produce less portable binaries)