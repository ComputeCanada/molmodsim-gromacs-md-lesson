---
title: "GROMACS Topology Files"
teaching: 0
exercises: 0
questions:
- "How are GROMACS topology files organized?"
- "What are typical pitfalls when editing topologies manually?"
objectives:
- "First learning objective. (FIXME)"
keypoints:
- "First key point. Brief Answer to questions. (FIXME)"
---

## A first look at a GROMACS topology file.

A standalone topology file, usually called `topol.top`,  often looks something like this:

~~~
;
;       File 'topol.top' was generated
;       By user: user (1000)
;       On host: login1.mun.ace-net.ca
;       At date: Wed Jul 19 17:02:44 2023
;
;       This is a standalone topology file
;
;       Created by:
;        :-) GROMACS - gmx pdb2gmx, 2023.2-EasyBuild_4.7.2_r59d2f54bdcdfe06afe5574d9d28d40b929cdbfb1 (-:
;       
;       Executable:   /cvmfs/soft.computecanada.ca/easybuild/software/2020/avx512/MPI/gcc9/openmpi4/gromacs/2023.2/bin/gmx
;       Data prefix:  /cvmfs/soft.computecanada.ca/easybuild/software/2020/avx512/MPI/gcc9/openmpi4/gromacs/2023.2
;       Working dir:  /gpfs/project/6002406/molmodsim_workshop_gromacs/topology/ldh
;       Command line:
;         gmx pdb2gmx -f 3LDH_tetramer.pdb -o 3ldh_processed.gro -water tip3p -ff oplsaa
;       Force field was read from the standard GROMACS share directory.
;

; Include forcefield parameters
#include "oplsaa.ff/forcefield.itp"

; Include chain topologies
#include "topol_Protein_chain_A.itp"

; Include water topology
#include "oplsaa.ff/tip3p.itp"

#ifdef POSRES_WATER
; Position restraint for each water oxygen
[ position_restraints ]
;  i funct       fcx        fcy        fcz
   1    1       1000       1000       1000
#endif

; Include topology for ions
#include "oplsaa.ff/ions.itp"

[ system ]
; Name
Protein

[ molecules ]
; Compound        #mols
Protein_chain_A     4
~~~

### Comment lines

Lines starting with a semicolon `;` are treated as comments.

The file usually starts with a comment section that contains how the file was initially generated (e.g. when, by whom, which program and version was used, etc.).
GROMACS tools, like `pdb2gmx`, almost always record the exact command that was used, while Other 3rd-party tools may record less information.

Often additional comments can be found that give information about the (multi-column-)data in the following lines.

### Section Names

Next we see several section headers `[ section_name ]` which each are followed by one or more lines of data.
The main-sections need to appear in a fixed order and can then be followed by certain other sections.

### Preprocessor Directives

We also notice several pre-processor directives, that `grompp` uses to assemble the final
topology.  For that [`grompp`][grompp] has a built-in preprocessor that implements a subset of
the `cpp` processor that programmers most C and C++ programmers are familiar with.

#### `#include "filename.itp"`

* This directive looks for an additional file, in this example called `filename.itp` 
  and will internally paste its content of the file replacing this directive.  
* Any pre-processor directives inside the included file are processed as well.
* Included files are also called "include-topology" and by convention have the `.itp` file extension

#### `#ifdef SOME_WORD` ... `#endif`
* This will include the lines in between these statements only if `SOME_WORD` has been defined.
* Defining a word can be done either by adding a line  `define = -DSOME_WORD` to the `.mdp` file
  or by adding a `#define SOME_WORD` statement into a processed `.top` or `.itp` file.




This file was taken from the data-set "GAFF-BCC-2018" which is available on:
https://virtualchemistry.org/ff.php

~~~
; Topology generated from /home/user/Liquids/MOLECULES/OEP/methane/methane-3-oep.log.gz
; Charge method used bcc
; MOL_GMX.top created by acpype (Rev: 0) on Wed Feb 14 23:09:47 2018

[ defaults ]
; nbfunc        comb-rule       gen-pairs       fudgeLJ fudgeQQ
1               2               yes             0.5     0.8333

[ atomtypes ]
;name   bond_type     mass     charge   ptype   sigma         epsilon       Amb
 c3       c3          0.00000  0.00000   A     3.39967e-01   4.57730e-01 ; 1.91  0.1094
 hc       hc          0.00000  0.00000   A     2.64953e-01   6.56888e-02 ; 1.49  0.0157

[ moleculetype ]
;name            nrexcl
 MOL              3

[ atoms ]
;   nr  type  resi  res  atom  cgnr     charge      mass       ; qtot   bond_type
     1   c3     1   MOL    C1    1    -0.106800     12.01000 ; qtot -0.107
     2   hc     1   MOL    H1    2     0.026700      1.00800 ; qtot -0.080
     3   hc     1   MOL    H2    3     0.026700      1.00800 ; qtot -0.053
     4   hc     1   MOL    H3    4     0.026700      1.00800 ; qtot -0.027
     5   hc     1   MOL    H4    5     0.026700      1.00800 ; qtot -0.000

[ bonds ]
;   ai     aj funct   r             k
     1      2   1    1.0969e-01    2.7665e+05 ;     C1 - H1    
     1      3   1    1.0969e-01    2.7665e+05 ;     C1 - H2    
     1      4   1    1.0969e-01    2.7665e+05 ;     C1 - H3    
     1      5   1    1.0969e-01    2.7665e+05 ;     C1 - H4    

[ angles ]
;   ai     aj     ak    funct   theta         cth
     2      1      3      1    1.0758e+02    3.2970e+02 ;     H1 - C1     - H2    
     2      1      4      1    1.0758e+02    3.2970e+02 ;     H1 - C1     - H3    
     2      1      5      1    1.0758e+02    3.2970e+02 ;     H1 - C1     - H4    
     3      1      4      1    1.0758e+02    3.2970e+02 ;     H2 - C1     - H3    
     3      1      5      1    1.0758e+02    3.2970e+02 ;     H2 - C1     - H4    
     4      1      5      1    1.0758e+02    3.2970e+02 ;     H3 - C1     - H4    

[ system ]
 MOL

[ molecules ]
; Compound        nmols
 MOL              1     
~~~


[grompp]: https://manual.gromacs.org/current/onlinehelp/gmx-grompp.html#gmx-grompp

{% include links.md %}