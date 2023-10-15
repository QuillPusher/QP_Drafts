## cppyy: Python-C++ bindings interface based on Cling/LLVM

cppyy provides fully automatic, dynamic Python-C++ bindings by
leveraging the Cling C++ interpreter and LLVM. It supports both PyPy
(natively), CPython, and C++ language standards through C++17 (and parts
of C++20).

## Background

Early Python enthusiasts may recognize cppyy as a successor to the
PyRoot component in the ROOT Framework, that was developed in the early
2000s, and after the introduction of Cling interpreter, it evolved into
cppyy.

## Recent interoperablilty Enhancements

### Reducing Dependencies

Recent work done on cppyy has been focused on removing unnecessary
dependencies on domain-specific infrastructure (since initial research
was done for the High Energy Physics (HEP) field, but its usefulness was
discovered beyond that as well).

Only a small set of APIs are needed to connect to the interpreter, since
other APIs are already available in the standard compiler. This is what
led to the creation of LibInterOp (a library of helper functions), that
helped extact out things that were unnecessary for cppyy, etc.

The API surface is now incomparably smaller and simpler than what it
used to be.

## Making C++ More Social

cppyy serves as a great proof of concept for other languages to become
interoperable with C++ (using LibInterOp). This helps a lot of data
scientists that are working with legacy C++ code and would like to
migrate to simpler, more interactive languages.

The goal is to eventually land these interoperablilty tools (including
LibInterOp) to greater communities like LLVM and Clang, to enable C++ to
interact with other languages besides Python.

## Where does the cppyy code reside?

Following are the main components where cppyy logic (with Compiler Research
Organization's customizations started by [sudo-panda]) resides:

- [cppyy](https://github.com/compiler-research/cppyy)
- [cppyy-backend](https://github.com/compiler-research/cppyy-backend)
- [CPyCppyy](https://github.com/compiler-research/CPyCppyy)

> Note: These are forks of the [original cppyy] repos created by [wlav].

CppInterOp is an additional library used with Cling/Clang interpreters that
helps these packages communicate with C++ code.

- [CppInterOp](https://github.com/compiler-research/CppInterOp)

### cppyy-backend

The `cppyy-backend` forms a layer over `cppyy`, modifying some functionality to
provide the functions required for `CPyCppyy`. It also adds some [utilities] to
help with repackaging and redistribution.

For example, it initializes the interpreter (using the
`clingwrapper::ApplicationStarter` function), adds the required `include` paths,
and adds the headers required for cppyy to work. It also adds some checks and
combines two or more functions to help CPyCppyy work.

These changes help ensure that any change in `cppyy` doesn't directly affect
`CPyCppyy`, and the API for `CPyCppyy` remains unchanged.

### CPyCppyy

`CPyCppyy` uses the functionality provided by `cppyy-backend` and provides
Python objects for C++ entities. `CPyCppyy` uses separate proxy classes for each
type of object. It also includes helper classes, for example, `Converters.cxx`
helps convert Python type objects to C++ type objects, while `Executors.cxx` is
used to execuate a function and convert its return value to a Python object, so
that it can be used inside Python.

### cppyy

cppyy provides the front-end for Python. It is [included] in code to import
cppyy in Python. It initializes things on the backend side, provides helper
functions (e.g., `cppdef()`, `cppexec()`, etc.) that the user can utilize, and
it calls the relevant backend functions required to initialize cppyy. 

---

## Further Reading

Details and performance are described in [this paper], originally presented at
PyHPC'16, but since updated with improved performance numbers. A more 
[recent paper] was presented in April 2023 focuses on efficiency and accuracy
improvement.

cppyy documentation: [cppyy.readthedocs.io](http://cppyy.readthedocs.io/).

Notebook-based tutorial: [Cppyy Tutorial](https://github.com/wlav/cppyy/blob/master/doc/tutorial/CppyyTutorial.ipynb).


[this paper]: http://cern.ch/wlav/Cppyy_LavrijsenDutta_PyHPC16.pdf

[recent paper]: https://arxiv.org/abs/2304.02712

[wlav]: https://github.com/wlav

[original cppyy]: https://github.com/wlav/cppyy

[sudo-panda]: https://github.com/sudo-panda

[utilities]: https://cppyy.readthedocs.io/en/latest/utilities.html

[included]: https://cppyy.readthedocs.io/en/latest/starting.html