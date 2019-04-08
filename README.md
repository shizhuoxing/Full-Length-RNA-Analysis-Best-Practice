# Full-Length-RNA-Analysis-Best-Practice
*This is a Full-Length RNA Analysis pipeline developted by BGI RD group.*

As we all knows, with the progress of single molecule sequencing technology, full-length transcript sequencing will become more popular. Compared to the second generation sequencing technology, the three generations sequencing technology can detect full-length transcript from 5-end to polyA tail, this enables us to take the more accurate way to quantifying gene and isoform expression, and can take more accurate way to research isoform structure, such as alternative splicing(AS), alternative polyadenylation(APA), allele specific expression(ASE), fusion gene, UTR length and UTR secondary structure, etc.   
Here, we provide a command line's version bioinformatics pipeline for PacBio IsoSeq data analysis from raw subreads.bam, this pipeline was work well in both PacBio official IsoSeq library construction protocol and BGI patented mutil-isoform in one ZMW and full-length polyA tail library construction protocol. This pipeline contain quality control, basic statistics, full-length transcripts identification, isoform clustering, error correction and isoform quantification, and it is very easy to install and use.   

# Dependencies   
SMRTlink 6.0 or later   
`we suggest install in command line only model:``smrtlink-*.run --rootdir smrtlink --smrttools-only`   
blast   
R

# Usage
set `smrtlink` `blast` to you path.
```
exprot ....
```

## Step1 raw data chunking
Chunk and parallel processing of the raw data can significantly reduce computing time.
```
dataset create --type SubreadSet raw.subreadset.xml *.subreads.bam
dataset split --zmws --chunks 3 raw.subreadset.xml
```
## Step2 CCS for each chunk
```
perl creat_chunk_rtc.pl raw.chunk1.subreadset.xml ./ > resolved-tool-contract-1.json && ccs --resolved-tool-contract resolved-tool-contract-1.json   
perl creat_chunk_rtc.pl raw.chunk2.subreadset.xml ./ > resolved-tool-contract-2.json && ccs --resolved-tool-contract resolved-tool-contract-2.json  
perl creat_chunk_rtc.pl raw.chunk3.subreadset.xml ./ > resolved-tool-contract-3.json && ccs --resolved-tool-contract resolved-tool-contract-3.json  
```
## Step3 classify ccs by primer blast
### 3.1) cat ccs result in bam format from each chunk
```
ls ccs.bam > ccs.bam.list && bamtools merge ccs.bam.list
samtools view ccs.bam > ccs.sam
bamtools convert -format fasta -in ccs.bam -out ccs.fa 
samtools view ccs.bam | awk '{print $1"\t"length($11)"\t"$13"\t"$14}' | sed 's/np:i://' | sed 's/rq:f://' > ccs.stat 
```
### 3.2) make primer blast to ccs
```
makeblastdb -in primer.fa -out primer.fa
blastn -query ccs.fa -db primer.fa -outfmt 7 -word_size 5 > mapped.m7 
```
### 3.3) classify ccs by primer
```
perl classify_by_primer.pl mapped.m7 ccs.fa ./ 
```
## Step4 isoform cluster
```
perl fl_to_sam.pl ccs.sam isoseq_flnc.fasta > isoseq_flnc.sam 
```
## Step5 isoform expression quntify


# Contact
If you have any questions, encounter problems or potential bugs, don’t hesitate to contact us. Either report issues on github or write an email to:

Zhuoxing Shi - shizhuoxing@bgi.com
