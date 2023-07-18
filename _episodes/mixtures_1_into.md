---
title: "Simulating a Mixture of Organic Liquids"
teaching: 0
exercises: 0
questions:
- "How do I simulate a mixture of small molecules?"
objectives:
- "Learn about forcefields for general small molecules."
keypoints:
- "First key point. Brief Answer to questions. (FIXME)"
---

## Introduction

Sometimes we don't want to simulate a protein in water, but rather a mixture 
of small molecules that form a liquid.

In this example we want to create a system of a mixture consisting of:

* 100 molecules of [n-heptane][heptane] (SMILES: `CCCCCCC`), 
* 100 molecules of [iso-octane][isooctane] (*2,2,4-Trimethylpentane*; SMILES: `CC(C)CC(C)(C)C`) and 
*  50 molecules of [phenol][phenol] (SMILES: `Oc1ccccc1`).

FIXME: add image of a box with this mixture

### Drafting a Plan
We will need:

* 3D-structure files for all of the different molecules
* Forcefield topologies for all of the different molecules
* create a large simulation box in which we insert the desired molecules in 
  the desired numbers
* run a short simulation at high pressure to compress (shrink) the box and force
  the molecules into a liquid state
* equilibrate the system at the desired temperature and pressure

First we need files with the 3D structures for those molecules. While we
could create them manually with a molecule editor, or download structure
files from one of the many databases like ChemSpider or the NIST Chemistry WebBook.
Instead we will use the tool [OpenBabel][openbabel] to generate MDL-Mol-files 
from [SMILES][SMILES] strings that we have looked up from Wikipedia.

For generating forcefield topologies for the molecules we have several options:

1. We can go with the *Generalized AMBER Force Field* (GAFF). The [*AmberTools*][ambertools],
   which are available as a module in the software stack on all of the Alliances HPC systems,
   contain AnteChamber to generate GAFF topologies in a format suitable for AMBER and NAMD.
   [ACPYPE][acpype] is a tool written in Python, that can be used as an interface for
   AnteChamber and convert topologies into a format suitable for GROMACS.

2. The official [*CHARMM General Force Field* (CGenFF) server][cgenff] can be used to 
   generate topologies that are compatible with the CHARMM family of force fields.
   These can be downloaded in various formats, including for GROMACS.

3. The [LigParGen server][ligpargen] from the Jorgensen-group can generate topologies
   for *OPLS-AA* (Optimized Potential for Liquid Simulations - All-Atoms) forcefield
   These can be downloaded in various formats, including for GROMACS.
   A command-line version for this exists as well, but requires access to the group's
   BOSS software, which is currently not available in our software-stack.

The [ATB-server][atb] can be used to generate topologies for the *GROMOS* forcefield.
However the GROMOS force fields have been parameterized with a physically incorrect 
multiple-time-stepping scheme using a twin-range cut-off.
The ability to use this GROMOS scheme for cut-offs was removed in GROMACS 4.5
released in 2010.  All subsequent versions of GROMACS can therefore no longer be used
to reproduce all properties of the GROMOS force fields.
For this reason, using the GROMOS force field in GROMACS is no longer physically valid.

#### SMILES
The [simplified molecular-input line-entry system (SMILES)][SMILES] is a line notation
for chemical structures that basically specifies a number of atoms and how they are
bonded.  SMILES strings are often used in the field of cheminformatics to represent
chemical structures in a compact form.

For the purpose of obtaining structure files with 3D-coordinate for the molecules
of interest, they have the advantage of encoding the atoms and bonds, which can be
used by Open Babel to generate 3D-coordinates by applying standard bond-lengths and
-angles without the need of internet connection to query an external database.

Using the IUPAC InChi would have served the same purpose. 

## Prepare structure files and molecule-topologies
### Load Modules and Prepare a Directory

We start by logging in to one of the Alliance clusters, loading
the `openbabel` module and creating a new directory where we will
build our system.

~~~
$ module load gcc/9.3.0 openbabel/3.1.1
$ mkdir -p ~/scratch/molmodsim_workshop_gromacs/mixtures_amber
$ cd       ~/scratch/molmodsim_workshop_gromacs/mixtures_amber
$ mkdir structures/
~~~
{: .language-bash }

### Use OpenBabel to generate MDL-Mol-files

