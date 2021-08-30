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

But, we can alter the time for one hour with `-t` and ask for more cores with `-c`. Example below:

```
si -t 1:00:00 -c 6
```

Interactive sessions are limited to maximum 2 hours. See at the bottom how to submitted **passive** jobs.

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

