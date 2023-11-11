---
layout: post
title: "Peptide Docking and OpenMM Minimization with ADCPv1.1, Interfacing RDKit, and MM/GBSA Calculation in Amber"
author: "rwxayheee"
categories: journal
tags: [documentation]
image: coming-soon.jpg
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
* [Example 1-2 Basic Minimization: Using OpenMM for a Two-step Minimization](#example-1-2-basic-minimization-using-openmm-for-a-two-step-minimization)

* [Example 2-1 Interfacing RDKit: Exporting ADCP raw outputs into RDKit Molecules and Using Vina for Local Optimization]
* [Example 2-2 Interfacing AMBER: Exporting minmized ADCP outputs into AMBER for MM/GBSA Calculation]

* [Example 3 Advanced Docking: Docking a Cyclice Peptide Containing a Disulfide Bond and Pose Selection](#example-3-advanced-docking-docking-a-cyclice-peptide-containing-a-disulfide-bond-and-pose-selection)

* [Example 4 Advanced Docking: Docking a Peptide Containing D-Amino Acids and Backbone Dihedral Validation](#example-4-advanced-docking-docking-a-peptide-containing-d-amino-acids-and-backbone-dihedral-validation)

## Example 1-1 Basic Docking: Docking a Standard AA, 5-mer Peptide in ADCP v1.1

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

And the command for **docking calculation** was - 

```shell
adcp -O -T 2xpp.trg -s "FFEIF" -N 400 -n 20000000 -o dock1 -L swiss -w dock1 -ref 2xpp_pepH.pdb -nc 0.8 -c 40 &> dock1.log;
```

ADCP v1.1 uses slightly different syntax to read TRG file (`-T`) and there are more options (`-L`, `-w`) to support the additional features. In addition, *the default clustering method has changed from RMSD to contact-based clustering* with a cutoff occupancy of 0.8. A reference structure is optional, but might help to measure the reproducibility or track possible improvements if the docking calculations are run in replicates. 

The above calculation could take around 1 hours 40 minutes to 2 hours on a 40-core Intel(R) Xeon(R) Gold 6148 CPU @ 2.40GHz. 

## Example 1-2 Basic Minimization: Using OpenMM for a Two-step Minimization

A completed docking calculation will generate a folder named `dock1` (as specified by the `-o` option in the docking calculation), containing the docking output PDB and DLG files: 

```
dock1
├── dock1_out.pdb
└── dock1_summary.dlg
```

For the reported 100 poses (default if not changed by the `-m` option in the docking calculation), we will first perform a gas phase minimization to eliminate the bad contacts in the complex and reduce the strain energies within the peptide ligand - 

```shell
adcp -k -T 2xpp.trg -s "FFEIF" -pdmin -o dock1 -L swiss -w dock1 -nmin 100 -nitr 100 -env vacuum &> min1.log;
```

I let the number of iterations to be 100, as I believe it can yield a reasonable geometry, relatively stable `E_peptide` without changing the docking pose too much. See below for outputs I collected with `-nitr` ranging from 5 to 1000: 

| nitir | E_Complex | E_Receptor | E_Peptide | dE_Interaction | dE_Complex-Receptor |
| --- | --- | --- | --- | --- | --- |
| 5 (Default) | 1038.39 | 676.60 | 562.69 | -200.89 | 361.79 |
| 10 | -1200.51 | -1317.06 | 305.66 | -189.10 | 116.56 |
| 25 | -2362.51 | -2218.40 | 103.79 | -247.91 | -144.11 |

nitr = 5 (default)
OMM Energy: E_Complex =   1038.39; E_Receptor =    676.60; E_Peptide  =    562.69
OMM Energy: dE_Interaction =   -200.89; dE_Complex-Receptor =    361.79

nitir = 10
OMM Energy: E_Complex =  -1200.51; E_Receptor =  -1317.06; E_Peptide  =    305.66
OMM Energy: dE_Interaction =   -189.10; dE_Complex-Receptor =    116.56

nitir = 25
OMM Energy: E_Complex =  -2362.51; E_Receptor =  -2218.40; E_Peptide  =    103.79
OMM Energy: dE_Interaction =   -247.91; dE_Complex-Receptor =   -144.11

nitr = 50
OMM Energy: E_Complex =  -2606.76; E_Receptor =  -2429.05; E_Peptide  =     87.96
OMM Energy: dE_Interaction =   -265.68; dE_Complex-Receptor =   -177.71

nitir = 100
OMM Energy: E_Complex =  -2755.49; E_Receptor =  -2552.56; E_Peptide  =     82.50
OMM Energy: dE_Interaction =   -285.43; dE_Complex-Receptor =   -202.93

nitir = 500
OMM Energy: E_Complex =  -2846.32; E_Receptor =  -2569.50; E_Peptide  =     79.23
OMM Energy: dE_Interaction =   -356.05; dE_Complex-Receptor =   -276.82

nitir = 1000
OMM Energy: E_Complex =  -2865.19; E_Receptor =  -2581.05; E_Peptide  =     42.53
OMM Energy: dE_Interaction =   -326.66; dE_Complex-Receptor =   -284.14

## Example 3 Advanced Docking: Docking a Cyclice Peptide Containing a Disulfide Bond and Pose Selection

```shell
adcp -O -T 2xpp.trg -s "FCEIFC" -cys -N 400 -n 20000000 -o dock1 -L swiss -w dock1 -nc 0.8 -c 40 &> dock1.log;
```

## Example 4 Advanced Docking: Docking a Peptide Containing D-Amino Acids and Backbone Dihedral Validation

```shell
adcp -O -T 2xpp.trg -s "&FFEIF" -N 400 -n 20000000 -o dock1 -L swiss -w dock1 -nc 0.8 -c 40 &> dock1.log;
```