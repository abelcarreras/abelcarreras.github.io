On development features
=======================

RAS-CI implementation scheme
----------------------------
Fortran functions only

.. image :: images/rasci_scheme.pdf
    :align: center
    :target: ../images/rasci_scheme.pdf


.. include:: rasci_contract.rst

.. image :: images/rasci_contract_even.pdf
    :target: ../images/rasci_contract_even.pdf


.. image :: images/rasci_contract_odd.pdf
    :target: ../images/rasci_contract_odd.pdf

Fragment localization
---------------------
* *ras_nfrag*: Number of fragments
* *ras_nfrag_atoms*: Number of atoms in each fragment (atoms should be correlative in input file)
* *ras_frag_st*: localization algorithm to use (0: Boys,  1: Sequential [https://aip.scitation.org/doi/10.1063/1.4904292])
* *ras_frag_sets*: Define orbitals to localize (default: localize all)

example::

	ras_nfrag        4 !NFrag   ! # of fragments
	ras_nfrag_atoms  [34,32,34,32]! # atoms in each fragment
	RAS_FRAG_ST      1        ! 0: Boys   1: Sequential
	ras_frag_sets    [86,191,4,4] ! sets of orbitals to localize

Spin polarization
-----------------
Perform spin polarization treatment for srDFT using the method described in: Coulsonm C. A. Fisher, I. Notes on the molecular Orbital Tratment of the Hydrogen Molecule. Philos. Mag. 1949, 40, 386-393. 

* *ras_srdft_spinpol*: Activate spin polarization (False: Deactivate(default), True: Activate)

Sequential Diabatization
------------------------

To activate the  diabatization analysis of excited states in ras-ci method it is necessary to define
a section block and 3 **REM** keywords in the qchem input. The section block defines the excited states
number to be included in the diabatization (by increasing order). This is equivalent to the one used
in **TDDFT diabatization method** (*check qchem manual for further information*).

This block can be placed at the end of the usual input (after the **$END** of the **$REM** section) and
has an structure like this ::

    $localized_diabatization
    adiabatic states
    n1 n2 n3 n4 n5 n6
    $end

where *n1, n2, n3,* etc.. are the excited state number (sorted by increasing energy) to be used as
reference. Also the use of this block requires a keyword to indicate the number of adiabatic states
defined ::

    sts_multi_nroots N_ad

where *N_ad* are the number of adibatic states defined in **$localized_diabatization** block.

Once the states to be used in the diabatization analysis are defined it is necessary to request
a diabatization calculation using the keyword ::

    cis_diabath_decompose N

where *N* is the number of sequential steps in the diabatization process (if *N=true*, this is equivalent to 1).
A sequential step is complete diabatization process using a particular set of states and a particular
diabatization method. In usual diabatization calculations this is just 1, but in the current implementation
it is possible to link sequential diabatization steps using a different number of states and method to obtain
more sophisticated results.

Then it is necessary to define the diabatization method and the excited states to use (with respect to the
reference states). To do this we have to define the keyword ::

  ras_diab_seq_data    [n1_1,n2_1,n3_1,meth_1,data_1,n1_2,n2_2,meth_2,data_2]

where *n1_1, n2_1, n3_1* are the state numbers to be used in the 1st step of the diabatization. These
step number are taken respect to the reference states defined in $localized_diabatization section
(so the 1st reference state is 1, 2nd reference state is 2, etc..). *meth_1* indicates the diabatization
method to be used in the 1st step, this method is defined by an integer number where ::

   1  -> ED method   (data ignored)
   2  -> Boys method (data ignored)
   3  -> DQ method   (data: float between 0.0-1.0, fraction of quadrupole/dipole)

finally, *data_1* indicates the additional parameters for the method. Currently only **DQ** makes use of this
parameter, but *data* value must be defined for all methods (even if it is ignored in the diabatization method).

this structure is repeated for each diabatization step requested. In the above example *n1_2,n2_2,meth_2,data_2*
are the parameters of the second diabatization step.
In order to keep track of which states/data correspond to each diabatization step it is necessary to define
another keyword that contains the list of number of states for each diabatization step ::

   ras_diab_seq_list    [sn1 ,sn2]

where *sn1* and *sn2* are number of states to use in each diabatization state (not counting for method and data,
just number of diabatic states)


After each diabatization step the resulting diabatic states are reordered by increasing energy, and this
order will be the one used to select the states for the next diabatization step.


This is an example of diabatization input::

   $rem
   ras_diab_seq_list    [7,4]
   ras_diab_seq_data    [1,2,3,4,5,6,7,2,0.0,4,5,6,7,3,1.0]
   sts_multi_nroots  7
   $end

   $localized_diabatization
   adiabatic states
   2 3 4 5 6 7 8
   $end

In this example 2 diabatization steps are performed, the first one uses all 7 diabatic states defined
in the **$localized_diabatization** block (1-7) using *method* **2** (Boys) and *data* **0.0** (which
is ignored for this method). The second step uses the 4 highest energy diabatic states (4-7) obtained
from the previous step and performs a diabatization using *method* **3** (DQ) with a *parameter* **1.0**
(100% quadrupole).
