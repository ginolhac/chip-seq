# Chip-seq practical session

Running all analyses is computationally intensive and despite the power of the current laptops, jobs should be run on high-performance-clusters (HPC).

## log in `gaia`

[`gaia`](https://hpc.uni.lu/systems/gaia/) is one of the [HPC of the UNI](https://hpc.uni.lu).

### connect to the frontend

To connect to it, you need an account and an authorized ssh key. After the setting up of your account, the following should work:

```
ssh gaia-cluster
```

You should the prompt of the gaia frontend:
```
===============================================================================
 /!\ NEVER COMPILE OR RUN YOUR PROGRAMS FROM THIS FRONTEND !
     First reserve your nodes (using oarsub(1))
Linux access.gaia-cluster.uni.lux 3.2.0-4-amd64 unknown
 16:45:49 up 126 days,  1:28, 34 users,  load average: 0.96, 1.58, 1.73
0 16:45:49 your_login@access(gaia-cluster) ~ $
```
Note that your are on the `access` frontend.

The frontend is meant for browsing / transfer your files only and you **MUST** connect to a node for any computational work using the utility `oarsub` described [here](https://hpc.uni.lu/users/docs/oar.html). This program managed the queuing system and dispatch jobs among the resources according to the demands.

 Software are organized into modules that provide you with the binaries but also all the environment required for their running processes.

### connect to a node

Connecting to a computing node is anyway required to use modules.

For Thursday:  
`oarsub -I -t inner=3505160 -l nodes=1,walltime=10`

For Friday:  
`oarsub -I -t inner=3505161 -l nodes=1,walltime=10`

Without entering into the details of [submitting a job](https://hpc.uni.lu/users/docs/oar.html#request-hierarchical-resources-with-oarsub), here is the explanation for the above command:

- `-I` is for interactive, default is passive
- `-t inner=xxx`, connect to a `container`, specific for today because we booked resources for the course
- `-l` define the resources you need. The less you ask for, the more high up you are in the queue. A `node` is composed of 12 `cores`, so 12 tasks could be run in parallel. `walltime` define for how long in hours your job will last.

Once logged in, the prompt changes for:
```
09:14:48 your_login@gaia-66(gaia-cluster)[OAR3511326->717]
```

where you see the node you are logged to (here `gaia-66`), the job ID (3511326) and the time in minutes before your job will be killed (717 minutes).

## monitoring your the resources used

On a shared cluster, you have to take of **three** things:
1. memory usage
2. cores used
3. disk space

### memory

Each node has
On a interactive session, use the command `htop` to see if the memory is not full. If the system is swapping (using hard drives for memory storage) it becomes super slow and eventually stalled.

For passive sessions, you can use [ganglia](https://hpc.uni.lu/gaia/ganglia/) to check out the nodes you are using.

### cores

even if you book 10 cores, nothing will prevent you from starting 100 jobs. They will run but then tasks are distributed on the available resources. In this example, each task will use 1/10th of a core, then runs very slowly.  
On a interactive session, use the command `htop` to see if a process is correctly using close to 100% of a core.

### disk space

Like on your local machine, you need to check how much data you used.
Using a command line, you could use
```
du -sh ~
```

to display your disk usage (`du`) for your home folder (`~`).

## load necessary software as modules

### Add location of these modules

```
module use $RESIF_ROOTINSTALL/lcsb/modules/all
module use /home/users/aginolhac/.local/easybuild/modules/all/
```

### Load the modules

```
module load bio/FastQC
module load bio/AdapterRemoval
module load bio/pysam
module load bio/paleomix
module load bio/SAMtools/0.1.19-goolf-1.4.10
module load bio/BWA
module load bio/mapDamage
```

### Tweak for the `picard-tools`

To get all jars available

```
mkdir -p ~/install/jar_root/
cp /opt/apps/sources/p/picard/picard-tools-1.100.zip ~/install/jar_root/
cd ~/install/jar_root/
unzip picard-tools-1.100.zip
mv  picard-tools-1.100/*.jar .
cd
```

you need to see `yes` to overwrite one file.

#### Final tweak for `Gatk`

```
cp /home/users/aginolhac/install/jar_root/GenomeAnalysisTK.jar ~/install/jar_root/
```

## prepare your working environment

go to your home directory:  
`cd`
create a new folder to work in:  
`mkdir chip-seq`
go inside:  
`cd chip-seq`
create and go in a sub-folder:
`mkdir raw ; cd raw`
symbolic link:
`ln -s /work/users/aginolhac/chip-seq/raw/C* .`


## check integrity of files

Just as a side note, such large files are usually a pain to download. Since they are the very raw files after the sequencer (despite basecalling) checking their integrity is worth doing. Computing the `md5um` ensure you have the same file as your sequence provider. Then `paleomix` will check the FASTQ is correct, *i. e* has 4 lines in a correct format.

`md5sum -c C53CYACXX_TC1-I-A-D3_14s006682-1-1_Sinkkonen_lane114s006682_sequence.txt.md5 `

## FASTQ Quality controls

Using [FastQC](http://www.bioinformatics.babraham.ac.uk/projects/fastqc/) you can perform the necessary controls over fastq files.

```
fastqc C51C3ACXX_TC1-H3K4-A-D3_14s006647-1-1_Sinkkonen_lane514s006647_sequence.txt.gz
```

### running serial

A tidy bit of `bash` programming to do it for all files
```
for f in *.gz
  do fastqc $f
done
```

### running in parallel

If you have booked **2** nodes, otherwise updae the `-j` option:

```
parallel -j 2 "fastqc {}" ::: *.gz
```

the `{}` instruction will be replaced by all occurences of the pattern `*.gz`, everything that ends by `.gz`. [`parallel`](http://www.gnu.org/software/parallel/) takes care of submitting a new job so the number of parallel remains the same.

### visualize the results

collect the `html` files using either `rsync`, `scp` for command lines or [FileZilla](https://filezilla-project.org/download.php?type=client) for  GUI tool.

You should observe some issues that needs to be solve.

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
  mouse_nuclear:
    # Path to .fasta file containg a set of reference sequences.
    Path: Path: /work/users/aginolhac/chip-seq/references/GRCm38.p3.fasta
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
```
