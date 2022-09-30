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
### 5. Run checkV

There could be some non-viral sequences or regions in the VirSorter2 results with a minimal score cutoff of 0.5. Here we use CheckV to quality control the VirSorter2 results and also to trim potential host regions left at the ends of proviruses. You can adjust the "-t" option based on the availability of CPU cores.
```
checkv end_to_end vs2-pass1/final-viral-combined.fa checkv -t 28 -d /path/to/check_database/db/checkv-db-v1.0
```
### 6. Run VirSorter2 again
Then we run the checkV-trimmed sequences through VirSorter2 again to generate "affi-contigs.tab" files needed by DRAMv to identify AMGs. You can adjust the "-j" option based on the availability of CPU cores. Note the "--seqname-suffix-off" option preserves the original input sequence name since we are sure there is no chance of getting >1 proviruses from the same contig in this second pass, and the "--viral-gene-enrich-off" option turns off the requirement of having more viral genes than host genes to make sure that VirSorter2 is not doing any screening at this step. The above two options require VirSorter2 version >=2.2.1.
``` 
cat checkv/proviruses.fna checkv/viruses.fna > checkv/combined.fna
virsorter run --seqname-suffix-off --viral-gene-enrich-off --provirus-off --prep-for-dramv -i checkv/combined.fna -w vs2-pass2 --include-groups dsDNAphage,ssDNA --min-length 5000 --min-score 0.5 -j 28 all
```
> Note: if you obtain viral sequences from other tools and you want annotate your sequences with DRAM-v, you should also run Virsorter2 with --min-score set to 0. 

