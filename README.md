## Chip-seq practical session


### Public data



Go on [NCBI GEO](http://www.ncbi.nlm.nih.gov/sra?linkname=bioproject_sra_all&from_uid=125035)

Then select RunSelector.

Chose samples as Lasses' instructions

From a _Cell_ paper from 2010 and belong to this dataset:
 
http://www.ncbi.nlm.nih.gov/geo/query/acc.cgi?acc=GSE20752
 
Specifically I would look at these samples:
 
`3T3L1_t2_H3K4me3`
http://www.ncbi.nlm.nih.gov/geo/query/acc.cgi?acc=GSM535748
 
`3T3L1_t3_H3K4me3`
http://www.ncbi.nlm.nih.gov/geo/query/acc.cgi?acc=GSM535755
 
`3T3L1_t2_H3K27ac`
http://www.ncbi.nlm.nih.gov/geo/query/acc.cgi?acc=GSM535751
 
`3T3L1_t3_H3K27ac`
http://www.ncbi.nlm.nih.gov/geo/query/acc.cgi?acc=GSM535758
 
I think this is the input control they used for normalization:
http://www.ncbi.nlm.nih.gov/geo/query/acc.cgi?acc=GSM535740


Download `SRR_Acc_List.txt` in `/work/users/aginolhac/chip-seq/doctoral_school/Mikkelsen`

then fetch and compress

`parallel -j 6 --progress "fastq-dump --gzip {}" :::: SRR_Acc_List.txt`

mapped with paleomix

`paleomix bam_pipeline --bwa-max-threads=4 --max-threads=12 mikkelsen.yml`

last for ~ 3 hours 30 minutes

