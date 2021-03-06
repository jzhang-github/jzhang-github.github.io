About this document
========================================================================

This document describes how to build VASP.6.X.
It is a rough summary of the information you will find at:

http://www.vasp.at/wiki/index.php/Installing_VASP

If you can access the aformentioned link we recommend
you do so right now and forget about this document
(since the information on our wiki is more complete
and up-to-date), if not, please continue reading.



# 1. Build system
------------------------------------------------------------------------

The build system of this VASP distribution has the following structure:


                   vasp.X.X.X (root directory)
                                |
          ------------------------------------------------
         |        |        |         |          |         |
        arch     bin     build      src     testsuite   tools

* `root/`

	Holds the high-level makefile and several subdirectories.

* `root/src`

	Holds the source files of VASP and a low-level makefile.

* `root/arch`

	Holds a collection of `makefile.include.*` files.

* `root/build`

	The different versions of VASP, i.e., the standard, gamma-only,
	non-collinear, and CUDA-GPU versions will be build in separate
	subdirectories of this directory.

* `root/bin`

	Here make will store the binaries.

* `root/testsuite`

	Holds a suite of correctness tests to check your build.

* `root/tools`

	Holds several python scripts related to the (optional) use
	of HDF5 input/output files.



# 2. How to make VASP
------------------------------------------------------------------------

Copy one of the `makefile.include.*` files in `root/arch` to
`root/makefile.include`.

Take one that most closely reflects your system (hopefully).
For instance, on a linux box with the Intel Composer suite:

    cp arch/makefile.include.linux_intel  ./makefile.include

In many cases these `makefile.include` files will have to be adapted
to the particulars of your system (see section 3).

When you've finished setting up makefile.include, build VASP:

    make all

This will build the standard, gamma-only, and non-collinear version
of VASP one after the other.

Alternatively on may build these versions individually:

    make std
    make gam
    make ncl


To compile the CUDA-GPU ports of VASP:

    cp arch/makefile.include.linux_intel ./makefile.include

and adapt it to the particulars of your system (see section 3),
followed by:

    make gpu
    make gpu_ncl

to built the CUDA-GPU ports of the standard and non-collinear versions,
respectively.

**N.B.**: Unfortunately we do not offer a CUDA GPU port of the
	gamma-only version.

After building the `vasp_std`, `vasp_gam`, and `vasp_ncl`
executables (e.g. by means of `make all`), you can test your
build by means of:

    make test

Running the validation testsuite is described in some detail
in `root/testsuite/README.md` and on our wiki:
http://www.vasp.at/wiki/index.php/Validation_tests.



# 3. Adapting makefile.include
------------------------------------------------------------------------

## 3.1. Precompiler variables:

* `CPP_OPTIONS`:

	Specify the precompiler flags: `[-Dflag1 [-Dflag2] ... ]`  
	Take a lead from the `makefile.include.*` files in `/arch`
	and consult the VASP manual.

	**N.B.I**: `-DNGZhalf`, `-DwNGZhalf`, `-DNGXhalf`, and `-DwNGXhalf`
    are deprecated options.  Building the standard, gamma-only,
    or non-collinear version of the code is specified through
    an additional argument to the make command (see "make" section below).

	**N.B.II**: `CPP_OPTIONS` is only used in the `makefile.include` file,
    where it should be added to CPP (see below).


* `CPP`:

	The command to invoke the precompiler you want to use, for instance:

	Using Intel's Fortran precompiler:

      CPP=fpp -f_com=no -free -w0 $*$(FUFFIX) $*$(SUFFIX) $(CPP_OPTIONS)

	Using cpp:

      CPP=/usr/bin/cpp -P -C -traditional $*$(FUFFIX) >$*$(SUFFIX) $(CPP_OPTIONS)

	**N.B.**: This variable has to include `$(CPP_OPTIONS)`.
    If it does not, `CPP_OPTIONS` will be ignored!



## 3.2. Compiler variables:

The Fortran compiler will be invoked as:

    $(FC) $(FREE) $(FFLAGS) $(OFLAG) $(INCS)

* `FC`:

	The command to invoke your Fortran compiler
	(e.g. gfortran, ifort, mpif90, mpiifort, ... ).


* `OFLAG`:

	The general level of optimization (default: `OFLAG=-O2`).


* `FFLAGS`:

	Additional compiler flags.


* `OFLAG_IN`:

	(default: `-O2`) In the vast majority of makefiles this
	variable is set:

      OFLAG_IN=$(OFLAG)


* `DEBUG`:

	The optimization level with which the main program (main.F)
	will be compiled, usually:

      DEBUG=-O0


* `INCS`:

	Use this variable to specify objects to be included in the
	sense of:

      INCS=-I/path/to/directory-with-files-to-be-included


