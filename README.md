# JUPITER Benchmark Suite: ParFlow

[![DOI](https://zenodo.org/badge/831431317.svg)](https://zenodo.org/badge/latestdoi/831431317) [![Static Badge](https://img.shields.io/badge/DOI%20(Suite)-10.5281%2Fzenodo.12737073-blue)](https://zenodo.org/badge/latestdoi/764615316)

This benchmark is part of the [JUPITER Benchmark Suite](https://github.com/FZJ-JSC/jubench). See the repository of the suite for some general remarks.

This repository contains the ParFlow benchmark. [`DESCRIPTION.md`](DESCRIPTION.md) contains details for compilation, execution, and evaluation.  
This workflow will execute [ParFlow](https://parflow.org) on GPUs to simulate an idealised infiltration problem into a 3D domain with homogeneous soil. It is the similar test problem as reported by [Hokkanen et al, 2021](https://doi.org/10.1007/s10596-021-10051-4).

A JUBE workflow is included which will clone the code, build it and run the benchmark.

The source code of ParFlow is included in the `./src/` subdirectory as a submodule from the upstream ParFlow repository at [github.com/parflow/parflow](https://github.com/parflow/parflow). The [support for the Rapids Memory Manager](https://github.com/hokkanen/rmm) is also included.