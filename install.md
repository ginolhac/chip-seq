
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

### IGV install locally

can be connected to the galaxy server to display bams / bigwig etc.

## gaia container

book 2 nodes for the session

```
oarsub -t container -r "2016-09-08 08:00:00" -l nodes=2,walltime=10:00:00
oarsub -t container -r "2016-09-09 08:00:00" -l nodes=2,walltime=10:00:00
```

containers booked:
```
0 11:26:21 aginolhac@access(gaia-cluster) ~ $ oarstat -u aginolhac
Job id     Name           User           Submission Date     S Queue
---------- -------------- -------------- ------------------- - ----------
3950979                   aginolhac      2016-08-26 11:10:57 W default   
3950983                   aginolhac      2016-08-26 11:12:17 W default   
3950992                   aginolhac      2016-08-26 11:25:12 W default   
3950993                   aginolhac      2016-08-26 11:25:26 W default   
3950994                   aginolhac      2016-08-26 11:25:53 W default   
3950995                   aginolhac      2016-08-26 11:26:09 W default
```


