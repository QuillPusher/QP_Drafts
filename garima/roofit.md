##  How to extend the use of Automatic Differentiation in RooFit

### What is RooFit?

RooFit is a statistical data analysis tool, widely used in scientific 
research, especially in the high-energy physics (HEP) field. It is an 
extension of the ROOT framework, a C++ based data analysis framework that 
provides tools for data storage, analysis, and visualization. RooFit  
provides a set of tools/classes to define probability density functions 
(PDFs), perform maximum likelihood fits, perform statistical tests, etc.


### Proof of Concept: Speeding up RooFit using Automatic Differentiation

One of the applications of RooFit is reducing statistical models (functions) 
to find a set of parameters that minimize the value of the function. This is 
accomplished using derivatives. In Numeric Differentiation (ND), derivatives 
become a bottleneck, while Automatic Differentiation (AD) is more efficient.

Following is an example performance comparison of gradient generation in 
TFormula/TF1 class in RooFit.

![AD vs ND comparison](ad_performance_graph.png "ND vs. AD Benchmarks")

RooFit is an extensive toolkit that depends on ND, which has its limitations. 
The idea is to apply AD to a few components of RooFit as a proof of concept 
and then provide a template for other data scientists to expand AD 
application in different parts of RooFit.

This can be accomplished by extending relevant RooFit classes (that a 
statistical model is comprised of) with a Translate function, which will 
extract all the mathematical differentiable properties out of these 
classes, use AD techniques and return the results.

### Advantages of using AD with RooFit

AD provides a better way to compute of derivatives of mathematical functions.
 Following are some of its main advantages:

- Efficient and more precise derivatives: It computes derivatives with high 
precision, avoiding the errors that may arise from approximating derivatives 
using finite differences. 

- Flexible model definition: AD can automatically compute derivatives while 
using a large number of parameters in a complex model. This prevents 
error-prone manual efforts.

- Seamless integration: AD can be seamlessly integrated in RooFit to empower 
its large userbase without any additional learning/ implementation. It would 
allow users to create and fit models based on their specific needs and 
experimental data.

- Maintainability: AD ensures consistent gradient computations, minimizing 
errors and making the code easier to maintain and debug.


### AD using Source Code Transformation in Clad

AD can be accomplished using Clad (a C++ plugin for Clang), that helps 
implement a technique called Source Code Transformation (SCT). 

