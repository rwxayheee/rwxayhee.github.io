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

The intent of the post is to introduce *to our lab members* the new features in ADCP v1.1, including the most useful **OpenMM minimization** and the incorporation of nonstandard residues, followed by some additional steps that might benefit our own work such as **parsing and exporting the output to RDKit** for post-processing, and finally **to Amber for the MM/GBSA calculation**. 

# Overview

This post includes our own examples of docking calculations for standard AA peptides, cyclic peptides and peptides with D-amino acids which have better support in ADCP v 1.1. The post-processing in AutoDock Vina, which we used for ADCP v1.0, is compared with the new post-processing protocol in this post using the OpenMM funtionalities that were embedded in ADCP v1.1. Lastly, because the minimization is performed in-vacuo, we want to recalculate the binding energy and free energy in Amber using an implicit solvation model. 

# Table of Contents

* [Example 1-1: Docking a Standard AA, 5-mer Peptide and Using OpenMM for Minimization]
* [Example 1-2: Exporting Parameters and Coordinates from OpenMM into Amber for MM/GBSA Calculation]

* [Example 2-1: Docking a Standard AA, 5-mer Peptide and Using Vina for Local Optimization]
* [Example 2-2: Exporting Molecules from RDKit into Amber for MM/GBSA Calculation]

* [Example 3: Docking a cyclice peptide containing a Disulfide Bond and Pose Selection]

* [Example 4: Docking a peptide containing D-amino acids and Backbone Dihedral Validation]