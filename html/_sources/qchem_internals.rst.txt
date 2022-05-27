Development Q-Chem
==================

How to add new REM variable
---------------------------
REM variables are, in principle, integer only variables. However some 
tricks can be done to use them as real.

New REM variables should be defined in file::

   config/rem.list


this file contains the list of all REM variables used in q-chem. Make sure your new
variable does not already exist. Each variable is associated to a number, the range
of numbers available for each user must be reserved in the upper section of the same
file. Make sure to not use more variables than the number of reserved numbers.

Each defined variable should be initializated in file::

   libgen/rem_default.C

The way to initalize a variable is by writting a line as::

   rem_write(0,NAME_OF_VARIABLE);

where 0 is the initial value(integer).

To use REM variables as real, the variable should be define as :math:`10^n` times the
intended value, where *n* is the maximum number of decimal places of accepted real.
This is defined in **RemDoubleToValue** function placed in file::
   
   qparser/read_rem.C

which should be edited to add the variable definition like::

   if (Loc == NAME_OF_VARIABLE){
     double dVal = TL[1].GetDouble();
     Val = INTEGER(dVal*10000);
     OK = 1;
   }

Then, within your code, in order to recover the intended real number the REM variable
should be divided by :math:`10^n` ::

   myrealvariable = rem_read(NAME_OF_VARIABLE) / 10000


How to add new scratch FILE
---------------------------
scratch files are store momory that can be used globally along q-chem code. These files
are intended to be used to store large matrices (avoiding to have them in RAM) but are
also useful to access data from different parts of the code.

This files are defined in ::

   include/fileman.h

where a positive integer number must be assigned to each file. Be careful to not set a
number already in use.

Manage scratch files
--------------------

**FileMan** function manages the scratch data stored in files. They are useful to store large
arrays in one section of the code to be used in another part.

Before using a file it has to be defined in *include/fileman.h*. This file contains a list of
all files defined along Q-Chem where each one is associated to a number. This number has to be
unique and is more or less taken in a first come first served basis. In order to organize this file
a little, developers usually reserve some range of numbers for themselves by writing a short
comment.

Opening file
^^^^^^^^^^^^
To read/store data the file first should be opened ::

    # Open to write data
    FileMan(FM_OPEN_W, FILE_NAME, 0, 0, 0, 0, 0);

    # Open to read data
    FileMan(FM_OPEN_R, FILE_NAME, 0, 0, 0, 0, 0);

