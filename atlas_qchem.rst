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

Note: QCSCRATCH environment variable defines the directory where the scratch files will be generated  during qchem execution. In batch jobs ideally this should be under the scratch directory tree. If this variable is not defined it will be set to current directory.
