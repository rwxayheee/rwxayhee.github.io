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
  + [Structure and Target Preparation](#structure-and-target-preparation)
  + [Docking Calculations](#docking-calculations)

## Step 1: Docking Calculation with ADCP

In this step, we will perform docking calculations for a peptide from sequence `FFEIF` and the receptor Iws1 from crystal structure 2XPP. A top-ranked binding mode will be chosen for post-processing and optimization in the further steps. 

### Structure and Target Preparation

Following the steps in the [ADCP documentation](https://ccsb.scripps.edu/adcp/tutorial-redocking/), we will **generate the PDBQT files for the peptide and the receptor, and the TRG file for the receptor** - 

```shell
reduce 2xpp_iws1.pdb > 2xpp_recH.pdb;
```

It should be noted that reduce will not protonate any of the imidazole nitrogens on HIS sidechains, unless with option `-NOFLILP` (or `-HIS` or `-BUILD`). 

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

Running the docking calculations with insufficient scope (number of independnt GA runs, as specified by `-N`) and depth (number of max. MCsteps, as specified by `-n`) will result in less reproducible outcomes. Here is my recipe for the presented system, which I think will produce relatively reproducible results for 5-mer to 7-mer peptides - 

```shell
#!/bin/zsh
adcp -t 2xpp.trg -s FFEIF -N 400 -n 20000000 -o dock1 -ref 2xpp_pepH.pdb -c 40 &> dock1.log;
  
adcp -t 2xpp.trg -s FFEIF -N 400 -n 20000000 -o dock2 -ref dock1_ranked_1.pdb -c 40 &> dock2.log;

dock1_rank="$(grep -m1 "| ref. |" dock1.log -a3 | tail -n1)";
S1="$(echo "${dock1_rank}" | awk '{split($0,a," "); print a[2]}')";
s1=$((S1));
dock2_rank="$(grep -m1 "| ref. |" dock2.log -a3 | tail -n1)";
S2="$(echo "${dock2_rank}" | awk '{split($0,b," "); print b[2]}')";
s2=$((S2));
  
cp dock2_ranked_1.pdb nc_ref.pdb;
if (( $s1 < $s2 )); then
  cp dock1_ranked_1.pdb nc_ref.pdb;
  else
fi
  
adcp -t 2xpp.trg -s FFEIF -N 400 -n 20000000 -o dock3 -ref nc_ref.pdb -nc 0.8 -c 40 &> dock3.log;
```

*Method explained*

+ **The population of initial conformer** is set to be all-helical, as specified by the sequence in ALL-CAPS. Three docking calculations were performed in serial, each containing 400 independent GA runs with max. 20,000,000 MC steps per GA run. 
+ **Docking Calculation #1** may use an arbitrarily placed (preferably folded in the desired way, if helical, coil or a specific secondary structure is expected) conformer of the peptide. The default RMSD clustering will be used after completion of all searches. 
+ **Docking Calculation #2** uses the top-ranked conformer from Docking Calculation #2 and the default RMSD clustering. 
+ **Docking Calculation #3** uses the conformer from Docking Calculations #1 and #2 that has the lowest printed affinity. The default contact-based clustering is used, assuming the selected conformer from the previous docking calculations has the native contacts. Outcomes from Docking Calculation #3 will be considered for post-processing and further optimization. 

I perform the docking calculations *in three replicates*, but with different reference structures for clustering, because I think choice of reference might affect clustering and ranking, and often times there is no prior knowledge what the best conformation (folding) would be like for the sequences of interest (unless the calculation is strictly a re-docking, while this example is not), and no reference structure to start with for the native contact analysis. However, if the top-ranked binding modes can be found in three replicates and are ranked invariantly with choice of reference structure, then I would take them as reproducible top-ranked binding modes. 

Below are the standard outputs I got for the top 10 binding modes from the example docking calculations - 

*dock1.log*

````shell
mode |  affinity  | clust. | ref. | clust. | rmsd | best | energy | best |
     | (kcal/mol) | rmsd   | rmsd |  size  | avg. | rmsd |  avg.  | run  |
-----+------------+--------+------+--------+------+------+--------+------+
   1        -15.3     0.0     5.3    4873     5.3   3.7   -13.1    9724
   2        -15.0     4.7     2.6     113     2.7   1.8   -13.3    18101
   3        -15.0     3.5     3.9    3132     3.7   2.2   -12.8    19274
   4        -14.9     5.9     3.7    1753     3.7   2.5   -13.2    17920
   5        -14.8     6.4     5.3     385     5.2   4.7   -12.8    19910
   6        -14.8     6.4     5.0    1333     4.9   4.2   -12.6    35129
   7        -14.6     6.9     4.1    1023     4.0   2.6   -12.5    36704
   8        -14.6     4.1     3.4     554     3.4   1.6   -11.4    32717
   9        -14.4     5.9     2.2     748     2.5   1.5   -11.7    66771
  10        -14.2     5.2     4.9    1237     4.9   4.0   -12.3    55850
````

From the above outputs, we should see that compared to how it positioned in the crystal structure 2XPP, the peptide `FFEIF` adopts a different conformation or binding mode in the top-ranked pose with RMSD = 5.3 to the reference structure which is essentially from the crystal structure. 

*dock2.log*

````shell
mode |  affinity  | clust. | ref. | clust. | rmsd | best | energy | best |
     | (kcal/mol) | rmsd   | rmsd |  size  | avg. | rmsd |  avg.  | run  |
-----+------------+--------+------+--------+------+------+--------+------+
   1        -15.3     0.0     0.1    4757     0.9   0.1   -13.2    17300
   2        -15.1     4.9     4.9     215     5.3   4.4   -12.7    66267
   3        -14.9     6.1     6.1    1464     6.1   5.3   -13.0    1767
   4        -14.8     4.0     3.9    3054     3.5   2.7   -12.9    24942
   5        -14.7     6.3     6.3    1549     6.5   6.0   -12.8    46071
   6        -14.5     6.5     6.5     539     6.5   5.6   -12.4    64089
   7        -14.3     6.9     6.9     869     6.9   6.0   -12.6    45899
   8        -14.2     6.8     6.8      40     6.9   6.4   -13.0    21812
   9        -14.2     5.1     5.1    1107     5.4   5.1   -12.3    65626
  10        -14.1     4.1     4.1     603     4.3   3.0   -11.4    63408

````

From the above outputs, we should see that the top-ranked pose in Docking Calculation #2 is highly consistent with the top-ranked pose in Docking Calculation #1, which is the reference structure here, with RMSD = 0.1. 

*dock3.log*

````shell
mode |  affinity  | ref. | clust. | rmsd | energy | best |
     | (kcal/mol) | fnc  |  size  | stdv |  stdv  | run  |
-----+------------+------+--------+------+--------+------+
   1        -15.7    1.000    4190      NA      NA    33189
   2        -14.9    0.900    2183      NA      NA    41742
   3        -14.9    0.550    1546      NA      NA    42954
   4        -14.8    0.500    1238      NA      NA    66033
   5        -14.6    0.650      68      NA      NA    60254
   6        -14.5    0.550      43      NA      NA    53970
   7        -14.5    0.450      29      NA      NA    35465
   8        -14.4    0.600     618      NA      NA    51413
   9        -14.3    0.400     121      NA      NA    42604
  10        -14.3    0.550     138      NA      NA    16287
````

From the above outputs, we should see that the top-ranked pose in Docking Calculation #3 is still consistent with the top-ranked pose in Docking Calculations #1 and #2, for having the same (fraction = 1.000) native contacts. 