where **FM_OPEN_R**/**FM_OPEN_W** labels indicate the type. These labels are already defined in Q-Chem
and are available everywhere. **FILE_NAME** is the name of the file defined in *fileman.h* file.

.. note::
    It is interesting to be noted that all these labels are just variables that contain integer numbers
    defined in the internals of Q-Chem. Its probably done this way for interoperability C/Fortran.

Writing data
^^^^^^^^^^^^

To store data in the file ::

    double data[N];
    FileMan(FM_WRITE,FILE_NAME,FM_DP,N,0,FM_BEG, &data);

where **FM_WRITE** labels indicates that the data will be written, **FM_DP** label indicates the data type
(double), **FM_BEG** indicates that the data will be stored from the beginning of the file and **data** is
the array of data to be stored.

.. note::
    Note that *data* argument should be passed to *FileMan* as a memory address.


Reading data
^^^^^^^^^^^^

To determine the length of the array ::

    int N;
    FileMan(FM_GET_LEN,FILE_NAME,FM_INT,0,0,0,&N);

where **FM_GET_LEN** indicates that the length will be retrieved, **FM_INT** indicates the data type
(integer in this case) and **N** is the variable that will contain the array length.

To read the data form file ::

    double data[N];
    FileMan(FM_READ,FILE_NAME,FM_DP,N,0,FM_BEG,&data);

where **FM_READ** indicated reading mode, and **N** is the length of the array.


Closing file
^^^^^^^^^^^^

Once the data is read/written the file should be close by ::

    FileMan(FM_CLOSE, FILE_NAME, 0, 0, 0, 0, 0);


Final notes
^^^^^^^^^^^

When working with armadillo objects (matrices & vectors) the raw data can be accessed
by the methods *.begin()* & *.memptr*. This exposes the memory address which allows
to use FileMan directly on these arrays. The difference between this methods is that *.memptr*
is read only, so it is safer than *.begin()* if the data have only to be read from the array ::

    arma::Col<double> data_array(N);

    FileMan(FM_WRITE,FILE_NAME,FM_DP,N,0,FM_BEG,data_array.begin());
    FileMan(FM_READ,FILE_NAME,FM_DP,N,0,FM_BEG,data_array.memptr);

Memory info inquisition
-----------------------
To obtain the amount of (static) memory available in mega-array use function: ::

    available_mem = MegLen()

this returns the memory in Qwords (blocks of 8 bytes). To transform to megabytes (MB)
(like in Q-Chem the input) units you can use something like: ::

    available_mem = float(MegLen())/(125*1000)

or alternatively to transform it to mebibytes (MiB) which is common in UNIX based systems use ::

    available_mem = float(MegLen())/(128*1024)

The total static memory of the mega-array in the same units can be obtained using the keyword: ::

    total_mem = MegTot()
    total_mem = float(MegTot())/(125*1000)

which corresponds to the requested static memory in the Q-Chem input contained in the REM variable (in MB): ::

    rem_read(REM_MEM_STATIC)

The total requested memory (static + dynamic) for the calculation in MB is found in the REM variable: ::

    rem_read(REM_MEM_TOTAL)


Adding new DFT functionals
--------------------------

The files containing the main functions of the functional are placed in either::

    *libfunc* or *libdft/libfunc*

To be callable everywhere these functions should be included in ::

    include/extref.list

The C functions prototypes are written in the header ::

    libdft/libdft.h

The definition of components of the functional (evaluations, derivatives, 2nd deribatives)
are writen here ::

    libdft/evalxc.C

The connection with the Q-Chem input interface is defined here ::

    libdft/Functionals.C
    libdft/dftcodes.C


Basis in tests
--------------
::

    1 3 3 3  # number of atoms, lines in block 1, lines in block 2, number of function blocks

      0.0 0.0 0.0   # coordinates x y z of atom 1

      # bock 1 (exponents)
      1.61277758800000e-01
      7.27000000000000e-01
      1.05700000000000e+00

      # block 2
      1.81380649178652e-01
      9.56881375059566e-01
      1.81359656261779e+00

    # function blocks
    0 0 0 0 1 1 1   # atom center, unknown, unknown, starting coeff block 1, num coeff block 1, unknown, unknown
    0 0             # starting coeff in block 2, num coeff block 2
    1 0             # function type (0: s, 1:p, 2:d ..), type (0: Cartesian, 1:pure)

    0 0 0 1 1 3 3
    1 1
    1 0

    0 0 0 2 1 5 6
    2 2
    1 1




Angular Momentum
----------------
Calling angular momentum:

.. code-block:: fortran

    REAL*8  Lx(NBas, NBas), Ly(NBas, NBas), Lz(NBas, NBas) ! Angmom matrices (output)
    REAL*8 cx, cy, cz  ! Center coordinates (input)

    CALL angmom_mat(Lx, Ly, Lz, cx, cy, cz)

    write(6,*) 'Lx'
    DO i=1, NBas
        write(6,*) (Lx(i, j), j = 1, NBas)
    ENDDO

    write(6,*) 'Ly'
    DO i=1,NBas
        write(6,*) (Ly(i, j), j = 1, NBas)
    ENDDO

    write(6,*) 'Lz'
    DO i=1,NBas
        write(6,*) (Lz(i, j), j = 1, NBas)
    ENDDO


Other
-----

.. code-block:: c++

    extern "C" void ras_test_() {

        cout << " ----- STARTING TEST -----" << endl;
        using libqints::qchem::aobasis;

        INTEGER bCode   = rem_read(REM_IBASIS);
        BasisSet s1(bCode);
        OneEMtrx oneE(bCode);
        int NBas   = s1.getNBasis();
        s1.PrintInfo();
        int NOrb   = oneE.getNOrb();
        cout << "NOrb/NBas:" << NOrb << "/" << NBas << endl;


        RasCIStates ras;

        double energy_j, energy_k, energy_h;
        double energy_n = GeometryData().get_nuclear_repulsion();

        DensityMatrix canonic;
        energy_j = CoulombMatrix(canonic).get_full_contraction(canonic);
        energy_k = ExchangeMatrix(canonic).get_full_contraction(canonic);
        energy_h = HCoreMatrix().get_full_contraction(canonic);


        cout << "J: " << energy_j << endl;
        cout << "K: " << energy_k << "/" << -0.5 * energy_k << endl;
        cout << "H: " << energy_h << endl;

        double tot_energy2 = energy_j - 0.5 * energy_k + energy_h + energy_n;
        cout << "Total energy: " << tot_energy2 << endl;
        cout << CoulombMatrix(canonic).get_matrix_ao() << endl;

        energy_j = CoulombMatrix2(canonic).get_full_contraction(canonic);
        cout << "J2: " << energy_j << endl;
        cout << CoulombMatrix2(canonic).get_matrix_ao() << endl;

        energy_k = ExchangeMatrix2(canonic).get_full_contraction(canonic);
        cout << "K2: " << energy_k << "/" << -0.5 * energy_k << endl;

        return;

        //  Set up memory and threading (also only need to be done once)
        libqints::dev_omp qints_dev;
        qints_dev.init(8UL *1024*1024);

        int NBasis   = bSetMgr.crntShlsStats(STAT_NBASIS);

        //int NBasis = b1.get_nbsf();

        // These should be initialized properly before creating arrays below
        arma::mat J(NBasis, NBasis), K(NBasis, NBasis);
        arma::mat Ja(NBasis, NBasis), Ka(NBasis, NBasis);
        arma::mat Jb(NBasis, NBasis), Kb(NBasis, NBasis);

        arma::mat P = arma::eye(NBasis, NBasis);
        arma::mat Pa = arma::eye(NBasis, NBasis);
        arma::mat Pb = arma::eye(NBasis, NBasis);

        P = canonic.get_matrix_ao();
        Pa = canonic.get_matrix_ao_alpha();
        Pb = canonic.get_matrix_ao_beta();


        //P(1, 1) = -1;

        //  Reorder density matrix in-place for libqints ordering of basis functions
        libqints::gto::reorder_cc(P, aobasis.b1, true, true, libqints::gto::korder, libqints::gto::lex);
        libqints::gto::reorder_cc(Pa, aobasis.b1, true, true, libqints::gto::korder, libqints::gto::lex);
        libqints::gto::reorder_cc(Pb, aobasis.b1, true, true, libqints::gto::korder, libqints::gto::lex);

        //  Density matrix array (input)
        libqints::array_view<double> av_p(P.memptr(), NBasis*NBasis);
        libqints::array_view<double> av_Pa(Pa.memptr(), NBasis*NBasis);
        libqints::array_view<double> av_Pb(Pb.memptr(), NBasis*NBasis);

        //  J matrix array (output)
        libqints::array_view<double> av_j(J.memptr(), NBasis*NBasis);
        libqints::array_view<double> av_Ja(Ja.memptr(), NBasis*NBasis);
        libqints::array_view<double> av_Jb(Jb.memptr(), NBasis*NBasis);

        //  K matrix array (output)
        libqints::array_view<double> av_k(K.memptr(), NBasis*NBasis);
        libqints::array_view<double> av_Ka(Ka.memptr(), NBasis*NBasis);
        libqints::array_view<double> av_Kb(Kb.memptr(), NBasis*NBasis);

        int justAlTemp = rem_read(REM_JUSTAL);
        int spinTemp = rem_read(REM_SET_SPIN);
        //rem_write(1,REM_JUSTAL);
        //rem_write(1,REM_SET_SPIN);

        //  Run the JK-build job
        libqints::op_coulomb op;
        libfock::jk_engine<double>(op, aobasis.b4, qints_dev).compute(av_p, av_j, av_k);
        libfock::jk_engine<double>(op, aobasis.b4, qints_dev).compute(av_Pa, av_Pb, av_Ja, av_Jb, av_Ka, av_Kb);


        std::cout << "Orbitals" << std::endl;
        std::cout << canonic.get_matrix_ao_alpha() << std::endl;
        std::cout << canonic.get_matrix_ao_beta() << std::endl;
        std::cout << canonic.get_matrix_mo_alpha() << std::endl;
        std::cout << canonic.get_matrix_mo_beta() << std::endl;
        std::cout << canonic.get_matrix_mo() << std::endl;


        libqints::gto::reorder_cc(P, aobasis.b1, true, true, libqints::gto::lex, libqints::gto::korder);
        libqints::gto::reorder_cc(J, aobasis.b1, true, true, libqints::gto::lex, libqints::gto::korder);
        libqints::gto::reorder_cc(K, aobasis.b1, true, true, libqints::gto::lex, libqints::gto::korder);

        std::cout << "P" << std::endl;
        std::cout << P << std::endl;
        std::cout << "J" << std::endl;
        std::cout << J << std::endl;
        std::cout << "K" << std::endl;
        std::cout << K << std::endl;

        std::cout << "Jtot(full): " << arma::dot(P, J)/2 << std::endl;
        std::cout << "Ja: " << arma::dot(Pa, Ja) << std::endl;
        std::cout << "Jb: " << arma::dot(Pb, Jb) << std::endl;
        std::cout << "Jtot: " << arma::dot(Pa, Ja) + arma::dot(Pb, Jb) << std::endl;

        std::cout << "Ktot: " << arma::dot(P, K) << std::endl;
        std::cout << "Ka: " << -0.5 * arma::dot(Pa, Ka) << std::endl;
        std::cout << "Kb: " << -0.5 * arma::dot(Pb, Kb) << std::endl;
        //std::cout << "Ktot: " << arma::dot(Pa, Ka) + arma::dot(Pb, Kb) << std::endl;

        cout << "H: " << energy_h << endl;
        cout << "nuc: " << energy_n << endl;

        double tot_energy3 = arma::dot(P, J)/2 - 0.5 * arma::dot(P, K)/2 + energy_h + energy_n;
        cout << "Total energy: " << tot_energy3 << endl;
        double tot_energy4 = arma::dot(P, J)/2 - 0.5 * (arma::dot(Pa, Ka) + arma::dot(Pb, Kb)) + energy_h + energy_n;
        cout << "Total energy(f): " << tot_energy4 << endl;


        std::vector<double> Lx(NBasis * NBasis, 0.0);
        std::vector<double> Ly(NBasis * NBasis, 0.0);
        std::vector<double> Lz(NBasis * NBasis, 0.0);
        libqints::array_view<double> av_Lx(&Lx[0], Lx.size());
        libqints::array_view<double> av_Ly(&Ly[0], Ly.size());
        libqints::array_view<double> av_Lz(&Lz[0], Lz.size());
        libqints::multi_array<double> ma_L(3);
        ma_L.set(0, av_Lx);
        ma_L.set(1, av_Ly);
        ma_L.set(2, av_Lz);

        double cx = (std::rand() % 1000) / 500.0 *0.0;
        double cy = (std::rand() % 1000) / 500.0 *0.0;
        double cz = (std::rand() % 1000) / 500.0 *0.0;

        std::cout << "Cx" << cx << ", " << cy << ", " << cz << std::endl;

        libqints::basis_1e2c_cgto<double> b2i(aobasis.b1, aobasis.b1);


        libqints::dig_passthru_1e2c_cgto<double, 3> dig(b2i, ma_L);
        libqints::op_angmomentum L(cx, cy, cz);
        // libqints::qints_job qj(L, aobasis.b2, qints_dev);
        libfock::make_angmommat(aobasis.b1, aobasis.b1, av_Lx, av_Ly, av_Lz, cx, cy, cz);

        libqints::scr_null<libqints::bat_1e2c_cgto<double> > scr;
        //libqints::qints(L, b2i, scr, dig, qints_dev);


        arma::Mat<double> arma_Lx(&Lx[0], NBasis, NBasis);
        arma::Mat<double> arma_Ly(&Ly[0], NBasis, NBasis);
        arma::Mat<double> arma_Lz(&Lz[0], NBasis, NBasis);

        // Reorder to Q-Chem ordering
        libqints::gto::reorder_cc(arma_Lx, aobasis.b1, true, true, libqints::gto::lex, libqints::gto::korder);
        libqints::gto::reorder_cc(arma_Ly, aobasis.b1, true, true, libqints::gto::lex, libqints::gto::korder);
        libqints::gto::reorder_cc(arma_Lz, aobasis.b1, true, true, libqints::gto::lex, libqints::gto::korder);

        // Explicitely antisymmetrize integrals to kill numeric noise...
        arma_Lx = 0.5 * (arma_Lx - arma_Lx.t());
        arma_Ly = 0.5 * (arma_Ly - arma_Ly.t());
        arma_Lz = 0.5 * (arma_Lz - arma_Lz.t());

        std::cout << "Lx" << std::endl;
        std::cout << arma_Lx << std::endl;
        std::cout << arma::dot(arma_Lx, P) << std::endl;

        std::cout << "Ly" << std::endl;
        std::cout << arma_Ly << std::endl;
        std::cout << arma::dot(arma_Ly, P) << std::endl;

        std::cout << "Lz" << std::endl;
        std::cout << arma_Lz << std::endl;
        std::cout << arma::dot(arma_Lz, P) << std::endl;

        AngularMomentOperator angmom(1.0, 1.0, 1.0);

        std::cout << "Lx2" << std::endl;
        std::cout << angmom.get_matrix_ao(0) - arma_Lx << std::endl;
        std::cout << "Ly2" << std::endl;
        std::cout << angmom.get_matrix_ao(1) - arma_Ly << std::endl;
        std::cout << "Lz2" << std::endl;
        std::cout << angmom.get_matrix_ao(2) - arma_Lz << std::endl;

    }

    // test function
    void fer_test() {

        int NB2Car = rem_read(REM_NB2CAR);
        int NBas   = bSetMgr.crntShlsStats(STAT_NBASIS);

        arma::vec pdm1_v;
        arma::vec J_v;

        arma::mat pdm1;
        arma::mat pdm2;
        arma::mat J;

        pdm1.set_size(NBas, NBas);
        pdm2.set_size(NBas, NBas);
        pdm1.zeros();
        pdm2.zeros();

        pdm1.at(0,0) = 0.523;
        pdm1.at(5,5) = -0.523;
        pdm1.at(0,5) = 0.0;
        pdm1.at(5,0) = 0.0;

        pdm2.at(14,14) = 0.523;
        pdm2.at(19,19) = -0.523;
        pdm2.at(14,19) = 0.0;
        pdm2.at(19,14) = 0.0;

        J.set_size(NBas, NBas);

        pdm1_v.set_size(NB2Car);
        J_v.set_size(NB2Car);

        ScaV2M(pdm1.memptr(), pdm1_v.begin(), true, false);

        AOIntsJobInfo JobInfo(20, TenMin(rem_read(REM_ITHRSH)));
        //JobInfo.TurnIntScreenOn();
        ShlPrs s2Bra(DEF_ID);

        AOints(J_v.begin(),NULL, NULL,NULL,pdm1_v.memptr(),
               NULL, NULL, NULL,NULL, JobInfo, s2Bra, s2Bra);


        ScaV2M(J.begin(), J_v.memptr(), true, true);
        cout << "J: " << J << endl;
        cout << "energy test: " << arma::accu(pdm2 % J) << " : " << arma::accu(pdm2 % J) * ConvFac(HARTREES_TO_EV) << endl;

        MultipoleMomentOperator mp(false);
        double dp1 = -arma::accu(pdm1 % mp.get_quadrupole_matrix(0));
        double dp2 = -arma::accu(pdm1 % mp.get_quadrupole_matrix(1));
        double dp3 = -arma::accu(pdm1 % mp.get_quadrupole_matrix(2));
        cout << "dipole1 " << dp1 << " "<< dp2 << " " << dp3 << endl;

        dp1 = -arma::accu(pdm2 % mp.get_quadrupole_matrix(0));
        dp2 = -arma::accu(pdm2 % mp.get_quadrupole_matrix(1));
        dp3 = -arma::accu(pdm2 % mp.get_quadrupole_matrix(2));
        cout << "dipole2 " << dp1 << " "<< dp2 << " " << dp3 << endl;

        MultipoleMomentOperator mp2(true);
        DensityMatrix pdm1_gs; // HF density
        cout << "quadrupole: " << mp2.get_quadrupole_moment(pdm1_gs) << endl;

    }
