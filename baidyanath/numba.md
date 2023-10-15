## Introducing Numba

Numba is a JIT compiler for (a subset of) Python functions that can be
statically typed based on their input arguments. It essentially solves
the problem of slow inner loops, since data is often dealt with within
loops, creating performance bottlenecks.

Numba can compile python functions into LLVM IR, helping them execute
much faster than vanilla python.

### How Numba deals with the Python related Challenges?

Python uses the Duck Typing conventions, where each type is stored in a wrapper,
making it harder to deduce the types of stored artifacts. This is also one of
the contributing factors for its lack of efficiency and speed.

Numba converts the code into LLVM IR. Doing this removes the need for conversion
from basic types to their wrapped counterparts.

### Where does the Numba code reside?

Most of the Numba logic resides in the [numba_ext.py] file in the 
[cppyy repository].

> Note that this is a fork of the [original cppyy repo].

#### C++ to Numba Mappings

This file includes manual C++ type conversions (`_cpp2numba`). This manual
mapping is included since Numba doesn't inherently capture C++ types (as opposed
to Python types, which it does support). 

#### C++ to LLVM IR Mappings

Similarly, the mapping between C++ and LLVM IR (`_cpp2ir`) is also manually
defined. It uses the values from the [llvmlite] library to fetch relevant LLVM
types.

> `llvmlite` is a lightweight LLVM-Python binding for writing JIT compilers. 

#### Support for Functions and Classes

Using the mappings defined in this file, support for functions
(`CppFunctionNumbaType`) and classes (`CppClassNumbaType`) is available.

> Note that constructing classes inside Numba is not yet supported.

> Numba doesn't know the overload of the function before the user inputs their
> arguments. `CppFunctionNumbaType` tries to deduce the overload of the chosen
> function through the arguments supplied by the user. It then provides the
> signature of the chosen function to Numba. This is needed because Python doesn't
> have types, while Numba has its own types for each Python argument that is
> supplied. 


### How to start using Numba in my code?

To start using Numba, specific [decorators] are used above the relevant
functions to help Numba identify and JIT compile them. Numba will convert it
into LLVM IR the first time you execute that function. 

### Overcoming Limitations

Numba currently only supports a subset of Python. For example, it didn't
natively support the types of structures that are created by cppyy, which is why
cppyy support needed to be explicitly added to Numba, to be able to keep using
cppyy within Python, while also benefitting with the conversion to LLVM IR that
Numba can provide for specific functions.

This is challenging since three entities (C++, cppyy, and Numba) that
communicate differently need to work together, leading to a a more strict set
of restrictions.

For now, Numba supports primitive types and some class types in C++, but support
for other core C++ features (e.g., reference types) is still in the works.

### Long Term Outlook

Working with cppyy, Numba can look into C++ code. This enables a whole new
understanding of how two languages can work together. If an optimizer can see
the same representation coming from both, Numba and C++, it can optimize it a
lot better.

The next step would be to merge those runtimes together for better optimization
to take place. This is because normally, when two languages are communicating
with each other, at some point, the low-level code will request a function
call at some address with some parameters. This crossing of the language
boundary is slow, especially when large computations are involved.

The proposed solution is, that instead of incorporating the slow function call
(where the function's progress is invisible), incorporate the body of the
function call itself, where it is supposed to be called. This gives better
visibility to the optimizer, enabling it to remove indirection and scope-related
parts.

This is what the current POC has paved the way for, that it is possible to
combine cppyy and Numba, empowering greater language interoperability.

---

## Further Reading

Numba documentation:
[numba.readthedocs.io](https://numba.readthedocs.io/en/stable/user/index.html).

[numba_ext.py]: https://github.com/compiler-research/cppyy/blob/master/python/cppyy/numba_ext.py

[decorators]: https://numba.readthedocs.io/en/stable/user/jit.html

[cppyy repository]: https://github.com/compiler-research/cppyy/tree/master

[original cppyy repo]: https://github.com/wlav/cppyy

[llvmlite]: https://github.com/numba/llvmlite

