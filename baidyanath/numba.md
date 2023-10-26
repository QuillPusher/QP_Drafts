## Introduction

Numba is a JIT compiler for (a subset of) Python functions that can be
statically typed based on their input arguments. It essentially solves the
problem of slow inner loops, since data is often dealt with within loops,
creating performance bottlenecks.

## Latest enhancements in Numba from Compiler Research Org's contributors

There are two notable enhancements:

1. Enabling Numba to digest C++ code. This work builds on top of the principles
   established during Python/C++ interoperability development. This was the
   next step in adding a third programming entity to the mix. This meant more
   rules and restrictions since all three entities needed to agree on the
   specific operations.

2. Enabling PyROOT objects to work with Numba, addressing a major pain-point in
   the world of High Energy Physics research. Now there is an extension for
   PyROOT, that enables the use of PyROOT objects inside Numba JIT-ed
   functions, allowing Numba to determine object types and efficiently convert
   them into machine code.


### How CPPYY and PyROOT relate to Numba

cppyy is an automatic runtime Python-C++ bindings generator. It is the core of
PyROOT, which is why the Numba extension was originally developed with cppyy,
and later ported to PyROOT. Now, the Numba extension can be used with both,
cppyy and PyROOT.

### How Numba deals with the Python related Challenges

Python uses the Duck Typing conventions, where each type is stored in a
wrapper, making it harder to deduce the types of stored artifacts. This is also
one of the contributing factors for its lack of efficiency and speed.

Numba converts the code into LLVM IR. Doing this removes the need for
conversion from basic types to their wrapped counterparts. In other words,
Numba translates python functions into native code (using the LLVM framework),
helping numerically heavy programs execute much faster than vanilla python.

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
  cppyy.numba_ext and then you can use C++ functions in Numba directly. In the
  example shown below sqrt is a C++ function that can be used directly inside
  the Numba JIT-ed function with the help of the extension.

  ```
  import numba
  import cppyy
  import cppyy.numba_ext # <------- Imports the necessary information for numba to work with cppyy
  import math
  @numba.jit(nopython=True)
  def cpp_sqrt(x):
   return cppyy.gbl.sqrt(x) # <------------ Direct use, no extra setup required
  print("Sqrt of 4: ", cpp_sqrt(4.0))
  print("Sqrt of Pi: ", cpp_sqrt(math.pi))
  ```
  
  Output:

  Sqrt of 4: 2.0

  Sqrt of Pi: 1.7724538509055159

- **Template instantiation**:Cppyy supports template instantiation which gives
  you access to an important feature set in C++ that is used abundantly in lot
  of codebases. This extension extends that support to Numba too so any
  templated C++ function can be used in Numba. Below we have a templated square
  function and depending on the type of the matrix the extension instantiates
  the required template argument.

  ```
  import cppyy
  import cppyy.numba_ext
  import numba
  import numpy as np
  cppyy.cppdef("""
  template<typename T>
  T square(T t) { return t*t; }
  """)
  @numba.jit(nopython=True)
  def tsa(a):
    total = type(a[0])(0)
    for i in range(len(a)):
      total += cppyy.gbl.square(a[i])
    return total
  a = np.array(range(10), dtype=np.float32)
  print("Float array: ", a)
  print("Sum of squares: ", tsa(a))
  print()
  a = np.array(range(10), dtype=np.int32)
  print("Integer array: ", a)
  print("Sum of squares: ", tsa(a))
  ```

  Output:

   Float array: [0. 1. 2. 3. 4. 5. 6. 7. 8. 9.]

   Sum of squares: 285.0
  
   
  
   Integer array: [0 1 2 3 4 5 6 7 8 9]
  
   Sum of squares: 285


