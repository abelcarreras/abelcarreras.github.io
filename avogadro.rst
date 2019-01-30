Custom version of Avogadro visualization software
=================================================

This is a fork from the original repository at gitHub: https://github.com/cryos/avogadro that adds some
extra characteristics to read data from fchk files generated with qchem using the RAS-CI method.

Characteristics
---------------

* Natural orbitals
* Spin density 
* Fractional occupnacy density (FOD)


Repository
----------
This version can be downloaded from GitHub (spin_density branch) at: 

https://github.com/abelcarreras/avogadro/tree/spin_density

General instructions
--------------------

Donwload and compile. You may need to install the following libraries:

* QT4 
* open-babel

Special mac instructions
------------------------

An already compiled version of Avogadro for mac is avalable under request.
To run this version you still need to install the requiered libraries.
A simple way to do it is using brew (https://brew.sh):

Before running avogadro, open a terminal and type:

1. /usr/bin/ruby -e "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/master/install)”
2. brew tap cartr/qt4
3. brew tap-pin cartr/qt4
4. brew install qt@4
5. brew install open-babel

Once the installation is finished you should be able to run avogadro normally.


