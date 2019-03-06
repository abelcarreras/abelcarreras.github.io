On development features
=======================

RAS-CI implementation scheme
----------------------------
Fortran functions only

.. image :: images/rasci_scheme.pdf
    :align: center

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
