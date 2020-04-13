---
title: "Extending OpenMP to Facilitate Loop Optimization"
collection: publications
permalink: /publication/2018-01-01-Extending-OpenMP-to-Facilitate-Loop-Optimization
date: 2018-01-01
venue: 'In the proceedings of Evolving OpenMP for Evolving Architectures'
paperurl: 'http://link.springer.com/10.1007/978-3-319-98521-3_4'
citation: ' Ian Bertolacci,  Michelle Strout,  Bronis Supinski,  Thomas Scogland,  Eddie Davis,  Catherine Olschanowsky, &quot;Extending OpenMP to Facilitate Loop Optimization.&quot; In the proceedings of Evolving OpenMP for Evolving Architectures, 2018.'
---

# Abstract
OpenMP provides several mechanisms to specify parallel source-code transformations.
Unfortunately, many compilers perform these transformations early in the translation process, often before performing traditional sequential optimizations, which can limit the effectiveness of those optimizations.
Further, OpenMP semantics preclude performing those transformations in some cases prior to the parallel transformations, which can limit overall application performance.

In this paper, we propose extensions to OpenMP that require the application of traditional sequential loop optimizations.
These extensions can be specified to apply before, as well as after, other OpenMP loop transformations.
We discuss limitations implied by existing OpenMP constructs as well as some previously proposed (parallel) extensions to OpenMP that could benefit from constructs that explicitly apply sequential loop optimizations.
We present results that explore how these capabilities can lead to as much as a 20% improvement in parallel loop performance by applying common sequential loop optimizations.


[Access paper here](http://link.springer.com/10.1007/978-3-319-98521-3_4){:target="_blank"}
