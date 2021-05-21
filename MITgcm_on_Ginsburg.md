On Ginsburg, you want to perform most actions in the scratch storage space: `/burg/abernathey/users/<your_username>`. So navigate there first.

# Setup
Find basic instructions for getting the code and compiling [here](https://mitgcm.readthedocs.io/en/latest/getting_started/getting_started.html). 

*Do not modify the code tree*, i.e. make a separate directory for your project outside the code tree. Inside this directory, create four folders:
- build
- code
- input
- run

Modifications to the code tree reside in the `code` directory. To start with, you need to put the *build options* file in `code`. A *build options* file that should work on Ginsburg is pasted below. You should name it `linux_ia64_ifort_ginsburg`.  
```
#!/bin/bash
#
# Tested on uv100.awi.de (SGI UV 100, details:
#                         http://www.sgi.com/products/servers/uv/specs.html)
# a) For more speed, provided your data size does not exceed 2GB you can
#    remove -fPIC which carries a performance penalty of 2-6%.
# b) You can replace -fPIC with '-mcmodel=medium -shared-intel' which may
#    perform faster than -fPIC and still support data sizes over 2GB per
#    process but all the libraries you link to must be compiled with
#    -fPIC or -mcmodel=medium
# c) flags adjusted for ifort 12.1.0

if test "x$MPI" = xtrue ; then
  CC=mpiicc
  FC=mpiifort
  F90C=mpiifort
else
  CC=icc
  FC=ifort
  F90C=ifort
fi

CPP='/lib/cpp  -traditional -P'
EXTENDED_SRC_FLAG='-132'

DEFINES='-DWORDLENGTH=4'
LIBS="$(nf-config --flibs)"
INCLUDES="$(nf-config --fflags)"
INCLUDEDIRS="$(nf-config --includedir)"

if test "x$MPI" = xtrue ; then
   DEFINES+=' -DALLOW_USE_MPI -DALWAYS_USE_MPI'
   LIBS+=" -L$I_MPI_ROOT/lib64 -lmpi"
   INCLUDES+=" -I$I_MPI_ROOT/intel64/include"
   INCLUDEDIRS+=" $I_MPI_ROOT/intel64/include"
fi

if test "x$IEEE" = x ; then
    #  No need for IEEE-754
    FFLAGS="$FFLAGS -WB -convert big_endian -assume byterecl -mcmodel=medium -shared-intel"
    FOPTIM='-O3 -align '
    NOOPTFLAGS='-O1'
    NOOPTFILES='calc_oce_mxlayer.F fizhi_lsm.F fizhi_clockstuff.F'
else
    #  Try to follow IEEE-754
    FFLAGS="$FFLAGS -W0 -WB -convert big_endian -assume byterecl -mcmodel=medium -shared-intel"
    FOPTIM='-O0 -noalign'
fi

if test "x$DEVEL" = xtrue ; then
    FFLAGS="$FFLAGS -g -O0 -convert big_endian -assume byterecl -mcmodel=medium -shared-intel -traceback"
        FOPTIM='-O0 -noalign'
fi
```
You will also need some other files, including your own copy of SIZE.h. I recommend copying these from an example in the verifications directory of the code tree if you just want to get started. 

# Compiling

I recommend creating an executable file called *compile*, and putting the following lines in it:
```
#!/bin/bash

module load intel-parallel-studio/2020
module load netcdf-fortran-intel/4.5.3


make CLEAN
../MITgcm/tools/genmake2 -rootdir ../MITgcm -mpi -mods ../code -of ../code/linux_ia64_ifort_ginsburg
make depend
make
```
Modify `../MITgcm` wherever it appears in this file, to point to the location of your actual code tree. 

The command `./compile` will compile your code, and generate the executable file `mitgcmuv`. This is the file you need to run the MITgcm. 

# Input

If you are running an example from the `verification` directory, copy the files from their input file into your own input file (having a copy of these makes it easier to start a clean run if things go wrong). Also copy them into your run directory. 

# Running

Copy `mitgcmuv` along with input fields into the `run` directory (you can also use soft links here). To run the MItgcm on ginsburg, you will need to create the file `batch.sh` in your run directory:
```
#!/bin/sh
#
# Simple "Hello World" submit script for Slurm.
#
# Replace <ACCOUNT> with your account name before submitting.
#
#SBATCH --account=abernathey            # The account name for the job.
#SBATCH --job-name=JOB-NAME    # The job name.
#SBATCH -N 3                     # The number of nodes to use
                                 #(note there are 32 cores per node)
#SBATCH --exclusive                                 
#SBATCH --time=00:10:00              # The time the job will take to run.
#SBATCH --mail-type=ALL
#SBATCH --mail-user=<your-email>

module load intel-parallel-studio/2020
module load netcdf-fortran-intel/4.5.3

mpirun -n 96 ./mitgcmuv

# End of script
```

Information on job submission is [here](https://confluence.columbia.edu/confluence/display/rcs/Ginsburg+-+Submitting+Jobs).
