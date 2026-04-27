Undocumented Q-chem features
============================

Multi-state FED
---------------

FED can be calculated using more than 2 states. This is using diabatization
interface so it is needed to create a **$localized_diabatization** section to
define the states to be used. When using multi-state FED a standard FED is also calculated.

Minimum set of keywords needed for multi-FED  ::


    $rem
    ...
    sts_fed True        ! activate FED calculation
    sts_multi_nroots 4  ! number of states to be included in the FED
    sts_multi_print  1  ! (or 2 for more information)
    sts_donor 1-18      ! range of atoms in donor fragment
    sts_acceptor 19-36  ! range of atoms in acceptor fragment
    $end

    $localized_diabatization
      adiabatic states
      1 2 3 4           ! state number of the sates to be included in the calculation
    $end



full example for naphthalene dimer::

    $molecule
    0 1
    C	        0.0000000000         0.7076000000         0.0001000000
    C	        0.0000000000        -0.7076000000         0.0000000000
    C	        1.2250000000         1.3944000000         0.0000000000
    C	        1.2250000000        -1.3944000000         0.0001000000
    C	       -1.2250000000         1.3943000000         0.0000000000
    C	       -1.2250000000        -1.3943000000         0.0000000000
    C	        2.4327000000         0.6959000000        -0.0001000000
    C	        2.4327000000        -0.6958000000         0.0000000000
    C	       -2.4327000000         0.6958000000         0.0000000000
    C	       -2.4327000000        -0.6958000000        -0.0001000000
    H	        1.2489000000         2.4821000000        -0.0001000000
    H	        1.2489000000        -2.4822000000         0.0001000000
    H	       -1.2490000000         2.4821000000         0.0001000000
    H	       -1.2489000000        -2.4822000000        -0.0001000000
    H	        3.3732000000         1.2391000000        -0.0001000000
    H	        3.3733000000        -1.2390000000        -0.0001000000
    H	       -3.3732000000         1.2390000000         0.0000000000
    H	       -3.3733000000        -1.2390000000        -0.0001000000
    C	        0.0000000000         0.7076000000         4.0001000000
    C	        0.0000000000        -0.7076000000         4.0000000000
    C	        1.2250000000         1.3944000000         4.0000000000
    C	        1.2250000000        -1.3944000000         4.0001000000
    C	       -1.2250000000         1.3943000000         4.0000000000
    C	       -1.2250000000        -1.3943000000         4.0000000000
    C	        2.4327000000         0.6959000000         3.9999000000
    C	        2.4327000000        -0.6958000000         4.0000000000
    C	       -2.4327000000         0.6958000000         4.0000000000
    C	       -2.4327000000        -0.6958000000         3.9999000000
    H	        1.2489000000         2.4821000000         3.9999000000
    H	        1.2489000000        -2.4822000000         4.0001000000
    H	       -1.2490000000         2.4821000000         4.0001000000
    H	       -1.2489000000        -2.4822000000         3.9999000000
    H	        3.3732000000         1.2391000000         3.9999000000
    H	        3.3733000000        -1.2390000000         3.9999000000
    H	       -3.3732000000         1.2390000000         4.0000000000
    H	       -3.3733000000        -1.2390000000         3.9999000000
    $end

    $rem
    jobtype sp
    exchange b3lyp
    basis sto-3g
    cis_convergence 14
    cis_n_roots 6
    cis_singlets True
    cis_triplets False
    sts_fed True
    sts_multi_nroots 4
    sts_multi_print 1
    sts_donor 1-18
    sts_acceptor 19-36
    $end

    $localized_diabatization
     adiabatic states
     1 2 3 4
    $end

expected multi-FED output ::

                          FED MULTISTATE ANALYSIS

     Within CIS/TDA Singlet Excited States:

     Localized diabatization specification
       1   2   3   4

     ------------------------------------------------------------------------------
     4 excited states are requested ...
         1 LE (donor)    state(s) found
         2 CT            state(s) found
         1 LE (acceptor) state(s) found
     ------------------------------------------------------------------------------
     Excitation Energy (eV) and Fragment-X Difference of   4 Diabatic States
     ------------------------------------------------------------------------------
       n    Excitation       dX
     ----- ------------ ------------
        1   4.52780500  -0.00000449
        2   4.54916620   0.00000479
        3   5.19733066  -1.93705111
        4   5.19733075   1.93705270
     ------------------------------------------------------------------------------
     Matrix Elements of Diabatic Hamiltonain
     ------------------------------------------------------------------------------
     diabatH(1,2) =  -2.672379E-17
     diabatH(1,3) =  -8.865129E-06
     diabatH(1,4) =    9.37914E-06

     diabatH(2,3) =  -4.560166E-07
     diabatH(2,4) =  -1.565519E-06

     diabatH(3,4) =     0.01839572


