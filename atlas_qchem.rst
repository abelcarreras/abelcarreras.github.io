How to use qchem in ATLAS cluster (DIPC)
========================================

Requirements to acces to qchem
------------------------------

**Account**
- Have an atlas account (atlas.sw.ehu.es)
- Be a member of dccqhem linux group

**Available qhem compilations**
- qchem_trunk: Current master version of qchem (will be released in next stable version)
- qchem_group: Current in development version of qchem (includes new features developed in the group) 

Load custom modules
-------------------

o acces to qchem you should be able to load custom modules located in /scratch/abel/SOFTWARE/privatemodules. To do this you may modify the MODULEPATH environment variable as follows: 

export MODULEPATH=/scratch/abel/SOFTWARE/privatemodules:$MODULEPATH

Once this is done, you should be able to see the custom modules (qchem_trunk & qchem_group) executing the command::

    module avail

To use qchem, just load the corresponding module as usual::

    module qchem_trunk

or::

    module qchem_group

*Note: QCSCRATCH environment variable defines the directory where the scratch files will be generated  during qchem execution. In batch jobs ideally this should be under the scratch directory tree. If this variable is not defined it will be set to current directory.*

Run interactive qchem
---------------------

In order to run simple SHORT (< 1min) tests you may execute them directly in shell environment (once everything is loaded properly)::

    qchem input_file.inp output_file.out

For proper run long calculations you should use the queue system to submit batch calculations.
For this purpose you should write a batch script.

Batch job example
-----------------

All commands discussed previously can be gathered in a simple script to run in batch. This is a simple example:

``#!/bin/bash

#PBS -q parallel   #    “parallel" o “qchem” depending on the queue you have access
#PBS -l ncpus=8  # number of CPI
#PBS -l cput=80:00:00  # CPU time (WallTime * Ncpus) maximum running time (hh:mm:ss)
#PBS -N qchem_calc  # job name
#PBS -e error.log  # queue system error output file
#PBS -o output.log  # queue system standard output file

export MODULEPATH=/scratch/abel/SOFTWARE/privatemodules:$MODULEPATH

module load qchem_group

cd $PBS_O_WORKDIR   #  set workdir to submission directory 
qchem -nt 8 test.inp  test.out`
