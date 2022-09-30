# VIP

Pipline for identifying viral sequences in metagenomic data.  

Modified by Changhai from Meng's lab at 2022-09-30, referring to [virus identification SOP](https://www.protocols.io/view/viral-sequence-identification-sop-with-virsorter2-5qpvoyqebg4o/v3?step=3) from Sullivan lab written by Jiarong Guo et al.  

This pipline shows how to use VirSorter2, checkV, DRAMv, HHblits and some manual curation criteria for viral sequence identification.  

Also, there are a lot of viral sequences identification tools can be applied, such as [VIBRANT](https://github.com/AnantharamanLab/VIBRANT), [VirFinder](https://github.com/jessieren/VirFinder), [PhiSpy](https://github.com/linsalrob/PhiSpy) etc., you can selecte whatever as you prefer, [here](https://github.com/linsalrob/ProphagePredictionComparisons) is the comparison of these mainstream tools.  

### 1. Before starting
>This tutorial requires Linux OS with "conda" installed.  If you do not "conda" installed, you can follow the instructions [here](https://docs.conda.io/projects/conda/en/latest/user-guide/install/linux.html).  

### 2. Install dependencies
We need following three tools for this SOP:
