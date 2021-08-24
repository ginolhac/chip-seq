# ChIP-seq practical session

Running all analyses is computationally intensive and despite the power of the current laptops, jobs should be run on high-performance clusters (HPC).


## Lecture

slides are available as a pdf, click below

[![](https://rawgit.com/ginolhac/chip-seq/master/slides.png)](https://rawgit.com/ginolhac/chip-seq/master/chip-seq.pdf)


## Log in `iris`

[`iris`](https://hpc-docs.uni.lu/systems/iris/) is one of the [High Performance Computer (HPC) of the UNI](https://hpc.uni.lu).

You should use your account or one the student account prepared for you.

### Connect to the frontend

To connect to it, you need an account and an authorized `ssh` key. Actually, a pair of keys, one public and one private.
The public key is sent over when connecting to the remote and compared to the authorized private key.
A match allows the sender to log in. No password required.

After the setting up of your account, the following should work if you are using mac or GNU/Linux:

```bash
ssh iris-cluster
```

Otherwise, on Windows, use `xmobaterm`. In the terminal, log as your username, such as `student15`.

You should see the following prompt of the `iris` frontend:

```console
==================================================================================
 /!\ NEVER COMPILE OR RUN YOUR PROGRAMS FROM THIS FRONTEND !
     First reserve your nodes (using srun/sbatch(1))
username@access1.iris-cluster.uni.lux(11:30:46): ~ $
```

Note that you are on the `access` frontend.

The frontend is meant for browsing / transferring your files only and you **MUST** connect to a node for any computational work 
using the utility `slurm` described [here](https://hpc-docs.uni.lu/slurm/). This program managed the queuing system and dispatch jobs among the resources according to the demands.

Softwares are organized into **modules** that provide you with the binaries but also all the environment required for their running processes.
However, we will use a [container](https://www.docker.com/resources/what-container) that will ease our analyses.

The login procedure can be depicted as:

![](https://rawgit.com/ginolhac/chip-seq/master/access.jpg)

## TMUX

log in to a remote computer is great, all computation, heat generation is happening elsewhere but this comes with a price: disconnection.  
This happens all the time. The way to get around it is to have a `screen` system that stores your terminal, commands, environment in which 
you can easily detach and re-attach.

Two systems exist, `screen` and `tmux`. Both work well, but `tmux` has a nicer interface IMHO.

a short tutorial is [accessible here.](https://www.sitepoint.com/tmux-a-simple-start/)

Briefly, on the **access** frontend, start a `tmux` instance with

```
tmux
```

to detach (press CTRL and B together, release then use the next key):

```
CTRL + B, then D
```

to re-attach:

```
tmux attach
```

or the alias

```
tmux at
```


### Useful commands

once in an instance,

- **create** a new tab

```
CTRL + B, then C
```

- move to the **next** tab

```
CTRL + B, then N
```

- move to the **previous** tab

```
CTRL + B, then P
```

- **rename** to the current tab

```
CTRL + B, then ,
```

then type the new name

### Quit

```
exit
```

in all tabs kills the `tmux` session


Of note, `tmux` instances live until the frontend is rebooted.

### Book resources on a computing node

Connecting to a computing node is required to use resources.

You need to book resources by specifying how many cores, optionally if they are on a same node, the memory required, and a wall time clock. A job can never get extended.

Without entering into the details of [submitting a job](https://hpc-docs.uni.lu/slurm/commands/). The less you ask for, the more high up you are in the queue. Here is the explanation for the above command:

- `srun` is for interactive, `sbatch` for passive
- `--time=` following by _hour:minute:second_ for wall time clock
- `--mem=` with `12GB` for booking 12 gigabytes
- `-c` cores, A `node` is usually composed of 28 `cores`

Once logged in, the prompt changes for:

```bash
username@iris-001(11:17:55)
```

where you see the node you are logged to (here `iris-001`).

## Monitoring the resources used

On a shared cluster, you have to take care of **three** things:

1. memory usage
2. cores used
3. disk space

### Memory

Each node has
On an interactive session, use the command `htop` to see if the memory is not full. If the system is swapping (using hard drives for memory storage)
it becomes super slow and eventually stalled.

For passive jobs, you can join a computing node by using the `sjoin nodeid jobid` command. Where `nodeid` and `jobid` can be autocomplemented by <key>TAB</key>

### Cores

even if you book 10 cores, nothing will prevent you from starting 100 jobs. They will run but then tasks are distributed on the available resources.
In this example, each task will use 1/10th of a core, then runs very slowly.  
On an interactive session, use the command `htop` to see if a process is correctly using close to 100% of a core.

### Disk space

Like on your local machine, you need to check how much data you used.
Using a command line, you could use

#### Disk usage

```
du -sh ~
```

to display your disk usage (`du`) for your home folder (`~`). In a form readable by human (`-h`)

#### Disk free

```
df-ulphc
```

disk free scans all disks mounted. Could takes time to display the global usage. 
Please worry if only few Mb are available on the disk you are planning to write to.

check also your own quota with `df-ulhpc` on the frontend.

### Closing connection

When you are done, you can kill yourself your job by either doing `CTRL + D` or typing `exit`.
That will free your booked resources for others. Once done, you will still log on the frontend and normally inside a `tmux`.
The best is to detach from the `tmux` instance and log off from the gaia frontend using  `CTRL + D` or typing `exit`.
