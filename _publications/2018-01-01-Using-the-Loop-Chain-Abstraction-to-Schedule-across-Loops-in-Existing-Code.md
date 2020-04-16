---
title: "Using the Loop Chain Abstraction to Schedule across Loops in Existing Code"
collection: publications
permalink: /publication/2018-01-01-Using-the-Loop-Chain-Abstraction-to-Schedule-across-Loops-in-Existing-Code
date: 2018-01-01
venue: 'International Journal of High Performance Computing and Networking'
paperurl: 'https://www.inderscienceonline.com/doi/abs/10.1504/IJHPCN.2019.097053'
citation: ' Ian Bertolacci,  Michelle Strout,  Jordan Riley,  Stephen Guzik,  Eddie Davis,  Catherine Olschanowsky, &quot;Using the Loop Chain Abstraction to Schedule across Loops in Existing Code.&quot; International Journal of High Performance Computing and Networking, 2018.'
---
# Abstract
Exposing opportunities for parallelisation while explicitly managing data locality is the primary challenge to porting and optimising computational science simulation codes to improve performance.
OpenMP provides mechanisms for expressing parallelism, but it remains the programmer's responsibility to group computations to improve data locality.
The loop chain abstraction, where a summary of data access patterns is included as pragmas associated with parallel loops, provides compilers with sufficient information to automate the parallelism versus data locality trade-off.
We present the syntax and semantics of loop chain pragmas for indicating information about loops belonging to the loop chain and specification of a high-level schedule for the loop chain.
We show example usage of the pragmas, detail attempts to automate the transformation of a legacy scientific code written with specific language constraints to loop chain codes, describe the compiler implementation for loop chain pragmas, and exhibit performance results for a computational fluid dynamics benchmark.

[Access paper here](https://www.inderscienceonline.com/doi/abs/10.1504/IJHPCN.2019.097053){:target="_blank"}
