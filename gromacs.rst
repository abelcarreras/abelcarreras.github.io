GROMACS
=======

python API installation
-----------------------

Installing gromacs python API can be tricky.
First gromacs has to be compiled using: ::

    DGMXAPI=one
    DGMX_BUILD_SHARED_LIBS=on
    DGMX_PYTHON_PACKAGE=on

also it has to be installed so make sure that ::

    DCMAKE_INSTALL_PREFIX

is set to the desired install directory

.. note::
    make sure gromacs is compiled in simple precision, if not gmxapi
    installation gives problems.

Then there are some environment variables that should be set: ::

    export GMXTOOLCHAINDIR=/installation_path/share/cmake/gromacs/
    export GROMACS_DIR=/installation_path/share/cmake/gromacs/
    export gmxapi_DIR=/installation_path/


In Gromacs 2025to install the API (at lest in MAC) it is necessary to modify these files:

session.h

.. image :: images/session_h.png

session.cpp

.. image :: images/session_cpp.png
