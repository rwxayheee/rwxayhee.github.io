---
layout: post
title: "ADFR Suite v1.0 rc1 and ADT on Apple Silicon"
author: "rwxayheee"
categories: journal
tags: [documentation]
image: ADFR-tutorial-test.jpg
---

# Intro

This post is a guide to **setting up a Linux [virtual machine (VM)](https://en.wikipedia.org/wiki/Virtual_machine) with desktop on Mac computers with [Apple Silicon (M1/M2)](https://en.wikipedia.org/wiki/Apple_silicon)**, for the purpose of **installing and running the [x86_64](https://en.wikipedia.org/wiki/X86-64) programs in the [ADFR suite](https://ccsb.scripps.edu/adfr/)** and [AutoDockTools (ADT)](https://autodocksuite.scripps.edu/adt/) on such devices. 

**Apple Silicon uses the [ARM64](https://en.wikipedia.org/wiki/AArch64) architechture**. Programs that were not built in this architecture cannot be run directly on Macs with Apple Silicon. At present, a common solution to the mismatch of architectures is **[Rosetta](https://en.wikipedia.org/wiki/Rosetta_(software)), a compatibility layer** that translates software that were built for Intel processors so that they could be run on Apple Silicon. While in some situations, having a Linux VM with Rosetta emulation to run the x86_64 programs may be useful, when: 

1. The program is currently **lacking ARM64 support**, 

and, 

2. The Mac OS version/platform is for some reason (see below and more in [Motives](#motives) of this post) **not the preferred** choice, even though in theory it could be run natively on Apple Silicon with Rosetta. 

The Mac OS version issue mainly originates from the lack of support for 32-bit software since Mac OS Catalina. With that, programs with 32-bit components will not run on newer Mac OS. Without setting up a VM, another possible & elegant walkaround to this is to run the programs in [Docker](https://docs.docker.com/get-started/overview/) containers. To run [Vina & the python binding](https://github.com/ccsb-scripps/AutoDock-Vina), [Meeko](https://github.com/forlilab/Meeko), or [MGLTools](https://ccsb.scripps.edu/mgltools/) & the [ADFR suite](https://ccsb.scripps.edu/adfr/) in [Docker](https://docs.docker.com/get-started/overview/) images, please consider this user-contributed solution: 

<a href="https://github.com/Metaphorme/AutoDock-Vina-Docker" target="_blank">https://github.com/Metaphorme/AutoDock-Vina-Docker</a>

Hoping this guide would be some kind of complementary to the docker solution, for those who wish to use the ADFR suite and perhaps run some lightweight calculations on Apple Silicon (check out the performance in [Tests with sample data](#tests-with-sample-data)) in a VM, to escape from the hustle of privacy & network settings if that cannot be changed for the native OS... 

# Overview

Following this guide, we will create a **[Ubuntu](https://en.wikipedia.org/wiki/Ubuntu) VM** in **[UTM](https://mac.getutm.app/)** with a desktop that serves as the graphical user interface (GUI). The two enabled features of our VM, **Apple virtualization** and **Rosetta emulation**, will allow us to **install and run the x86_64 programs**, including *ADFR*, *AGFR*, *AGFRGUI*, and *ADCP*, from the current major version of ADFR suite (v1.0 rc1, as of October 2023), and *ADT*, from the current major version of MGLTools (1.5.7). 

The procedure generally follows the logic of the [UTM documentation on Rosetta](https://docs.getutm.app/advanced/rosetta/), with the addition of installing a desktop GUI and the specific [AMD64 (another name for x86_64)](https://en.wikipedia.org/wiki/X86-64) libraries for programs in the ADFR suite. Finally, to complete the tasks in the [ADCP tutorial](https://ccsb.scripps.edu/adcp/tutorial-redocking/), the current major version of program reduce is made from [source](https://github.com/rlabduke/reduce). 

## Table of Contents

1. [Step 1: Setting up a Ubuntu VM in UTM](#step-1-setting-up-a-ubuntu-vm-in-utm)
  + [Download UTM and the disc image of Ubuntu-for-ARM](#download-utm-and-the-disc-image-of-ubuntu-for-arm)
  2. [Set up the VM from UTM](#set-up-the-vm-from-utm)
  3. [Install the desktop GUI for Ubuntu]
2. [Step 2: Enabling Rosetta]
  1. [Make Rosetta accessible through VirtioFS]
  2. [Add Rosetta to the filesystem table fstab]
  3. [Register Rosetta using update-binfmts]
3. [Enabling the *Multiarch* and *Multilib* support]
  1. [*Multiarch*: Add AMD64 as a foreign arch to dpkg]
  2. [*Multilib*: Install specific AMD64 libraries for the ADFR suite]
4. [Installing the ADFR suite]
  1. [Make program reduce from source]
  2. [Tests with sample data]

## Step 1: Setting up a Ubuntu VM in UTM

**UTM** is a free application to create virtual machines on Mac with support for Apple Silicon. **Ubuntu** is a free & open source Linux operating system that we will be using for our VM. To begin with, we will set up a Ubuntu VM in UTM with the desired features to enable Rosetta emulation. 

### Download UTM and the disc image of Ubuntu-for-ARM

UTM (Version 4.3.5) - <a href="https://mac.getutm.app/" target="_blank">https://mac.getutm.app/</a>

Ubuntu (22.04.3-live-server-arm64) - <a href="https://ubuntu.com/download/server/arm" target="_blank">https://ubuntu.com/download/server/arm</a>

### Set up the VM from UTM

## Tests with sample data

### Motives

The ADFR suite is developed by the Sanner lab in the Center for Computational Structural Biology (CCSB) at Scripps Research. It has been tremendously useful for our group research in providing a set of tools to streamline, automate, and customize the docking routine. I am no longer actively involved in research projects that use AutoDockFR. However, there has been a considerable amount of unpublished work and efforts that we, together with several lab alumni, put into constructing a docking pipeline with ADFR that is specific to the biological system of our interest. 

The guide is written with the hope of helping our junior students get a smooth start with the ADFR suite, run some interactive trial calculations before deploying their production jobs on the supercomputer center, and hopefully carry on some of our work in the future. To students who are assigned to a university-owned Mac with Apple Silicon, having a Linux VM on it provides a good lot of freedom to install software they need for research. 

## Credits

## Contact
