## FASTQ Quality controls

Using [FastQC](http://www.bioinformatics.babraham.ac.uk/projects/fastqc/) you can perform the necessary controls over FASTQ files.

```
fastqc C51C3ACXX_TC1-H3K4-A-D3_14s006647-1-1_Sinkkonen_lane514s006647_sequence.txt.gz
```

but one file at a time, does not benefit the HPC.

### running in parallel

If you have booked at least **4** cores, otherwise update the `-j` option:

```
parallel -j 4 "fastqc {}" ::: fastq/*.gz
```

the `{}` instruction will be replaced by all occurrences of the pattern `*.gz`, everything that ends by `.gz`. [`parallel`](http://www.gnu.org/software/parallel/) 
takes care of submitting a new job so the number of parallel remains the same.

### running serial (for information)

A tidy bit of `bash` programming to do it for all files **NOT** in parallel

```
for f in *.gz
  do fastqc $f
done
```

### visualize the results

collect the `html` files using either `rsync`, `scp` for command lines or [FileZilla](https://filezilla-project.org/download.php?type=client) for a GUI tool.

You should observe some issues that needs to be solve.
