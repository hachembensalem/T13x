<img src="/Logo/Klessydra_Logo.png" width="400">

# Merging T13x User Guide

Intro: The Klessydra processing core family is a set of processors featuring full compliance with the RISC-V, and pin-to-pin compatible with the PULPino riscy cores. Klessydra cores fully support the RV32IM Base Integer Instruction set, and one instruction from the RV32A extension. 'T1' further extends the instruction set with a set of vector instructions as described above. The only privilege level supported in klessydra is Machine mode "M".

This guide explains how one can download and install Pulpino, and it's 
modified version of the riscv-gnu toolchain. It also demonstrates
how to patch the offcial riscv-toolchain in order to add the klessydra 
extensions. And then it shows how you can easily merge the Klessydra-Core 
in the Pulpino project.

###########################################################################################
- Prerequisites as indicated by the pulpino group
	- ModelSim in reasonably recent version (we tested it with versions 10.2c)
	- CMake >= 2.8.0, versions greater than 3.1.0 recommended due to support for ninja
	- riscv-toolchain, there are two choices for getting the toolchain: 

  		1) RECOMENDED OPTION: Use the custom version of the RISC-V toolchain from ETH. 
  		The ETH versions supports all the ISA extensions that were incorporated 
	  	into the RI5CY core as well as the reduced base instruction set for zero-riscy.
	        " https://github.com/pulp-platform/ri5cy_gnu_toolchain.git "

		2) Or download the official RISC-V toolchain supported by Berkeley.
 	       	" https://github.com/riscv/riscv-gnu-toolchain "


	  	Please make sure you are using the newlib version of the toolchain.
	- python2 >= 2.6
	
###########################################################################################

- IF you already have pulpino and their own version of the riscv-toolchain, then skip ahead to step.4


PROCEDURE:
1.	Install the following packeges:
		
		sudo apt-get install git cmake python-yaml tcsh autoconf automake autotools-dev curl libmpc-dev libmpfr-dev libgmp-dev gawk build-essential bison flex texinfo gperf libtool patchutils bc zlib1g-dev libexpat-dev

2.	Download and build the "ri5cy_gnu_toolchain"

		a) git clone https://github.com/pulp-platform/ri5cy_gnu_toolchain.git
		
		b) cd ri5cy_gnu_toolchain
		
		c) make ZERORISCY=1

		d) in case you need to build for RISCY cores, then just do "make" instead, and then add the symbolic links as shown from step 4.
		
	When the build is done, add the path **_<path_to_toolchain>/ri5cy_gnu_toolchain/install/bin_** to the environmental variables

3.	Download the PULPino suite:

		a) git clone https://github.com/pulp-platform/pulpino.git
		
		b) cd pulpino
		
		c) ./update-ips.py	


4.	If you want to run the klessydra specific tests, you have to download and patch the official riscv-toolchain, and then build it. Instructions for doing so are included in the README.md file
	inside the folder called toolchain_files.

5.	To merge the Klessydra core, and tests:

		a) git clone https://github.com/klessydra/T13x.git
		
		b) cd T13x
		
		c) ./runMErge.sh <pulpino_path>

6.	OPTIONAL: After merging is done, this is how you will be able to test Klessydra-t1-3th.
		-Open a terminal and navigate to "sw" folder inside pulpino and execute the following commands

		a) e.g. mkdir build
		
		b) cp cmake_configure.klessydra-t1-3th.gcc.sh build/
		
		c) cd build
		
		d) ./cmake_configure.klessydra-t1-3th.gcc.sh
		   (Execute the above script twice if you ever change the variable that changes the riscv-compiler, since changing the compiler flushes the values of the variables in the cmake cache and gives an error. Executing the script for a second time after changing the riscv-compiler will let the variables be redfined again )
		   
		e) make vcompile

		For running Klessydra tests; the variable "USE_KLESSYDRA_TEST" in the above shell file is set to '1' by default. You only need to build and run your test
		f) (e.g.  make vect_sum_single_funct_call_all_test_perf.vsimc) FOR KLESSYDRA T1x ONLY!!!
		General tests for all "Txx" versions of Klessydra are also available
		g) (e.g.  make barrier_test.vsimc)
		
		For running a PULPino test, set the variable "USE_KLESSYDRA_TEST" inside the shell file to 0, and re-execute the shell file again, and then run
		h) (e.g. make testALU.vsimc)
			
	IT"S DONE!!!!!!

