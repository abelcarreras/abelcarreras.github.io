How to use qchem in Mare Nostrum (BSC)
======================================

Preliminar information
----------------------

**Requirements to acces to qchem**

* Have a MareNostrum account (mn1.bsc.es or mn2.bsc.es)

**Available qhem compilations**

* qchem_group_mpi: Current in development version of qchem parallel MPI [mpi intel mkl version]


Load custom modules
-------------------

To acces to qchem you should be able to load custom modules located in /home/hpce17/hpce17274/privatemodules privatemodules. To do this you may modify the MODULEPATH environment variable as follows:: 

    export MODULEPATH=/home/hpce17/hpce17274/privatemodules/:$MODULEPATH

Once this is done, you should be able to see the custom modules (qchem_trunk & qchem_group) executing the command::

    module avail

To use qchem, just load the corresponding module as usual::

    module qchem_group_mpi

*Note: $MODULEPATH environment variable definition can be defined in user's ~/.bashrc configuration file so
it becomes defined for all calculation by default.*

Run interactive qchem
---------------------

SLURM queue system allows to run calculation interactivelly for testing purposes using a single node up to 4 cores. 
To enter to them use the command::

    salloc --partition=interactive -n 4

there is a time limit in interactive mode of aprox. 1 hour. For similar purposes debug queue can also be uses with similar time limitations but allowing to use a larger number of cores::

    salloc --qos=debug -n 96


Batch job example parallel MPI
------------------------------

All commands described previously can be gathered in a simple script to run in batch. This is a simple example::

	#!/bin/bash
	#SBATCH --job-name="test qchem"                     
	#SBATCH --workdir=/home/hpce17/hpce17274/test/test1  
	#SBATCH --output=test_water.log      # standard output dile
	#SBATCH --error=test_water.err       # standard error file
	#SBATCH --ntasks-per-node=4          # number of cores per node
	#SBATCH --nodes=1                    # number of nodes (1 recomended)
	#SBATCH --time=01:00:00              # Max time for this job hh:mm:ss     

	# load module qchem
	export MODULEPATH=/home/hpce17/hpce17274/privatemodules/:$MODULEPATH
	module load qchem_group_mpi

	# set scrach directory
	export QCSCRATCH=/home/hpce17/hpce17274/scratch/qchem_scratch/

	# launch qchem
	qchem -np $SLURM_NTASKS_PER_NODE water_dimer.in water_dime.out
