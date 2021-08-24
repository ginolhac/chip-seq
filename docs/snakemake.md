## Snakemake


The installation is from [Sarah Peter](https://wwwen.uni.lu/lcsb/people/sarah_peter), bioinformatician at the **LCSB**.
Her [tutorial](https://r3.pages.uni.lu/school/snakemake-tutorial/) is summarised here.

Main differences are we don't use a virtualbox VM, nor conda environments.

### Install Miniconda

[`Miniconda`](https://docs.conda.io/en/latest/miniconda.html) will provide you with a base conda environment.
If you are a regular use of the HPC, you may want to remove it at the end of this tutorial.

```bash
wget https://repo.anaconda.com/miniconda/Miniconda3-latest-Linux-x86_64.sh
chmod u+x Miniconda3-latest-Linux-x86_64.sh
./Miniconda3-latest-Linux-x86_64.sh
```

Follow the instructions prompted (use spacebar to scrool down the license), of note you need to specify your installation destination, e.g. `/home/users/username/miniconda3`. You must use the full path and cannot use `$HOME/miniconda3`. Answer `yes` to initialize Miniconda3.

Activate `conda` by reloading your BASH configuration

```bash
source ~/.bashrc
```

Notice that from now, you have an extra `(base)` written at the beginning of your prompt


### Finalize installation

- Update the permissions

```bash
(base) chmod +x $(which conda)
(base) chmod +x $(which conda-env)
```

- Update `conda`

```bash
(base) conda update conda
```

- Install `mamba` as recommended by Johannes Köster (`snakemake` author)

```bash
(base) conda install -c conda-forge mamba
```

- Ensure enclosed environments 

`conda` may use python modules if already installed be default. To avoid this behaviour, 
you need to add this line in your `.bashrc`. You need to edit it like with `vim ~/.bashrc`.
If you are not comfortable with editing files, see this [page](cli.md)

```bash
export PYTHONNOUSERSITE=True
```


### Install snakemake in dedicated environment

It is also recommended to leave the `base` environment as clean as possible,

Create a new conda environment and activate it:

```bash
(base) conda create -n snakemake
(base) conda activate snakemake
```

Now the prompt becomes `(snakemake)` and we can install `snakemake` inside it

```bash
(snakemake) mamba install -c conda-forge -c bioconda snakemake
```

Check that the snakemake is now installed

```bash
(snakemake) snakemake --version
```

Should return `6.7.0`


### (Optional) Revert the changes to your environment

From **Sarah Peter**, if you want to stop `conda` from always being active:

```bash
(base) conda init --reverse
```

In case you want to get rid of conda completely, you can now also delete the directory where you installed it (default is `$HOME/miniconda3`).