> [Clad](https://compiler-research.org/clad/) is a source transformation AD 
tool implemented as a plugin to the clang compiler, which automatically 
generates the derivative code for input C++ functions. 

SCT takes the source code (that needs to be differentiated) as the input and 
generates an output code that represents the derivative of the input. This 
output code can be used instead of the input code for more efficient 
compilation.

In case of RooFit, this is done by extending RooFit classes using a 
`translate()` function, which can extract all the mathematical differentiable 
properties out of all the RooFit classes that make up the statistical model.

For more technical details, please see the following paper:

> [Automatic Differentiation of Binned Likelihoods With RooFit and Clad](https://arxiv.org/abs/2304.02650)


## Steps to add AD support in RooFit classes

1 - Locate the relevant directory

2 - Review relevant classes where AD implementation logic resides:

   - `CodeSquashContext`: helps create a C++ function out of the compute graph
  
   - `RooFuncWrapper`: helps wrap the generated C++ code in a RooFit object
    
3 - Add AD support to a RooFit Class: 
  
   - select a Class
   - create `translate()` using Helper Functions to translate existing logic
 into AD-supported logic. 
   - update `evaluate()` to expose it to Clad

> See Appendix C for a list of available Helper Functions.

### 1. Locate Relevant Directory

Let us look at one of the RooFit directories. Each class name starts with a 
"Roo" prefix (e.g., RooAddPdf). 

> [roofit/roofitcore/src](https://github.com/root-project/root/tree/cb08bb7445a0b8db0a64a505399844c85ed048a4/roofit/roofitcore/src)

Here we will find physical mathematical notations implemented as independent 
classes, which can be used to build a compute graph.

> A compute graph defines a complex model as an interconnected set of simpler
 building blocks (variables, functions, etc.). These components are combined 
to form a graph, where the nodes are the variables, etc. and the edges 
represent the dependencies between them. This compute graph makes up a model 
on which further data analysis can be executed.

### 2. Review AD Logic Implementation

Following classes provide several Helper Functions to translate existing logic 
into AD-supported logic.

a - CodeSquashContext

b - RooFuncWrapper

#### a. CodeSquashContext

> [roofit/roofitcore/inc/RooFit/Detail/CodeSquashContext.h](https://github.com/root-project/root/blob/6136be0d4514591d8ab93815be941702f5509298/roofit/roofitcore/inc/RooFit/Detail/CodeSquashContext.h)

It handles how to create a C++ function out of the compute graph (which is 
created with different RooFit classes). This C++ function will be independent 
of these RooFit classes.

CodeSquashContext helps traverse the compute graph received from RooFit and 
then it translates that into a single piece of code (a C++ function), that can 
then be differentiated using Clad. It also helps evaluate the model.

In RooFit, evaluation is done using the 'evaluate()' function. It also 
performs a lot of book-keeping, caching, etc. that is required for RooFit (but 
not necessarily for AD). 

##### `translate()` function

A new `translate()` function is added to RooFit classes that includes a call 
to the `evaluate()` function (that most RooFit classes already include). It 
helps implement the Code Squashing logic that is used to optimize numerical 
evaluations. Using helper functions, it helps convert a RooFit expression into 
a form that can be efficiently evaluated.

It returns an `std::string` representing the underlying mathematical notation
 of the class as code, that can later be concatenated into a single string 
representing the entire model. This string of code is then just-in-time 
compiled by Cling (a C++ interpreter for Root).

##### Helper Functions

- **CodeSquashContext()**: this class maintains the context for squashing of 
RooFit models into code.  It keeps track of the results of various 
expressions to avoid redundant calculations.

- **Loop Scopes()**: `beginloop()` and `endloop()` are used to create a scope 
for iterating over vector observables (collections of data). This is 
especially useful when dealing with data that comes in sets or arrays.

- **addToGlobalScope()**: helps add code statements to the global scope 
(e.g., to declare variables).

- **addToCodeBody()**: adds the input string to the squashed code body. If a 
class implements a translate function that wants to emit something to the 
squashed code body, it must call this function with the code it wants to 
emit. In case of loops, it automatically determines if the code needs to be
stored inside or outside the scope of that loop.

- **makeValidVarName()**: takes a string (e.g., a variable name) and converts
 it into a valid C++ variable name by replacing any forbidden characters with
 underscores. 

- **buildArg()**: helps convert RooFit objects into arrays or other C++ 
representations for efficient computation.

- **addResult()**: adds (or overwrites) the string representing the result of
 a node.

> For each `translate()` function, it is important to call `addResult()` since 
this is what enables the squashing to happen. 

- **getResult()**: gets the result for the given node using the node name. 
This node also performs the necessary code generation through recursive calls
 to `translate()`.

- **assembleCode()**: combines the generated code statements into the final 
code body of the squashed function.

These functions will appear again in this document with more contextual 
examples. For detailed in-line documentation (code comments), please see:

> [roofit/roofitcore/src/RooFit/Detail/CodeSquashContext.cxx](https://github.com/root-project/root/blob/a50450c2701bbef8756c20ff8deaf6a48f42205b/roofit/roofitcore/src/RooFit/Detail/CodeSquashContext.cxx)


#### b. RooFuncWrapper

> [roofit/roofitcore/inc/RooFuncWrapper.h](https://github.com/root-project/root/blob/6136be0d4514591d8ab93815be941702f5509298/roofit/roofitcore/inc/RooFuncWrapper.h)

This class wraps the generated C++ code in a RooFit object, so that it can be
 used like other RooFit objects.

It takes a function body as input and creates a callable function from it. 
This allows users to evaluate the function and its derivatives efficiently.

##### Helper Functions

- **loadParamsAndData()** extracts parameters and observables from the 
provided data and prepares them for evaluation.

- **declareAndDiffFunction()**: declare the function and create its 
derivative.

- **gradient()**: calculates the gradient of the function with respect to its
 parameters.

- **buildCode()**: generates the optimized code for evaluating the function 
and its derivatives. 

- **dumpCode()**: prints the squashed code body to console (useful for 
debugging).

- **dumpGradient()**: prints the derivative code body to console (useful for 
debugging).

These functions will appear again in this document with more contextual 
examples. For detailed in-line documentation (code comments), please see:

> [roofit/roofitcore/src/RooFuncWrapper.cxx](https://github.com/root-project/root/blob/8d03a461ff8cf1b2ac3b20277cd962328b340e09/roofit/roofitcore/src/RooFuncWrapper.cxx)


### 3. Adding AD support to a RooFit Class

Let us take the `RooPoisson.cxx` class as an example. 

> [roofit/roofit/src/RooPoisson.cxx](https://github.com/root-project/root/blob/6136be0d4514591d8ab93815be941702f5509298/roofit/roofit/src/RooPoisson.cxx)

First step is to locate the `evaluate()` function. Most RooFit classes 
implement this function.

> RooFit internally calls the `evaluate()` function to evaluate a single node
 in a compute graph.

#### Before AD Support

Following is a code snippet from `RooPoisson` *before* it had AD support.

```C++
double RooPoisson::evaluate() const
{
  ...
  }										
  return TMath::Poisson(k, mean);
}
```
`TMath::Poisson()` is a simple mathematical function. To translate the 
`RooPoisson` class, create a translate function and in it include a call to 
this `TMath::Poisson()` function.

#### After AD Support

Following is a code snippet from `RooPoisson` *after* it has AD support.

##### Added `translate()` Function (using Helper Functions)

```C++
void RooPoisson::translate(RooFit::Detail::CodeSquashContext &ctx) const
{
   std::string xName = ctx.getResult(x);
     ...
   ctx.addResult(this, ctx.buildCall("RooFit::Detail::EvaluateFuncs::poissonEvaluate", xName, mean));
}
```
Following Helper Functions were used:

- `getResult()` helps lookup the result of a child node (the string that the 
child node previously saved in a variable using the `addResult()` function).  

- `addResult()` It may include a function call, an expression, or something 
more complicated. For a specific class, it will add whatever is represented on
 the right hand side to the result of that class, which can then be propagated
 in the rest of the compute graph.

> For each `translate()` function, it is important to call `addResult()` since 
this is what enables the squashing to happen. 

##### Updated `evaluate()` Function

```C++
double RooPoisson::evaluate() const
{
  ...
  }
  return RooFit::Detail::EvaluateFuncs::poissonEvaluate(k, mean);
}
```

Note that the `evaluate()` function was refactored in such a way that the 
mathematical parts were moved to an inline function in a separate header file 
named `EvaluateFuncs`, so that Clad could see and differentiate that function.
 See Appendix A for a detailed example.

> All contents of the `evaluate()` function don't always need to be pulled 
out, only the required parts (mathematical  logic) should be moved to 
`EvaluateFuncs`.

###### What is EvaluateFuncs?

Moving away from the class-based hierarchy design, `EvaluateFuncs.h` a simply 
a flat file of function implementations. 

This file is required since Clad will not be able to see anything that is not 
inlined and explicitly available to it during compilation (since it has to be 
in the same translation). So other than of generating these functions on the 
fly, your only other option is to place these functions in a separate header 
file and make them inline. 

Theoretically, multiple header files can also be used and then mashed 
together.

> Directory path: [roofit/roofitcore/inc/RooFit/Detail/EvaluateFuncs.h](https://github.com/root-project/root/blob/4e8c577dfd6a19d7c38a74e3074b406a598bf76a/roofit/roofitcore/inc/RooFit/Detail/EvaluateFuncs.h)


##### More `translate()` Examples

**Example 1:** Following is a code snippet from `RooGaussian.cxx` *after* it 
has AD support.

```C++
void RooGaussian::translate(RooFit::Detail::CodeSquashContext &ctx) const
{
   ctx.addResult(this, ctx.buildCall("RooFit::Detail::EvaluateFuncs::gaussianEvaluate", x, mean, sigma));
}
```

Following helper function can be seen here.

- `buildCall()` helps build a function call. Requires the fully qualified name
 (`RooFit::Detail::EvaluateFuncs::gaussianEvaluate`) of the function. When 
this external `buildCall()` function is called, internally, the `getResult()` 
function is called on the input RooFit objects (e.g., x, mean, sigma). That's 
the only way to propagate these upwards into the compute graph.

**Example 2:** A more complicated example of a `translate()` function can be 
seen here: 

```C++
void RooNLLVarNew::translate(RooFit::Detail::CodeSquashContext &ctx) const
{
   std::string weightSumName = ctx.makeValidVarName(GetName()) + "WeightSum";
   std::string resName = ctx.makeValidVarName(GetName()) + "Result";
   ctx.addResult(this, resName);
   ctx.addToGlobalScope("double " + weightSumName + " = 0.0;\n");
   ctx.addToGlobalScope("double " + resName + " = 0.0;\n");

   const bool needWeightSum = _expectedEvents || _simCount > 1;

   if (needWeightSum) {
      auto scope = ctx.beginLoop(this);
      ctx.addToCodeBody(weightSumName + " += " + ctx.getResult(*_weightVar) + ";\n");
   }
   
 ... 

}
```

> Source: - [RooNLLVarNew](https://github.com/root-project/root/blob/6136be0d4514591d8ab93815be941702f5509298/roofit/roofitcore/src/RooNLLVarNew.cxx#L298)


Helper functions from the above example:

- `makeValidVarName()` helps get a valid name from the name of the respective 
RooFit class. It then helps save it to the variable that represents the result
 of this class (the squashed code/ C++ function that will be created). 

- `addToGlobalScope()` helps declare and initialize the results variable, so 
that it can be available globally (throughout the function body). For local 
variables, the `addToCodeBody()` function can be used to keep the variables in
 the respective scope (for example, within a loop).

- `beginLoop()` helps build the start and the end of a For loop for your 
class. Simply place this function in the scope and place the contents of the 
`For` loop below this statement. The code squashing task will automatically 
build a loop around the statements that follow it. There's no need to worry 
about the index of these loops, because they get propagated. For example, if 
you want to iterate over a vector of RooFit objects using a loop, you don't 
have to think about indexing them properly because the `beginLoop()` function 
takes care of that. Simply call this function, place your function call in a 
scope and after the scope ends, the loop will also end.

- `addToCodeBody()` helps add things to the body of the C++ function that 
you're creating. It takes whatever string is computed in its arguments and 
adds it to the overall function string (which will later be just-in-time 
compiled). The `addToCodeBody()` function is important since not everything 
can be added in-line and this function helps split the code into multiple 
lines.

---

### Appendix A - `evaluate()` and `analyticalIntegral()` updates tutorial

> Besides the `evaluate()` function, this tutorial illustrates how the 
`analyticalIntegral()` can be updated. This is a more advanced effort that is 
highly dependent on the class that is being transformed for AD support, but 
will be necessary in specific instances.

Let's consider a fictional class RooFoo, that performs some arbitrary 
mathematical operations called 'Foo' (as seen in doFoo() function below).

> Note that doFoo is a simplified example, in many cases the mathematical 
operations are not limited to a single function, so they need to be spotted 
within the `evaluate()` function.

```C++
#include <iostream>
#include <string>
#include <limits>

class RooFoo : public RooAbsReal {
    int a;
    int b;
    int doFoo() { return a* b + a + b; }
    int integralFoo() { return /* whatever */;}
    public: 
    // Other functions...
    double evaluate() override { 
        // Do some bookkeeping
        return doFoo(); 
    }; 
    double analyticalIntegral(Int_t code, const char* rangeName) override {
        // Select the right paths for integration using codes or whatever.
        return integralFoo();
    }
};
```

> Note that all RooFit classes are deriving from the RooAbsReal object, but 
its details are not relevant to the current example. 

Note how the `evaluate()` function overrides the `RooAbsReal` for the RooFoo 
class. Similarly, the `analyticalIntegral()` function has also been overridden
 from the `RooAbsReal` class. These two functions can typically be found in a 
RooFit class (otherwise, you will have to make your own).

The `evaluate()` function includes some bookkeeping steps (commented out in 
above example) that are not relevant to AD. The important part is that it 
calls a specific function (doFoo() in this example), and returns the results. 

Similarly, the `analyticalIntegral()` function calls a specific function (
`integralFoo()` in this example), and returns the results. It may also include
 some code that may need to be looked at, but for simplicity, its contents are
 commented out in this example.

#### Adding AD Support Example 1 (RooFoo)

Besides creating the `translate()` function, the 
`buildCallToAnalyticIntegral()` function also needs to be added (to help call 
`analyticalIntegral()`).

Before creating the translate() function, move the mathematical logic (
`doFoo()` function in this example) out of the source class (RooFoo in this 
example) and into a separate header file called `EvaluateFuncs.h`. Also note 
that the parameters a and b have been defined as inputs, instead of them just 
being class members.

```C++
///// The EvaluateFuncs.h file 
int doFoo(int a, int b) { return a* b + a + b; }
```

> Directory path: [roofit/roofitcore/inc/RooFit/Detail/EvaluateFuncs.h](https://github.com/root-project/root/blob/4e8c577dfd6a19d7c38a74e3074b406a598bf76a/roofit/roofitcore/inc/RooFit/Detail/EvaluateFuncs.h)

So now that the `doFoo()` function exists in the `EvaluateFuncs` namespace, we
 need to comment out its original function definition in the RooFoo class and 
also add the namespace `EvaluateFuncs` to wherever `doFoo()` it is referenced 
(and also define input parameters for it). 

```C++
class RooFoo : public RooAbsReal {
    ...
    // int doFoo() { return a* b + a + b; }
    
    double evaluate() override { 
        ...
        return EvaluateFuncs::doFoo(a, b); 
    };
 ```

Similarly, update the translate function. Most translate functions include a 
`buildCall()` function, that includes the fully qualified name (including 
'EvaluateFuncs') of the function to be called along with the input parameters 
as they appear in the function (a,b in the following example).

Also, each `translate()` function requires the `addResult()` function. It will add 
whatever is represented on the right hand side to the result (saved in the `res` 
variable in the following example) of this class, which can then be propagated in 
the rest of the compute graph. 

 ```C++
     void translate(RooFit::Detail::CodeSquashContext &ctx) const override {
            std::string res = ctx.buildCall("EvaluateFuncs::doFoo", a, b);
            ctx.addResult(this, res);
    }

```

#### Adding AD Support Example 2 (RooGaussian)

Here again, we see that the original `evaluate()` function in RooGaussian was 
replaced with a reference to the one in `EvaluateFuncs` file, along with 
relevant input parameters.

```C++
double RooGaussian::evaluate() const
{
   return RooFit::Detail::EvaluateFuncs::gaussianEvaluate(x, mean, sigma);
}
```
Source: [roofit/roofit/src/RooGaussian.cxx](https://github.com/root-project/root/blob/4e8c577dfd6a19d7c38a74e3074b406a598bf76a/roofit/roofit/src/RooGaussian.cxx)

While the original `evaluate()` function is moved the to the `EvaluateFuncs` 
file.

```C++
/// @brief Function to evaluate an un-normalized RooGaussian.
inline double gaussianEvaluate(double x, double mean, double sigma)
{
   const double arg = x - mean;
   const double sig = sigma;
   return std::exp(-0.5 * arg * arg / (sig * sig));
}
```

Source: [roofit/roofitcore/inc/RooFit/Detail/EvaluateFuncs.h](https://github.com/root-project/root/blob/4e8c577dfd6a19d7c38a74e3074b406a598bf76a/roofit/roofitcore/inc/RooFit/Detail/EvaluateFuncs.h)

Remember that the `translate()` function needs to call the `evaluate()` function. So, 
similar to the Example 1 above, the `translate()` function is built using the fully 
qualified name of the 'gaussianEvaluate' and its input parameters (x, mean, sigma), 
while using the `addResult()` and `buildCall()` functions.

```C++

void RooGaussian::translate(RooFit::Detail::CodeSquashContext &ctx) const
{
   // Build a call to the stateless gaussian defined later.
   ctx.addResult(this, ctx.buildCall("RooFit::Detail::EvaluateFuncs::gaussianEvaluate", x, mean, sigma));
}
```

#### buildCallToAnalyticIntegral()

When `analyticalIntegral()` is found in your class, then you can implement the
 `buildCallToAnalyticIntegral()` function. Depending on the code, you can call
 one or more integral functions using the `code` parameter. Our RooFoo example
 above only contains one integral function (`integralFoo()`).

Similar to `doFoo()`, comment out `integralFoo()' in the original file and 
move it to a separate file called 'AnalyticalIntegrals.h' (this is not the 
same file as the `EvaluateFuncs.h`, but it works in a similar manner). 

As with `doFoo()`. add the relevant inputs (a,b) as parameters, instead of 
just class members.

```C++
///// The AnalyticalIntegrals.h file
int integralFoo(int a, int b) { return /* whatever */;}
```

> Directory path: [hist/hist/src/AnalyticalIntegrals.h](https://github.com/root-project/root/blob/ee996658194dc8dca32551b1e2df34f3250fae9a/hist/hist/src/AnalyticalIntegrals.h)

Next, in the original RooFoo class, update all references to the 
`integralFoo()` function with its new fully qualified path (
`EvaluateFunc::integralFoo`) and include the input parameters as well (
`EvaluateFunc::integralFoo(a, b)`).

```C++
    double analyticalIntegral(Int_t code, const char* rangeName) override {
        // Select the right paths for integration using codes or whatever.
        return EvaluateFunc::integralFoo(a, b);
    }
```

Next, in the `buildCallToAnalyticIntegral()` function, simply return the 
output using the `buildCall()` function.

```C++
    std::string
    buildCallToAnalyticIntegral(Int_t code, const char *rangeName, RooFit::Detail::CodeSquashContext &ctx) const override {
        return ctx.buildCall("EvaluateFunc::integralFoo", a, b);
    }
```

> Note that implementation of the `buildCallToAnalyticIntegral()` function is 
quiet similar to the `translate()` function, except that in `translate()`, you 
have to add to the result (using `addResult()`), while for 
`buildCallToAnalyticIntegral()`, you only have to return the string (using 
`buildCall()`).

**Consolidated Code changes in RooFoo example**

Final RooFoo code:

```C++
#include <iostream>
#include <string>
#include <limits>

class RooFoo : public RooAbsReal {
    int a;
    int b;
    // int doFoo() { return a* b + a + b; }
    // int integralFoo() { return /* whatever */;}
    public: 
    // Other functions...
    double evaluate() override { 
        // Do some bookkeeping
        return EvaluateFunc::doFoo(a, b); 
    }; 
    double analyticalIntegral(Int_t code, const char* rangeName) override {
        // Select the right paths for integration using codes or whatever.
        return EvaluateFunc::integralFoo(a, b);
    }

    //// ************************** functions for AD Support ***********************
    void translate(RooFit::Detail::CodeSquashContext &ctx) const override {
        std::string res = ctx.buildCall("EvaluateFunc::doFoo", a, b);
        ctx.addResult(this, res);
    }

    std::string
    buildCallToAnalyticIntegral(Int_t code, const char *rangeName, RooFit::Detail::CodeSquashContext &ctx) const override {
        return ctx.buildCall("EvaluateFunc::integralFoo", a, b);
    }
    //// ************************** functions for AD Support ***********************
};

```

Mathematical code moved to `EvaluateFuncs.h` file.

```C++
int doFoo(int a, int b) { return a* b + a + b; }
```

Integrals moved to the 'AnalyticalIntegrals.h' file.

```C++
int integralFoo(int a, int b) { return /* whatever */;}
```

> Remember, as long as your code is supported by Clad (e.g., meaning there are
 custom derivatives defined for all external Math library functions used in 
your code), it should work for AD support efforts. Please view Clad 
documentation for more details.

### Appendix B - What could go wrong (FAQs)

#### Will my analyticalIntegral() function support AD?

Both scenarios are possible:

1 - where `analyticalIntegral()` will be able to support AD

2 - where `analyticalIntegral()` will *not* be able to support AD

This requires further research.

#### What if my evaluate() function cannot support AD?

In some cases. the `evaluate()` function is written in a piece-wise format 
(multiple evaluations based on multiple chunks of code). You can review the 
`EvaluateFuncs.h` file to find AD support for several piece-wise (`if code==1 
{...} else if code==2 {...}` ) code snippets. 

However, there may still be some cases where AD support may not be possible 
due to the way that `evaluate()` function works in that instance.

#### What if my evaluate() function depends heavily on caching?

For simple caching, the caching logic can be separated from the 
mathematical code that is being moved to `EvaluateFuncs.h`, so that it can 
retained in the original file. 

For more complicated scenarios, the `code` variable can be used to identify 
use cases (parts of the mathematical code in `evaluate()`) that should be 
supported, while other parts that are explicitly not be supported (e.g., using
 `if code==1 {...} else if code==2 {...}`).

#### Can classes using Numerical Integration support AD?

So far, no. This needs further exploration. Hint: classes using Numerical 
Integration can be identified with the absence of the `analyticalIntegral()` 
function.

#### Why is my code falling back to Numeric Differentiation?

If you call in to an external Math library, and you use a function that has a 
customized variant with an already defined custom derivative, then you may see
 a warning like "falling back to Numeric Differentiation". In most such cases,
 your derivative should still work, since Numeric Differentiation is already 
well-tested in Clad.

To handle this, either define a custom derivative for that external function, 
or find a way to expose it to Clad.

An example of this can be seen with `gamma_cdf()` in AnalyticalIntegrals.h`, 
for which the custom derivative is not supported, but in this specific 
instance, it falls back to Numeric Differentiation and works fine, since `
gamma_cdf()` doesn't have a lot of parameters. 

> In such cases, Numeric Differentiation fallback is only used for that 
specific function. In above example, `gamma_cdf()` falls back to Numeric 
Differentiation but other functions in `AnalyticalIntegrals.h` will still be 
able to use AD. This is because Clad is going to assume that you have a 
derivative for this `gamma_cdf()` function, and the remaining functions will 
use AD as expected. In the end, the remaining functions (including 
`gamma_cdf()`) will try to fall back to Numeric Differentiation.

However, if you want to add pure AD support, you need to make sure that all 
your external functions are supported by Clad (meaning there is a custom 
derivative defined for each of them).

#### How do I test my new class while adding AD support?

Please look at the test classes that test the derivatives, evaluates, 
fixtures, etc. (defined in 'roofit/roofitcore/test'). You can clone and adapt 
these tests to your class as needed. For example:

> [roofit/roofitcore/test/testRooFuncWrapper.cxx](https://github.com/root-project/root/blob/ee996658194dc8dca32551b1e2df34f3250fae9a/roofit/roofitcore/test/testRooFuncWrapper.cxx)

> Tip: Tests like above can be referenced to see which parts of RooFit already
 support AD.

#### How do I control my compile time?

This is an area of research that still needs some work. In most cases, the 
compile times are reasonable, but with an increase in the level of complexity,
 higher compile times may be encountered.

### Appendix C - Helper functions discussed in this document

- **addResult()**: It may include a function call, an expression, or something
 more complicated. For a specific class, it will add whatever is represented 
on the right hand side to the result of this class, which can then be 
propagated in the rest of the compute graph. For each of the `translate()` 
functions, it is important to call `addResult()` since this is what enables 
the squashing to happen.

- **addToCodeBody()**: It helps add things to the body of the C++ function 
that you're creating. It takes whatever string is computed in its arguments 
and adds it to the overall function string (which will later be just-in-time 
compiled). The `addToCodeBody()` function is important since not everything 
can be added in-line and this function helps split the code into multiple 
lines.

- **addToGlobalScope()**: It helps declare and initialize the results 
variable, so that it can be available globally (throughout the function body).
 For local variables, the `addToCodeBody()` function can be used to keep the 
variables in the respective scope (for example, within a loop).

- **assembleCode()**: combines the generated code statements into the final 
code body of the squashed function.

- **beginLoop()**: It helps build the start and the end of a For loop for your
 class. Simply place this function in the scope and place the contents of the 
For loop below this statement. The code squashing task will automatically 
build a loop around the statements that follow it. There's no need to worry 
about the index of these loops, because they get propagated. For example, if 
you want to iterate over a vector of RooFit objects using a loop, you don't 
have to think about indexing them properly because the `beginLoop()` function 
takes care of that. Simply call this function, place your function call in a 
scope and after the scope ends, the loop will also end.

- **buildArg()**: helps convert RooFit objects into arrays or other C++ 
representations for efficient computation.

- **buildCall()**: Helps build a function call. Requires the fully qualified 
name of the function. When this external `buildCall()` function is called, 
internally, the `getResult()` function is called on the input RooFit objects 
(e.g., x, mean, sigma). That's the only way to propagate these upwards into 
the compute graph.

- **buildCode()**: generates the optimized code for evaluating the function 
and its derivatives. 

- **declareAndDiffFunction()**: declare the function and create its 
derivative.

- **dumpCode()**: prints the squashed code body to console(useful for 
debugging).

- **dumpGradient()**: prints the derivative code body to console (useful for 
debugging).

- **evaluate()**: All RooFit classes implement this function. It performs a 
lot of book-keeping, caching, etc. that is required for RooFit (but not 
necessarily for AD). 

- **getResult()**: It helps lookup the result of a child node (the string that
 the child node previously saved in a variable using the `addResult()` 
function). 

- **gradient()**: calculates the gradient of the function with respect to its
 parameters.

- **loadParamsAndData()** extracts parameters and observables from the 
provided data and prepares them for evaluation. 

- **makeValidVarName()**: It helps get a valid name from the name of the 
respective RooFit class. It then helps save it to the variable that represents
 the result of this class (the squashed code/ C++ function that will be 
created). 

- **translate()**: All RooFit classes that should support AD need to use this 
function. It creates a string of code, which is then just-in-time compiled 
using Cling (C++ interpreter for ROOT). For each of the `translate()` 
functions, it is important to call `addResult()` since this is what enables 
the squashing to happen.
