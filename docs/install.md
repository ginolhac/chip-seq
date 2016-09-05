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
module load bio/paleomix
module load bio/SAMtools/0.1.19-goolf-1.4.10
module load bio/BWA
module load bio/mapDamage
module load bio/MACS2
module load lang/Java
module load lang/R/3.3.0-ictce-7.3.5-bare
```

### Tweak for the `picard`


```
mkdir -p ~/install/jar_root/
cp /home/users/aginolhac/install/jar_root/picard.jar ~/install/jar_root/
```

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
`mkdir raw`
go inside:
`cd raw`
symbolic link the fastq files:
`ln -s /work/users/aginolhac/chip-seq/doctoral_school/raw/C* .`


## check integrity of files

Just as a side note, such large files are usually a pain to download. Since they are the very raw files
after the sequencer (despite basecalling) checking their integrity is worth doing.
Computing the `md5sum` ensure you have the same file as your sequence provider.
Then `paleomix` will check the FASTQ are correct, *i. e* have 4 lines in a correct format.

`md5sum -c C53CYACXX_TC1-I-A-D3_14s006682-1-1_Sinkkonen_lane114s006682_sequence.txt.md5`
