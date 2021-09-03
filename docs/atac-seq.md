
The workflow of all steps is summarised below:

![](https://rawgit.com/ginolhac/chip-seq/main/atac.png)

and the template is hosted on the [LCSB Gitlab](https://gitlab.lcsb.uni.lu/aurelien.ginolhac/snakemake-atac-seq)


## Book resources on iris

`si` is a shortcut for booking a short **interactive** session (30 minutes, 1 core).

But, we can alter the time for one hour with `-t` and ask for more cores with `-c`. Example below where we ask for 1 jour and 6 cores:

```
si -t 1:00:00 -c 6
```

Interactive sessions are limited to maximum 2 hours.

### (Optional) Reservations

Two nodes were reserved for this course. This will work if you have a student accounts or if you are part of the Department of Life Sciences.
To access them you need to add the following parameter for Thursday:

```
si -t 1:0:0 -c 6 --reservation=aginolhac-teaching-20210902
```

and for Friday:

```
si -t 1:0:0 -c 6 --reservation=aginolhac-teaching-20210903
```

## Install the snakemake template

We will work in the `scratch` partition.

- the alias to go there is: 

```bash
cds
```

Type `pwd` to ensure you are in `/scratch/users/username`

- create the directory and go inside:

```bash
mkdir snakemake-atac-seq
cd snakemake-atac-seq
```

and run the following commands:

```bash
VERSION="v0.0.2"
wget -qO- https://gitlab.lcsb.uni.lu/aurelien.ginolhac/snakemake-atac-seq/-/archive/${VERSION}/snakemake-atac-seq-${VERSION}.tar.gz | tar xfz - --strip-components=1
```

this command will download, extract (without the root folder) the following files:

```
config/
Dockerfile
README.md
workflow/
```

you may want to delete the `Dockerfile` and `README.md` if you wish, 
they are not used by `snakemake` for runtime.

## Load necessary tools

- load `singularity`

the command once on a `node` is:

```bash
(base) user@access module load tools/Singularity
```

- load the `snakemake` environment

```bash
(base) user@access $ conda activate snakemake
```

Of note, the 2 above steps can be replaced by the alias `smk` if you added the alias in your `.bashrc`

It should look like this from accessing the access machine to getting the resources and activating the environment:

```bash
(base) aginolhac@access1.iris-cluster.uni.lux(14:05:02)-> 20:56): ~ $ si -c 6 -t 1:00:00
# salloc -p interactive --qos debug -C batch 
salloc: Pending job allocation 2424900
salloc: job 2424900 queued and waiting for resources
salloc: job 2424900 has been allocated resources
salloc: Granted job allocation 2424900
salloc: Waiting for resource configuration
salloc: Nodes iris-139 are ready for job
(base) aginolhac@iris-139(14:17:21)-> 29:51)(2424900 1N/T/1CN): ~ $ smk
(snakemake) aginolhac@iris-139(14:17:23)-> 29:49)(2424900 1N/T/1CN): ~ $
```

# Run on human data


> **Disclaimer**: those sequence files are of human origin. You **must not** copy them except for this teaching exercise.


## Fetch data

Make sure you are in `/scratch/users/username/snakemake-atac-seq/`, then copy the 12 fastq files:

```bash
mkdir data
rsync -v /scratch/users/aginolhac/snakemake-atac-seq/data/*gz data/
```

### Set-up the cache

To avoid spending too much time fetching the same reference genome and indexing it for mapping, we will share our work for these parts.

```bash
export SNAKEMAKE_OUTPUT_CACHE=/scratch/users/aginolhac/snakecache
```

### Dry-run

First a dry-run as we did before:

```bash
snakemake -j 6 -n
```

### Run

If all correct, run the workflow with `cache` activated. Of note, my scratch is shared for fetching the databases of `fastq_screen`.

```bash
snakemake --use-singularity --singularity-args "-B /scratch/users/aginolhac:/scratch/users/aginolhac"  --cache -j 6
```


### Fetch data back on your computer

The needed files are:

```
report.html
results/qc/multiqc/multiqc.html
results/big_wig/mDAN_D30_1.bigWig
results/big_wig/mDAN_D30_2.bigWig
results/big_wig/mDAN_D30_3.bigWig
results/big_wig/smNPC_1.bigWig
results/big_wig/smNPC_2.bigWig
results/big_wig/smNPC_3.bigWig
```