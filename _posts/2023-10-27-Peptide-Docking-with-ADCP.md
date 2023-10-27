---
layout: post
title: "Peptide Docking with ADCP, Post-processing, and Induced-fit Optimization with Vina: A Python Scripting Example"
author: "rwxayheee"
categories: journal
tags: [documentation]
image: adcp-our-opt.jpg
---

# Intro

This post describes **peptide docking** using [Autodock CrankPep (ADCP)*](https://ccsb.scripps.edu/adcp) (Version 1.0 rc1), some **post-processing** including [converting the output structure of the docked peptide into a charged RDKit molecule](https://github.com/rwxayheee/prepare_peptide_ligand), and finally a very primitive **"induced-fit" refinement** using the local optimization function, `optimize`, [from AutoDock Vina's Python bindings](https://autodock-vina.readthedocs.io/en/latest/vina.html#vina.vina.Vina.optimize). The efforts were based on the current major version of ADCP as of Oct 2023 with the intent to **interface ADCP with Vina and RDKit in Python**. 

The provided example is based on an experimental structure of Spt6-Iws1(Spn1) complex from *E. cuniculi* ([PDB ID: 2XPP](https://www.rcsb.org/structure/2xpp)). The crystal structure 2XPP contains two chains: The longer one is Iws1 and will be the receptor in the docking calculation. The shorter one is a truncated form of Spt6. In addition to the mentioned structure, the Spt6-Iws1 complex is also observed in a high-res EM structure of actively working RNA polymerase II elongation complex ([PDB ID: 7XN7](https://www.rcsb.org/structure/7xn7)). The peptide sequence `FFEIF` is part of the IWS1-interacting domain of Spt6 in *E. cuniculi*. Similar sequences present in Spt6 across different species, with a highly conserved F residue that fits the binding site on the Iws1 receptor. The precense of multiple F residues makes the 5-mer sequence from *E. cuniculi* an interesting sequence to explore in peptide docking calculations. 

# Overview

