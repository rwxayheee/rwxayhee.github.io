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

The intent of the post is to introduce to our lab members the new features in ADCP v1.1, including the most useful **OpenMM minimization** and the incorporation of D-amino acids, followed by some additional steps that might benefit our own work such as **parsing and exporting the output to RDKit** for post-processing, and finally **to Amber for the MM/GBSA calculation**. 

# Overview

This post includes our own examples of docking calculations for standard amino acid (AA) peptides, cyclic peptides and peptides with D-amino acids which have better support in ADCP v1.1. The post-processing in AutoDock Vina, which we used for ADCP v1.0, is compared with the new post-processing protocol in this post using the OpenMM funtionalities that were embedded in ADCP v1.1. Lastly, because the minimization is performed *in vacuo*, we want to recalculate the binding energy and free energy in Amber using an implicit solvation model. 

## Associated Files

<a href="{{ site.url }}/files/peptide-docking.zip" download>peptide-docking.zip</a>

Contains input PDB files, `2xpp_iws1.pdb` and `2xpp_FFEIF.pdb`, for the peptide docking examples. 

# Table of Contents

* [Example 1-1: Docking a Standard AA, 5-mer Peptide and Using OpenMM for Minimization](#example-1-1-docking-a-standard-aa-5-mer-peptide-and-using-openmm-for-minimization)
* [Example 1-2: Exporting Parameters and Coordinates from OpenMM into Amber for MM/GBSA Calculation]

* [Example 2-1: Docking a Standard AA, 5-mer Peptide and Using Vina for Local Optimization]
* [Example 2-2: Exporting Molecules from RDKit into Amber for MM/GBSA Calculation]

* [Example 3: Docking a Cyclice Peptide Containing a Disulfide Bond and Pose Selection](#example-3-docking-a-cyclice-peptide-containing-a-disulfide-bond-and-pose-selection)

* [Example 4: Docking a Peptide Containing D-Amino Acids and Backbone Dihedral Validation](#example-4-docking-a-peptide-containing-d-amino-acids-and-backbone-dihedral-validation)

## Example 1-1: Docking a Standard AA, 5-mer Peptide and Using OpenMM for Minimization

The preparation steps for the docking calculation can be performed the same way as described in section [Structure and Target Preparation](https://rwxayheee.github.io/Peptide-Docking-with-ADCP#structure-and-target-preparation) in the previous post. Additionally, because the tautomers of histidine residues (HIE/HID) will be discriminated in ADCP v1.1 (at least in the post-processing step that uses molecular mechanics), the `NOFLIP` or `FLIP` options may be used to protonate the histidine sidechains in the protein receptor and peptide ligands. 

Here beginning from the receptor PDB file, `2xpp_iws1.pdb`, and a peptide PDB file in the aligned position, `2xpp_FFEIF.pdb`, the following commands were used for **docking preparation** - 

```s
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

```s
adcp -O -T 2xpp.trg -s "FFEIF" -N 400 -n 20000000 -o dock1 -L swiss -w dock1 -ref 2xpp_pepH.pdb -nc 0.8 -c 40 &> dock1.log;
```

ADCP v1.1 uses slightly different syntax to read TRG file (`-T`) and there are more options (`-L`, `-w`) to support the additional features. In addition, the default clustering method has changed from RMSD to contact-based clustering with a cutoff occupancy of 0.8. A reference structure is optional, but might help to measure the reproducibility or track possible improvements if the docking calculations are run in replicates. 

The above calculation took about 1 hours 40 minutes on a 40-core Intel(R) Xeon(R) Gold 6148 CPU @ 2.40GHz. 

## Example 3: Docking a Cyclice Peptide Containing a Disulfide Bond and Pose Selection


```s
adcp -O -T 2xpp.trg -s "FCEIFC" -cys -N 400 -n 20000000 -o dock1 -L swiss -w dock1 -nc 0.8 -c 40 &> dock1.log;
```

## Example 4: Docking a Peptide Containing D-Amino Acids and Backbone Dihedral Validation

```s
adcp -O -T 2xpp.trg -s "&FFEIF" -N 400 -n 20000000 -o dock1 -L swiss -w dock1 -nc 0.8 -c 40 &> dock1.log;
```