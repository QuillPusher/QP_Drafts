## Introduction

cppyy provides fully automatic, dynamic Python-C++ bindings by leveraging the
Cling C++ interpreter and LLVM. It differs from other Python-C++ binders since
it creates bindings at runtime (using Cling interpreter for C++ side runtime),
which is a lot more natural for Python, where most actions are performed at
runtime. 

## Latest enhancements in cppyy from Compiler Research Org's contributors

- Reducing dependencies of cppyy to minimize code bloat and make it faster.

- Creating a cppyy-style CppInterOp library that enables interoperability with
  C++ code, bringing the speed and efficiency of C++ to simpler, more
  interactive languages like Python.

### Reducing Dependencies

Recent work done on cppyy has been focused on removing unnecessary dependencies
on domain-specific infrastructure (e.g., the ROOT framework). The idea was to
convert the cppyy-backend to use Cling directly (instead of ROOT meta), and
then use it in cppyy.

Only a small set of APIs are needed to connect to the interpreter, since
other APIs are already available in the standard compiler. This is what
led to the creation of CppInterOp (a library of helper functions), that
helped extract out things that were unnecessary for cppyy, etc.

The API surface is now incomparably smaller and simpler than what it
used to be.

### CppInterOp library

CppInterOp can be adopted incrementally. While the rest of the framework is the
same, a small part of CppInterOp can be utilized. More components may be
adopted over time.

It is designed to be simple and robust (simple function calls, no inheritance,
etc.). The goal is to make it as close to the compiler API as possible, and
each routine to do just one thing that it was designed for.

## Making C++ More Social

cppyy serves as a great proof of concept for other languages to become
interoperable with C++ (using LibInterOp). This helps a lot of data
scientists that are working with legacy C++ code and would like to
migrate to simpler, more interactive languages.

The goal of this research is to eventually land these interoperability tools
(including CppInterOp) to greater communities like LLVM and Clang, to enable
C++ to interact with other languages besides Python.

## Example: Template Instantiation

The developmental Cppyy version can run basic examples such as the one here.
Features such as standalone functions and basic classes are also supported.

C++ code (Tmpl.h)

```
template <typename T>
struct Tmpl {
  T m_num;
  T add (T n) {
    return m_num + n;
}
};
```

Python Interpreter

```
>>> import cppyy
>>> import cppyy.gbl as Cpp
>>> cppyy.include("Tmpl.h")
>>> tmpl = Tmpl[int]()
>>> tmpl.m_num = 4
>>> print(tmpl.add(5))
9
>>> tmpl = Tmpl[float]()
>>> tmpl.m_num = 3.0
>>> print(tmpl.add(4.0))
7.0
```

## Where does the cppyy code reside?

Following are the main components where cppyy logic (with Compiler Research
Organization's customizations started by [sudo-panda]) resides:

- [cppyy](https://github.com/compiler-research/cppyy)
- [cppyy-backend](https://github.com/compiler-research/cppyy-backend)
- [CPyCppyy](https://github.com/compiler-research/CPyCppyy)

> Note: These are forks of the [original cppyy] repos created by [wlav].

CppInterOp is an additional library used with Cling/Clang interpreters that
helps these packages communicate with C++ code.

- [CppInterOp]

## How cppyy components interact with each other

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
used to execute a function and convert its return value to a Python object, so
that it can be used inside Python.

### cppyy

cppyy provides the front-end for Python. It is [included] in code to import
cppyy in Python. It initializes things on the backend side, provides helper
functions (e.g., `cppdef()`, `cppexec()`, etc.) that the user can utilize, and
it calls the relevant backend functions required to initialize cppyy. 

---

## Further Reading

- [High-performance Python-C++ bindings with PyPy and Cling]

- [Efficient and Accurate Automatic Python Bindings with cppyy & Cling] 

- cppyy documentation: [cppyy.readthedocs.io].

- Notebook-based tutorial: [Cppyy Tutorial].

- [C++ Language Interoperability Layer]

## Credits:

- [Baidyanath Kundu] (Princeton University) for his research work on cppyy and
  Numba for [Compiler Research Organization].

- [Vassil Vasilev] (Princeton University) for mentoring Baidyanath and
  continuing this research with [Aaron Jomy].

- [Wim Lavrijsen] (Lawrence Berkeley National Lab) cppyy's original contributor.


[Vassil Vasilev]: https://github.com/vgvassilev

[Aaron Jomy]: https://github.com/maximusron

[Baidyanath Kundu]: https://github.com/sudo-panda

[Wim Lavrijsen]: https://github.com/wlav

[Compiler Research Organization]: https://compiler-research.org/

[High-performance Python-C++ bindings with PyPy and Cling]: http://cern.ch/wlav/Cppyy_LavrijsenDutta_PyHPC16.pdf

[Efficient and Accurate Automatic Python Bindings with cppyy & Cling]: https://arxiv.org/abs/2304.02712

[Cppyy Tutorial]: https://github.com/wlav/cppyy/blob/master/doc/tutorial/CppyyTutorial.ipynb

[wlav]: https://github.com/wlav

[original cppyy]: https://github.com/wlav/cppyy

[sudo-panda]: https://github.com/sudo-panda

[utilities]: https://cppyy.readthedocs.io/en/latest/utilities.html

[included]: https://cppyy.readthedocs.io/en/latest/starting.html

[cppyy.readthedocs.io]: http://cppyy.readthedocs.io/

[CppInterOp]: https://github.com/compiler-research/CppInterOp/tree/main

[C++ Language Interoperability Layer]: https://compiler-research.org/libinterop/