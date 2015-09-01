## Chip-seq practical session

Running all analyses is computationally intensive and despite the power of the current laptops, jobs should be run on high-performance-clusters (HPC).

### log in `gaia`

[`gaia`](https://hpc.uni.lu/systems/gaia/) is one of the [HPC of the UNI](https://hpc.uni.lu).

#### connect to the frontend

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

The frontend is meant for browsing / transfer your files only and you **MUST** connect to a node for any computational work using the utility `oarsub` described [here](https://hpc.uni.lu/users/docs/oar.html).

 Software are organized into modules that provide you with the binaries but also all the environment required for their running processes.

#### connect to a node

Connecting to a computing node is anyway required to use modules.

For Thursday:  
`oarsub -I -t inner=3505160 -l nodes=1,walltime=10`

For Friday:  
`oarsub -I -t inner=3505161 -l nodes=1,walltime=10`

### load necessary software as modules

 1. Add location of these modules

```
module use $RESIF_ROOTINSTALL/lcsb/modules/all
module use /home/users/aginolhac/.local/easybuild/modules/all/
```

 2. Load the modules

```
module load bio/FastQC
module load bio/AdapterRemoval
module load bio/pysam
module load bio/paleomix
module load bio/SAMtools
module load bio/BWA
module load bio/mapDamage
 ```

### prepare your working environment

go to your working directory:  
`cdw`
create a new folder to work in:  
`mkdir chip-seq`
go inside:  
`cd chip-seq`
create a sub-folder:
`mkdir raw`
