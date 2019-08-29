How to use qchem in IQTC cluster (UB)
=====================================

Preliminar information
----------------------

**Requirements to acces to qchem**

* Have an iqtc account (portal.qt.ub.edu)
* Be a member of g8 group (Prof. Pere Alemany's group)

**Available qchem compilations**

* qchem_group: Development version of qchem [openmp gcc mkl] (iqtc08 only)
* qchem_mpi:   Development version of qchem [mpi intel mkl]  (iqtc04 only)

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

Run interactive qchem in batch queue
------------------------------------
All commands described previously can be gathered in a simple script to run in batch.
In the following section you can find some example scripts for both qchem compilations
available in IQTC.


Batch job example parallel OpenMP (single node) [pe SMP] IQTC08
---------------------------------------------------------------
This is the recommended version for single node calculations. This version should be run in IQTC08. Example::

	#!/bin/bash
	#$ -N test_openmp  # job name
	#$ -S /bin/bash
	#$ -q iqtc08.q     # custer where to run (iqtc08 only)
	#$ -pe smp 8       # define the parallel environment and number of cores

    # load module qchem
	source /etc/profile
	export MODULEPATH=/home/g8abel/privatemodules:$MODULEPATH
	module load qchem_group

	# define envirotment qchem
	export QCSCRATCH=$TMPDIR

	# run qchem
	qchem -nt 8 inputfile.inp outputfile.out

Batch job example parallel MPI (multiple nodes) [pe MPI] IQTC04
---------------------------------------------------------------
This version is used to run on multiple nodes. Use it ony if you plan to
use more processors than available in a single node (>12 in IQTC04).
This version is compiled in IQTC04 and should work best in this cluster.
Here a simple example::

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



Batch job example parallel MPI (single node) [pe SMP] IQTC04
------------------------------------------------------------
For calculations using less than the number of processors available in one node it is strongly
recommended to use the OpenMP version. Still some features of qchem may run faster in MPI compilation.
For this you still can run qchem_mpi version in SMP environment but this requires to setup a "machines"
file manually. To do this, just create a plain text file named $machines$ containing only one line::

    localhost

This file should be placed in a directory accessible from the calculation nodes (e.g. the same directory
that contains your input files). Then you should modify $export QCMACHINEFILE$ line to specify the proper
path to your machines file. This file can be reused for all your calculations, it is not necessary to
create a new "machines" file for each one.

Submit script example::

	#!/bin/bash
	#$ -S /bin/bash
	#$ -N test_mpi    # job name
	#$ -q iqtc04.q    # custer where to run (iqtc04 recommended)
	#$ -pe smp 8      # define environment to SMP and number of processors


    # load module qchem
	. /etc/profile
	export MODULEPATH=/home/g8abel/privatemodules:$MODULEPATH
	module load qchem_mpi

	# define envirotment qchem
	export QCSCRATCH=$TMPDIR
	export QCMACHINEFILE=machine_file_directory/machines

	# run qchem (-np indicates the number of processors. You may want to use the same as in "-pe")
	qchem -np 8 inputfile.inp outputfile.out