Supplimentary Information:

7.	In order to run tests in Modelsim, go to the build folder and do the following:
		make nameofthetest.vsim (or .vsimc to run a test without modelsim GUI)

8. many Klessydra-T13 tests areavailable, and they will be described in the test_guide

# T13x Extensions illustration

This small section illustrates briefly the extensions of the T13

Extensions of T13x core:

The T13 can configed in many ways in the from the "cmake_configure.klessydra-t1-3th.gcc.sh" found in the sw forlder:

You will find the following generics that will be passed to the RTL. Read the comments next to the variables before modifying:
1)  "THREAD_POOL_SIZE" sets the number of hardware threads. This should not be set less than 3, and the T13 perfroms best when it is equal to 3 and not grater.
2)  "LUTRAM_RF" this variable creates a LUTRAM based registerfile instead of a flip-flop based one, it is good for FPGA synthesis as it saves a lot of power
3)	"RV32E" this enables the embedded extension of the RISCV ISA, and makes the regfile to be half its original size.
4)	"RV32M" this enable the M-extension of the RISCV ISA. The mul instruction is a single cycle instructions, and the mulh/hu/hsu instructions need 3 cycles. divisions are slow, and can be up to 32 cycles, however fast single cycle divisions are availabe for special cases (div by 0, numerator < denominator, numerator is 0, and numerator equals the denminator).
5)	"superscalar_exec_en=1"  Enables superscalar execution when set to 1, else the stall of the pipeline will depend on tha latency of the instruction executing. This more than doubles the speed of the core in many apps, however if in the exceptional case the RTL is not simulating correctly, disable this and see whether the RTL will work again.
6)	"accl_en"  Enables the generation of the hardware accelerator of the T13
7)	"replicate_accl_en" Set to 1 to replicate the accelerator for every thread, this increases the parallelism of the T13 by allocating a dedicated accelerator for each hardware thread
8)	"multithreaded_accl_en" Set this to 1 to let the replicated accelerator have shared functional units, but maintain dedicated SPM memories for each hardware thread (note: replicate_accl_en must be set to '1').
9)	"SPM_NUM" The number of scratchpads available "Minimum allowed is 2". When the acclerator is replicated, each hardware thread will have scratchpads equal to SPM_NUM, so in a THREAD_POOL_SIZE of 3 we will have 3*SPM_NUM scratchpads in totals
10)	Addr_Width" This address is for scratchpads. Setting this will make the size of the spm to be: "2^Addr_Width -1"
11)	"SIMD" Changing the SIMD, would increase the DLP by increasing the number of the functional units in the accelerator, and the number of banks in the spms acordingly (can be power of 2 only e.g. 1,2,4,8) no more than SIMD 8 is allowed. Setting this to '8' with replicate_accl_en and superscalar_exec_en being set to '1' as well will make the accelerator run at peak performance.
12)	"MCYCLE_EN" Can be set to 1 or 0 only. Setting to zero will disable MCYCLE and MCYCLEH
13)	"MINSTRET_EN" Can be set to 1 or 0 only. Setting to zero will disable MINSTRET and MINSTRETH
14)	"MHPMCOUNTER_EN" Can be set to 1 or 0 only. Setting to zero will disable all performance counters except "MCYCLE/H" and "MINSTRET/H"
15)	"count_all" Perfomance counters count for all the harts instead of there own hart
16)	"debug_en" Generates the debug unit, the debug unit is elimentary and might need some further evaluation and testing


Hope you like it :D