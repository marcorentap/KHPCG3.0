# Prerequisites

You need:

- Valid install of trilinos with Kokkos and Tpetra packages enabled.
	(Tpetra requirement will be removed when KokkosCrsMatrix is moved out of Tpetra)
- CMake Version 3.0 or newer
- OpenMP (Not directly used but Kokkos can use this.)

# Usage

There are two CMake Keywords:

`KHPCG_EXECSPACE={OpenMP, Cuda, Serial (default)}` defines the execution space

`KHPCG_SYMGS={Color, Inexact, Level (default)}` defines the Gause-Seidel implementation (SymGS)

Example use:

```
mkdir build
cd build
cmake -DKHPCG_EXECSPACE=OpenMP ..
make -j$(nproc)
./KokkosHPCG.exe
```

The result should be in the `HPCG-Benchmark-3.0_*.yaml` file
