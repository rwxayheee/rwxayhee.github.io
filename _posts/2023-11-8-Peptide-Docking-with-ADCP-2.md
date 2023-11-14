---
layout: post
title: "Peptide Docking and OpenMM Minimization with ADCPv1.1, Interfacing RDKit, and MM/GBSA Calculation in Amber"
author: "rwxayheee"
categories: journal
tags: [documentation]
image: Vernons-Linking-Rings.jpg
---

# Intro

This post is based on some basic knowledge and familarity with **peptide docking** using [Autodock CrankPep (ADCP)](https://ccsb.scripps.edu/adcp), as described in the [previous post](https://rwxayheee.github.io/Peptide-Docking-with-ADCP). 

The intent of the post is to introduce to our lab members the new features in ADCP v1.1, including the most useful **OpenMM minimization** and the incorporation of D-amino acids, followed by some additional steps that might benefit our own work such as **parsing and exporting the output to RDKit** for post-processing, and finally **to AMBER for the MM/GBSA calculation**. 

# Overview

This post includes our own examples of docking calculations for [standard amino acid (AA)](https://en.wikipedia.org/wiki/Proteinogenic_amino_acid) peptides, cyclic peptides and peptides with D-amino acids which have better support in ADCP v1.1. The post-processing in AutoDock Vina, which we used for ADCP v1.0, is compared with the new post-processing protocol in this post using the OpenMM funtionalities that were embedded in ADCP v1.1. 

## Associated Files

<a href="{{ site.url }}/files/peptide-docking.zip" download>peptide-docking.zip</a>

Contains input PDB files, `2xpp_iws1.pdb` and `2xpp_FFEIF.pdb`, for the peptide docking examples:  

```
peptide-docking/
├── 2xpp_FFEIF.pdb
└── 2xpp_iws1.pdb
```

# Table of Contents

* [Example 1-1 Basic Docking: Docking a Standard AA, 5-mer Peptide in ADCP v1.1](#example-1-1-basic-docking-docking-a-standard-aa-5-mer-peptide-in-adcp-v11)
  + [Docking Preparation](#docking-preparation)
  + [Docking Calculation](#docking-calculation)
* [Example 1-2 Basic Minimization: Using OpenMM for a Two-step Minimization](#example-1-2-basic-minimization-using-openmm-for-a-two-step-minimization)
  + [Gas Phase Minimization](#gas-phase-minimization)
  + [2nd Minimization Preparation](#2nd-minimization-preparation)
  + [Minimization with Implicit Solvent](#minimization-with-implicit-solvent)

* [Example 2-1 Interfacing RDKit: Exporting ADCP raw outputs into RDKit Molecules and Using Vina for Local Optimization]
* [Example 2-2 Interfacing AMBER: Exporting minmized ADCP outputs into AMBER for MM/GBSA Calculation]

* [Example 3 Advanced Docking: Docking a Cyclice Peptide Containing a Disulfide Bond and Pose Selection](#example-3-advanced-docking-docking-a-cyclice-peptide-containing-a-disulfide-bond-and-pose-selection)

* [Example 4 Advanced Docking: Docking a Peptide Containing D-Amino Acids and Backbone Dihedral Validation](#example-4-advanced-docking-docking-a-peptide-containing-d-amino-acids-and-backbone-dihedral-validation)

## Example 1-1 Basic Docking: Docking a Standard AA, 5-mer Peptide in ADCP v1.1

### Docking Preparation
The preparation steps for the docking calculation can be performed the same way as described in section [Structure and Target Preparation](https://rwxayheee.github.io/Peptide-Docking-with-ADCP#structure-and-target-preparation) in the previous post. Additionally, because the tautomers of histidine residues (HIE/HID) will be discriminated in ADCP v1.1 (at least in the post-processing step that uses molecular mechanics), the `NOFLIP` or `FLIP` options may be used to protonate the histidine sidechains in the protein receptor and peptide ligands. 

Here beginning from the receptor PDB file, `2xpp_iws1.pdb`, and a peptide PDB file in the aligned position, `2xpp_FFEIF.pdb`, the following commands were used for **docking preparation** - 

```shell
# initialisation of ADCP v1.1
source ~/.bashrc; # mamba initialize
micromamba activate adcpsuite # activate micromamba env

# ligand preparation
reduce 2xpp_FFEIF.pdb > 2xpp_pepH.pdb; # protonate peptide ligand
prepare_ligand -l 2xpp_pepH.pdb -o 2xpp_pepH.pdbqt # generate peptide ligand PDBQT

# receptor prapration
reduce -NOFLIP 2xpp_iws1.pdb > 2xpp_recH.pdb; # protonate receptor
prepare_receptor -r 2xpp_recH.pdb -o 2xpp_recH.pdbqt; # generate receptor PDBQT
agfr -r 2xpp_recH.pdbqt -l 2xpp_pepH.pdbqt -asv 1.1 -o 2xpp -ng # generate receptor TRG, skip gradient calculation
```

### Docking Calculation

And the command for **docking calculation** was - 

```shell
adcp -O -T 2xpp.trg -s "FFEIF" -N 400 -n 20000000 -o dock1 -L swiss -w dock1 -ref 2xpp_pepH.pdb -nc 0.8 -c 40 &> dock1.log;
```

ADCP v1.1 uses slightly different syntax to read TRG file (`-T`) and there are more options (`-L`, `-w`) to support the additional features. In addition, *the default clustering method has changed from RMSD to contact-based clustering* with a cutoff occupancy of 0.8. A reference structure is optional, but might help to measure the reproducibility or track possible improvements if the docking calculations are run in replicates. 

The above calculation (-N 400 -n 20000000) could take around **1 hours 40 minutes to 2 hours on a 40-core Intel(R) Xeon(R) Gold 6148 CPU @ 2.40GHz**. 

## Example 1-2 Basic Minimization: Using OpenMM for a Two-step Minimization

### Gas Phase Minimization

A completed docking calculation will generate a folder named `dock1` (as specified by the `-o` option in the docking calculation), containing the docking output PDB and DLG files: 

```
dock1
├── dock1_out.pdb
└── dock1_summary.dlg
```

For the reported 100 poses (default if not changed by the `-m` option in the docking calculation), we will first perform a **gas phase minimization** to eliminate the bad contacts in the complex and reduce the strain energies within the peptide ligand - 

```shell
adcp -k -T 2xpp.trg -s "FFEIF" -pdmin -o dock1 -L swiss -w dock1 -nmin 100 -nitr 100 -env vacuum &> min1.log;
```

I set the number of iterations to be 100, as I believe it can yield a reasonable geometry and relatively stable `E_peptide` without changing the docking pose too much. See below for the energy outputs I collected with `-nitr` ranging from 5 to 1000: 

| nitr | E_Complex | E_Receptor | E_Peptide | dE_Interaction | dE_Complex-Receptor |
| --- | --- | --- | --- | --- | --- |
| 5 (Default) | 1038.39 | 676.60 | 562.69 | -200.89 | 361.79 |
| 10 | -1200.51 | -1317.06 | 305.66 | -189.10 | 116.56 |
| 25 | -2362.51 | -2218.40 | 103.79 | -247.91 | -144.11 |
| 50 | -2606.76 | -2429.05 | 87.96 | -265.68 | -177.71 |
| 100 | -2755.49 | -2552.56 | 82.50 | -285.43 | -202.93 |
| 500 | -2846.32 | -2569.50 | 79.23 | -356.05 | -276.82 |
| 1000 | -2865.19 | -2581.05 | 42.53 | -326.66 | -284.14 |

See below for an overlay of minimized structures with `-nitr` equals 5 (green), 25 (magenta), and 100 (yellow): 

![omm-minimize-1](/assets/img/omm-minimize-1.jpg)

When `-nitr` equals 5 (green) or 25 (magenta), the benzene ring of F residue at the N terminal is visibly distorted. This can also be inferred from the high `E_Peptide` values. 

See below for an overlay of minimized structures with `-nitr` equals 100 (yellow), 500 (orange), and 1000 (blue): 

![omm-minimize-2](/assets/img/omm-minimize-2.jpg)

When `-nitr` equals 500 (orange) or 1000 (blue), the conformation and position of the peptide could change. Although `E_Complex` could be minimized even more, the minimization and the energies were computed in *vacuo*, so somewhere between 100 and 500 we should consider **switching the environment to solvent**. 

However, it should be noted that **not all poses can be properly minimized within 100 steps**. For example, the following linking-ring docked pose (light green), which cannot be minimized without substantial conformation change after 5000 steps (dark green): 

![omm-linking-benzene](/assets/img/omm-linking-benzene.jpg)

In the 100 processed modes, that seemed to be the only problematic structure that remained unphysical after the 100-step minimization in vacuo. 

The above minimization calculation (-nmin 100 -nitr 100 -env vaccum) will take **about 1 hour** on a 40-core Intel(R) Xeon(R) Gold 6148 CPU @ 2.40GH. The completed minimization calculation will generate `dock1_omm_rescored_out.pdb` under the work folder `dock1`. With the `-k` option, subfolder `dock1_omm_amber_parm` will be kept under `dock1`. 

### 2nd Minimization Preparation

To start another minimization, I will do the following 3 things: 

(1) **Relocate output files** under `dock1` to `dock1_min1`

```shell
cp -r dock1 dock1_min1
```

(2) **Write a new PDB file** of the minimized peptide coordinates and docking info to replace `dock1_out.pdb` in the next minimization

```python
# 1. Read Docking Poses
dock_dict = {}

with open('dock1_min1/dock1_out.pdb','r') as f:
    line = f.readline()
    
    while line:
        header = line[:6]
        if header=='MODEL ':
            modnum = line.split()[-1]
            dock_dict[modnum] = []
        else:
            if header not in ['ATOM  ','ENDMDL']:
                dock_dict[modnum].append(line)
                
        line = f.readline()

# 2. Read Minimized Poses
omm_dict = {}
is_pep = False

with open('dock1_min1/dock1_omm_rescored_out.pdb','r') as f:
    line = f.readline()
    
    while line:
        header = line[:6]
        if header=='MODEL ':
            is_pep = False
            modnum = line.split()[-1]
            omm_dict[modnum] = []
        if header=='TER   ':
            is_pep = True
        if header=='ATOM  ' and is_pep:
            omm_dict[modnum].append(line)
            # for each model
            # record all ATOM lines appear after TER
        
        line = f.readline()

# 3. Read Rank Mapping
rank_list = []
rank_dict = {}

with open('min1.log','r') as f:
    line = f.readline()
    
    while line:
        if line[:12]=='OMM Ranking:':
            rank_list.append(line)
            
        line = f.readline()

for rank in rank_list[6:-1]:
    rank_dict[rank.split()[4]] = rank.split()[3]
    # mapping ADCP Rank (key) with OMM Rank (value)

# 4. Write New PDB

with open('dock1/dock1_min.pdb','w') as fw:
    for mode in dock_dict.keys():
        fw.write('MODEL '+mode+'\n')
        for line in dock_dict[mode]:
            fw.write(line)
        for line in omm_dict[rank_dict[mode]]:
            fw.write(line)
        fw.write('ENDMDL\n')

```

(3) **Clean Up work folder** `dock1`

```shell
rm -r dock1/dock1_omm*;
mv dock1/dock1_min.pdb dock1/dock1_out.pdb;
```

Now the work forder `dock1` contains only: 

```shell
dock1
├── dock1_out.pdb
└── dock1_summary.dlg
```

While `dock1_out.pdb` contains the coordinates from the previous minimization and information from the docking calculation. 

### Minimization with Implicit Solvent

The command I used to perform the **2nd minimization with implicit solvent** is - 

```shell
adcp -k -T 2xpp.trg -s "FFEIF" -pdmin -o dock1 -L swiss -w dock1 -nmin 100 -nitr 1000 -env implicit &> min2.log;
```

In which I set `-nitr` to be 1000. See below for the energy outputs I collected with `-nitr` ranging from 5 to 5000: 

| nitr | E_Complex | E_Receptor | E_Peptide | dE_Interaction | dE_Complex-Receptor |
| --- | --- | --- | --- | --- | --- |
| 5 (Default) | -3084.70 | -2994.62 | -96.17 | 6.09 | -90.08 |
| 50 | -4771.29 | -4570.70 | -182.15 | -18.44 | -200.59 |
| 100 | -4844.07 | -4631.38 | -187.98 | -24.71 | -212.69 |
| 200 | -4872.04 | -4651.11 | -189.32 | -31.60 | -220.93 |
| 500 | -4895.56 | -4668.38 | -193.22 | -33.96 | -227.18 |
| 1000 | -4900.30 | -4673.12 | -193.34 | -33.84 | -227.18 |
| 2000 | -4896.49 | -4669.85 | -193.37 | -33.27 | -226.64 |
| 5000 | -4883.27 | -4657.35 | -191.20 | -34.72 | -225.92 |

You will see that the energies, most importantly `dE_Interaction`, became generally invariant after `-nitr` reached 1000. But again we should keep in mind that the required number of minimization steps always depends on how good the initial structure is... 

The above minimization calculation (-nmin 100 -nitr 1000 -env implicit) will take ??? on a 40-core Intel(R) Xeon(R) Gold 6148 CPU @ 2.40GH. 

## Example 3 Advanced Docking: Docking a Cyclice Peptide Containing a Disulfide Bond and Pose Selection

```shell
adcp -O -T 2xpp.trg -s "FCEIFC" -cys -N 400 -n 20000000 -o dock1 -L swiss -w dock1 -nc 0.8 -c 40 &> dock1.log;
```

## Example 4 Advanced Docking: Docking a Peptide Containing D-Amino Acids and Backbone Dihedral Validation

```shell
adcp -O -T 2xpp.trg -s "&FFEIF" -N 400 -n 20000000 -o dock1 -L swiss -w dock1 -nc 0.8 -c 40 &> dock1.log;
```