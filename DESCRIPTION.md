# ParFlow benchmark

## Purpose
[ParFlow](https://parflow.org/) is a massively parallel, physics-based integrated watershed model. It simulates fully coupled dynamic 2D/3D hydrological, groundwater and land-surface processes for large scale problems. Saturated and variably saturated subsurface flow in heterogeneous porous media are simulated in three spatial dimensions. The code employs a discretization of the three dimensional surface-subsurface flow which allows for hillslope runoff and channel routing (Kollet and Maxwell, 2006).

ParFlow was originally developed at Lawrence Livermore National Lab (LLNL) in California (Ashby and Falgout, 1996). Parallelism in ParFlow is implemented via the MPI2 protocol. From version 3.7, an embedded Domain Specific Language approach was implemented to enable a CUDA backend ([Hokkanen et al., 2021](https://doi.org/10.1007/s10596-021-10051-4
)). ParFlow parallel I/O is via tasklocal and shared files in a custom, open-source format. 


## Getting the source code and building
ParFlow is available at [https://github.com/parflow/parflow](https://github.com/parflow/parflow). 

ParFlow can be built for both CPU and GPU usage. The benchmark is only intended to for GPUs.
ParFlow has a few dependencies (see the [ParFlow documentation](https://github.com/parflow/parflow/blob/master/README.md) for details), not all of which are required in this benchmark.
For this benchmark the following dependencies are required: GCC, OpenMPI, CUDA, CMake, TCL, Hypre, NetCDF (module switch Stages/2023 Stages/2022; ml GCC OpenMPI CUDA UCX-settings/RC-CUDA CMake Tcl Hypre netCDF). The Rapids Memory Manager is also a dependency, and should be cloned from [RMM](https://github.com/hokkanen/rmm.git).

Here is a recipe to build ParFlow

```shell
mkdir parflow_benchmark
cd parflow_benchmark
export PFDIR=$(pwd)
export PARFLOW_DIR=$PFDIR/install

git clone -b branch-0.10 --single-branch --recurse-submodules https://github.com/hokkanen/rmm.git rmm

export RMM_ROOT=$PFDIR/rmm
mkdir rmm/build/
cd rmm/build/
cmake .. -DCMAKE_INSTALL_PREFIX=$PFDIR/rmm
make 
make install

cd $PFDIR
git clone https://github.com/parflow/parflow.git 
cd $PFDIR/parflow

rm -rf $PFDIR/build/
mkdir $PFDIR/build/
cd $PFDIR/build/
CC=mpicc CXX=mpicxx FC=mpif90 cmake ../parflow \
	-DMPIEXEC_EXECUTABLE=$(which srun) \
       	-DPARFLOW_AMPS_LAYER=mpi1 \
	-DCMAKE_C_FLAGS='-fopenmp -Wall -Werror -Wno-unused-result -Wno-unused-function' \
	-DCMAKE_BUILD_TYPE=Release \
	-DPARFLOW_ENABLE_TIMING=TRUE \
	-DPARFLOW_HAVE_CLM=ON \
	-DCMAKE_INSTALL_PREFIX=${PARFLOW_DIR} \
	-DPARFLOW_ENABLE_NETCDF=TRUE \
	-DNETCDF_DIR=$EBROOTNETCDF \
	-DTCL_TCLSH=${EBROOTTCL}/bin/tclsh8.6 \
	-DPARFLOW_AMPS_SEQUENTIAL_IO=on \
	-DPARFLOW_ENABLE_SLURM=TRUE \
	-DPARFLOW_ACCELERATOR_BACKEND=cuda \
	-DRMM_ROOT=$RMM_ROOT \
	-DCMAKE_CUDA_RUNTIME_LIBRARY=Shared
make -j 4
make install
```


The JUBE step `COMPILE` will fetch the source code and build ParFlow.

## Running the benchmark
The benchmarks is run in two steps. First, generate the necessary case setup files
```bash
tclsh ./clayLmod_strong.tcl 2 2 1 1008
```

and then run ParFlow
```bash
parflow clayLmod_strong
```

The JUBE step `RUN` will execute the benchmark for you. 

## JUBE execution
The JUBE workflow requires Python (aside from JUBE and ParFlow).

Run the JUBE workflow with
```
jube -vvv run jube_clayL_benchmark.xml 
```
If the compilation step was already executed by JUBE, and there is no need to repeat it, then you can skip it with
```
jube -vvv run jube_clayL.xml --tag=skip_compile
```

## Verification
ParFlow should run without errors and report `Problem solved`. There should be a number of output files, including the solution NetCDF file `*.00001.nc` and a timing file `*.out.timings.csv`. 

A further detailed verification of the solution can be achieved done by obtaining the reference solution with the `getReferenceSolution.sh` script which will download a reference solution, and can be compared using the  supplied Python scripts.

The `CHECK` step of the JUBE workflow will perform the detailed check.
 
## Results

Performance is reported in the `out.timing.csv` file.
The JUBE workflow will also read this file and create a summary table by invoking the analysis and result steps
```
jube analyse ./bench_run  
jube result ./bench_run  
```

## Baseline
Results were obtained on JUWELS Booster system at JSC. Each node provides 4 Nvidia A100 GPUs. 
| nodes | taskspernode | total_tasks | total_gpus | Total_Time [s] | Stable_solver [s] |
|-------|--------------|-------------|------------|------------|---------------|
| 1     | 4            | 4           | 4          | 732.78     | 265.88        |
| 4     | 4            | 16          | 16         | 319.96     | 105.08        |
| 9     | 4            | 36          | 36         | 294.02     | 71.98         |
| 16    | 4            | 64          | 64         | 328.92     | 63.61         |
