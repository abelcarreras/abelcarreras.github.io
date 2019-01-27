How to use qchem in IQTC cluster (UB)
=====================================

Preliminar information
----------------------

**Requirements to acces to qchem**

* Have an iqtc account (portal.qt.ub.edu)
* Be a member of g8 group (Prof. Pere Alemany's group)

**Available qhem compilations**

* qchem_group: Current in development version of qchem (includes new features developed in the group) [serial intel_compilers mkl compilation]
* qchem_mpi: parallel MPI compilation of qchem_group [mpi intel mkl version]


*All qchem compilations are compilated in iqtc04 cluster. It is not guaranteed to work properly on other clusters.*

Load custom modules
-------------------

o acces to qchem you should be able to load custom modules located in /home/g8abel/privatemodules. To do this you may modify the MODULEPATH environment variable as follows:: 

    export MODULEPATH=/home/g8abel/privatemodules:$MODULEPATH

Once this is done, you should be able to see the custom modules (qchem_trunk & qchem_group) executing the command::

    module avail

To use qchem, just load the corresponding module as usual::

    module qchem_group

or::

    module qchem_mpi

*Note: *$QCSCRATCH* environment variable defines the directory where the scratch files will be generated  during qchem execution. In batch jobs ideally this should be under the scratch directory tree. If this variable is not defined it will be set to current directory.*


Define hostfile (MPI parallel version only)
-------------------------------------------

In order to run MPI version of qchem the location of hostfile must be defined. The complete path (including the filename) should be defined in the environment variable **$QCMACHINEFILE**

Under queue system environemnt (SGE) this file is generated automatically and placed in a temporal directory under the node /scratch directory. This directory is written in **$TMPDIR** environtment variable.
Therefore, in batch calcultions, a convenient way is to define **$QCMACHINEFILE** as::

    export QCMACHINEFILE=$TMPDIR/machines

Run interactive qchem
---------------------

SGE queue system allows to run calculation interactivelly using special queues designed for this purpose.
To enter to them use the command::

    qrsh -q interactive04.q

where  interactive04.q is the interactive queue in iqt04 cluster. Other similar queues are available for iqtc01 and iqtc02: interactive01.q and interactive02.q. Keep in mind that when working with interactive queues MPI hostfile must be created manually. In simple case use this file may only containg one line::

   localhost

Batch job example serial
------------------------

All commands discussed previously can be gathered in a simple script to run in batch. This is a simple example::

	#!/bin/bash
	#$ -S /bin/bash
	#$ -N test_serial  # job name
	#$ -q iqtc04.q     # custer where to run (iqtc04 recommended)
	#$ -pe smp 1       # define the parallel environment (single node) and number of cores

    # load module qchem
	. /etc/profile
	export MODULEPATH=/home/g8abel/privatemodules:$MODULEPATH
	module load qchem_group

	# define envirotment qchem
	export QCSCRATCH=$TMPDIR

	# run qchem
	qchem inputfile.inp outputfile.out

Batch job example parallel MPI
------------------------------

All commands discussed previously can be gathered in a simple script to run in batch. This is a simple example::

	#!/bin/bash
	#$ -S /bin/bash
	#$ -N test_mpi    # job name
	#$ -q iqtc04.q    # custer where to run (iqtc04 recommended)
	#$ -pe mpi* 8     # define enviroment to MPI and number of processors


    # load module qchem
	. /etc/profile
	export MODULEPATH=/home/g8abel/privatemodules:$MODULEPATH
	module load qchem_mpi

	# define envirotment qchem
	export QCSCRATCH=$TMPDIR
	export QCMACHINEFILE=$TMPDIR/machines

	# run qchem (-np indicates the number of processors. You may want to use the same as in "-pe")
	qchem -np 8 inputfile.inp outputfile.out
