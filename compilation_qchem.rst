Qchem compilation notes
=======================

To compile qchem in ATLAS cluster in DIPC using MKL it is necessary to modify FinMKL.cmake placed in cmake directory in order to set the correct library paths and environment variables. A convenient  way is to modify line 213 from::

	set(MKL_VERSION “10.1")

to::

	set(MKL_VERSION "11.0")


To compile MPI version in IQTC it is necessary to modify file /bin/qchem to force activation of  **-np** flag::

    set WITH_MPI = 1  

Also it is necessary to modify /bin/parallel.csh file to manually define the environment variable $QCMACHINEFILE to properly set “machines” file location by commenting the following lines::

	if ( $?QCGMAKE ) then
	   set QCMACHINEFILE=$QCBIN/machines
	else
	   set QCMACHINEFILE=$QC/bin/mpi/machines
	fi
