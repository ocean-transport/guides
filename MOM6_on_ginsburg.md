On Ginsburg, you want to perform most actions in the scratch storage space: `/burg/abernathey/users/<your_username>`. So navigate there first. 
*Part of the reason to do this is the limit on number of files and storage in home directory.*

The instructions are based on the getting started page in [MOM6_examples](https://github.com/NOAA-GFDL/MOM6-examples/wiki/Getting-started#compiling-the-models), which is the super repo for the model and associated examples. Refer to this linked page for more details, such as getting input files for some experiments and linking to them. 

# Get the model
You can either fork the repo into your github account (if you plan to develop the code) or directly get the code from the main repo. Here we choose the second method

```
cd <scratch>
git clone --recursive https://github.com/NOAA-GFDL/MOM6-examples.git MOM6-examples
```
> Note: 
> - Make sure to do this in scratch.
> - Make sure to use the `--recursive` flag.

# Use interactive node
We need an interactive node to do the compiling. 
```
srun --pty -t 0-03:00 -A abernathey /bin/bash
```

# Compiling
There are two crucial steps for compiling:
- Set up the right environment, which involves loading the appropriate modules for fortran compiler and netcdf.
- Making a mkmf template that recognizes and links to the right compiler and libraries.

First setup a build directory, where all the compiling will take place. 
```
cd <scratch>/MOM6_examples
mkdir build
cd build
```

## Setup compile environment
Load the netcdf and compiler modules:
```
module load netcdf-fortran-intel/4.5.3
module load intel-parallel-studio/2020
```
If you then run `module list`, you should see the following:
![Screen Shot 2022-11-18 at 11 27 46 PM](https://user-images.githubusercontent.com/18236610/202862192-6cccb47e-c73e-4bb1-8148-b7a0d23e6e85.png)


## Create/get the mkmf template
Create a file names ginsburg_mpi.mk and copy the following text in it. This file was created using the example templates available at: https://github.com/NOAA-GFDL/mkmf/tree/master/templates, starting from linux-intel.mk. 
> The main action was identifying the name of the compilers that had mpi support, and linking to the right locations with netcdf and mpi includes (see some commented out lines, which were replaced by hard coded links. This was done because the shell was not appending paths easily. Can be streamlined in future.). 
```
vim ginsburg_mpi.mk # This will create a new file. Make sure you are in build directory here.
```
Copy and paste the the following into the file.
```
# Template for the Intel Compilers on Linux systems
#
# Typical use with mkmf
# mkmf -t linux-intel.mk -c"-Duse_libMPI -Duse_netCDF" path_names /usr/local/include

############
# Command Macros
FC = mpiifort
CC = mpiicc
CXX = mpiicpc
LD = mpiifort #mpif90 $(MAIN_PROGRAM)

#######################
# Build target macros
#
# Macros that modify compiler flags used in the build.  Target
# macrose are usually set on the call to make:
#
#    make REPRO=on NETCDF=3
#
# Most target macros are activated when their value is non-blank.
# Some have a single value that is checked.  Others will use the
# value of the macro in the compile command.

DEBUG =              # If non-blank, perform a debug build (Cannot be
                     # mixed with REPRO or TEST)

REPRO =              # If non-blank, erform a build that guarentees
                     # reprodicuibilty from run to run.  Cannot be used
                     # with DEBUG or TEST

TEST  =              # If non-blank, use the compiler options defined in
                     # the FFLAGS_TEST and CFLAGS_TEST macros.  Cannot be
                     # use with REPRO or DEBUG

VERBOSE =            # If non-blank, add additional verbosity compiler
                     # options

OPENMP =             # If non-blank, compile with openmp enabled

NO_OVERRIDE_LIMITS = # If non-blank, do not use the -qoverride-limits
                     # compiler option.  Default behavior is to compile
                     # with -qoverride-limits.

NETCDF =             # If value is '3' and CPPDEFS contains
                     # '-Duse_netCDF', then the additional cpp macro
                     # '-Duse_LARGEFILE' is added to the CPPDEFS macro.

INCLUDES =           # A list of -I Include directories to be added to the
                     # the compile command.

SSE = -xsse2         # The SSE options to be used to compile.  If blank,
                     # than use the default SSE settings for the host.
                     # Current default is to use SSE2.

COVERAGE =           # Add the code coverage compile options.

# Need to use at least GNU Make version 3.81
need := 3.81
ok := $(filter $(need),$(firstword $(sort $(MAKE_VERSION) $(need))))
ifneq ($(need),$(ok))
$(error Need at least make version $(need).  Load module gmake/3.81)
endif

# REPRO, DEBUG and TEST need to be mutually exclusive of each other.
# Make sure the user hasn't supplied two at the same time
ifdef REPRO
ifneq ($(DEBUG),)
$(error Options REPRO and DEBUG cannot be used together)
else ifneq ($(TEST),)
$(error Options REPRO and TEST cannot be used together)
endif
else ifdef DEBUG
ifneq ($(TEST),)
$(error Options DEBUG and TEST cannot be used together)
endif
endif

MAKEFLAGS += --jobs=$(shell grep '^processor' /proc/cpuinfo | wc -l)

# Required Preprocessor Macros:
CPPDEFS += -Duse_netCDF

# Additional Preprocessor Macros needed due to  Autotools and CMake
CPPDEFS += -DHAVE_SCHED_GETAFFINITY

# Macro for Fortran preprocessor
FPPFLAGS = -fpp -Wp,-w $(INCLUDES) -I/burg/opt/netcdf-fortran-intel-2020/include
# Fortran Compiler flags for the NetCDF library
#FFPPLAGS += $(shell nf-config --fflags)
#FPPLAGS += $(nf-config --fflags)
#FPPLAGS += $(nf-config --fflags)
# Fortran Compiler flags for the MPICH MPI library
#FFPPLAGS += $(shell pkg-config --cflags-only-I mpich2-c)
#FFPPLAGS += "-I/burg/opt/parallel_studio_xe_2020/compilers_and_libraries_2020.4.304/linux/mpi/intel64/include/"

# Base set of Fortran compiler flags
FFLAGS := -fno-alias -stack_temps -safe_cray_ptr -ftz -assume byterecl -i4 -r8 -nowarn -g -sox -traceback

# Flags based on perforance target (production (OPT), reproduction (REPRO), or debug (DEBUG)
FFLAGS_OPT = -O2
FFLAGS_REPRO = -fpmodel source -O2
FFLAGS_DEBUG = -O0 -check -check noarg_temp_created -check nopointer -warn -warn noerrors -debug variable_locations -fpe0 -ftrapuv

# Flags to add additional build options
FFLAGS_OPENMP = -qopenmp
FFLAGS_OVERRIDE_LIMITS = -qoverride-limits
FFLAGS_VERBOSE = -v -V -what -warn all
FFLAGS_COVERAGE = -prof-gen=srcpos

# Macro for C preprocessor
#CPPFLAGS = -D__IFC $(INCLUDES)
CPPFLAGS = -D__IFC $(INCLUDES) -I/burg/opt/netcdf-fortran-intel-2020/include -I/burg/opt/netcdf-c-4.7.4//include
# C Compiler flags for the NetCDF library
#CPPFLAGS += $(shell nc-config --cflags)
#CPPFLAGS += $(nf-config --cflags)
# C Compiler flags for the MPICH MPI library
#CPPFLAGS += $(shell pkg-config --cflags-only-I mpich2-c)

# Base set of C compiler flags
CFLAGS := -sox -traceback

# Flags based on perforance target (production (OPT), reproduction (REPRO), or debug (DEBUG)
CFLAGS_OPT = -O2
CFLAGS_REPRO = -O2
CFLAGS_DEBUG = -O0 -g -ftrapuv

# Flags to add additional build options
CFLAGS_OPENMP = -qopenmp
CFLAGS_VERBOSE = -w3
CFLAGS_COVERAGE = -prof-gen=srcpos

# Optional Testing compile flags.  Mutually exclusive from DEBUG, REPRO, and OPT
# *_TEST will match the production if no new option(s) is(are) to be tested.
FFLAGS_TEST = $(FFLAGS_OPT)
CFLAGS_TEST = $(CFLAGS_OPT)

# Linking flags
LDFLAGS :=
LDFLAGS_OPENMP := -qopenmp
LDFLAGS_VERBOSE := -Wl,-V,--verbose,-cref,-M
LDFLAGS_COVERAGE = -prof-gen=srcpos

# Start with a blank LIBS
#LIBS = 
LIBS = -L/burg/opt/netcdf-fortran-intel-2020/lib -lnetcdff
# NetCDF library flags
#LIBS += $(nf-config --flibs)
# MPICH MPI library flags
#$(shell pkg-config --libs mpich2-f90)

# Get compile flags based on target macros.
ifdef REPRO
CFLAGS += $(CFLAGS_REPRO)
FFLAGS += $(FFLAGS_REPRO)
else ifdef DEBUG
CFLAGS += $(CFLAGS_DEBUG)
FFLAGS += $(FFLAGS_DEBUG)
else ifdef TEST
CFLAGS += $(CFLAGS_TEST)
FFLAGS += $(FFLAGS_TEST)
else
CFLAGS += $(CFLAGS_OPT)
FFLAGS += $(FFLAGS_OPT)
endif

ifdef OPENMP
CFLAGS += $(CFLAGS_OPENMP)
FFLAGS += $(FFLAGS_OPENMP)
LDFLAGS += $(LDFLAGS_OPENMP)
endif

ifdef SSE
CFLAGS += $(SSE)
FFLAGS += $(SSE)
endif

ifdef NO_OVERRIDE_LIMITS
FFLAGS += $(FFLAGS_OVERRIDE_LIMITS)
endif

ifdef VERBOSE
CFLAGS += $(CFLAGS_VERBOSE)
FFLAGS += $(FFLAGS_VERBOSE)
LDFLAGS += $(LDFLAGS_VERBOSE)
endif

ifeq ($(NETCDF),3)
  # add the use_LARGEFILE cppdef
  ifneq ($(findstring -Duse_netCDF,$(CPPDEFS)),)
    CPPDEFS += -Duse_LARGEFILE
  endif
endif

ifdef COVERAGE
ifdef BUILDROOT
PROF_DIR=-prof-dir=$(BUILDROOT)
endif
CFLAGS += $(CFLAGS_COVERAGE) $(PROF_DIR)
FFLAGS += $(FFLAGS_COVERAGE) $(PROF_DIR)
LDFLAGS += $(LDFLAGS_COVERAGE) $(PROF_DIR)
endif

LDFLAGS += $(LIBS)

#---------------------------------------------------------------------------
# you should never need to change any lines below.

# see the MIPSPro F90 manual for more details on some of the file extensions
# discussed here.
# this makefile template recognizes fortran sourcefiles with extensions
# .f, .f90, .F, .F90. Given a sourcefile <file>.<ext>, where <ext> is one of
# the above, this provides a number of default actions:

# make <file>.opt	create an optimization report
# make <file>.o		create an object file
# make <file>.s		create an assembly listing
# make <file>.x		create an executable file, assuming standalone
#			source
# make <file>.i		create a preprocessed file (for .F)
# make <file>.i90	create a preprocessed file (for .F90)

# The macro TMPFILES is provided to slate files like the above for removal.

RM = rm -f
TMPFILES = .*.m *.B *.L *.i *.i90 *.l *.s *.mod *.opt

.SUFFIXES: .F .F90 .H .L .T .f .f90 .h .i .i90 .l .o .s .opt .x

.f.L:
	$(FC) $(FFLAGS) -c -listing $*.f
.f.opt:
	$(FC) $(FFLAGS) -c -opt_report_level max -opt_report_phase all -opt_report_file $*.opt $*.f
.f.l:
	$(FC) $(FFLAGS) -c $(LIST) $*.f
.f.T:
	$(FC) $(FFLAGS) -c -cif $*.f
.f.o:
	$(FC) $(FFLAGS) -c $*.f
.f.s:
	$(FC) $(FFLAGS) -S $*.f
.f.x:
	$(FC) $(FFLAGS) -o $*.x $*.f *.o $(LDFLAGS)
.f90.L:
	$(FC) $(FFLAGS) -c -listing $*.f90
.f90.opt:
	$(FC) $(FFLAGS) -c -opt_report_level max -opt_report_phase all -opt_report_file $*.opt $*.f90
.f90.l:
	$(FC) $(FFLAGS) -c $(LIST) $*.f90
.f90.T:
	$(FC) $(FFLAGS) -c -cif $*.f90
.f90.o:
	$(FC) $(FFLAGS) -c $*.f90
.f90.s:
	$(FC) $(FFLAGS) -c -S $*.f90
.f90.x:
	$(FC) $(FFLAGS) -o $*.x $*.f90 *.o $(LDFLAGS)
.F.L:
	$(FC) $(CPPDEFS) $(FPPFLAGS) $(FFLAGS) -c -listing $*.F
.F.opt:
	$(FC) $(CPPDEFS) $(FPPFLAGS) $(FFLAGS) -c -opt_report_level max -opt_report_phase all -opt_report_file $*.opt $*.F
.F.l:
	$(FC) $(CPPDEFS) $(FPPFLAGS) $(FFLAGS) -c $(LIST) $*.F
.F.T:
	$(FC) $(CPPDEFS) $(FPPFLAGS) $(FFLAGS) -c -cif $*.F
.F.f:
	$(FC) $(CPPDEFS) $(FPPFLAGS) -EP $*.F > $*.f
.F.i:
	$(FC) $(CPPDEFS) $(FPPFLAGS) -P $*.F
.F.o:
	$(FC) $(CPPDEFS) $(FPPFLAGS) $(FFLAGS) -c $*.F
.F.s:
	$(FC) $(CPPDEFS) $(FPPFLAGS) $(FFLAGS) -c -S $*.F
.F.x:
	$(FC) $(CPPDEFS) $(FPPFLAGS) $(FFLAGS) -o $*.x $*.F *.o $(LDFLAGS)
.F90.L:
	$(FC) $(CPPDEFS) $(FPPFLAGS) $(FFLAGS) -c -listing $*.F90
.F90.opt:
	$(FC) $(CPPDEFS) $(FPPFLAGS) $(FFLAGS) -c -opt_report_level max -opt_report_phase all -opt_report_file $*.opt $*.F90
.F90.l:
	$(FC) $(CPPDEFS) $(FPPFLAGS) $(FFLAGS) -c $(LIST) $*.F90
.F90.T:
	$(FC) $(CPPDEFS) $(FPPFLAGS) $(FFLAGS) -c -cif $*.F90
.F90.f90:
	$(FC) $(CPPDEFS) $(FPPFLAGS) -EP $*.F90 > $*.f90
.F90.i90:
	$(FC) $(CPPDEFS) $(FPPFLAGS) -P $*.F90
.F90.o:
	$(FC) $(CPPDEFS) $(FPPFLAGS) $(FFLAGS) -c $*.F90
.F90.s:
	$(FC) $(CPPDEFS) $(FPPFLAGS) $(FFLAGS) -c -S $*.F90
.F90.x:
	$(FC) $(CPPDEFS) $(FPPFLAGS) $(FFLAGS) -o $*.x $*.F90 *.o $(LDFLAGS)
```

## Compile the fms library
Before we can compile MOM6, we need to compile the fms library. 

```
mkdir fms # do this in build
cd fms
../../src/mkmf/bin/list_paths -l ../../src/FMS # will create a file called path_names
../../src/mkmf/bin/mkmf -t ../ginsburg_mpi.mk -p libfms.a -c "-Duse_libMPI -Duse_netCDF" path_names # creates a file called MakeFile
make NETCDF=3 libfms.a -j 
# repro option set in original doc doesn't work on ginsburg yet. 
# remove the -j flag if wanting to run line by line, rather than in parallel. 
# this should create a file called libfms.a if everything worked properly, with many other files.
```

## Compile MOM6 in ocean only mode 
Compile and build an executable for the model. 
```
mkdir ocean_only # do this in build directory
../../src/mkmf/bin/list_paths -l ./ ../../src/MOM6/{config_src/infra/FMS1,config_src/memory/dynamic_symmetric,config_src/drivers/solo_driver,config_src/external,src/{*,*/*}}/
# The above will create a file named path_names
../../src/mkmf/bin/mkmf -t ../ginsburg_mpi.mk -o '-I../fms' -p MOM6 -l '-L../fms -lfms' path_names
# The above makes a Makefile
make MOM6 -j
# The above should create an executable called MOM6, if everything goes well. 
```

> Another way compile would have been to use autoconf (for ocean only mode), and not have to most of the above steps.  
> The following is the message from HPC-support on how to do this:  
> I was successful in using the Autoconf Build Configuration from https://github.com/NOAA-GFDL/MOM6/blob/dev/gfdl/ac/README.md. 
> I loaded netcdf/gcc/64/gcc/64/4.7.4 and netcdf-fortran/4.5.3, along with openmpi/gcc/64/4.0.3rc4.  
> Then, for the ../ac/configure command, in my ~/build directory I then loaded anaconda/3-2020.11.  
> I have a MOM6 executable now.  

# Running a test case
Now we will run a test case to make sure that the executable is working. 

```
cd <scratch>/MOM6_examples/ocean_only
cp -r Phillips_2layer Phillips_2layer_test # copy an example to play with
cd Phillips_2layer_test
mkdir -p RESTART
mpirun -n 4 ../../build/ocean_only/MOM6 # don't use mpi if using single core. 
```
If the above works, you will start seeing output that looks like:
![Screen Shot 2022-11-19 at 12 24 55 PM](https://user-images.githubusercontent.com/18236610/202863637-faf4532f-120e-46d4-b105-9764c65af4dd.png)
