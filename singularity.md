## singularity

in `docker`, start from a ubuntu image and start an interactive session

```
docker pull ubuntu
docker run -i -t ubuntu bash
```

once in the container, do changes

```
root@ apt-get install python2.7 python-pip wget vim unzip less locales default-jre nano parallel
root@ echo -e "LANG=en_US.UTF-8\nLC_ALL=en_US.UTF-8" > /etc/default/locale
root@ pip install numpy
root@ pip install paleomix
root@ pip install MACS2
root@ wget https://github.com/samtools/samtools/releases/download/1.9/samtools-1.9.tar.bz2
root@ apt-get install liblzma-dev libbz2-dev  zlib1g-dev
root@ wget https://github.com/lh3/bwa/archive/v0.7.17.tar.gz ; tar xvfz v0.7.17.tar.gz
root@ cd bwa-0.7.17 ; make ; cp bwa /usr/local/bin/
root@ wget https://github.com/MikkelSchubert/adapterremoval/archive/v2.2.2.tar.gz;
root@ tar xvfz v2.2.2.tar.gz
root@ cd adapterremoval-2.2.2 ; make ; make install
root@ wget https://github.com/broadinstitute/picard/releases/download/2.18.13/picard.jar ; mv picard.jar usr/local/bin/
root@ wget https://www.bioinformatics.babraham.ac.uk/projects/fastqc/fastqc_v0.11.7.zip
root@ unzip fastqc_v0.11.7.zip; ln -s /FastQC/fastqc /usr/local/bin/fastqc
root@ wget http://hgdownload.soe.ucsc.edu/admin/exe/linux.x86_64/bedGraphToBigWig; chmod +x bedGraphToBigWig;mv bedGraphToBigWig /usr/local/bin 
```

for samtools 1.9 `./configure --without-curses  --prefix=/usr/local/; make; make install`
for R

```
root@ gpg --keyserver hkp://keyserver.ubuntu.com:80 --recv-keys 51716619E084DAB9
root@ gpg -a --export 51716619E084DAB9 | sudo apt-key add -
root@ echo 'deb https://cloud.r-project.org/bin/linux/ubuntu bionic-cran35/' >> /etc/apt/sources.list
root@ apt install r-base
```


after exiting, fetch the container ID and commit the changes, giving a new name

```
docker container ls -l
docker commit eb45dce12638 ubuntu-chip-seq
```

then connect with `docker run -i -t ubuntu-chip-seq bash`

### publish image to docker hub

```
export DOCKER_ID_USER="ginolhac"
docker login
docker tag ubuntu-chip-seq ginolhac/ubuntu-chip-seq
docker push ginolhac/ubuntu-chip-seq
```


#### create singularity image

on iris, sept 2018 version 2.5.2-dist

```
module load tools/Singularity
singularity pull docker://ginolhac/ubuntu-chip-seq
singularity shell -s /bin/bash --bind /scratch/users:/scratch/users /scratch/users/aginolhac/ubuntu-chip-seq.simg
```

`srun --cpu-bind=none -p interactive --time=XX -c 8 --pty bash -i`

### issues

- install `nano`