* `FREE`:

	Specify the options that your Fortran compiler needs for it to
	accept free-form source layout, without line-length limitation.
	For instance:

	Using Intel's Fortran compiler:

      FREE=-free -names lowercase

	Using gfortran:

      FREE=-ffree-form -ffree-line-length-none



## 3.3. Libraries:

The linker will be invoked as:

    $(FCL) -o vasp  ..all-objects.. $(LLIBS)


* `FCL`:

	The command that invokes the linker: in most cases identical to:

      FCL=$(FC) [+ some options]

	Using the Intel composer suite (Fortran compiler + MKL libraries),
	typically:

      FCL=$(FC) -mkl=sequential


* `LLIBS`:

	Specify libraries and/or objects to be linked against,
	in the usual ways:

      LLIBS=[-Ldirectory -llibrary] [path/library.a] [path/object.o]

	Usually one has to specify several numerical libraries
	(BLAS, LAPACK or scaLAPACK, etc).

	For instance using the Intel composer suite
	(and compiling with `CPP_OPTIONS= .. -DscaLAPACK ..`):

      MKL_PATH   = $(MKLROOT)/lib/intel64
      BLACS      = -lmkl_blacs_openmpi_lp64
      SCALAPACK  = $(MKL_PATH)/libmkl_scalapack_lp64.a $(BLACS)
      LLIBS      += $(SCALAPACK)

	For other configurations please take a lead from the
	`makefile.include.*` files under `/arch`.



## 3.4. The list of objects:

The standard list of objects needed to compile VASP is given by the
variable `SOURCE` in the `root/src/.objects` file that is part of the
distribution (you should not need to edit this variable).

Objects to be added to this list can be specified in `makefile.include`
by means of:

    OBJECTS= .. your list of objects ..


