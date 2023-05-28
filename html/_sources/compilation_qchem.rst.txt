Qchem compilation notes
=======================

To compile Q-Chem in atlas using intel compilers the following modules are necessary: ::

    # ATLAS - FDR
    module load Boost/1.71.0-iimpi-2019b
    module load imkl/2019.5.281-iimpi-2019b
    module load iccifort/2019.5.281
    module load CMake/3.15.3-GCCcore-8.3.0
    module load HDF5/1.10.5-intel-2019b-serial
    module load Python/3.7.4-GCCcore-8.3.0

    # ATLAS - EDR
    module load Boost/1.71.0-iimpi-2019b
    module load imkl/2019.5.281-iimpi-2019b
    module load iccifort/2019.5.281
    module load CMake/.3.15.3-GCCcore-8.3.0
    module load HDF5/1.10.5-intel-2019b-serial
    module load Python/3.7.6-Anaconda3-2020.02


To use MKL library the following line should be executed in order to load the proper environment ::

    # ATLAS - FDR
    source /scratch/scicomp/easybuild/CentOS/7.3.1611/Haswell/software/imkl/2019.5.281-iimpi-2019b/mkl/bin/mklvars.sh intel64

    # ATLAS - EDR
    source /scicomp/EasyBuild/CentOS/7.4.1708/Skylake/software/imkl/2019.5.281-iimpi-2019b/mkl/bin/mklvars.sh intel64

To compile Q-Chem in ATLAS cluster in DIPC using MKL it is necessary to modify FinMKL.cmake placed in cmake directory in
order to set the correct library paths and environment variables. A convenient  way is to modify line 213 from ::

    set(MKL_VERSION “10.1")

to ::

	set(MKL_VERSION "11.0")

also in the recent version of Q-Chem there is a problem in *libham/libham/CMakeLists.txt* file using an *old version* of
intel compiler (2017). This can be solved (https://gitlab.kitware.com/cmake/cmake/-/issues/17829) by changing ::

    cxx_generalized_initializers

by ::

    cxx_std_14

To compile MPI version in IQTC it is necessary to modify file /bin/qchem to force activation of **-np** flag ::

    set WITH_MPI = 1  

Also it is necessary to modify /bin/parallel.csh file to manually define the environment variable $QCMACHINEFILE to
properly set “machines” file location by commenting the following lines ::

	if ( $?QCGMAKE ) then
	   set QCMACHINEFILE=$QCBIN/machines
	else
	   set QCMACHINEFILE=$QC/bin/mpi/machines
	fi


.. note::
    The amount of RAM memory by default in ATLAS queue system is insufficient to compile Q-Chem, at least 16GB should
    be explicitly specified!!

.. note::
    The locale variable may produce some issues. If that is the case define::

        LC_ALL=en_US.UTF-8


MAC compilation
---------------
Compilation in MAC using Intel Compiler 19.x and MKL has some issues finding iomp5 library. A temporary
workaround is to manually edit *FindMKL.cmake* file by adding ::

    set(INTEL_IOMP5_PATH /opt/intel/compilers_and_libraries_2020/mac/lib/libiomp5.a)

just after this line ::

    find_library(INTEL_IOMP5_PATH iomp5 PATHS ENV ${LD_LIBRARY_PATH_NAME})


Example script
--------------
ATLAS - FDR ::

    #!/bin/bash
    #SBATCH --partition=regular
    #SBATCH --job-name=comp_G_fdr_qchem
    #SBATCH --cpus-per-task=8
    #SBATCH --mem=16gb
    #SBATCH --nodes=1
    #SBATCH --ntasks-per-node=1

    module load Boost/1.71.0-iimpi-2019b
    module load imkl/2019.5.281-iimpi-2019b
    module load iccifort/2019.5.281
    module load CMake/3.15.3-GCCcore-8.3.0
    module load HDF5/1.10.5-intel-2019b-serial
    module load Python/3.7.4-GCCcore-8.3.0

    source /scratch/scicomp/easybuild/CentOS/7.3.1611/Haswell/software/imkl/2019.5.281-iimpi-2019b/mkl/bin/mklvars.sh intel64

    export QC=/scratch/abel/SOFTWARE/qchem/

    cd $QC
    ./configure intel mkl openmp > ../configure_g.log


    cd $QC/build
    make -j 8 > compile.log 2> compile.err
    make install > install.log


ATLAS - EDR ::

    #!/bin/bash
    #SBATCH --partition=regular
    #SBATCH --job-name=comp_qchem
    #SBATCH --cpus-per-task=8
    #SBATCH --mem=32gb
    #SBATCH --nodes=1
    #SBATCH --ntasks-per-node=1

    module load Boost/1.71.0-iimpi-2019b
    module load imkl/2019.5.281-iimpi-2019b
    module load iccifort/2019.5.281
    module load CMake/.3.15.3-GCCcore-8.3.0
    module load HDF5/1.10.5-intel-2019b-serial
    module load Python/3.7.6-Anaconda3-2020.02

    export OMP_NUM_THREADS=$SLURM_CPUS_PER_TASK

    source /scicomp/EasyBuild/CentOS/7.4.1708/Skylake/software/imkl/2019.5.281-iimpi-2019b/mkl/bin/mklvars.sh intel64

    export QC=/scratch/abel/SOFTWARE/qchem/

    cd $QC
    ./configure intel mkl openmp > ../configure_g.log

    cd $QC/build
    make -j 8  > compile.log 2> compile.err
    make install > install.log

