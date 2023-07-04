## Execution Results Handling in Clang-REPL 

Following are the major feautres contributed by Jun Zhang.

#### 1. Automatic Printf
The `Automatic Printf` feature makes it easy to display variable values during 
program execution. Using the `printf` function repeatedly is not required. 

[Click here to view Automatic PrintF Feature details](Automatic_PrintF.md)

#### 2. 'Value' Interface

In many cases, it is useful to bring back the program execution result to the 
compiled program. This result can be stored in an object of type 'Value' and 
stored to be further processed or to be displayed to the user.

[Click here to view Value Interface details](Value_Interface.md)

#### 3. Pretty Printing

This feature helps create a temporary dump to display the value and type 
(pretty print) of the desired data. This is a good way to interact with the 
interpreter during interactive programming.

[Click here to view Pretty Printing details](Pretty_Printing.md)

### Getting to know the developer

Jun is a software engineering undergraduate student. Working on the Clang/LLVM 
infrastructure, he has contributed ~70 patches. More details here: 

[Jun Zhang's Profile](JunProfile.md)

### Detailed RFC and Discussion

For more technical details and community discussion on these features, please 
visit:

[RFC on LLVM Discourse](https://discourse.llvm.org/t/rfc-handle-execution-results-in-clang-repl/68493)
