---
title: "Success Stories"
layout: gridlay
excerpt: "Student Success Stories"
sitemap: false
permalink: /carpe-diem/
---

## Success Stories

This page is dedicated to showcasing the notable achievements of our students and contributors. Under the guidance of mentors from the Compiler Research Group, these talented individuals have achieved remarkable success in open-source software development and research advancement. 

### 1. Jun Zhang

Jun is a 3rd year software engineering student at Anhui Normal University, 
WuHu, China. He was originally introduced to the Compiler Research team through 
Google Summer of Code. Since then, working on the Clang/LLVM infrastructure, he 
has contributed ~70 patches.

Github Profile: [junaire](https://github.com/junaire)

#### Handle Execution Results in Clang-REPL

Jun's major contributions are summed up in the following RFC:

- [Handle Execution Results in clang-repl](https://discourse.llvm.org/t/rfc-handle-execution-results-in-clang-repl/68493)

It builds on top of previous initiative to bring Cling-REPL functionality in 
the upstream Clang repo for more generalized applications.

Background: Cling is an interpretive technology that is based on Clang. 
Cling-REPL provides REPL (read–eval–print loop) functionality for interactive 
programming. Moving parts of Cling-REPL to upstream Clang repository in the 
Clang-REPL component helps push the advancements achieved in Cling-REPL to a 
wider audience.

Following are the relevant patches:

- [Add a new annotation token: annot_repl_input_end](https://reviews.llvm.org/D148997)

- [Introduce Value to capture expression results](https://reviews.llvm.org/D141215)

- [Implement Value pretty printing](https://reviews.llvm.org/D146809)

A brief explanation of these changes can be found here:

- [Execution Results Handling](Execution_Results.md)

#### Technical debt alleviation

Besides making useful contributions to help the High Energy Physics (HEP) 
field, he was also able to land critical patches to the upstream LLVM 
repositories, that will reduce the technical debt and help Compiler Research 
Org adapt the upcoming LLVM versions more seamlessly. Some examples can be
seen [here](https://reviews.llvm.org/D131241) 
and [here](https://reviews.llvm.org/D130831).

#### How leading companies can utilize this technology

Clang-REPL/Cling are most commonly used by engineers at CERN for High Energy 
Physics (HEP) computations, but wider applications are possible for different 
research and data science areas, especially when **fast prototyping** is a 
major requirement. 

These features have opened up a new paradigm of interactive possibilities in 
C++ programming and as it gains traction, more users are expected to adopt this
 approach to achieve better performance.

#### Why new programmers would want to learn about these features

REPL is a beginner-friendly approach to programming. Runtime value capture and 
pretty printing features are essential parts of REPL interactivity. Programmers
 who use Clang-REPL command line are very likely to use these features to help 
them to code and debug.

Runtime value capture **connects the Compiled Code & Interpreted Code**, which 
makes it easier for Clang-REPL enable C++ interoperability with other 
user-firendly languages like Python.

> Clang-REPL is a general-purpose adaptation of Cling (which is more 
HEP-centric). Anyone who wants to learn about about interactive features of 
Clang-REPL will also benefit from learning about how Cling works and how it 
evolved over time.

#### Soft skills and community outreach

One of the areas of interest for Jun was 'how to demonstrate my work to 
others'. Besides accomplishing the technical aspects of each project, a major 
personal achievement for Jun was pitching these ideas to the broader LLVM 
community (e.g., in LLVM Developers Meeting, Request for Comment (RFC) forum 
discussions, exhaustive code reviews, etc.) and demonstrating to them that 
these ideas would add value to their overall codebase. 

Effectively communicating technical direction to a diverse audiences from 
different cultures, while speaking in English as a second language was also 
a valueable experience.

Intellectual discourse with, and acceptance from, the veterans of Clang project
 provided Jun with valuable insights into the inner workings of 
open source collaboration.

Finally, Jun's practive approach in engaging reviewers and mentors was a great 
asset in pushing through these major changes in a large, remote, and 
decentralized open source project such as LLVM, where it is hard for a new 
comer to establish themselves as an expert (as opposed to brick-and-mortar 
companies, where it is easy to identify star performers).
