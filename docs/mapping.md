## paleomix, Next-Generation Sequencing wrapper

this framework is open-source and available on [GitHub](https://github.com/MikkelSchubert/paleomix) and 
wraps all steps from `fastq` to `bam` files. 
Actually, this tool can do much more but the rest is out of scope. 
Its major drawback is that it is dedicated to one machine. For clusters, you are then limited to one node since memory are not shared by default.
Actually, not entirely true since independent tasks can be spawn on a separate machine.
Full documentation available [here](http://paleomix.readthedocs.io/en/latest/)


check if paleomix is available
```
paleomix
```

### test your install

fetch the example, reference is the human mitochondrial genome
```
mkdir -p ~/install/paleomix/example
cp -r /work/users/aginolhac/chip-seq/paleomix/examples/bam_pipeline/00* ~/install/paleomix/example
cd ~/install/paleomix/example
```
run the example, start by a `dry-run`, adjust the number of threads accordingly.
```
paleomix bam_pipeline run --bwa-max-threads=1 --max-threads=2 --dry-run 000_makefile.yaml
```

If all fine, re-rerun the command without the `--dry-run` option

Of note, calling `mapDamage` was disabled to limit the computation time (~ 35 min when included).
Anyway, this tool is not used for ChIP-seq analysis.

### Generate a makefile

Trimming, mapping imply a lot of steps and it is hard to be sure that everything goes well. 
Paleomix works in temporary folders, check the data produced and then copy back files that are complete. 
Plus, you want to test different parameters, add a new reference without having to redo earlier steps while being sure that all files are up-to-date. 
This goes through a `YAML` makefile. The syntax is pretty straight-forward.
What matters is, that you use **SPACES** and not TABS.

Create a generic makefile (extension, `yml` or `yaml` to get syntax highlights)
```
cd ~/chip-seq
paleomix bam_pipeline mkfile > mouse.yaml
```

### Edit the makefile

using your favorite text editor, edit the `mouse.yaml`. For example `vim mouse.yaml` or `kate` or `nano`.

#### Options

For duplicates, change the default behavior from `filter` to `mark`  
```
  PCRDuplicates: mark
```

#### Features

Under the `Features` section, update the featurse that need to be performed.
Change `yes/no` to match the following:

```
  Features:
    RawBAM: yes         # Generate BAM from the raw libraries (no indel realignment)
                        #   Location: {Destination}/{Target}.{Genome}.bam
    RealignedBAM: no    # Generate indel-realigned BAM using the GATK Indel realigner
                        #   Location: {Destination}/{Target}.{Genome}.realigned.bam
    mapDamage: no       # Generate mapDamage plot for each (unrealigned) library
                        #   Location: {Destination}/{Target}.{Genome}.mapDamage/{Library}/
    Coverage: yes       # Generate coverage information for the raw BAM (wo/ indel realignment)
                        #   Location: {Destination}/{Target}.{Genome}.coverage
    Depths: no          # Generate histogram of number of sites with a given read-depth
                        #   Location: {Destination}/{Target}.{Genome}.depths
    Summary: yes        # Generate summary table for each target
                        #   Location: {Destination}/{Target}.summary
    DuplicateHist: no   # Generate histogram of PCR duplicates, for use with PreSeq
                        #   Location: {Destination}/{Target}.{Genome}.duphist/{Library}/
```

In detail, the `RealignedBAM` are important for calling variants, we only need to `RawBAM`.
Moreover, the `Depths` also help to define which upper limit could be used for variant calling.
This is not in the scope of ChIP-seq analysis. Same for mapDamage, only relevant for ancient DNA.

#### Prefixes

These are the references to align read to. You could notice that we are going to use only one chromosome
to save computational time.

```
Prefixes:
  mouse_19:
    Path: /work/users/aginolhac/chip-seq/doctoral_school/references/chr19.fasta
```


#### Samples

enter at the end of the makefile, the following lines, according to your login.
Again, do use **spaces** and not tabs for the indentation. For those who are lazy and use copy/paste in `vim`
 use the trick to `:set paste` to avoid extra spaces, comment hashes etc to be automatically added.

 Be careful to **replace** `student01` by the relevant username

```
TC1-I-A-D3:
  TC1-I-A-D3:
    TC1-I-A-D3:
      "14s006680-1-1":
        /home/users/student01/chip-seq/raw/C53CYACXX_TC1-I-A-D3_14s006682-1-1_Sinkkonen_lane114s006682_sequence.txt.gz

TC1-H3K4-A-D3:
  TC1-H3K4-A-D3:
    TC1-H3K4-A-D3:
      "14s006647-1-1":
        /home/users/student01/chip-seq/raw/C51C3ACXX_TC1-H3K4-A-D3_14s006647-1-1_Sinkkonen_lane514s006647_sequence.txt.gz

TC1-I-ST2-D0:
  TC1-I-ST2-D0:
    TC1-I-ST2-D0:
      "14s006677-1-1":
        /home/users/student01/chip-seq/raw/C51C3ACXX_TC1-I-ST2-D0_14s006677-1-1_Sinkkonen_lane814s006677_sequence.txt.gz

TC1-H3K4-ST2-D0:
  TC1-H3K4-ST2-D0:
    TC1-H3K4-ST2-D0:
      "14s006644":
        /home/users/student01/chip-seq/raw/C51C3ACXX_TC1-H3K4-ST2-D0_14s006644-1-1_Sinkkonen_lane514s006644_sequence.txt.gz
```

### Perform the trimming / mapping

First use the option `--dry-run` to spot mistakes.

Please **adapt** the `--max-threads` option to the #cpus actually booked

```
paleomix bam_pipeline run --bwa-max-threads=4 --max-threads=12 --dry-run mouse.yaml
```

when all green lights are on, remove the `dry-run` and perform the mapping.

## check trimming

First of all, check using `fastqc` that the trimming did remove the adapters that were contaminated the reads.

Again, with `parallel` specify the **max** number of jobs with the option `-j` to fit the #cpus booked

```
find . -name "reads.truncated.bz2" | parallel -j 12 "fastqc {}" &
```

using the character `&` tells the shell that we want the processes to run in the background. Meaning that you can still run more things while the 4 tasks are running. Check them using `htop`.

check especially, the input for ST2, day0 before and after trimming. Did it solve the issue with adapters?

## filter for unique reads

*Uniqueness* of reads refers to mappability. The fewer locations a read has in a genome, the higher is mappability will be.
A common filter is to use **30** as a threshold for filtering reads:

```
samtools view -b -q 30 file.bam > file.q30.bam
```

Filter in parallel

```
parallel "samtools view -b -q 30 {} > {.}.q30.bam" ::: *.bam
```

Since we are using only the chr19 for this tutorial, do you think the mappability score is correct? Why?

## filter for duplicates?

Duplication is a bias that comes from PCR amplification. Reads then stack at the same location and create artificial high depth of coverage.
Duplicates have an unclear definition in a mapped file. Usually, single-end reads that are mapped
at the same 5' end are considered as duplicates. External coordinates are used for paired-end reads.  
For regular NGS, filtering for duplicates is mandatory. However, for ChIP-seq since the reads are,
by nature, clustered at one location this is not recommended. If duplication is observed at the reads level, 
such as in `fastqc` output, then filtering may be necessary. Marking duplicates allow keeping track of them without losing them.
