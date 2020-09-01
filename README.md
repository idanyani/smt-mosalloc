# Build and Run
```sh
$ cmake .
$ make
$ ctest -VV
$ ./runMosalloc.py -aps 2MB -as2 0 -ae2 2MB -bps 1200MB -bs1 40MB -be1 1064MB -bs2 20MB -be2 40MB -- <app>
```

# What is Mosalloc

The Mosaic Memory Allocator – Mosalloc – allows its users to back the memory of an application with some arbitrary predetermined heterogeneous collection of pages with different sizes (4KB, 2MB, and/or 1GB) on an x86 machine. All memory allocations of the application that links against Mosalloc are fulfilled by Mosalloc.
The allocator is primarily useful for architectural virtual memory studies. It allows researchers to construct mathematical models that predict the runtime of applications executing on top of some newly proposed virtual memory design. This methodology requires a partial simulation of only the said newly proposed memory subsystem, as opposed to a full "cycle accurate" simulation of the entire processor. The resulting mathematical models are much more accurate than preexisting
models.
For more information about Mosalloc, its usages, and motivation, please refer to the paper which was published in [MICRO '20](https://www.microarch.org/micro53/).

Mosalloc is implemented as a dynamic library and can be pre-loaded before glibc (using LD_PRELOAD environment variable) and hooks all memory requests made by an application. First, Mosalloc intercepts malloc() requests by hooking the morecore() function, which malloc() calls when it needs to extend the heap. Second, Mosalloc intercepts direct invocations of brk(), mmap() and munmap(), the primary memory system calls in Linux, by overriding their glibc wrapper functions.
Mosalloc is an independent library so it does not require modifying the existing source code or rebuilding the application. Additionally, Mosalloc is implemented in user-space and does not require kernel modification.
Mosalloc allows the user to back the address space of applications with arbitrary combinations of page sizes. To accomplish this, Mosalloc manages three pools that serve the three types of memory requests in Linux: (1) brk() calls, (2) anonymous mmap() calls, and (3) file-backed mmap() calls. The user should specify the layout of the brk() pool and the layout of the anonymous mmap() pool through a set of environment variables, which Mosalloc takes as an
input. The brk() and anonymous mmap() pools can mix pages of different sizes. Mosalloc allocates these pools via the mmap() system call with the relevant flags (MAP_HUGETLB, MAP_HUGE_2MB, and MAP_HUGE_1GB). The file-backed pool is backed only with 4KB pages because Linux does not support file-backed mmap() calls with hugepages; these mmap() calls are served from the page cache, which Linux manages only with 4KB pages. These three pools are allocated (dynamically) on-demand.

# Mosalloc Configuration
Mosalloc pools can be configured using environment variables. All of these environment variables are mandatory to run Mosalloc (users can use the runMosalloc script which configures these environment variables using a user friendly arguments). Here is a table summarizes these environment variables, their corresponding arguments in runMosalloc script, and their meaning:
environment variable | runMosalloc argument | description
-------------------- | -------------------- | --------------------
HPC_MMAP_POOL_SIZE | anon_pool_size (aps) | The anonymous mmap() pool size
HPC_MMAP_1GB_START_OFFSET | anon_start_1gb (as1) | The start offset of the 1GB hugepages region in the anonymous mmap() pool
HPC_MMAP_1GB_END_OFFSET | anon_end_1gb (ae1) | The end offset of the 1GB hugepages region in the anonymous mmap() pool
HPC_MMAP_2MB_START_OFFSET | anon_start_2mb (as2) | The start offset of the 2MB hugepages region in the anonymous mmap() pool
HPC_MMAP_2MB_END_OFFSET | anon_end_2mb (ae2) | The end offset of the 2MB hugepages region in the anonymous mmap() pool
HPC_BRK_POOL_SIZE | brk_pool_size (bps) | The anonymous brk() pool size
HPC_BRK_1GB_START_OFFSET | brk_start_1gb (bs1) | The start offset of the 1GB hugepages region in the brk() pool
HPC_BRK_1GB_END_OFFSET | brk_end_1gb (be1) | The end offset of the 1GB hugepages region in the brk() pool
HPC_BRK_2MB_START_OFFSET | brk_start_2mb (bs2) | The start offset of the 2MB hugepages region in the brk() pool
HPC_BRK_2MB_END_OFFSET | brk_end_2mb (be2) | The end offset of the 2MB hugepages region in the brk() pool
HPC_FILE_BACKED_POOL_SIZE | file_pool_size (fps) | The file-backed mmap() pool size
HPC_ANALYZE_HPBRS | analyze | Let Mosalloc analyzes the actual sizes of the three pools and write them to a separated file for each sub-process
HPC_MMAP_FIRST_FIT_LIST_SIZE | N/A (hardcoded to 1MB) | The size of the first-fit list which manages the anonymous mmap() allocations. The first-fit list is statically allocated with a predefined size to prevent an allocation recursive calls.
HPC_FILE_BACKED_FIRST_FIT_LIST_SIZE | N/A (hardcoded to 10KB) | The size of the first-fit list which manages the file-backed mmap() allocations.
runMosalloc script can be used to initialize these environment variables with a simple command line. For example, to run <app> with a 2MB anonymous mmap() pool which is allocated with only 2MB huge pages, a 1200MB anonymous mmap() pool with a 2MB region [20MB, 40MB) and additional 1GB region [40MB, 1064MB), and without file-backed mmap() pool (size=0) we can run the following command line:
```sh
$ ./runMosalloc.py -aps 2MB -as2 0 -ae2 2MB -bps 1200MB -bs1 40MB -be1 1064MB -bs2 20MB -be2 40MB -- <app>
```


# Future work
Mosalloc, currently, supports only one window/region of each hugepage size in each pool. As a future work, we will add support for multiple windows/regions of each hugepage size for the brk() and anonymous mmap() pools.
Finally, we will be happy to get contributions.
