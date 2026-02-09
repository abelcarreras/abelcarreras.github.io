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
    #SBATCH --qos=long    # (chose according to your needs: regular, long, xlong, large, ..)
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
    #SBATCH --qos=regular
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

Launcher script
---------------
Creating a batch script can be tedious, especially if many similar calculation have to submitted. Here I prepared a
simple bash command-line app (script) to automate this process. This script creates a batch script from simple data
provided from command line flags and submits the job. This script is particularly designed to submit PyQchem python
scripts but can be easily modified to meet your particular needs ::

    #!/bin/bash

    # Conf varibles
    TEMP_FILE_NAME="temp_launch_script_98263"

    # Set default values for optional arguments
    PARTITION_ARG="regular"
    NAME_ARG="no_name"
    PROC_ARG="8"
    RAM_ARG="10Gb"

    # Function to display usage information
    function usage() {
      echo "Usage: $0 <script_to_run> [--name <no_name>] [--partition <regular>] [--n_proc <8>] [--ram <1Gb>]"
    }

    # Parse command-line arguments using getopts
    while [[ $# -gt 0 ]]; do
      key="$1"

      case "$key" in
        --partition)
          PARTITION_ARG="$2"
          shift # past argument
          shift # past value
          ;;
        --name)
          NAME_ARG="$2"
          shift # past argument
          shift # past value
          ;;
        --n_proc)
          PROC_ARG="$2"
          shift # past argument
          shift # past value
          ;;
        --ram)
          RAM_ARG="$2"
          shift # past argument
          shift # past value
          ;;
        -h|--help)
          usage
          exit 0
          ;;
        *)
          POSITIONAL_ARG="$1"
          shift
          ;;
      esac
    done

    # Check if the positional argument is provided
    if [ -z "$POSITIONAL_ARG" ]; then
      echo "Error: A positional argument is required."
      usage
      exit 1
    fi

    # Display the values of all arguments
    RED="\e[31m"
    GREEN="\e[32m"
    BLUE="\e[34m"
    ENDCOLOR="\e[0m"

    echo
    echo -e " Job name: ${GREEN}   $NAME_ARG ${ENDCOLOR}"
    echo -e " Run script: ${BLUE} $POSITIONAL_ARG ${ENDCOLOR}"
    echo -e " Partition: ${GREEN}  $PARTITION_ARG ${ENDCOLOR}"
    echo -e " Num procs: ${GREEN}  $PROC_ARG ${ENDCOLOR}"
    echo -e " RAM Memory: ${GREEN} $RAM_ARG ${ENDCOLOR}"
    echo

    read -p "Submit? " -n 1 -r
    echo    # (optional) move to a new line
    if [[ ! $REPLY =~ ^[Yy]$ ]]
    then
        exit
    fi

    # Generate launch script
    echo "#!/bin/bash
    #SBATCH --qos=${PARTITION_ARG}
    #SBATCH --job-name=${NAME_ARG}
    #SBATCH --cpus-per-task=${PROC_ARG}
    #SBATCH --mem=${RAM_ARG}
    #SBATCH --nodes=1
    #SBATCH --ntasks-per-node=1

    export MODULEPATH=/scratch/abel/SOFTWARE/privatemodules/:$MODULEPATH
    export OMP_NUM_THREADS=${PROC_ARG}
    #export PYQCHEM_CACHE=1

    module load qchem_group

    python ${POSITIONAL_ARG}
    " > $TEMP_FILE_NAME

    # Submit launch script to queue system
    sbatch $TEMP_FILE_NAME

    # Delete launch script
    rm $TEMP_FILE_NAME

A basic usage is as follows: ::

   $ launcher script.py --partition regular --n_procs 10 --ram 50Gb

.. Note::
    It may be convenient to place this script in some folder included in your $PATH environment variable such that it can be called from
    anywhere in the system.
