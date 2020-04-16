---
title: "Identifying and Scheduling Loop Chains Using Directives"
collection: publications
permalink: /publication/2016-01-01-Identifying-and-Scheduling-Loop-Chains-Using-Directives
date: 2016-01-01
venue: 'In the proceedings of 2016 Third Workshop on Accelerator Programming Using Directives (WACCPD)'
paperurl: 'https://ieeexplore.ieee.org/abstract/document/7836581/'
citation: ' I. Bertolacci,  M. Strout,  S. Guzik,  J. Riley,  C. Olschanowsky, &quot;Identifying and Scheduling Loop Chains Using Directives.&quot; In the proceedings of 2016 Third Workshop on Accelerator Programming Using Directives (WACCPD), 2016.'
---
# Abstract
Exposing opportunities for parallelization while explicitly managing data locality is the primary challenge to porting and optimizing existing computational science simulation codes to improve performance and accuracy.
OpenMP provides many mechanisms for expressing parallelism, but it primarily remains the programmer's responsibility to group computations to improve data locality.
The loopchain abstraction, where data access patterns are included with the specification of parallel loops, provides compilers with sufficient information to automate the parallelism versus data locality tradeoff.
In this paper, we present a loop chain pragma and an extension to the omp for to enable the specification of loop chains and high-level specifications of schedules on loop chains.
We show example usage of the extensions, describe their implementation, and show preliminary performance results for some simple examples.

[Access paper here](https://ieeexplore.ieee.org/abstract/document/7836581/){:target="_blank"}
