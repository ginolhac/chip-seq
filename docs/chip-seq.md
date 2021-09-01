## Workflow


The workflow of all steps is summarised below:

![](https://rawgit.com/ginolhac/chip-seq/master/workflow.png)

and the template is hosted on the [LCSB Gitlab](https://gitlab.lcsb.uni.lu/aurelien.ginolhac/snakemake-chip-seq)


### Singularity containers

[Singularity](https://www.sylabs.io/) allows to use containers (from _i.e_ [Docker](https://www.docker.com/)) on High-Performance Computer.
For more details see the lecture [by HPC team](https://ulhpc-tutorials.readthedocs.io/en/latest/containers/singularity/)

Shortly, a container with all the necessary tools, softwares and all libraries are embeded. Hence, you need to book the HPC resources but not install those softwares. **Snakemake** will load the container for every  **action** it runs.



### Book resources on iris

`si` is a shortcut for booking a short **interactive** session (30 minutes, 1 core).

But, we can alter the time for one hour with `-t` and ask for more cores with `-c`. Example below where we ask for 1 jour and 6 cores:

```
si -t 1:00:00 -c 6
```

Interactive sessions are limited to maximum 2 hours. See at the bottom how to submit **passive** jobs (max walltime: 48 hours).

#### (Optional) 

Two nodes were reserved for this course. This will work if you have a student accounts or if you are part of the Department of Life Sciences.
To access them you need to add the following parameter for Thursday:

```
si -t 1:0:0 -c 6 --reservation=aginolhac-teaching-20210902
```

and for Friday:

```
si -t 1:0:0 -c 6 --reservation=aginolhac-teaching-20210903
```

### Install the snakemake template

We will work in the `scratch` partition.

- the alias to go there is: 

```bash
cds
```

Type `pwd` to ensure you are in `/scratch/users/username`

- create the directory and go inside:

```bash
mkdir snakemake-chip-seq
cd snakemake-chip-seq
``` 

and run the following commands:

```bash
VERSION="v0.1.1"
wget -qO- https://gitlab.lcsb.uni.lu/aurelien.ginolhac/snakemake-chip-seq/-/archive/${VERSION}/snakemake-chip-seq-${VERSION}.tar.gz | tar xfz - --strip-components=1
```

this command will download, extract (without the root folder) the following files:

```
CHANGELOG.md
config/
Dockerfile
LICENSE
README.md
resources/
workflow/
```

You can check the list of files using `ll` (alias of long list: `ls -l`)

you may want to delete the `LICENSE`, `Dockerfile`, `CHANGELOG.md` and `README.md` if you wish, 
they are not used by `snakemake` for runtime.

### Fetch test datasets

Using the [nextflow datasets](https://github.com/nf-core/test-datasets), clone it using `git`:

```bash
git clone -b chipseq --depth 1 https://github.com/nf-core/test-datasets.git
```

### Load necessary tools

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


### Test the workflow 

#### Config files

3 tabulated separated values file (`.tsv`) defined where are the data and how to process them. They are all in the `config` folder.
Everything in the `workflow` contains the snakemake machinery and can be ignored if you are not interested into this.

- units

This file has the key column in the first column, that must match the first colummn of `samples.tsv`.
Then `unit` are for **technical** replicates. 

`fq1` is the path the first pair of `FASTQ` or for single-end. In this last case, `fq2` needs to be empty.

The 2 last columns, `sra_accession` and `platform` are not going to be used.

Display the content of `config/units.tsv` (using `less config/units.tsv` for example) it should look like:

```
sample	unit	fq1	fq2	sra_accession	platform
Spt5_IN	1	test-datasets/testdata/SRR5204809_Spt5-ChIP_Input1_SacCer_ChIP-Seq_ss100k_R1.fastq.gz	test-datasets/testdata/SRR5204809_Spt5-ChIP_Input1_SacCer_ChIP-Seq_ss100k_R2.fastq.gz		ILLUMINA
Spt5_IN	2	test-datasets/testdata/SRR5204810_Spt5-ChIP_Input2_SacCer_ChIP-Seq_ss100k_R1.fastq.gz	test-datasets/testdata/SRR5204810_Spt5-ChIP_Input2_SacCer_ChIP-Seq_ss100k_R2.fastq.gz		ILLUMINA
Spt5	1	test-datasets/testdata/SRR5204807_Spt5-ChIP_IP1_SacCer_ChIP-Seq_ss100k_R1.fastq.gz	test-datasets/testdata/SRR5204807_Spt5-ChIP_IP1_SacCer_ChIP-Seq_ss100k_R2.fastq.gz		ILLUMINA
Spt5	2	test-datasets/testdata/SRR5204808_Spt5-ChIP_IP2_SacCer_ChIP-Seq_ss100k_R1.fastq.gz	test-datasets/testdata/SRR5204808_Spt5-ChIP_IP2_SacCer_ChIP-Seq_ss100k_R2.fastq.gz		ILLUMINA
```

- samples

Now look at `config/samples.tsv`:

```
sample	group	batch_effect	control	antibody	peak-analysis
Spt5_IN	SptA	batch1		Spt	
Spt5	SptA	batch1	Spt5_IN	Spt	narrow
```

`sample` are ID that must match the `units.tsv` `sample` column. 

`group` are for biological replicate, `batch_effect` is self-explanatory.

`control` allows to specify which **input** has to be used. Leave it empty for input DNA sample.

`antibody` is for antobody or histone mark used.

`peak-analysis` either `narrow` (for K4) or `broad` (K27 or K36). This will be used when running `macs2`.

- config

The file `config.yaml` contains the generic parameters for the analysis.

More specifically, it allows to specify the **reference genome** to use.

Here the section is designed for the **yeast** genome, that fits the test-data

```
resources:
  ref:
    # Ensembl species name
    species: saccharomyces_cerevisiae
    # Ensembl release
    release: 101
    # Genome build
    build: R64-1-1
    # for testing data a single chromosome can be selected (leave empty for a regular analysis)
    chromosome:
    # specify release version number of igenomes list to use (see https://github.com/nf-core/chipseq/releases), e.g. 1.2.2
    igenomes_release: 1.2.2
    # if igenomes.yaml cannot be used, a value for the mappable or effective genome size can be specified here, e.g. macs-gsize: 2.7e9
    macs-gsize:
    # if igenomes.yaml cannot be used, a path to an own blacklist can be specified here
    blacklist:
```

This section will need to be updated when we use the real human data

#### Run the test-data workflow

1. Check that your prompt is indicating that you are on a computing node, and with the snakemake environment loaded.

Something like that:

```
(snakemake) 0 [student20@iris-114 student20](2469485 1N/T/1CN)$
```

2. Dry-run

```
snakemake -n
```

```
[Wed Sep  1 10:53:50 2021]
localrule all:
    input: <TBD>
    jobid: 0
    resources: tmpdir=/tmp

Job stats:
job                  count    min threads    max threads
-----------------  -------  -------------  -------------
all                      1              1              1
generate_igenomes        1              1              1
get_gsize                1              1              1
total                    3              1              1

This was a dry-run (flag -n). The order of jobs does not reflect the order of execution.
```

3. Produce the Direct Acyclic Graph

With the alias, it is `dag`. You can then fetch and look at the `dag.pdf`. All jobs are with a plain line, meaning they are not done.

4. Run the real workflow

```bash
snakemake --use-singularity --singularity-args "-B /scratch/users/aginolhac:/scratch/users/aginolhac"  -j 6
```

First it download the `singularity` image (and will cache it). It takes a few minutes (feels long however).

Then you can see the workflow moving on. **107** jobs are scheduled.

Messages like

```
System has not been booted with systemd as init system (PID 1). Can't operate.
Failed to create bus connection: Host is down
```

can be ignored. The whole workflow takes ~ 15 minutes with 6 cores. 

5. Create the report

```
snakemake --report
```

You can now fetch both `report.html` and `results/qc/multiqc/multiqc.html` that summarise the run.

6. DAG again

if you wish, the lines are now dashed.

### Submit passive jobs

It consists in two steps:

- create a file `launcher.sh` at the root of your snakemake that contains the following lines:

```bash
#!/bin/bash -l
#SBATCH -N 1
#SBATCH -J JOB_NAME
#SBATCH --mail-type=begin,end,fail
#SBATCH --mail-user=yours@mail.lu
#SBATCH --ntasks-per-node=1
#SBATCH --mem-per-cpu=4096
#SBATCH -c 12
#SBATCH --time=0-06:00:00
#SBATCH -p batch

module load tools/Singularity
conda activate snakemake

srun snakemake --use-singularity --singularity-args "-B /scratch/users/aginolhac:/scratch/users/aginolhac" -j 12

```

You need to update the `JOB_NAME` and your email address. You can also delete those 3 lines (line 3 to 5) if you are interested into this monitoring.

The duration (here 6 hours) and number of cores can be adapted to your needs.

- run the submission with 

```bash
sbatch launcher.sh
```

your job will be queued and start whenever it's possible and according to the [fairshare](https://hpc-docs.uni.lu/slurm/fairsharing/).

You can monitor your scheduling with `squeue -la -u $USER`


See also the HPC team [tutorial](https://ulhpc-tutorials.readthedocs.io/en/latest/basic/scheduling/).

