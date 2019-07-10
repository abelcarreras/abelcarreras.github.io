RAS-CI Contraction scheme  :math:`(N_{\alpha}=N_{\beta})`
---------------------------------------------------------
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
		* IF(doLAHa & doLAPb)
			* Loop MNAHa: LAHa
				* Loop MNAPb: LAPb
					* (sp|s'h) = <Part|V|Act> <Act|V|Hole>
		* IF((M-2) :math:`\geq` (N-1))
			* Loop M2N1: LM2N1
				* :math:`F_{ss'}` = <Act|F|Act> <Act|Act>
				* IF(Hole)
					* :math:`F_{ss'}` = <Act|F|Act> <Hole|Hole>
					* (ss' | ij) = <Hole|V|Hole> <Act|V|Act>
				* IF(Part)
					* (ss' | ab) = <Part|V|Part> <Act|V|Act>
					* :math:`F_{ss'}` = <Act|F|Act> <Part|Part>
					* IF(Hole)
						* (ps|hs') = <Hole|V|Part> <Act|Act>
        * IF(Hole & (M-2) :math:`\geq` N)
            * Loop M2N: LM2N
                * :math:`F_{ss'}` = <Hole|F|Hole> <Act|Act>
                * (ss' | ij) = <Hole|V|Hole> <Act|Act>
                * (si | s'j) = <Hole|V|Hole> <Act|Act>
        * IF(Part & N :math:`\geq` 2)
            * Loop M2N2: LM2N2
                * (ss' | ab) = <Part|V|Part> <Act|Act>
                * (sa | s'b) = <Part|V|Part> <Act|Act>
                * :math:`F_{ss'}` = <Part|F|Part> <Act|Act>
    * IF(i1 = i2)
        * Loop M1N1: LM1N1
            * :math:`F_{ss}` = <Act|F|Act> <Act|Act>
            * IF(Hole)
                * :math:`F_{ss}` = <Act|F|Act> <Hole|Hole>
                * (ss | ij) = <Hole|V|Hole> <Act|V|Act>
            * IF(Part)
                * (ss | ab) = <Part|V|Part> <Act|V|Act>
				* :math:`F_{ss}` = <Act|F|Act> <Part|Part>

        * IF(Hole & (M-1) :math:`\geq` N)
            * Loop M1N: LM1N
                * :math:`F_{ss}` = <Hole|F|Hole> <Act|Act>
                * (ss | ij) = <Hole|V|Hole> <Act|Act>
                * (si | sj) = <Hole|V|Hole> <Act|Act>
        * IF(Part & N :math:`\geq` 2)
            * Loop M1N2: LM1N2
                * (ss | ab) = <Part|V|Part> <Act|Act>
                * (sa | sb) = <Part|V|Part> <Act|Act>
                * :math:`F_{ss}` = <Part|F|Part> <Act|Act>
    * Loop i3 = 1,i1
        * Loop i4 = 1,i4max
            * define: iXssss, Iijkl
            * FormXij: Lb,SgnB;LHb,SgnHb;LPb,SgnPb
            * Loop La
                * Loop Lb
                    * (ss|ss) = <Act|V|Act> <Act|V|Act>
                * IF(Hole & MNHb>0)
                    * Loop LHb
                        * (ss|ss) = <Hole|V|Hole> <Act|V|Act>
                * IF(Part & MNPb>0)
                    * Loop LPb
                        * (ss|ss) = <Part|V|Part> <Act|V|Act>
            * IF(:math:`I_{ijk}` :math:`\neq` 1,7 & (Hole.OR.Part))
                * Loop Lb
                    * IF(Hole & MNHa>0)
                        * Loop LHa
                            * (ss|ss) = <Hole|V|Hole> <Act|V|Act>
                    * IF(Part & MNPa>0)
                        * Loop LPa
                            * (ss|ss) = <Part|V|Part> <Act|V|Act>
    * IF( :math:`I_{ijkl}` = 1,3,5)
        * Free LB, LHb, LPb
        * cycle i4
    * ELSEIF( :math:`I_{ijkl}` = 2,7 & M :math:`\geq` 2)
        * IF(N :math:`\geq` 2)
            * Loop LM2N2
                * (ss|ss) = <Act|V|Act> <Act|Act>
                * IF(Hole)
                    * (ss|ss) = <Act|V|Act> <Hole|Hole>
                * IF(Part)
                    * (ss|ss) = <Act|V|Act> <Part|Part>
        * IF(Hole & (M-2) :math:`\geq` (N-1))
            * Loop LM2N1
                * (ss|ss) = <Hole|V|Hole> <Act|Act>
        * IF(Part & N :math:`\geq` 3)
            * Loop LM2N3
                * (ss|ss) = <Part|V|Part> <Act|Act>
    * ELSEIF( :math:`I_{ijkl}` = 4,6,8,9,10 & M :math:`\geq` 3)
        * IF(N :math:`\geq` 2 & (M-3) :math:`\geq` (N-2))
            * Loop LM3N2
                * (ss|ss) = <Act|V|Act> <Act|Act>
                * IF(Hole)
                    * (ss|ss) = <Act|V|Act> <Hole|Hole>
                * IF(Part)
                    * (ss|ss) = <Act|V|Act> <Part|Part>
        * IF(Hole & (M-3) :math:`\geq` (N-1))
            * Loop LM3N1
                * (ss|ss) = <Hole|V|Hole> <Act|Act>
        * IF(Part & N :math:`\geq` 3)
            * Loop LM3N3
                * (ss|ss) = <Part|V|Part> <Act|Act>
    * ELSEIF( :math:`I_{ijkl}` = 11 & M :math:`\geq` 4)
        * IF(N :math:`\geq` 2 & (M-4) :math:`\geq` (N-2))
            * Loop LM4N2
                * (ss|ss) = <Act|V|Act> <Act|Act> (2 times)
                * IF(Hole)
                    * (ss|ss) = <Act|V|Act> <Hole|Hole> (2 times)
                * IF(Part)
                    * (ss|ss) = <Act|V|Act> <Part|Part> (2 times)
        * IF(Part & N  :math:`\geq` 3 & (M-4) :math:`\geq` (N-3))
            * Loop LM4N3
                * (ss|ss) = <Part|V|Part> <Act|Act> (2 times)
    * Free Lb, LHb, LPb
    * END i4, i3
    * IF(Hole .OR. Part)
        * Loop i3 = 1,M
            * IF(N < 1) cycle i3
            * Loop La(MNa)
                * IF(Hole & (M-1) :math:`\geq` N)
                    * Loop LM1N
                        * (ss|si) = <Act|V|Hole> <Act|V|Act>
                        * IF( :math:`I_{ijkl}\geq` 9)
                            *  (ss|si) = <Act|V|Hole> <Act|V|Act>
                * IF(Part)
                    * Loop LM1N1
                        * (ss|sa) = <Act|V|Part> <Act|V|Act>
                        * IF( :math:`I_{ijkl}\geq` 9)
                            * (ss|sa) = <Act|V|Part> <Act|V|Act>
            * NMin = 1; IF(NO Hole) NMin = 2
            * IF(N < NMin .OR. :math:`I_{ijkl}` = 5)
                * cycle i3
            * ELSEIF( :math:`I_{ijkl}` = 6,9,10)
                * IF(Hole & (M-2) :math:`\geq` (N-1))
                    * Loop LM2N1
                        * (ss|si) = <Act|V|Hole> <Act|V|Act>
                * IF(Part & N :math:`\geq` 2)
                    * Loop LM2N2
                        * (ss|sa) = <Act|V|Part> <Act|V|Act>
            * ELSEIF( :math:`I_{ijkl}` = 11 & M :math:`\geq` 3)
                * IF(Hole & (M-3) :math:`\geq` (N-1))
                   * Loop LM3N1
                        * (ss|si) = <Act|V|Hole> <Act|V|Act> (2 times)
                * IF(Part & N :math:`\geq` 2 & (M-3) :math:`\geq` (N-2))
                    * Loop LM3N2
                        * (ss|sa) = <Act|V|Part> <Act|V|Act> (2 times)
    * END i3, i2, i1
    * IF(Part)
        * Fab = <Part|F|Part> <Act|Act>
    * IF(Hole)
        * Fij = <Hole|F|Hole> <Act|Act>

    * Write response vector to disk
* End Roots