- **Overload selection**: Similar to template instantiation the extension helps
  select the appropriate overload based on the type of the input provided to
  the function.

  ```
  cppyy.cppdef("""
  int mul(int x) { return x * 2; }
  float mul(float x) { return x * 3; }
  """)
  @numba.jit(nopython=True)
  def oversel(a):
    total = type(a[0])(0)
    for i in range(len(a)):
      total += cppyy.gbl.mul(a[i])
    return total
  
  a = np.array(range(10), dtype=np.float32)
  print("Array: ", a)
  print("Overload selection output: ", oversel(a))
  
  a = np.array(range(10), dtype=np.int32)
  print("Array: ", a)
  print("Overload selection output: ", oversel(a))
  ```

  Output:

   Array: [0. 1. 2. 3. 4. 5. 6. 7. 8. 9.]
  
   Overload selection output: 135.0
  
   Array: [0 1 2 3 4 5 6 7 8 9]
  
   Overload selection output: 90

### Demos

#### 1. Numba physics example

Taken from: [numba_scalar_impl.py]

```
import numba
import cppyy
import cppyy.numba_ext

cppyy.cppdef("""
#include <vector>
struct Atom {
    float x;
    float y;
    float z;
};

std::vector<Atom> atoms = {{1, 2, 3}, {2, 3, 4}, {3, 4, 5}, {4, 5, 6}, {5, 6, 7}};
""")

@numba.njit
def lj_numba_scalar(r):
    sr6 = (1./r)**6
    pot = 4.*(sr6*sr6 - sr6)
    return pot

@numba.njit
def distance_numba_scalar(atom1, atom2):
    dx = atom2.x - atom1.x
    dy = atom2.y - atom1.y
    dz = atom2.z - atom1.z

    r = (dx * dx + dy * dy + dz * dz) ** 0.5

    return r

 def potential_numba_scalar(cluster):
    energy = 0.0
    for i in range(cluster.size() - 1):
      for j in range(i + 1, cluster.size()):
        r = distance_numba_scalar(cluster[i], cluster[j])
        e = lj_numba_scalar(r)
        energy += e

    return energy

print("Total lennard jones potential =", potential_numba_scalar(cppyy.gbl.atoms))
```
Output:

Total lennard jones potential = -0.5780277345740283

#### 2. Using the extension with PyROOT

To use the extension with PyROOT, just as we do with Cppyy, we need to import
`cppyy.numba_ext`. In the example we use the TLorentzVector class from ROOT. It
has with four properties: `Px` , `Py` , `Pz` and `E`. It also provides the
transverse momentum `Pt` to the user which can be calculated by:

```
####################### Setup Code #############################
import numba
import math
import ROOT
import cppyy.numba_ext # <----------- Import the Numba extension
import time

ROOT.gInterpreter.Declare("""
std::vector<TLorentzVector> vec_lv;

const int no_of_samples = 100;

void fill() {
  vec_lv.reserve(no_of_samples);
  TRandom3 R(111);

  for (int i = 0; i < no_of_samples; ++i) {
    double Px = R.Gaus(0,10);
    double Py = R.Gaus(0,10);
    double Pz = R.Gaus(0,10);
    double E = R.Gaus(100,10);
    vec_lv.push_back(TLorentzVector(Px, Py, Pz, E));
  }
}
""")
ROOT.gInterpreter.ProcessLine("""
fill();
""")

print()
```
Output:

Welcome to JupyROOT 6.27/01

In this example we calculate the same using Python and show how we can speed up
the calculation using Numba. The `calc_pt` function uses pure Python to calculate
`Pt` whereas the `numba_calc_pt` uses Numba to do the same. As before the only
difference between the two is the `numba.njit` decorator so you do not need to
change anything.

```
def calc_pt(lv):
  return math.sqrt(lv.Px() ** 2 + lv.Py() ** 2)

def calc_pt_vec(vec_lv):
  pt = []
  for i in range(vec_lv.size()):
    k pt.append((calc_pt(vec_lv[i]), vec_lv[i].Pt()))
  return pt

@numba.njit
def numba_calc_pt(lv):
  return math.sqrt(lv.Px() ** 2 + lv.Py() ** 2)

def numba_calc_pt_vec(vec_lv):
  pt = []
  for i in range(vec_lv.size()):
      pt.append((numba_calc_pt(vec_lv[i]), vec_lv[i].Pt()))
  return pt
```

