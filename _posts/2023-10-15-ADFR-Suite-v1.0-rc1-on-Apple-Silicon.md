---
layout: post
title: "ADFR Suite v1.0 rc1 on Apple Silicon"
author: "rwxayheee"
categories: journal
tags: [documentation]
image: ADFR-tutorial-test.jpg
---

### Motives

The [ADFR suite](https://ccsb.scripps.edu/adfr/) is developed by the Sanner lab in the Center for Computational Structural Biology (CCSB) at Scripps Research. It has been tremendously useful for our group research in providing a set of tools to streamline, automate, and customize the docking routine. I am no longer actively involved in research projects that use AutoDockFR. However, there has been a considerable amount of unpublished work and efforts that we, together with several lab alumni, put into constructing a docking pipeline with ADFR that is specific to the biological system of our interest. 

The guide is written with the hope of helping our junior students get a smooth start with the ADFR suite, run some interactive trial calculations before deploying their production jobs on the supercomputer center, and hopefully carry on some of our work in the future. To students who are assigned to a university-owned Mac with [Apple Silicon (M1/M2)](https://en.wikipedia.org/wiki/Mac_transition_to_Apple_silicon), having a Linux [virtual machine (VM)](https://en.wikipedia.org/wiki/Virtual_machine) on it provides a good lot of freedom to install software they need for research. At the moment, not all programs have [ARM64](https://en.wikipedia.org/wiki/AArch64) support and can not be directly run on Macs with Apple Silicon. [Rosetta](https://en.wikipedia.org/wiki/Rosetta_(software)) is a compatibility layer that translates such software that were exclusively built in different architectures so that they could be run on Apple Silicon which is the ARM64 architecture. 

The general idea of running [x86_64](https://en.wikipedia.org/wiki/X86-64) programs in a Linux VM with Rosetta may be useful when: 
1. The program is currently lacking ARM64 support. 
and,  
2. The Mac OS version or the platform is for some reason (see above) not the preferred choice, even though in theory it could be run natively on Apple Silicon with Rosetta. 

To run [Vina & the python binding](https://github.com/ccsb-scripps/AutoDock-Vina), [Meeko](https://github.com/forlilab/Meeko), or [MGLTools](https://ccsb.scripps.edu/mgltools/) & the [ADFR suite](https://ccsb.scripps.edu/adfr/) in [Docker](https://docs.docker.com/get-started/overview/) images, please consider this <a href="https://github.com/Metaphorme/AutoDock-Vina-Docker" target="_blank">user-contributed solution</a>. 

At present, this guide would be a complementary to the docker solution, for those who wish to use the ADFR suite and perhaps run some lightweight calculations on Apple Silicon. 

# Overview

Following this guide, we will create a **[Ubuntu](https://en.wikipedia.org/wiki/Ubuntu) VM** in **[UTM](https://mac.getutm.app/)** with a desktop that serves as the graphical user interface (GUI). The two enabled features of our VM, **Apple virtualization** and **Rosetta emulation**, will allow us to **install and run the x86_64 programs**, including *ADFR*, *AGFR*, *AGFRGUI*, and *ADCP*, from the current major version of ADFR suite (v1.0 rc1, as of October 2023). 

The procedure generally follows the logic of the [UTM documentation on Rosetta](https://docs.getutm.app/advanced/rosetta/), with the addition of installing a desktop GUI and the specific [AMD64 (another name for x86_64)](https://en.wikipedia.org/wiki/X86-64) libraries for programs in the ADFR suite. Finally, to complete the tasks in the [ADCP tutorial](https://ccsb.scripps.edu/adcp/tutorial-redocking/), the current major version of program reduce is made from [source](https://github.com/rlabduke/reduce). 

## Table of Contents

1. [Setting up a Ubuntu VM in UTM](#Setting-up-a-Ubuntu-VM-in-UTM)
  1. [Download UTM and the disc image of Ubuntu-for-ARM](#Download-UTM-and-the-disc-image-of-Ubuntu-for-ARM)
  2. [Set up the VM from UTM](#Set-up-the-VM-from-UTM)
  3. [Install the desktop GUI for Ubuntu (optional)](#Install-the-desktop-GUI-for-Ubuntu-(optional))
2. [Enabling Rosetta](#Enabling-Rosetta)
  1. [Make Rosetta accessible through VirtioFS](#Make-Rosetta-accessible-through-VirtioFS)
  2. [Add Rosetta to the filesystem table fstab](#Add-Rosetta-to-the-filesystem-table-fstab)
  3. [Register Rosetta using update-binfmts](#Register-Rosetta-using-update\-binfmts)
3. [Enabling the *Multiarch* and *Multilib* support](#Enabling-the-Multiarch-and-Multilib-support)
  1. [*Multiarch*: Add AMD64 as a foreign arch to dpkg](#Multiarch:-Add-AMD64-as-a-foreign-arch-to-dpkg)
  2. [*Multilib*: Install specific AMD64 libraries for the ADFR suite](#Multiplib:-Install-specific-AMD64-libraries-for-the-ADFR-suite)
4. [Installing the ADFR suite](#Installing-the-ADFR-suite)
  1. [Make program reduce from source](#Make-program-reduce-from-source)
  2. [Tests with sample data](#Tests-with-sample-data)

## Setting up a Ubuntu VM in UTM

**UTM** is a free application to create virtual machines on Mac with support for Apple Silicon. **Ubuntu** is a free & open source Linux operating system. To begin with, we will set up a Ubuntu VM in UTM with the desired features to enable Rosetta emulation. 

## Download UTM and the disc image of Ubuntu-for-ARM

UTM (Version 4.3.5) - <a href="https://mac.getutm.app/" target="_blank">https://mac.getutm.app/</a>

Ubuntu (22.04.3-live-server-arm64) - <a href="https://ubuntu.com/download/server/arm" target="_blank">https://ubuntu.com/download/server/arm</a>

## Credits

## Contact
