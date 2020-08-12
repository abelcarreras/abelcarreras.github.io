Qchem compilation notes
=======================

To compile Q-Chem in atlas using intel compilers the following modules are necessary: ::

    module load CMake/3.9.1-intel-2017a
    module load Boost/.1.64.0-intel-2017a

To use MKL library the following line should be executed in order to load the proper environment ::

    source /scicomp/easybuild/CentOS/7.3.1611/Haswell/software/imkl/2017.3.196-iimpi-2017b/mkl/bin/mklvars.sh intel64

To compile Q-Chem in ATLAS cluster in DIPC using MKL it is necessary to modify FinMKL.cmake placed in cmake directory in order to set the correct library paths and environment variables. A convenient  way is to modify line 213 from::

	set(MKL_VERSION “10.1")

to::

	set(MKL_VERSION "11.0")

also in the recent version of Q-Chem there is a problem in *libham/libham/CMakeLists.txt* file using an *old version* of intel compiler (2017).
This can be solved (https://gitlab.kitware.com/cmake/cmake/-/issues/17829) by changing::

    cxx_generalized_initializers

by::

    cxx_std_14

To compile MPI version in IQTC it is necessary to modify file /bin/qchem to force activation of **-np** flag::

    set WITH_MPI = 1  

Also it is necessary to modify /bin/parallel.csh file to manually define the environment variable $QCMACHINEFILE to properly set “machines” file location by commenting the following lines::

	if ( $?QCGMAKE ) then
	   set QCMACHINEFILE=$QCBIN/machines
	else
	   set QCMACHINEFILE=$QC/bin/mpi/machines
	fi


Note: The amount of RAM memory by default in ATLAS queue system is insufficient to compile Q-Chem, at least 16GB should be explicitly
specified!!

Example script
--------------
::

    #!/bin/bash
    #cores=8
    #wallt=5
    #cput=$(($cores * $wallt))

    #PBS -q parallel
    #PBS -l ncpus=8
    #PBS -l cput=120:00:00
    #PBS -l mem=16gb
    #PBS -N comp_qchem
    #PBS -e error.log
    #PBS -o output.log

    module load CMake/3.9.1-intel-2017a
    module load Boost/.1.64.0-intel-2017a

    source /scicomp/easybuild/CentOS/7.3.1611/Haswell/software/imkl/2017.3.196-iimpi-2017b/mkl/bin/mklvars.sh intel64

    export QC=/scratch/abel/SOFTWARE/qchem/

    cd $QC
    ./configure intel mkl openmp > ../configure_g.log


    cd $QC/build
    make -j 8 > compile.log 2> compile.err
    make install > install.log

