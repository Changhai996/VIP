# VIP

Pipline for identifying viral sequences in metagenomic data.  

Modified by Changhai from Meng's lab at 2022-09-30, referring to [virus identification SOP](https://www.protocols.io/view/viral-sequence-identification-sop-with-virsorter2-5qpvoyqebg4o/v3?step=3) from Sullivan lab written by Jiarong Guo et al.  

This pipline shows how to use VirSorter2, checkV, DRAMv, HHblits and some manual curation criteria for viral sequence identification.  

Also, there are a lot of viral sequences identification tools can be applied, such as [VIBRANT](https://github.com/AnantharamanLab/VIBRANT), [VirFinder](https://github.com/jessieren/VirFinder), [PhiSpy](https://github.com/linsalrob/PhiSpy) etc., you can selecte whatever as you prefer, [here](https://github.com/linsalrob/ProphagePredictionComparisons) is the comparison of these mainstream tools.  

### 1. Before starting
>This tutorial requires Linux OS with "conda" installed.  If you do not "conda" installed, you can follow the instructions [here](https://docs.conda.io/projects/conda/en/latest/user-guide/install/linux.html).  

### 2. Install dependencies
>This is the most annoying step,  the configuration of the software database is likely to be done manually for network issues, and it takes a lot of time  


We need following tools for this SOP:

[VirSorter2](https://github.com/jiarong/VirSorter2)(version >=2.2.3)  
[CheckV](https://bitbucket.org/berkeleylab/checkv/src/master/)(version >=0.7.0)  
[DRAMv](https://github.com/WrightonLabCSU/DRAM)(version >=1.2.0)  
[HH-suite3](https://github.com/soedinglab/hh-suite)(version >=3.3.0)  

### 3. Prep test data
We need a small test data for this tutorial. Here we grab seven sequences identified by VirSorter2 from a soil sample the sequence names are named after their categories in manual curation step that we will discuss later.

```
wget -O test.fa https://bitbucket.org/MAVERICLab/virsorter2/raw/15a21fa9c1ee1d2af52b0148b167292e549d356e/test/test-for-sop.fa
```

### 4.Run VirSorter2
First, run VirSorter2 with a loose cutoff of 0.5 for maximal sensitivity. We are only interested in phages (dsDNA and ssDNA phage). A minimal length 5000 bp is chosen since it is the minimum required by downstream viral classification. You can adjust the "-j" option based on the availability of CPU cores. Note that the "--keep-original-seq" option preserves the original sequence of circular and (near) fully viral contigs (score >0.8 as a whole sequence) and we are passing them to checkV to trim possible host genes left at ends and handle duplicate segments of circular contigs.
```
virsorter run --keep-original-seq -i test.fa -w vs2-pass1 --include-groups dsDNAphage,ssDNA --min-length 5000 --min-score 0.5 -j 28 all
```

