How to use qchem in ATLAS cluster (DIPC)
========================================

Preliminar information
----------------------

**Requirements to acces to qchem**

* Have an atlas account (atlas.sw.ehu.es)
* Be a member of dccqchem linux group

**Available qchem compilations**

* qchem_trunk: Current master version of Q-Chem [openmp intel_compilers mkl boost]
* qchem_group: Current in development version of Q-Chem (includes new features developed in the group) [openmp intel_compilers mkl boost]

Load custom modules
-------------------

The access to Q-Chem you should be able to load custom modules located in /scratch/abel/SOFTWARE/privatemodules.
To do this you may modify the MODULEPATH environment variable as follows::

    export MODULEPATH=/scratch/abel/SOFTWARE/privatemodules:$MODULEPATH

Once this is done, you should be able to see the custom modules (qchem_trunk & qchem_group) executing the command::

    module avail

To use qchem, just load the corresponding module as usual::

    module qchem_trunk

or::

    module qchem_group


Define scratch dir
------------------
Q-Chem requires to define a scratch directory to store temporal files during execution.
It is recommended to create this directory somewhere under your /scratch/user/ folder.
In principle Q-chem temp files will be deleted after a successful run, however if you
calculation ends with error some (maybe useful) data may remain in this folder. Take care
to clean up this folder from time to time to avoid reaching the max storage quota for your user.

To define your scratch directory in your submit script write the following line ::

    export QCSCRATCH=/scratch/user/myscratchfolder        # set scratch folder for qchem

*Note: QCSCRATCH environment variable defines the directory where the scratch files will be
generated during qchem execution. If this variable is not defined it will be set
to current directory but it is strongly recommended to define it for batch jobs.*

Run interactive qchem
---------------------

In order to run simple SHORT (< 1min) tests you may execute them directly in shell environment (once everything is loaded properly)::

    qchem input_file.inp output_file.out

For proper run long calculations you should use the queue system to submit batch calculations.
For this purpose you should write a batch script.

Batch job example
-----------------

All commands discussed previously can be gathered in a simple script to run in batch.
Check http://dipc.ehu.es/cc/computing_resources/ for more detailed technical info.
This is a simple example::


    #!/bin/bash
    #SBATCH --partition=long    # (chose according to your needs: regular, long, xlong, large, ..)
    #SBATCH --job-name=jobname  # job name
    #SBATCH --cpus-per-task=8   # number of CPU's to use
    #SBATCH --mem=10gb          # RAM memory to use (this has to be coherent with Qchem input script)
    #SBATCH --nodes=1           # run on a single node (for Q-Chem always 1)
    #SBATCH --ntasks-per-node=1 # tasks per node (for Q-Chem always 1)
    #SBATCH -e error.log        # queue system custom error output file (optional)
    #SBATCH -o output.log       # queue system custom standard output file (optional)

    export MODULEPATH=/scratch/abel/SOFTWARE/privatemodules:$MODULEPATH  # load custom modules
    export OMP_NUM_THREADS=$SLURM_CPUS_PER_TASK                          # define OMP_NUM_THREADS
    export QCSCRATCH=/scratch/user/myscratchfolder        # set scratch folder for qchem

    module load qchem_group     # load q-qchem module (this can be either qchem_group or qchem_trunk)

    qchem -nt ${SLURM_CPUS_PER_TASK} input_qchem.in output_qchem.out  # Run Q-chem

For some calculations you may want to use *-save* option in qchem. (https://manual.q-chem.com/5.0/sect-running.html)
If you use this option remember that the generated data folder will be stored in the $QCSCRATCH folder you have defined
(and not removed)

.. Note::
    Warning! to run jobs on batch your data must be copied under the directory **/scratch/username**.
    It is recommended to also run your jobs from the same directory.

Local scratch
-------------
According to the documentation in ATLAS cluster users can make use of local storage of the calculation nodes to
put (for example) Q-Chem scratch files. Using local scratch can improve the performance of Q-Chem calculations
up to 4 times. Local scratch is mounted for all nodes in /lscratch directory. This directory has w/r permission
for all users. **Local scratch should be used with care since the users are responsible of cleaning the remaining
data after the calculation is finished.** This cleaning can be written in the submitting script. Here an example: ::

    #!/bin/bash
    #SBATCH --partition=regular
    #SBATCH --job-name=complete
    #SBATCH --cpus-per-task=4
    #SBATCH --mem=16gb
    #SBATCH --nodes=1
    #SBATCH --ntasks-per-node=1

    export MODULEPATH=/scratch/abel/SOFTWARE/privatemodules:$MODULEPATH
    export OMP_NUM_THREADS=$SLURM_CPUS_PER_TASK

    # load Q-Chem module
    module load qchem_group

    # Define Local scratch
    mkdir -p /lscratch/${USER}_${SLURM_JOB_ID}/
    export QCSCRATCH=/lscratch/${USER}_${SLURM_JOB_ID}

    # run Q-Chem
    qchem -nt ${SLURM_CPUS_PER_TASK} input_qchem.in output_qchem.out

    # Clean local scratch
    rm -r /lscratch/${USER}_${SLURM_JOB_ID}/

.. Note::
    Keep in mind that if the calculation crashes or it is cancelled by the user before finishing the cleaning part of the script
    will not be executed. In this case the user should manually enter the node (by ssh *nodename*) and remove the scratch
    data.
