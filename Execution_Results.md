## Execution Results Handling in Clang-REPL 

### Automatic Printf
The `Automatic Printf` feature makes it easy to display variable values during 
program execution. Using the `printf` function repeatedly is not required. 
This is achieved using an extension in the `libclangInterpreter` library.

To automatically print the value of an expression, simply write the expression 
in the global scope **without a semicolon**.

![Automatic PrintF](https://github.com/QuillPusher/drafts/blob/main/img_PrintF.png)

Following are some examples:

```
clang-repl> int x = 42;
clang-repl> x // equivalent to calling printf("(int &) %d\n", x);
(int &) 42

clang-repl> std::vector<int> v = {1,2,3};
clang-repl> v // This syntax is fine after [D127284](https://reviews.llvm.org/D127284)
(std::vector<int> &) {1,2,3}

clang-repl> "Hello, interactive C++!"
(const char [24]) "Hello, interactive C++!"
```

![Capture Execution Results](https://github.com/QuillPusher/drafts/blob/main/img_ExecResults.png)
