## 1. Garima Singh

Garima was a software engineering student at Manipal Institute of Technology in Manipal, Karnataka, India when she started her first project with the compiler research organization. She has since made several contributions to compiler research, delivering 2 papers and presenting at four major conferences as oral presentations. She is currently (as of this writing) an MSc student at ETH Zurich.

Github Profile: [grimmmyshini](https://github.com/grimmmyshini)

### Major Contributions

#### Error Estimation Framework

Garima's interest in error estimation led to her exploration of floating point error estimation in High Performance Computing (HPC) applications. This also led to her research on its applications in High Energy Physics (HEP).

For more details, please see the [Error Estimation Framework] document.

#### Automatic Differentiation with C++ and Clad

Garima explored the the internal workings of the Clang compiler and applications of its Clad [^1] plugin, specifically for Automatic Differentiation. She issued several patches and bug fixes to these projects.

#### Enabling Clad at scale in HEP through RooFit

During her research, Garima came across the challenges faced by users when performing large-scale data analysis. She saw the potential of current state-of-the-art techniques to make analysis faster and more efficient. She also discovered that even small improvements to the workflow had significant impact on performance, when working at such a large scale. This work was particularly rewarding for Garima since she was able to delve deep into the theoretical properties of AD during this research.

[^1]: Clad is a source transformation Automatic Differentiation (AD) library for C++, implemented as a plugin for the Clang compiler.

[Error Estimation Framework]: https://compiler-research.org/tutorials/fp_error_estimation_clad_tutorial/

### How leading companies can utilize this technology

For the error estimation work, companies can use this technology to not only determine any numerical instabilities in their code, but also to perform approximate computing techniques to make their software more efficient.

Clad has a lot of practical applications, especially in companies or organizations with scientific software. Garima's work has demonstrated how Clad can be used with sufficiently complex codebases such as ROOT.

Opensource/ scientific software may benefit more from this work, as compared to the mainstream technology companies, since it is geared more towards exploratory science than consumer products.


### Why new programmers would want to learn about these features

Error estimation work is is an exciting, new research area and can also be used on smaller-scale projects. The customizable nature of the tool allows researchers to create new analyses and share them with others in the form of papers or other scientific contributions. 

On the RooFit-AD side, as RooFIt grows and more classes change and evolve, the underlying AD framework also needs to evolve. It is safe to assume that users of RooFit would be interested in following up on this work to keep using AD with their statistical models.

#### Soft skills and community outreach

The Compiler Research team attributes the community-wide acceptance of Automatic Differentiation to Garima's work in demonstrating its usefulness in large codebases such as that of the Root Framework [^2], specifically in its RooFit [^3] classes.

