On development features
=======================

RAS-CI implementation scheme
----------------------------
Fortran functions only

.. image :: images/rasci_scheme.pdf
    :align: center

RAS-CI Contraction scheme  :math:`(N_{\alpha}=N_{\beta})`
----------------------------
Scheme for the *RAS_Contrac(V,ItrRAS,M,N)* routine:

* USE Reduced_Lists
* USE Addressing
* Retrieve Fock, integrals
* Set parameters and dimensions
* Allocate amplitude vectors (jB) and responses (jR)

* Loop Roots
	* read jB
	* Loop i1 = 1,M
		* IF(Hole and (N :math:`\leq` (M-1)))
			* Loop MNAHa=M1N: LM1N
				* Fis: <Act|F|Hole>
				* build: LAHa(MNAHa), SgnAHa, doLAHa=true
		* IF(Part)
			* Loop MNAPa=M1N1: LM1N1
				* Fas: <Act|F|Part> 
				* build: LAPa(MNAPa), SgnAPa, doLAPa=true
	* Loop i2 = 1,i1
		* define: iFock, iXvv, iXoo1, iXoo2
		* IF(Hole and (N :math:`\leq` (M-1)))
			* RAS_FormXah: LAHb(MNAHb), SgnAHb, doLAHb=true
			* Loop LAHa
				* Loop LAHb
					* <HoleA|V|HoleB> (iXoo1)
					* IF(i1 :math:`\neq` i2) <HoleA|V|HoleB> (iXoo2)
		* IF(Part)
			* RAS_FormXap: LAPb(MNAPb), SgnAPb, doLAPb=true
			* Loop MNAPa: LAPa
				* Loop MNAPb: LAPb
					* <PartA|V|PartB> (iXvv)
					* IF(i1 :math:`\neq` i2) <PartA|V|PartB> (iXvv)
				* IF(doLAHb)
					* Loop MNAHb: LAHb
						* <PartA|V|HoleB>
		* IF(i1 :math:`\neq` i2)
			* IF(doLAHa and doLAPb)
				* Loop MNAHa: LAHa
					* Loop MNAPb: LAPb
						* <PartA|V|HoleB>
			* IF((M-2) :math:`\geq` (N-1))
				* Loop M2N1: LM2N1
					* :math:`F_{ss'}` = <Act|F|Act><Act|Act>
					* IF(Hole) 
						* :math:`F_{ss'}` = <Act|F|Act><Hole|Hole> 
						* (ss'|ij) = <Hole|V|Hole><Act|V|Act>
					* IF(Part)
						* :math:`F_{ss'}` = <Act|F|Act><Part|Part>
						* (ss'|ab) = <Part|V|Part><Act|V|Act>
						* IF(Hole)
							* (ps|hs') = <Hole|V|Part><Act|V|Act>

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
