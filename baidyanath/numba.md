## Introducing Numba

Numba is a JIT compiler for (a subset of) Python functions that can be
statically typed based on their input arguments. It essentially solves the
problem of slow inner loops, since data is often dealt with within loops,
creating performance bottlenecks.

Numba translates python functions into native code (using the LLVM framework),
helping numerically heavy programs execute much faster than vanilla python.

Previously, PyROOT objects were not supported by Numba, but recent developments
have changed that. Now there is an extension for PyROOT, that enables the use
of PyROOT objects inside Numba JIT-ed functions, allowing Numba to determine
object types and efficiently convert them into machine code.

### How Numba deals with the Python related Challenges?

Python uses the Duck Typing conventions, where each type is stored in a wrapper,
making it harder to deduce the types of stored artifacts. This is also one of
the contributing factors for its lack of efficiency and speed.

Numba converts the code into LLVM IR. Doing this removes the need for conversion
from basic types to their wrapped counterparts.

### How CPPYY and PyROOT relate to Numba

cppyy is an automatic runtime Python-C++ bindings generator. It is the core of
PyROOt, which is why the Numba extension was originally developed with cppyy,
and later ported to PyROOT. Now, the Numba extension can be used with both,
cppyy and PyROOT.

### Advantages of using Numba with cppyy/PyROOT

- Numba makes loops fast: When using Cppyy/PyROOT with Python, the loops in
Python are slower as compared to languages such as C/C++. Numba alleviates this
problem and can make it as fast as C without much code instrumentation.

- Flexibility to Code completely in python: This makes debugging easier. To
debug Numba instrumented code you can either comment out the instrumentation
line and debug the code as you would do in Python or use gdb using numba.gdb .
Numba also has a variety of flags that can be turned on to see tracebacks and
the intermediate steps taken by Numba. This is easier than to debug a code that
is setup in Python and uses RDF for hotspots.

- No extra conversions required for machine code: Cppyy can be converted to
machine code cleanly, that is no boxing and unboxing is required, so we do not
spend any time in type conversions and gain the maximum amount of speedup
possible.

- Easier transitioning from one language to the other : You can switch easily
  between C++ and Python as and when you want.

### Features of Numba Extension

- **Plug and Play**: To use the extension you just need to import
  cppyy.numba_ext and then you can use C++ functions in Numba directly.

- **Template instantiation**:Cppyy supports template instantiation which gives
  you access to an important feature set in C++ that is used abundantly in lot
  of codebases. This extension extends that support to Numba too so any
  templated C++ function can be used in Numba. 

- **Overload selection**: Similar to template instantiation the extension helps
  select the appropriate overload based on the type of the input provided to
  the function.

> For code examples and demos, please see: [Using C++ From Numba, Fast and Automatic, PyHEP 2022]

### Where does the Numba code reside?

Most of the Numba logic resides in the [numba_ext.py] file in the 
[cppyy repository].

> Note that this is a fork of the [original cppyy repo].

#### C++ to Numba Mappings

Among other things, the `numba_ext.py` file includes manual C++ type
conversions (`_cpp2numba`). This manual mapping is included since Numba doesn't
inherently capture C++ types (as opposed to Python types, which it does
support). 

#### C++ to LLVM IR Mappings

Similarly, the mapping between C++ and LLVM IR (`_cpp2ir`) is also manually
defined. It uses the values from the [llvmlite] library to fetch relevant LLVM
types.

> **llvmlite** is a lightweight LLVM-Python binding for writing JIT compilers. 

#### Support for Functions and Classes

Using the mappings defined in this file, support for functions
(`CppFunctionNumbaType`) and classes (`CppClassNumbaType`) is available.

> Note that constructing classes inside Numba is not yet supported.

> Numba doesn't know the overload of the function before the user inputs their
> arguments. `CppFunctionNumbaType` tries to deduce the overload of the chosen
> function through the arguments supplied by the user. It then provides the
> signature of the chosen function to Numba. This is needed because Python 
> doesn't have types, while Numba has its own types for each Python argument 
> that is supplied. 


### How to start using Numba in my code?

To start using Numba, specific [decorators] are used above the relevant
functions to help Numba identify and JIT compile them. Numba will convert it
into LLVM IR the first time you execute that function. 

### Overcoming Limitations

Numba currently only supports a subset of Python. For example, it didn't
natively support the types of structures that are created by cppyy, which is
why cppyy support needed to be explicitly added to Numba, to be able to keep
using cppyy within Python, while also benefitting with the conversion to LLVM
IR that Numba can provide for specific functions.

This is challenging since three entities (C++, cppyy, and Numba) that
communicate differently need to work together, leading to a stricter set of
restrictions.

For now, Numba supports primitive types and some class types in C++, but
support for other core C++ features (e.g., reference types) is still in the
works.

### Long Term Outlook

Working with cppyy, Numba also work with C++ code. This enables a whole new
understanding of how two languages can work together. If an optimizer can see
the same representation coming from both, Numba and C++, it can optimize it a
lot better.

The next step would be to merge those runtimes together for better optimization
to take place. This is because normally, when two languages are communicating
with each other, at some point, the low-level code will request a function call
at some address with some parameters. This crossing of the language boundary is
slow, especially when large computations are involved.

The proposed solution is, that instead of incorporating the slow function call
(where the function's progress is invisible), incorporate the body of the
function call itself, where it is supposed to be called. This gives better
visibility to the optimizer, enabling it to remove indirection and
scope-related parts.

This is what the current POC has paved the way for, that it is possible to
combine cppyy and Numba, empowering greater language interoperability.

---

## Further Reading

- Numba documentation: [numba.readthedocs.io](https://numba.readthedocs.io/en/stable/user/index.html).

- [Using C++ From Numba, Fast and Automatic, PyHEP 2022]
   - [How to try out this notebook](https://github.com/sudo-panda/PyHEP-2022)

- [Parallelism, Performance and Programming Model Meeting](https://indico.cern.ch/event/1196174/)
   - [Notebook](https://indico.cern.ch/event/1196174/contributions/5028203/attachments/2501253/4296735/PPP.ipynb)
   - [Slides](https://indico.cern.ch/event/1196174/contributions/5028203/attachments/2501253/4296778/PPP.pdf)

- [Using C++ From Numba, Fast and Automatic](https://compiler-research.org/assets/presentations/B_Kundu-PyHEP22_Cppyy_Numba.pdf)
   - [Video](https://www.youtube.com/watch?v=RceFPtB4m1I)


[numba_ext.py]: https://github.com/compiler-research/cppyy/blob/master/python/cppyy/numba_ext.py

[decorators]: https://numba.readthedocs.io/en/stable/user/jit.html

[cppyy repository]: https://github.com/compiler-research/cppyy/tree/master

[original cppyy repo]: https://github.com/wlav/cppyy

[llvmlite]: https://github.com/numba/llvmlite

[Using C++ From Numba, Fast and Automatic, PyHEP 2022]: https://compiler-research.org/presentations/#CppyyNumbaPyHEP2022