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


Fractional Orbital Density (FOD)
--------------------------------

Generate the fraction orbital density of a RAS-CI excited state.

* *gui 2* : Request generation of *.fchk* file
* *ras_natorb_state* i : Select *i* excited state for FOD calculation
* *ras_fod*: Activate FOD (False: Deactivate(default), True: Activate)

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

   1  -> ER method   (data ignored)
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

.. note:: In the recent versions of Q-Chem, Mulliken analysis of the diabatic/adiabatic states is not calculated by default
          (including Attachment/Detachment). To do this analysis use:
          **STATE_ANALYSIS=TRUE**


SOC Natural Transition Orbitals (Spinless triplet density matrix NTOs)
----------------------------------------------------------------------

The calculation of Natural Transition Orbitals (NTO) is requested using the following keywords: ::

    STATE_ANALYSIS = True
    GUI = 2

These keywords will print the NTO’s in the *fchk* file with titles: ::

    "NTOs occupancies (x,y)”
    "NTOs U coefficients (x,y)”
    "NTOs V coefficients (x,y)”

Where x and y denote the two states involved in the transition.
The format of the NTO's is the same as for the Natural Orbitals (NO) so they can be
visualized using any standard visualization software by changing the title names.


Wave function analysis of RAS-CI states
---------------------------------------

Analysis of RAS-CI states is requested using **STATE_ANALYSIS** keyword. This analysis include
a calculation of the Natural Orbitals (NO), Transition Natural Orbitals (NTO), Spinless triplet
density matrix NTOs (SOC-NTO), Fractional Occupation Density (FOD), Electronic Density, Spin Density
and Transition Density. These properties can be plotted in Cube format or Molden format for each
RAS-CI state, this is controlled by **PLOTS** keyword.

Molden format (NO, NTO and SOC-NTO only)::

    STATE_ANALYSIS = True
    PLOTS = 0     ! Note: This can be omitted, default is 0
    NTO_PAIRS = 2 ! Note: This can be omitted, default is 2
    GUI 0

Cube format ::

    STATE_ANALYSIS = True
    NTO_PAIRS = 2 ! Note: This can be omitted, default is 2
    PLOTS = 1
    GUI 0

    $plots
       grid_points                    50 50 50
       grid_range  (-8,8) (-8,8) (-8,8)
    $end

.. note::
   **PLOTS = 1** requires **$plots** section to be written in Q-Chem input using new plot format.
   (https://manual.q-chem.com/5.1/sect-plots.html). **grid_range** can be omitted to use automatic
   range adjusted to the molecular size.

.. warning::
    The number of NTO pairs written is controlled by **NTO_PAIRS** keyword. If not specified the value is set to 2.

.. warning::
    To plot data in both Cube and Molden files **GUI** should be set to 0. If not a **fchk** will be generated
    instead. At this moment is not possible to plot Molden/Cubes/fchk at the same time.

Using **STATE_ANALYSIS = 2**  will compute and plot interstate properties for all pairs of states.
This is required to compute and plot SOC-NTO.

Molden format (NO and NTO only)::

    STATE_ANALYSIS = 2
    PLOTS = 0  ! Note: This can be omitted, default is 0
    GUI 0


Cube format ::

    STATE_ANALYSIS = 2
    PLOTS = 1
    GUI 0

    $plots
       grid_points                    50 50 50
    $end


The NTO information will be written in the output as ::

   e-/hole pair  1 alpha:  ampl =  0.450353 ( 20.3%)   [3 : 4]
   e-/hole pair  2 alpha:  ampl =  0.203136 (  4.1%)   [2 : 5]
   e-/hole pair  1 beta :  ampl =  0.311186 (  9.7%)   [3 : 4]
   e-/hole pair  2 beta :  ampl =  0.279777 (  7.8%)   [2 : 5]

where the last two numbers "[3: 4]" indicate the cubefile numbers that corresponds of the NTO pair (electron/hole).


Notes about diabatization in TDDFT method
-----------------------------------------

Due to a possible bug in Q-Chem a change of behavior appeared in Qchem v5.x
respect to Mulliken analysis of diabatic states. Now in addition to *cis_ampl_anal* the keyword::

    NAMD_NSURFACES 0

is required to analyze the diabatc states. If not set, the adiabatic states are analyzed instead.

Example for ethene dimer::

    $molecule
    0 1
    C     0.0000000   0.0000000   0.6660120
    C     0.0000000   0.0000000   -0.6660120
    H     0.0000000   0.9228100   1.2279200
    H     0.0000000   -0.9228100  1.2279200
    H     0.0000000   -0.9228100  -1.2279200
    H     0.0000000   0.9228100   -1.2279200
    C     4.2000000   0.0000000   0.6660120
    C     4.2000000   0.0000000   -0.6660120
    H     4.2000000   0.9228100   1.2279200
    H     4.2000000   -0.9228100  1.2279200
    H     4.2000000   -0.9228100  -1.2279200
    H     4.2000000   0.9228100   -1.2279200
    $end

    $rem
    JOBTYPE      sp
    EXCHANGE     hf
    cis_n_roots  8
    cis_singlets true
    cis_triplets false
    RPA          false
    BASIS        6-31G
    cis_ampl_anal  true
    mem_static   900
    !Diabat
    NAMD_NSURFACES 0
    loc_cis_ov_separate    false
    er_cis_numstate        4
    cis_diabath_decompose  true
    $end

    $localized_diabatization
    On the next line, list which excited adiabatic states we want to mix.
    1 2 7 8
    $end

