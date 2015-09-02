## paleomix, Next-Generation Sequencing wrapper

this framework is open-source and available at [GitHub](https://github.com/MikkelSchubert/paleomix) and wrap all steps from `fastq` to `bam` files. Actually, this tol can do much more but the rest is out of scope.

check if paleomix is available
```
paleomix -h
```

### test your install

fetch the example, reference is the human mitochondrial genome
```
mkdir ~/install/paleomix/example
cp -r /work/users/aginolhac/chip-seq/paleomix/examples/bam_pipeline/00* ~/install/paleomix/example
cd ~/install/paleomix/example
```
run the example, start by a `dry-run`, adjust the number of threads accordingly.
```
paleomix bam_pipeline run --bwa-max-threads=1 --max-threads=2 --dry-run 000_makefile.yaml
```

If all fine, re-rerun the command without the `--dry-run` option

### Generate a makefile

Trimming, mapping imply a lot of steps and it is hard to be sure that everything goes well. Paleomix works in temporary folder, check the data produced and then copy back files that are complete. Plus, you want to test different parameters, add a new reference without having to redo earlier steps while being sure that all files are up-to-date. This goes through a `YAML` makefile. The syntax is pretty forward.

Create a generic makefile
```
cd ~/chip-seq
paleomix bam_pipeline mkfile > mouse.makefile
```

### Edit the makefile

using your favorite editor, edit the `mouse.makefile`. For example `vim mouse.makefile` or `kate` or `nano`.

#### Features

Under the `Features` section, comment with a `#` the part that should be run to fit the following
```
Features:
  - Raw BAM        # Generate BAM from the raw libraries (no indel realignment)
                   #   Location: {Destination}/{Target}.{Genome}.bam
#    - Realigned BAM  # Generate indel-realigned BAM using the GATK Indel realigner
                   #   Location: {Destination}/{Target}.{Genome}.realigned.bam
#    - mapDamage      # Generate mapDamage plot for each (unrealigned) library
                   #   Location: {Destination}/{Target}.{Genome}.mapDamage/{Library}/
  - Coverage       # Generate coverage information for the raw BAM (wo/ indel realignment)
                   #   Location: {Destination}/{Target}.{Genome}.coverage
#    - Depths         # Generate histogram of number of sites with a given read-depth
                   #   Location: {Destination}/{Target}.{Genome}.depths
  - Summary        # Generate target summary (uses statistics from raw BAM)
                   #   Location: {Destination}/{Target}.summary
```

#### Prefixes

These are the references to align read to.

```
Prefixes:
  # Name of the prefix; is used as part of the output filenames
  mouse_19:
    # Path to .fasta file containg a set of reference sequences.
    Path: Path: /work/users/aginolhac/chip-seq/references/chr19.fasta
```


#### Samples

enter at the end of the makefile, the following lines, according to your login.
Do use **spaces** and not tabs for the indentation.

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

```
paleomix bam_pipeline run --bwa-max-threads=1 --max-threads=12 --dry-run mouse.makefile
```
