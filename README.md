# Compressed-Sparse-Blocks

This is a directly port from http://people.eecs.berkeley.edu/~aydin/csb/html/index.html.

Compressed Sparse Blocks (CSB) Library (Cilk Plus implementation)
Author
Aydın Buluç (in collaboration with Hasan Metin Aktulga, James Demmel, Jeremy Fineman, Matteo Frigo, John Gilbert, Charles Leiserson, Lenny Oliker, Sam Williams).
This material is based upon work supported by the National Science Foundation under Grants No. 0540248, 0615215, 0712243, 0822896, and 0709385, by MIT Lincoln Laboratory under contract 7000012980, and by the Department of Energy, Office of Science, ASCR Contract No. DE-AC02-05CH11231. Any opinions, findings and conclusions or recommendations expressed in this material are those of the author(s) and do not necessarily reflect the views of the National Science Foundation (NSF) and the Department of Energy (DOE). This software is released under the MIT license.

Introduction
The Compressed Sparse Blocks (CSB) is a storage format for sparse matrices that does not favor rows over columns (and vice-versa), hence offering performance symmetry in shared-memory parallel systems for Ax and A'x. The format is originally described in this paper [1]. It has been later improved through the incorporation of bitmasked register blocks in this paper [2] where an algorithm for symmetric matrices is also proposed. Finally this recent paper [3] includes performance results for the multiple vector cases.

This library targets shared-memory parallel systems (ideally in a single NUMA domain for best performance) and implements:

Sparse Matrix-Vector Multiplication (SpMV)
Sparse Matrix-Transpose-Vector Multiplication (SpMV_T)
Sparse Matrix-Multiple-Vector Multiplication (SpMM)
Sparse Matrix-Transpose-Multiple-Vector Multiplication (SpMM_T)
Download
Download the library and drivers as a tarball including the source code.

All operations can be done on an arbitrary semiring by overloading add() and multiply(), though some optimizations might not work for specialized semirings. While the code is implemented using Intel Cilk Plus (which is available in Intel Compilers and GCC), it can be ported to any concurrency platform that supports efficient task-stealing such as OpenMP and TBB.

The driver will accept matrices in text-based triples format and a binary format for faster benchmarking (created using this matlab script). The library also includes functions to convert from the common CSC format though the conversion is serial and not optimized for performance yet. An example input in (compressed) ascii and in (compressed) binary.
How to run it?
Read the example makefile. Here is a README file.
Running this code on a 8-core Intel processor is done by the following way (similar for other executables):

make parspmv/both_d/parspmvt (the tarball includes sample makefiles as well)
CILK_NWORKERS=8 ./both_d ../BinaryMatrices/asic_320k.bin nosym binary
If you have multiple sockets (NUMA domains) in your machine, then you need to constrain the memory space to a single NUMA node (CSB is not designed for multiple NUMA domains - it will run, but slower).

export CILK_NWORKERS=8 (or 16 if hyperthreading turns out to be beneficial)
numactl –cpunodebind=0 ./parspmvt ../BinaryMatrices/asic_320k.bin nosym binary
if you don't set CILK_NWORKERS, then it will run with as many hardware threads available on your machine (or numactl constrained domain).

./parspmv ../BinaryMatrices/kkt_power.bin nosym binary (using the binary format for fast I/O)
./parspmv ../TextMatrices/kkt_power.mtx nosym text (using the matrix market format)
./spmm_d$$number runs on $$number right-hand-side vectors that are randomly generated using double precision
./spmm_s$$number uses single precision for the same case
./both_d runs both parspmv and parspmv_t one after other (simulating iterative methods such as BiCG and QMR)
What do those file names mean?

BiCSB: Original CSB code with minor performance fixes, nonsymmetric and without register blocking. Quite robust
BmCSB: Bitmasked register blocks in action. Modify RBDIM in utility.h to try different blocking sizes (8x8, 4x4, etc). May perform better.
CSC: Serial CSC implementation. For reference only.
Release notes:

1.2: Current version. Multiple vector support.
A performance bug affecting A'x scaling on certain matrices is fixed.
1.1: Bitmasked register blocks, symmetric algorithm with half the bandwidth, port to Intel Cilk Plus.
A performance bug affecting Ax scaling on certain matrices is fixed.
Minor: A bug with the parspmvt test driver is fixed; new default parspmv compilation target for those who don't have SSE5.
1.0: Initial version. Support for Ax and A'x using cilk++.
Troubleshooting:

We recommend a recent (version 14) Intel compiler. It is free for non-commercial use on Linux
If your processor doesn't support AVX instructions, just remove the -xAVX command from the makefile.
Here is an expected output
Citation:

[1] Aydın Buluç, Jeremy T. Fineman, Matteo Frigo, John R. Gilbert, and Charles E. Leiserson. Parallel sparse matrix-vector and matrix-transpose-vector multiplication using compressed sparse blocks. In SPAA'09: Proceedings of the 21st Annual ACM Symposium on Parallel Algorithms and Architectures, 2009.
[2] Aydın Buluç, Samuel Williams, Leonid Oliker, and James Demmel. Reduced-bandwidth multithreaded algorithms for sparse matrix-vector multiplication. In Proceedings of the IPDPS. IEEE Computer Society, 2011
[3] H.Metin Aktulga, Aydın Buluç, Samuel Williams, and Chao Yang. Optimizing sparse matrix-multiple vectors multiplication for nuclear configuration interaction calculations. In Proceedings of the IPDPS. IEEE Computer Society, 2014
