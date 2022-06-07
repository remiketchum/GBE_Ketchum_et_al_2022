GENOME ASSEMBLY PROTOCOL

1.1 Run CANU on PacBio Raw Subreads 

```
canu -p spez.canu.1300g.ml3k.def -d spez.canu.1300g.ml3k.def genomeSize=1300m minReadLength=3000 corOutCoverage=300 useGrid=false -pacbio-raw /projects/areitze2_research/dovet_spez/XDOVE_20200513_S64018_PL100156795-1_B01.subreads.fastq.gz
```

1.2 Run longranger to align 10x sequences to CANU Assembly 

```
/scratch/esmit245/10X/longranger-2.2.2/longranger basic --id=spez --fastqs=/projects/areitze2_research/dovet_spez/ --sample=DTG_DNA_134 --localcores 30
```

1.3 Run Pilon on CANU assembly, polished using 10x reads 

```
pilon -Xmx350G --genome spez.canu.1300g.ml3k.def.contigs.fasta --bam spez.contigs.barcoded.bam --changes --vcf --diploid --threads 16 --output spez.contigs.polished.bm
```

1.4 Run MiniMap to map long reads to assembly 

```
minimap2 -t30 -ax map-pb ../spez.contigs.polished.r1.fasta /projects/areitze2_research/dovet_spez/XDOVE_20200513_S64018_PL100156795-1_B01.subreads.fastq.gz --secondary=no | samtools sort -m 280G -o spez.canu.pol1x.aligned.pb.bam -T tmp.ali
```

1.5 generate histogram to determine cutoffs 

```
purge_haplotigs  hist  -b spez.canu.pol1x.aligned.pb.bam  -g ../spez.contigs.polished.r1.fasta -t 30
```

1.6 Classify contigs according to the cutoffs 

```
purge_haplotigs contigcov -i spez.canu.pol1x.aligned.pb.bam.gencov  -l 7  -m 100  -h 150
```

1.7 Run Purge: -a is the percent cutoff for identifying a contig as a haplotig (default=70), I used values ranging from 35-55 to account for the high heterozygosity in this genome. A value of 40 returned the most complete genome with the least amount of duplicated genes present in the assembly.

```
purge_haplotigs purge -t 30 -g ../spez.contigs.polished.r1.fasta  -c coverage_stats.csv -a 40 -o output.40
```

1.8 Run Clip on output from Purge to trim overlapping contig ends

```
purge_haplotigs clip -t 40  -p output.40.fasta  -h output.40.haplotigs.fasta -o output.40.clip
```

##Dovetail Genomics then ran HiRise on the genome assembly generated above.##