As of summer of 2023, the [OpenBabel website][openbabel], is rather outdated
and only links to outdated documentation.  Newer documentation can be found
at [https://open-babel.readthedocs.io/][openbabel-docs]. 
For this we are especially interested in the [`obabel` command][obabel-manual].

That page describes the general syntax as:

`obabel -:"<text>"  [-i <input-ID>] [-o <output-ID>] [-O outfile] [OPTIONS]`

Where `<text>` is our SMILES string. The `<input-ID>`  can be either `smi`
or `smiles`.  As output format (`<output-ID>`) we choose `mdl` for the MDL-MOL format.
We should use the --gen3D option to generate 3D-coordinates and use `--title`
to write a tile with the name of the molecule into the file.

~~~
$ obabel -:"CCCCCCC"        --gen3D --title "n-heptane"  -o mdl -O structures/n-heptane.mol
$ obabel -:"CC(C)CC(C)(C)C" --gen3D --title "iso-octane" -o mdl -O structures/iso-octane.mol
$ obabel -:"Oc1ccccc1"      --gen3D --title "phenol"     -o mdl -O structures/phenol.mol
$ ls structures/
~~~
{: .language-bash }

~~~
iso-octane.mol  n-heptane.mol  phenol.mol
~~~
{: .output}

### Load modules for AmberTools and Create Python Environment for Acpype

~~~
$ module load ambertools/23
$ virtualenv venv_acpype
$ source venv_acpype/bin/activate
(venv_acpype) $ pip install acpype
~~~
{: .language-bash }
~~~
Looking in links: /cvmfs/soft.computecanada.ca/custom/python/wheelhouse/gentoo/avx512, /cvmfs/soft.computecanada.ca/custom/python/wheelhouse/gentoo/avx2, /cvmfs/soft.computecanada.ca/custom/python/wheelhouse/gentoo/generic, /cvmfs/soft.computecanada.ca/custom/python/wheelhouse/generic
Collecting acpype
  Using cached acpype-2022.7.21-py3-none-any.whl (7.3 MB)
Installing collected packages: acpype
Successfully installed acpype-2022.7.21
~~~
{: .output}

At this point we should have two directories `structures/` and `venv_acpype/` in our
working directory:
~~~
(venv_acpype) $ ls
~~~
{: .language-bash }
~~~
structures/
venv_acpype/
~~~
{: .output}

### Generate GAFF topologies with `ACPYPE`

~~~
(venv_acpype)$ acpype --input ./structures/n-heptane.mol --basename heptane \
          --net_charge 0 --charge_method bcc --atom_type gaff2 --outtop gmx 
~~~
{: .language-bash }
~~~
===========================================================================
| ACPYPE: AnteChamber PYthon Parser interfacE v. 2022.7.21 (c) 2023 AWSdS |
===========================================================================
==> ... charge set to 0
==> Executing Antechamber...
==> * Antechamber OK *
==> * Parmchk OK *
==> Executing Tleap...
==> * Tleap OK *
==> Removing temporary files...
==> Using OpenBabel v.3.1.0

==> Writing GROMACS files

==> Disambiguating lower and uppercase atomtypes in GMX top file, even if identical.

==> Writing GMX dihedrals for GMX 4.5 and higher.

==> Writing pickle file heptane.pkl
==> Removing temporary files...
Total time of execution: 2s
~~~
{: .output}

Generating the topology for heptane was successful. Let's see what files have been generated by ACPYPE:
~~~
(venv_acpype)$ ls heptane.acpype/
~~~
{: .language-bash }
~~~
acpype.log                 ATOMTYPE.INF            heptane_GMX.gro       md.mdp
ANTECHAMBER_AC.AC          em.mdp                  heptane_GMX.itp       n-heptane.mol
ANTECHAMBER_AC.AC0         heptane_AC.frcmod       heptane_GMX_OPLS.itp  posre_heptane.itp
ANTECHAMBER_AM1BCC.AC      heptane_AC.inpcrd       heptane_GMX_OPLS.top  rungmx.sh
ANTECHAMBER_AM1BCC_PRE.AC  heptane_AC.lib          heptane_GMX.top       sqm.in
ANTECHAMBER_BOND_TYPE.AC   heptane_AC.prmtop       heptane.pkl           sqm.out
ANTECHAMBER_BOND_TYPE.AC0  heptane_bcc_gaff2.mol2  leap.log              sqm.pdb
~~~
{: .output}

Most of the files are not needed for use with GROMACS. It's probably a good idea to keep a copy of them for future reference. 
The `acpype.log` contains more detailed information on what steps ACPYPE performed to generate the topology. 
The file `sqm.out` contains the output of a QM calculation, using the semi-empirical AM1 method, to calculate the partial charges for the molecule.

The files that pertain to GROMACS are `heptane_GMX*`, `posre_heptane.itp`, `em.mdp`, `md.mdp` and `rungmx.sh`

| Filename                | Description                                 |
|:------------------------|:------------------------------------------  |
| `heptane_GMX.gro`       | coordinate file in GRO format               |
| `heptane_GMX.top`       | GROMACS system topology file                |
| `heptane_GMX.itp`       | "include" topology for the molecule         |
| `heptane_GMX_OPLS.itp`  | system topology using OPLS-AA forcefield    |
| `heptane_GMX_OPLS.top`  | include topology using OPLS-AA forcefield   |
| `posre_heptane.itp`     | position-restraints file                    |
| `em.mdp`                | MDP (MD-parameter) for energy minimization  |
| `md.mdp`                | MDP file for MD (10,000 steps)              |
| `rungmx.sh`             | script to quickly run em and md on this     |
{: width="750" }


Now we can continue with generating the topologies foe iso-octane and phenol:

~~~
(venv_acpype)$ acpype --input ./structures/iso-octane.mol --basename iso-octane \
      --net_charge 0 --charge_method bcc --atom_type gaff2 --outtop gmx 
(venv_acpype)$ acpype --input ./structures/phenol.mol --basename phenol \
      --net_charge 0 --charge_method bcc --atom_type gaff2 --outtop gmx 
(venv_acpype)$ ls
~~~
{: .language-bash }
~~~
heptane.acpype/    iso-octane.acpype/    phenol.acpype/  
structures/        venv_acpype/
~~~
{: .output}


[acpype]: https://github.com/alanwilter/acpype#readme
[ambertools]: http://ambermd.org/AmberTools.php
[atb]: https://atb.uq.edu.au/
[cgenff]: https://cgenff.umaryland.edu/
[heptane]: https://en.wikipedia.org/wiki/Heptane
[isooctane]: https://en.wikipedia.org/wiki/2,2,4-Trimethylpentane
[ligpargen]: http://zarbi.chem.yale.edu/ligpargen/
[obabel-manual]: https://open-babel.readthedocs.io/en/latest/Command-line_tools/babel.html
[openbabel-docs]: https://open-babel.readthedocs.io/en/latest/
[openbabel]: http://openbabel.org/
[phenol]:  https://en.wikipedia.org/wiki/Phenol
[SMILES]: https://en.wikipedia.org/wiki/SMILES