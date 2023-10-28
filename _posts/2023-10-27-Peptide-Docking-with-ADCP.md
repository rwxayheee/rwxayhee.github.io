---
layout: post
title: "Peptide Docking with ADCP, Post-processing, and Induced-fit Optimization with Vina"
author: "rwxayheee"
categories: journal
tags: [documentation]
image: adcp-our-opt.jpg
---

# Intro

This post describes **peptide docking** using [Autodock CrankPep (ADCP)](https://ccsb.scripps.edu/adcp) (Version 1.0 rc1), some **post-processing** including [converting the output structure of the docked peptide into a charged RDKit molecule](https://github.com/rwxayheee/prepare_peptide_ligand), and finally a very primitive **"induced-fit" refinement** using the local optimization function, `optimize`, [from AutoDock Vina's Python bindings](https://autodock-vina.readthedocs.io/en/latest/vina.html#vina.vina.Vina.optimize). The efforts were based on the current major version of ADCP as of Oct 2023 with the intent to **enable post-processing of ADCP outcomes by Vina and RDKit in Python**. 

The provided example is based on an experimental structure of Spt6-Iws1(Spn1) complex from *E. cuniculi* ([PDB ID: 2XPP](https://www.rcsb.org/structure/2xpp)). The crystal structure 2XPP contains two chains: The longer one is Iws1 and will be the receptor in the docking calculation. The shorter one is a truncated form of Spt6. In addition to the mentioned structure, the Spt6-Iws1 complex is also observed in a high-res EM structure of actively working RNA polymerase II elongation complex ([PDB ID: 7XN7](https://www.rcsb.org/structure/7xn7)). The peptide sequence `FFEIF` is part of the IWS1-interacting domain of Spt6 in *E. cuniculi*. Similar sequences present in Spt6 across different species, with a highly conserved F residue that fits the binding site on the Iws1 receptor. The precense of multiple F residues makes the 5-mer sequence from *E. cuniculi* an interesting sequence to explore in peptide docking calculations. 

# Overview

This post will walk you through a peptide docking project consisting of (1) docking calculation, (2) post-processing and (3) structure refinement. 

To reproduce the presented work, you must have the software dependencies: 
+ [ADFR Suite](https://ccsb.scripps.edu/adfr/downloads/) (v1.0 rc1, including `ADCP`, `reduce`, `prepare_ligand`, `prepare_receptor`)
+ [Meeko](https://github.com/forlilab/Meeko) (Python Bindings, v0.5)
+ [AutoDock Vina](https://github.com/ccsb-scripps/AutoDock-Vina) (Python Bindings, v1.2.5) 

# Table of Contents

* [Step 1: Docking Calculation with ADCP](#step-1-docking-calculation-with-adcp)

## Step 1: Docking Calculation with ADCP

In this step, we will perform docking calculations for a peptide from sequence `FFEIF` and the receptor Iws1 from crystal structure 2XPP. A top-ranked binding mode will be chosen for post-processing and optimization in the further steps. 

### Structure and Target Preparation

Following the steps in the [ADCP documentation](https://ccsb.scripps.edu/adcp/tutorial-redocking/), we will **generate the PDBQT files for the peptide and the receptor, and the TRG file for the receptor** - 

```shell
reduce 2xpp_iws1.pdb > 2xpp_recH.pdb;
```

It should be noted that this will not protonate any of the imidazole nitrogens on HIS sidechains, unless with option `-NOFLILP` (or `-HIS` or `-BUILD`). 

```shell
reduce 2xpp_FFEIF.pdb > 2xpp_pepH.pdb;
prepare_receptor -r 2xpp_recH.pdb -o 2xpp_recH.pdbqt;
prepare_ligand -l 2xpp_pepH.pdb -o 2xpp_pepH.pdbqt;
agfr -r 2xpp_recH.pdbqt -l 2xpp_pepH.pdbqt -asv 1.1 -o 2xpp;
```

In addition to running with Shell (preferably [zsh](https://en.wikipedia.org/wiki/Z_shell)) scripts, such commands can always be executed from Jupyter Notebook - 

```python
import subprocess
from subprocess import PIPE
def shell(cmd):
    process = subprocess.Popen(cmd, shell=True, stdout = subprocess.PIPE, stderr = subprocess.PIPE)
    out, err = process.communicate()
    outlines = out.decode("utf-8").split("\n")
    errlines = err.decode("utf-8").split("\n")
    for line in outlines+errlines:
        print(line)
```

### Docking Calculations

