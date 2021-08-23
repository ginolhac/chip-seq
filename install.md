
## INSTALL

### build modules

#### pysam

`pysam` requires `Cython`, load it first and easybuild too

```
module use $RESIF_ROOTINSTALL/lcsb/modules/all
module load lang/Cython/0.22-goolf-1.4.10-Python-2.7.3
module load base/EasyBuild
```

`cd ~/easybuild`


sometimes Cython is not found, reload cache by doing `module spider Cython`

#### paleomix


##### Dependencies

'nose>=1.3.0'
'flexmock>=0.9.7'
'coverage>=4.0.0'

solving `pysam` and `nose` was sufficient

`eb nose-1.3.7-goolf-1.4.10-Python-2.7.3.eb`
`eb pysam-0.9.1-goolf-1.4.10-Python-2.7.3.eb`


Need to add the path where pysam and nose are since paleomix depends on it
`eb -r /home/users/aginolhac/easybuild paleomix-1.2.5-goolf-1.4.10-Python-2.7.3.eb`

#### AdapterRemoval

version 2 is now published

eb AdapterRemoval-2.1.7-goolf-1.4.10.eb`


#### jar

picard and GATK can be used from resif 

#### mapDamage

`eb -r /home/users/aginolhac/easybuild mapDamage-2.0.6-goolf-1.4.10-Python-2.7.3.eb`

### load modules

```
module load bio/AdapterRemoval
module load bio/paleomix
module load bio/SAMtools/0.1.19-goolf-1.4.10
module load bio/BWA
module load bio/mapDamage
```


```
module load bio/AdapterRemoval
module load bio/paleomix
module load bio/SAMtools/0.1.19-goolf-1.4.10
module load bio/BWA
module load bio/mapDamage
```

### IGV install locally

can be connected to the galaxy server to display bams / bigwig etc.


## set-up ssh keys on Windows

**FIXME**: convert to `ppk` before for all students 

download `putty`, `puttygen` and `pageant` [here](http://www.chiark.greenend.org.uk/~sgtatham/putty/download.html)

`puttygen` will be useless once `ppk` keys are available.

1. Open `puttygen` to convert keys to `.ppk` format. 
    * load private key, should say imported with success
    * save private key with `.ppk` extension
1. Open `pageant` and open the ssh key in `ppk` format
1. Open `putty` with a new session
    * host: `access-gaia.uni.lu
    * port: `8022`
    * save as `gaia`
1. load the `gaia` session, a prompt asks for the login `student??` and should do


### Remenber galaxy works only with internal wi-fi

**FIXME**: external PhD cannot connect


### Test from a student account

using the `ssh` keys provided

`ssh -p 8022 -i ./student06.key student06@access-gaia.uni.lu`

student01 = Jonathan
student02 = Julia
student03 = Daniela
student04 = Xiabong
student05 = Turkan
student06 = test
student07 = Jonas
student08 = Nicolas

## mkdocs

set-up done in `mkdocs.yml`

### deploy

publish on [github](http://ginolhac.github.io/chip-seq/)
```
mkdocs gh-deploy --clean
```

### convert to pdf

install convertion to pandoc 

```
pip install mkdocs-pandoc
```

convert to PDF and EPUB

```
mkdocs2pandoc > mydocs.pd
pandoc --toc -f markdown+grid_tables+table_captions -o chip-seq_AG.pdf mydocs.pd
pandoc --toc -f markdown+grid_tables -t epub -o chip-seq_AG.epub mydocs.pd
```


