## Peak calling

Using [MACS2](https://github.com/taoliu/MACS/)

For both the day 0 and day 3 of differentiation into adipocytes, two files are available

* input, as control
* histone modification H3K4

**MACS2** is going to use both files to normalize the read counts and perform the peak calling.

### Retrieve the BAM files with all chromosomes

```
cd ~/chip-seq
mkdir bams
cd bams
ln -s /scratch/users/aginolhac/chip-seq/bams/*.bam .
```

### Perform peak calling

```
macs2 callpeak -t TC1-H3K4-ST2-D0.GRCm38.p3.q30.bam \
               -c TC1-I-ST2-D0.GRCm38.p3.q30.bam \
               -f BAM -g mm -n TC1-ST2-H3K4-D0 -B -q 0.01 --outdir TC1-ST2-H3K4-D0 &
macs2 callpeak -t TC1-H3K4-A-D3.GRCm38.p3.q30.bam \
               -c TC1-I-A-D3.GRCm38.p3.q30.bam \
               -f BAM -g mm -n TC1-A-H3K4-D3 -B -q 0.01 --outdir TC1-A-H3K4-D3

```

In case `macs2` gives `command not found`, your are certainly missing the module, please see the [set-up](http://ginolhac.github.io/chip-seq/install/)



### check model inferred by MACS2

execute R script.

```
Rscript TC1-A-H3K4-D3/TC1-A-H3K4-D3_model.r
Rscript TC1-ST2-H3K4-D0/TC1-ST2-H3K4-D0_model.r
```

fetch the pdf produced.

### sort per chromosomes and coordinates

```
find TC* -name '*.bdg' | parallel "sort -k1,1 -k2,2n {} > {.}.sort.bdg"

```

### convert to bigwig

in order to get smaller files

```
find TC* -name '*sort.bdg' | parallel -j 2 "bedGraphToBigWig {} \
  /scratch/users/aginolhac/chip-seq/references/GRCm38.p3.chom.sizes {.}.bigwig"

```

### Fetch the files and display them in IGV

IGV can be [downloaded](http://software.broadinstitute.org/software/igv/) from the broadinstitute. 

### Perform peak calling with broad option

```
macs2 callpeak -t TC1-H3K27-ST2-D0.GRCm38.p3.q30.bam \
               -c TC1-I-ST2-D0.GRCm38.p3.q30.bam \
               -f BAM --broad -g mm -n TC1-ST2-H3K27-D0-broad -B -q 0.01 --outdir TC1-ST2-H3K27-D0-broad &
macs2 callpeak -t TC1-H3K27-A-D3.GRCm38.p3.q30.bam \
               -c TC1-I-A-D3.GRCm38.p3.q30.bam \
               -f BAM --broad -g mm -n TC1-A-H3K27-D3-broad -B -q 0.01 --outdir TC1-A-H3K27-D3-broad

```

Get the `bigwig` files for **H3K27**.
Redo those sort and conversion steps but only for the folders that end with 'broad'

```
find TC*broad -name '*.bdg' | parallel "sort -k1,1 -k2,2n {} > {.}.sort.bdg"
find TC*broad -name '*sort.bdg' | parallel -j 2 "bedGraphToBigWig {} \
  /scratch/users/aginolhac/chip-seq/references/GRCm38.p3.chom.sizes {.}.bigwig"

```


## GREAT analysis

The website [GREAT](http://bejerano.stanford.edu/great/public/html/) allows pasting bed regions of enriched regions.

### predict functions of cis-regulatory regions

Using the `TC1-A-H3K4-D3_peaks.narrowPeak` file produced by MACS2.

This file has the different fields:

1. chromosome
1. start
1. end
1. peak name
1. integer score for display
1. strand
1. fold-change
1. -log<sub>10</sub> pvalue
1. -log<sub>10</sub> qvalue
1. relative summit position to peak start

Let's format the file as a 3 fields BED file and focus on more significant peaks filtering on *q-values*.

```
awk '$9>40'  TC1-A-H3K4-D3/TC1-A-H3K4-D3_peaks.narrowPeak | cut -f 1-3 | sed 's/^/chr/' >  TC1-A-H3K4-D3/TC1-A-H3K4_peaks.bed
cat TC1-A-H3K27-D3-broad/TC1-A-H3K27-D3-broad_peaks.broadPeak | cut -f 1-3 | sed 's/^/chr/' > TC1-A-H3K27-D3-broad/TC1-A-H3K27-D3-broad_peaks.broad.bed

```

then  

* load the BED in [GREAT](http://bejerano.stanford.edu/great/public/html/)  
* for the relevant genome, `mm10`  
* association rule:
    * _Single nearest gene_ for **H3K4** 
    * _Two nearest genes_ for **H3K27** 

### Differential peak calling

[THOR](http://www.regulatory-genomics.org/thor-2/basic-intrstruction/) allows comparing two conditions associated with their own controls and with replicates.

- first, index the bams

```
parallel "samtools index {}" ::: *bam
```

- second, create a config file `THOR.config` that contains:

```
#rep1
TC1-H3K4-ST2-D0.GRCm38.p3.q30.bam
#rep2
TC1-H3K4-A-D3.GRCm38.p3.q30.bam
#chrom_sizes
/scratch/users/aginolhac/chip-seq/references/GRCm38.p3.chom.sizes
#genome
/scratch/users/aginolhac/chip-seq/references/GRCm38.p3.fasta
#inputs1
TC1-I-ST2-D0.GRCm38.p3.q30.bam
#inputs2
TC1-I-A-D3.GRCm38.p3.q30.bam
```

A command line looks like
```
rgt-THOR -m -n TC1-I-A-D0vsD15 --output-dir=TC1-I-A-D0vsD1 THOR.config
```

takes ~ 25 minutes
