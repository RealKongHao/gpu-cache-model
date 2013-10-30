

A GPU Cache Model
=============

Date: 30-Oct-2013

Author: Cedric Nugteren (http://www.cedricnugteren.nl)

This repository provides a GPU cache model by extending reuse distance theory. This is achieved by modelling: 1) the GPU's hierarchy of threads, warps, threadblocks, and sets of active threads, 2) conditional and non-uniform latencies, 3) cache associativity, 4) miss-status holding-registers, and 5) warp divergence. The model is implemented in C++. It uses the Ocelot GPU emulator to create unordered memory traces, which are input to the model.

The repository contains 4 major parts:
1. The C++ model itself in *src/model*.
2. A memory tracer for the Ocelot emulator in *src/tracer*.
3. A script to visualise the reuse distances in *src/visualiser*.
4. A script to parse profiled cache miss rates (for comparison) in *src/profiler*.


The C++ model
-------------

The model first allocates threads to warps, warps to threadblocks, and threadblocks to cores (see section 4.1 of the CUDA programming guide). Next, a memory coalescing model follows the definitions of section G.4.2 of the CUDA programming guide.

The actual reuse distance theory uses the already available computational and memory efficient implementations for sequential processors. A naive implementation of a reuse distance stack has a computational complexity of O(NM), in which N is the trace length (the total number of memory accesses) and M the number of unique accesses. This model uses a more computationally efficient version: a binary-tree C++ implementation of Bennett and Kruskal's algorithm. This implementation has a computational complexity of O(N*log(N)). Associativity is modelled by creating a binary-tree for each set in the cache.

To reduce the overall complexity and execution time of the model, the amount of threads is limited. Two abstractions are made: 1) a limited amount of cores is modelled and the results are generalised across all cores, and 2) a limited number of threads is modelled. These core and thread counts are configurable parameters. Per default, they are set to a single core with 8192 threads at most.

More information on reuse distance theory:
	G. Almasi, C. Cascaval, and D. Padua
	Calculating Stack Distances Efficiently
	MSP'02: Memory System Performance


The Ocelot tracer
-------------

The Ocelot GPU emulator is used to produce memory access traces for CUDA kernels. A custom tracer is implemented on top of Ocelot, creating a trace containing for each access: 1) the thread ID, 2) whether it is a read or a write, 3) the memory address, and 4) the size of the memory access.

More information on Ocelot:
	G. Diamos, A. Kerr, S. Yalamanchili, and N. Clark
	Ocelot: A Dynamic Optimization Framework for Bulk-Synchronous Applications in Heterogeneous Systems
	PACT '10: International Conference on Parallel Architectures and Compilation Techniques


The visualisation script
-------------

An R script is provided to generate histograms to visualise the reuse distance profiles generated by the models. In the histograms, it is also shown what the cache hit/miss cut-off distance is.


The hardware profiler (for verification)
-------------

A verification method based on hardware counters is also included. NVIDIA's profiler can be used to output the measured number of cache-line hits and misses in the L1 data cache. These results are automatically compared to the model's result by several scripts.


Usage
=============

*	Compile the model:

		make build

	Compiles the GPU cache model.

* Run the model:

		make run NAME='example'

	This compiles and runs the GPU cache model for a benchmark named *example*. This assumes there is a folder with the name *example* in the subdirectory *output*, containing trace files. The trace files can be generated using the Ocelot tracer.

* Run the Ocelot tracer:

		make trace NAME='example' DIR='examples/example_dir/'

	This compiles a CUDA program with the Ocelot tracer and executes it to generate a trace. It assumes there is single CUDA source-file *examples/example_dir/example.cu*. This generates output traces (.trc) in the *output/example* folder, one for each kernel in the CUDA program.

* Run the profiler to generate verification data:

		make verify NAME='example' DIR='examples/example_dir/'

	This compiles a CUDA program and executes it with nvprof to generate profiling data. It assumes there is single CUDA source-file *examples/example_dir/example.cu*. This generates output files (.prof) in the *output/example* folder, one for each kernel in the CUDA program. If such files are created, the cache model will report automatically on the results.

* Run the visualisation script:

		make histogram NAME='example'

	This creates a reuse distance plot in histogram form from the data generated by the model.

* Change the cache configuration:

		make go16
		make go48

	Changes the default cache configuration to Fermi's 16KB or 48KB cache configuration. Of course, this can also be done manually by changing *configurations/default.conf*.

###################################################