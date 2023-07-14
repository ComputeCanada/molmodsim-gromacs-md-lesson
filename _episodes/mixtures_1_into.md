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

Sometimes we don't want to simulate a protein in water, but rather a mixture 
of small molecules that form a liquid.

In this example we want to create a system of a mixture consisting of:

* 100 molecules N-heptane (SMILES: `CCCCCCC`), 
* 100 molecules of iso-octane (SMILES: `CC(C)CC(C)(C)C`) and 
*  50 molecules of phenol (SMILES: `Oc1ccccc1`).

First we need files with the 3D structures for those molecules. While we
could create them manually with a molecule editor, or download structure
files from one of the many databases, we will instead use the tool
[OpenBabel][openbabel] to generate MDL-Mol-files from SMILES strings
that we have looked up from Wikipedia.


## Load Modules and Prepare a Directory

We start by logging in to one of the Alliance clusters, loading
the `openbabel` module and creating a new directory where we will
build our system.

~~~
$ module load gcc/9.3.0 openbabel/3.1.1
$ mkdir -p ~/scratch/molmodsim_workshop_gromacs/mixtures_amber/structures
$ cd       ~/scratch/molmodsim_workshop_gromacs/mixtures_amber/structures
~~~
{: .language-bash }


## Use OpenBabel to generate MDL-Mol-files

As of summer of 2023, the [OpenBabel website][openbabel], is rather outdated
and only links to outdated documentation.  Newer documentation can be found
at [https://open-babel.readthedocs.io/][openbabel-docs]. 
For this we are especially interested in the [obabel command][obabel-manual].

That page describes the general syntax as:

`obabel -:"<text>"  [-i <input-ID>] [-o <output-ID>] [-O outfile] [OPTIONS]`

Where `<text>` is our SMILES string. The `<input-ID>`  can be either `smi`
or `smiles`.  As output format (`<output-ID>`) we choose `mdl` for the MDL-MOL format.
We should use the --gen3D option to generate 3D-coordinates and use `--title`
to write a tile with the name of the molecule into the file.

~~~
$ obabel -:"CCCCCCC"        --gen3D --title "n-heptane"  -o mdl  -O n-heptane.mol
$ obabel -:"CC(C)CC(C)(C)C" --gen3D --title "iso-octane" -o mdl  -O iso-octane.mol
$ obabel -:"Oc1ccccc1"      --gen3D --title "phenol"     -o mdl  -O phenol.mol
~~~
{: .language-bash }



[openbabel]: http://openbabel.org/
[openbabel-docs]: https://open-babel.readthedocs.io/en/latest/
[obabel-manual]: https://open-babel.readthedocs.io/en/latest/Command-line_tools/babel.html