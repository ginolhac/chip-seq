## Workflow


The workflow of all steps is summarised below:

![](https://rawgit.com/ginolhac/chip-seq/master/workflow.png)

and the template is hosted on the [LCSB Gitlab](https://gitlab.lcsb.uni.lu/aurelien.ginolhac/snakemake-chip-seq)


### Singularity containers

[Singularity](https://www.sylabs.io/) allows to use containers (from _i.e_ [Docker](https://www.docker.com/)) on High-Performance Computer.
For more details see the lecture [by HPC team](https://ulhpc-tutorials.readthedocs.io/en/latest/containers/singularity/)

Shortly, a container with all the necessary tools, softwares and all libraries are embeded. Hence, you need to book the HPC resources but not install those softwares. **Snakemake** will load the container for every  **action** it runs.



### Book resources on iris

`si` is a shortcut for booking a short interactive session (30 minutes, 1 core).

But, we can alter the time for one hour with `-t` and ask for more cores with `-c`. Example below:

```
si -t 1:00:00 -c 6
```

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