**N.B.**: Several objects *have* to be added in this manner (see the
following section on "Fast-Fourier-Transforms).



## 3.5. Fast-Fourier-Transforms:

* `OBJECTS`:

	Add the objects to be compiled (or linked againts) that provide
	the FFTs (may include static libraries of objects .a).


* `INCS`:

	In case one compiles using the fftw-library, i.e.,

      OBJECTS= .. fftw3d.o fftmpiw.o ..

	then `INCS` can be set to include the directory that holds "fftw3.f":

      INCS += -I/path/to/directory-that-holds-fftw3f

	(needed because fftw3d.F and fftmpiw.F include "fftw3.f").

	**N.B.**: If in the aformentioned case `INCS` is not set,
    then fftw3.f has to be present in `root/src`.


Common choices are:

* When building with the Intel Composer suite it is best
	to link against the MKL fftw wrappers of Intel's FFTs:

      FCL = $(FC) -mkl=sequential

      OBJECTS = fftmpiw.o fftmpi_map.o fftw3d.o fft3dlib.o
      INCS   += -I$(MKLROOT)/include/fftw


* To explicitly link against an fftw library (in this case fftw-3.3.4):

      OBJECTS = fftmpiw.o fftmpi_map.o fftw3d.o fft3dlib.o
      FFTW       ?= /path/to/my/fftw-3.3.4
      LLIBS      += -L$(FFTW)/lib -lfftw3
      INCS       += -I$(FFTW)/include

* Or use use Juergen Furtmueller's FFT implementation (deprecated):

      OBJECTS= fftmpi.o fftmpi_map.o fft3dfurth.o fft3dlib.o
      INCS=

* For other configurations please take lead from the
	`makefile.include.*` files under `/arch`.



## 3.6. Special rules for the optimization level of FFT related objects:

Based on past experience the optimization level for the
compilation of the FFT related objects is set explicitly.
This is done as follows:

    OBJECTS_O1 += fft3dfurth.o fftw3d.o fftmpi.o fftmpiw.o
    OBJECTS_O2 += fft3dlib.o



## 3.7. Special rules in general:

The current `src/makefile` contains a set of recipes to allow for the
compilation of objects at different levels of optimization (other than
the general level specified by `OFLAG`).

In these recipes the compiler will be invoked as:

    $(FC) $(FREE) $(FFLAGS_x) $(OFLAG_x) $(INCS_x)

where x stands for: 1, 2, 3, or IN.


* `FFLAGS_x`:

	Default: `FFLAGS_x=$(FFLAGS)`, for x=1, 2, 3, and IN.


* `OFLAG_x`:

	Default: `OFLAG_x=-Ox` (for x=1, 2, 3), and `OFLAG_IN=-O2`


* `INCS_x`:

	Default: `INCS_x=$(INCS)`, for x=1, 2, 3, and IN.


The objects to be compiled in accordance with these recipes are
specified by means of the variables:

    OBJECTS_O1, OBJECTS_O2, OBJECTS_O3, and OBJECTS_IN


Several objects are compiled at `-O1` and `-O2` by default. These lists
of objects are specified in the .objects file through the variables:

    SOURCE_O1, SOURCE_O2, and SOURCE_IN

and reflect the special rules as they were present in most of the
makefiles of the old build system.

To completely overrule a default setting (for instance for the `-O1`
special rules) use the following construct:

    SOURCE_O1=
    OBJECTS_O1= .. your list of objects ..



## 3.8. For the library "lib":

* `CPP_LIB`:

	The command to invoke the precompiler.
	In most cases it will suffise to set:

      CPP_LIB=$(CPP)


* `FC_LIB`:

	The command to invoke your Fortran compiler.
	In most cases:

      FC_LIB=$(FC)

	**N.B.**: the library can be compiled without MPI support,
    i.e., when `FC=mpif90`, `FC_LIB` may specify a Fortran
    compiler without MPI supprt, e.g. `FC_LIB=ifort`.


* `FFLAGS_LIB`:

	Fortran compiler flags, including a specification of the
	level of optimization. In most cases:

      FFLAGS_LIB=-O1


* `FREE_LIB`:

	Specify the options that your Fortran compiler needs for it to
	accept free-form source layout, without line-length limitation.
	In most cases it will suffise to set:

      FREE_LIB=$(FREE)


* `CC_LIB`:

	The command to invoke your C compiler (e.g. gcc, icc, ..).
	N.B.: the library can be compiled without MPI support.


* `CFLAGS_LIB`:

	C compiler flags, including a specification of the
	level of optimization. In most cases:

      CFLAGS_LIB=-O

* `OBJECTS_LIB`:

	List of "non-standard" objects to be added to the library.
	In most cases:

      OBJECTS_LIB= linpack_double.o



## 3.9. For the library "parser"

* `CXX_PARS`:

	Specify your C++ compiler here and add the `stdc++` library to your link line.
	When using Intel's compilers this would amount to:

      CXX_PARS = icpc
      LLIBS += -lstdc++



## 3.10. For the CUDA-GPU port:

* `CUDA_ROOT`:

	Location of CUDA toolkit install.  For example:

      CUDA_ROOT := /opt/cuda


* `CUDA_LIB`:

	CUDA toolkit libraries to link to.  Typically:

      CUDA_LIB   := -L$(CUDA_ROOT)/lib64 -lnvToolsExt -lcudart -lcuda -lcufft -lcublas


* `NVCC`:

	Location of CUDA compiler and flags.  Typically:

      NVCC := $(CUDA_ROOT)/bin/nvcc -g


* `OBJECTS_GPU`:

	Add the objects to be compiled (or linked againts) that provide
	the FFTs (may include static libraries of objects .a).  For fftw:

      OBJECTS_GPU = fftmpiw.o fftmpi_map.o fft3dlib.o fftw3d_gpu.o fftmpiw_gpu.o


* `GENCODE_ARCH`:

	CUDA compiler options to generate code for your particular GPU architecture.  

	For Kepler, for example:

      GENCODE_ARCH := -gencode=arch=compute_35,code=\"sm_35,compute_35\"

	or for Maxwell:

      GENCODE_ARCH := -gencode=arch=compute_53,code=\"sm_53,compute_53\"

	Multiple `-gencode` statements can be compiled to create cross-platform executables.
	See http://docs.nvidia.com/cuda/cuda-compiler-driver-nvcc for details.


* `MPI_INC`:

	Path to MPI include files so the CUDA compiler can find them.  For example:

      MPI_INC := /opt/openmpi/include

	These can often be found with `mpicc --show`


* `CPP_GPU`:

	Preprocessor options for GPU compilation. Always include:

      -DCUDA_GPU              # Tells cross-platform sources to build for GPU
      -DRPROMU_CPROJ_OVERLAP  # Overlap communication and computation in RPROJ_MU

      -UscaLAPACK             # Do not use scaLAPACK
      -Ufock_dblbuf           # Do not use the newest HF routines

	Optional:

      -DUSE_PINNED_MEMORY  # Use pinned memory for transfer buffers
      -DCUFFT_MIN=<N>      # Intercept any FFT calls of size greater than N^3 and evaluate on GPU.

	Experimental:

      -DUSE_MAGMA  # use MAGMA for LAPACK-like calls on the GPU.

* `MAGMA_ROOT`:

	If using the experimental MAGMA support, path to MAGMA (>=1.6).  Typically:

      MAGMA_ROOT := /opt/magma/lib



# 4. Further information
------------------------------------------------------------------------

This document presents a rough summary of the information
you will find at:

http://www.vasp.at/wiki/index.php/Installing_VASP

For further information please follow the aforementioned link,
since the information on our wiki is more complete and up-to-date.
