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
ln -s /work/users/aginolhac/chip-seq/data/*.bam .
```

### Perform peak calling

```
macs2 callpeak -t TC1-H3K4-ST2-D0.GRCm38.p3.q30.bam \
               -c TC1-I-ST2-D0.GRCm38.p3.q30.bam \
               -f BAM -g mm -n TC1-ST2-H3K4 -B -q 0.01 --outdir TC1-ST2-H3K4
macs2 callpeak -t TC1-H3K4-A-D3.GRCm38.p3.q30.bam \
               -c TC1-I-A-D3.GRCm38.p3.q30.bam \
               -f BAM -g mm -n TC1-A-H3K4 -B -q 0.01 --outdir TC1-A-H3K4
```

### sort per chr and coordinates

```
find TC* -name '*.bdg' | parallel "sort -k1,1 -k2,2n {} > {.}.sort.bdg"
```

### convert to bigwig

```
find TC* -name '*sort.bdg' | parallel -j 1 "/work/users/aginolhac/chip-seq/bedGraphToBigWig {} /work/users/aginolhac/chip-seq/references/GRCm38.p3.chom.sizes {.}.bigwig"
```

### Fetch the files and display them in IGV