```
numba_warmup, _ = measure_execution(numba_calc_pt, (ROOT.vec_lv[0], ))
python_elapsed, _ = measure_execution(calc_pt_vec, (ROOT.vec_lv, ))
numba_elapsed, pt = measure_execution(numba_calc_pt_vec, (ROOT.vec_lv, ))

print(f"Numba'd : Warmup = {numba_warmup :.5f}s")
print(f"Pure Python: Elapsed = {python_elapsed:.5f}s")
print(f"Numba'd : Elapsed = {numba_elapsed :.5f}s")

print(f"Speedup = {python_elapsed / numba_elapsed:.5f}x")

no_of_samples = 3
print("\nCalc pT \tActual pT")
print("---------------------------")
print(*(f"{x:2.5f} \t{y:2.5f}" for x,y in pt[:no_of_samples]), sep="\n")

if False in tuple(x==y for x, y in pt):
    print("\nSome values do not match")
else:
    print("\nAll values for pT match")
```
Output:

 Numba'd : Warmup = 0.04820s

 Pure Python: Elapsed = 0.00813s

 Numba'd : Elapsed = 0.00037s

 Speedup = 22.21831x

 

 Calc pT     Actual pT

 8.95222     8.95222

 4.11973     4.11973

 25.97929    25.97929

> All values for pT match

#### 3. RDF

You can also use it inside RDF through `ROOT.Numba.Declare`. Underneath is a
simple example where it is used to calculate the power function.

```
import numba
import ROOT
import cppyy.numba_ext # <----- Import extension

ROOT.gInterpreter.Declare("""
double cpppow(double x, int y) { return pow(x, y); }
""")

@ROOT.Numba.Declare(['double', 'int'], 'double')
def pypownd(x, y):
    return ROOT.cpppow(x, y) # <--------- Numba.Declare supports PyROOT due to the Numba extension

ROOT.gInterpreter.ProcessLine("""
cout << "2^3 = " << Numba::pypownd(2, 3) << endl
    << "4^5 = " << Numba::pypownd(4, 5) << endl;""")
print()

# Or we can use the callable as well within a RDataFrame workflow.
data = ROOT.RDataFrame(4).Define('x', '(float)rdfentry_')\
                          .Define('x_pow3', 'Numba::pypownd(x, 3)')\
                          .AsNumpy()

print('pypownd({}, 3) = {}'.format(data['x'], data['x_pow3']))
```

Output:

 pypownd([0. 1. 2. 3.], 3) = [ 0. 1. 8. 27.]

 2^3 = 8

 4^5 = 1024


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


## Credits:

- [Baidyanath Kundu] (Princeton University) for his research work on cppyy and
  Numba for [Compiler Research Organization].

- [Wim Lavrijsen] (Lawrence Berkeley National Lab.) for originally working on cppyy.

- [Vassil Vasilev] (Princeton University) for mentoring Baidyanath and
  continuing this research with [Aaron Jomy].

[Vassil Vasilev]: https://github.com/vgvassilev

[Aaron Jomy]: https://github.com/maximusron

[Baidyanath Kundu]: https://github.com/sudo-panda

[Wim Lavrijsen]: https://github.com/wlav

[Compiler Research Organization]: https://compiler-research.org/

[numba_ext.py]: https://github.com/compiler-research/cppyy/blob/master/python/cppyy/numba_ext.py

[decorators]: https://numba.readthedocs.io/en/stable/user/jit.html

[cppyy repository]: https://github.com/compiler-research/cppyy/tree/master

[original cppyy repo]: https://github.com/wlav/cppyy

[llvmlite]: https://github.com/numba/llvmlite

[Using C++ From Numba, Fast and Automatic, PyHEP 2022]: https://compiler-research.org/presentations/#CppyyNumbaPyHEP2022

[numba_scalar_impl.py]: https://github.com/numba/numba-examples/blob/master/examples/physics/lennard_jones/numba_scalar_impl.py