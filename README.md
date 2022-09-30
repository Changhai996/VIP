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
### 6. Additional procedures (important)
> First, you need to make sure they are viruses. To my knowledge, if you can not find a viral structure protein in viral sequences, they are most likely to be non-viral, because vs2 has a high false positive rate.
> Here are some criteria for select the result form above procedures:  
> Keep1: viral_gene >0  (please look at checkv result quality_summary.tsv)  
> Keep2: viral_gene =0 AND (host_gene =0 OR score >=0.95 OR hallmark >2) (please look at vs2 result final-viral-score.tsv)  
> Note: CheckV will identify the terminal repeats in the viral sequnces you provide, if they detect DTR(direct terminal repeats) or ITR(invert terminal repeats), they would write the reuslt in checkv/complete_genomes.tsv file in step 5. You need to carefully check these virus with circular or linear genome in the annotation files.

### 7. Run VirSorter2 again
Then we run the checkV-trimmed and filted sequences by step 6 through VirSorter2 again to generate "affi-contigs.tab" files needed by DRAMv to identify AMGs. You can adjust the "-j" option based on the availability of CPU cores. Note the "--seqname-suffix-off" option preserves the original input sequence name since we are sure there is no chance of getting >1 proviruses from the same contig in this second pass, and the "--viral-gene-enrich-off" option turns off the requirement of having more viral genes than host genes to make sure that VirSorter2 is not doing any screening at this step. The above two options require VirSorter2 version >=2.2.1.
``` 
cat checkv/proviruses.fna checkv/viruses.fna > checkv/combined.fna
virsorter run --seqname-suffix-off --viral-gene-enrich-off --provirus-off --prep-for-dramv -i checkv/combined.fna -w vs2-pass2 --include-groups dsDNAphage,ssDNA --min-length 5000 --min-score 0.5 -j 28 all
```
> Note: if you obtain viral sequences from other tools and you want annotate your sequences with DRAM-v, you should also run Virsorter2 with --min-score set to 0 with above command. 

### 8. Run DRAMv
Then run DRAMv to annotate the identified sequences, which can be used for manual curation. You can adjust the "--threads" option based on availability of CPU cores.
```
# step 1 annotate
DRAM-v.py annotate -i vs2-pass2/for-dramv/final-viral-combined-for-dramv.fa -v vs2-pass2/for-dramv/viral-affi-contigs-for-dramv.tab -o dramv-annotate --skip_trnascan --threads 28 --min_contig_size 1000
#step 2 summarize anntotations
DRAM-v.py distill -i dramv-annotate/annotations.tsv -o dramv-distill
```
### 9. Run HHblits
According to my knowledge, some protein sequences of viruses cannot be annotated by traditional sequences alignment methods, so a more sensitive method needs to be introduced. We use the Profile-to-ptofile annotation method of HHblits, which can also be used in the [web version](https://toolkit.tuebingen.mpg.de/tools/hhpred), but unable to annotate multiple sequences.
In brief, you can copy the following shell script in your environemnt and run with:
```
sh /mnt/storage14/duanchanghai/tools/hhblit.sh your_protein_file.faa
```
The output file with suffix hhblit_annotation.tsv is the annotation file, you should carefully check the results with probability > 95 which need serious consideration (check the other hits in the list)

### 10. Manual curation

For those in “manual check” category, you can look through their annotations in "dramv-annotate/annotations.tsv" and "hhblit_annotation.tsv" in which each gene of every contig is a line and has annotation from multiple databases. This step is hard to standardize, but below are some criteria based on our experience.

Criteria for calling a contig viral:  
😁Structural genes, hallmark genes, depletion in annotations or enrichment for hypotheticals (~10% genes having non-hypothetical annotations)  
😄Lacking hallmarks but >=50% of annotated genes hit to a virus and at least half of those have viral bitcore >100 and the contig is <50kb in length  
😊Provirus: Integrase/recombinase/excisionase/repressor, enrichment of viral genes on one side （need further check out ）
☺️Provirus: “break” in the genome: gap between two genes corresponding to a strand switch, higher coding density, depletion in annotations, and an enrichment for phage genes on one side  need further check out ）


Criteria for callling a contig non-viral:  
😫/>3x cellular like genes than viral, nearly all genes annotated, no genes hitting to only viruses and no viral hallmark genes  
😩Lacking any viral hallmark genes and >50kb  
😣Strings of many obvious cellular genes, with no other viral hallmark genes. Examples encountered in our benchmarking include 1) CRISPR Cas, 2) ABC transporters, 3) Sporulation proteins, 4) Two-component systems, 5) Secretion system. Some of these may be encoded by viruses, but are not indicative of a viral contig without further evidence.  
😖Multiple plasmid genes or transposases but no clear genes hitting only to viruses  
☹️Few annotations, only ~1-3 genes hitting to both viruses and cellular genes but with stronger bitscores for the cellular genes.  

‼️Lastly, user beware that any provirus boundary predicted by VirSorter 2 and/or checkV is an approximate estimate only (calling “ends” is quite a challenging problem in prophage discovery), and needs to be manually inspected carefully too, especially for AMG studies.
