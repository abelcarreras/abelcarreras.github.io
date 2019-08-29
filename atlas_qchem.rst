How to use qchem in ATLAS cluster (DIPC)
========================================

Preliminar information
----------------------

**Requirements to acces to qchem**

* Have an atlas account (atlas.sw.ehu.es)
* Be a member of dccqchem linux group

**Available qchem compilations**

* qchem_trunk: Current master version of qchem [openmp intel_compilers mkl boost]
* qchem_group: Current in development version of qchem (includes new features developed in the group) [openmp intel_compilers mkl boost]

Load custom modules
-------------------

o acces to qchem you should be able to load custom modules located in /scratch/abel/SOFTWARE/privatemodules. To do this you may modify the MODULEPATH environment variable as follows:: 

    export MODULEPATH=/scratch/abel/SOFTWARE/privatemodules:$MODULEPATH

Once this is done, you should be able to see the custom modules (qchem_trunk & qchem_group) executing the command::

    module avail

To use qchem, just load the corresponding module as usual::

    module qchem_trunk

or::

    module qchem_group

*Note: QCSCRATCH environment variable defines the directory where the scratch files will be generated during qchem execution. In batch jobs ideally this should be under the scratch directory tree. If this variable is not defined it will be set to current directory.*

Run interactive qchem
---------------------

In order to run simple SHORT (< 1min) tests you may execute them directly in shell environment (once everything is loaded properly)::

    qchem input_file.inp output_file.out

For proper run long calculations you should use the queue system to submit batch calculations.
For this purpose you should write a batch script.

Batch job example
-----------------

All commands discussed previously can be gathered in a simple script to run in batch. This is a simple example::

	#!/bin/bash

	#PBS -q parallel       #  “parallel" o “qchem” depending on the queue you have access to
	#PBS -l ncpus=8        # number of CPI
	#PBS -l cput=80:00:00  # CPU time (WallTime * Ncpus) maximum running time (hh:mm:ss)
	#PBS -N qchem_calc     # job name
	#PBS -e error.log      # queue system error output file
	#PBS -o output.log     # queue system standard output file

	export MODULEPATH=/scratch/abel/SOFTWARE/privatemodules:$MODULEPATH

	module load qchem_group

	cd $PBS_O_WORKDIR     # set workdir to submission directory 
	qchem -nt 8 inputfile.inp  outputfile.out

Batch job example 2
-------------------

This is a more elaborated example showing how to retrieve additional data from qchem using **"-save"** flag
in the qchem call. Also how to use previous qchem data (ex. converged electronic structure data) in new
calculation (requires additional keywords in qchem input, check qchem manual for more info.) ::

    #!/bin/bash

    #PBS -q qchem
    #PBS -l ncpus=4
    #PBS -l mem=4000mb
    #PBS -l cput=800:00:00  # CPU time (Walltime / Ncores)
    #PBS -N test
    #PBS -e error.log
    #PBS -o output.log

    NProc=4       # Should be tha same as "-l ncpus="
    export MODULEPATH=/scratch/abel/SOFTWARE/privatemodules:$MODULEPATH

    #------------------------------------#
    # QChem and job variables
    #------------------------------------#
    module load qchem_group   # load qchem module
    export QCSCRATCH=.        # set scratch folder for qchem

    #------------------------------------#
    # Job set up
    #------------------------------------#
    folder=                 # path to the input file
    name=                   # input file without .in extension
    writetype=0             # 1: save additional data / 0: Do not save additional data
    readtype=0              # 1: read data from previous calculation / 0: Do not read additional data
    savename=name.in.save   # Path to the directory that contains previous calculation data (relative to workdir)
    cleansrc=1              # Cleanup scratch dir after calculation

    #------------------------------------#
    # Set read and write folders
    #------------------------------------#
    cd $folder

    mkdir -p $QCSCRATCH/qchem$$
    if [ $readtype = 1 ]; then
      cp $savename/* $QCSCRATCH/qchem$$/.
      if [ $writetype = 1 ]; then
         cd $folder
         rm -r $savename
      fi
    fi

    #------------------------------------#
    # Run QChem
    #------------------------------------#
    if [ $writetype = 1 -o $readtype = 1 ]; then
     qchem -save -nt $NProc $name.in $name.out qchem$$

    else
     qchem -nt $NProc $name.in $name.out
    fi

    #------------------------------------#
    # Select what you want to save (comment/uncomment)
    #------------------------------------#
    if [ $writetype = 1 ]; then
       mkdir $QCSCRATCH/$savename
       mv $QCSCRATCH/qchem$$/53.0  $QCSCRATCH/$savename/.    # MO coeff
       mv $QCSCRATCH/qchem$$/58.0  $QCSCRATCH/$savename/.    # Fock
       mv $QCSCRATCH/qchem$$/99.0  $QCSCRATCH/$savename/.    # MO energies
       # mv $QCSCRATCH/qchem$$/NTOs  $QCSCRATCH/$savename/.  # NTOs files
       # mv $QCSCRATCH/qchem$$/plots $QCSCRATCH/$savename/.  # plot files
       # mv $QCSCRATCH/qchem$$/*     $QCSCRATCH/$savename/.  # Everything
       # mv $QCSCRATCH/$savename $folder/.
    fi

    #------------------------------------#
    # Clean up
    #------------------------------------#
    if [$cleansrc = 1 ]; then
       rm -r $QCSCRATCH/qchem$$
    fi
