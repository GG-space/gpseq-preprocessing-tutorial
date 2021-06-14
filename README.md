# GPSeq pre-processing example

<!-- MarkdownTOC -->

- [Introduction](#introduction)
- [Pre-processing tutorial](#pre-processing-tutorial)
    - [0. Parameters](#0-parameters)
    - [1. QC](#1-qc)
    - [2. Extract flags and their frequency](#2-extract-flags-and-their-frequency)
    - [3. Manual check](#3-manual-check)
    - [4. Filter by prefix](#4-filter-by-prefix)
    - [5. Map](#5-map)
    - [6. Filter mapping](#6-filter-mapping)
    - [7. Correct mapping](#7-correct-mapping)
    - [8. Group reads](#8-group-reads)
    - [9. Assign read groups to sites](#9-assign-read-groups-to-sites)
    - [10. De-duplicate](#10-de-duplicate)
    - [11. Generate final BED file](#11-generate-final-bed-file)

<!-- /MarkdownTOC -->

## Introduction

Lorem ipsum dolor sit amet, consectetur adipisicing elit, sed do eiusmod
tempor incididunt ut labore et dolore magna aliqua. Ut enim ad minim veniam,
quis nostrud exercitation ullamco laboris nisi ut aliquip ex ea commodo
consequat. Duis aute irure dolor in reprehenderit in voluptate velit esse
cillum dolore eu fugiat nulla pariatur. Excepteur sint occaecat cupidatat non
proident, sunt in culpa qui officia deserunt mollit anim id est laborum.

## Pre-processing tutorial

### 0. Parameters

```bash
# Parameters
input="fastq/KG35_S6_LALL_R1_001.fastq.gz"
libid="KG35"
bowtie2_ref="/mnt/data/Resources/references/GRCm38.r95.dna/Mus_musculus.GRCm38.95.dna.fa"
cutsite_path="/mnt/data/Resources/mm10.r95/recognition_sites/mm10.r95.MboI.bed.gz"
threads=10
```

### 1. QC

```bash
# FASTQ quality control
mkdir -p fastqc
fastqc $input -o fastqc --nogroup
```

### 2. Extract flags and their frequency

```bash
# Extract flags and filter by UMI quality
mkdir -p fastq_hq
fbarber flag extract \
    "$input" fastq_hq/$libid.hq.fastq.gz \
    --filter-qual-output fastq_hq/$libid.lq.fastq.gz \
    --unmatched-output fastq_hq/$libid.unmatched.fastq.gz \
    --log-file fastq_hq/$libid.log \
    --pattern 'umi8bc8cs4' --simple-pattern \
    --flagstats bc cs --filter-qual-flags umi,30,.2 \
    --threads $threads --chunk-size 200000
```

### 3. Manual check

### 4. Filter by prefix

```bash
# Filter by prefix
mkdir -p fastq_prefix
fbarber flag regex \
    fastq_hq/$libid.hq.fastq.gz fastq_prefix/$libid.fastq.gz \
    --unmatched-output fastq_prefix/$libid.unmatched.fastq.gz \
    --log-file fastq_prefix/$libid.log \
    --pattern "bc,^(?<bc>GTCGTCGA){s<2}$" "cs,^(?<cs>GATC){s<2}$" \
    --threads $threads --chunk-size 200000
```

### 5. Map

```bash
# Align
mkdir -p mapping
bowtie2 \
    -x "$bowtie2_ref" fastq_prefix/$libid.fastq.gz \
    --very-sensitive -L 20 --score-min L,-0.6,-0.2 --end-to-end --reorder -p $threads \
    -S mapping/$libid.sam &> mapping/$libid.mapping.log
```

### 6. Filter mapping

```bash
# Filter alignment
sambamba view -q -S mapping/$libid.sam -f bam -t $threads > mapping/$libid.bam

rm -i mapping/$libid.sam
sambamba view -q mapping/$libid.bam -f bam \
    -F "mapping_quality<30" -c -t $threads \
    > mapping/$libid.lq_count.txt
sambamba view -q mapping/$libid.bam -f bam \
    -F "ref_name=='chrM'" -c -t $threads \
    > mapping/$libid.chrM.txt
sambamba view -q mapping/$libid.bam -f bam -t $threads \
    -F "mapping_quality>=30 and not secondary_alignment and not unmapped and not chimeric and ref_name!='chrM'" \
    > mapping/$libid.clean.bam
sambamba view -q mapping/$libid.clean.bam -f bam -c -t $threads > mapping/$libid.clean_count.txt
```

### 7. Correct mapping

```bash
# Correct aligned position
mkdir -p atcs
sambamba view -q -t $threads -h -f bam -F "reverse_strand" \
    mapping/$libid.clean.bam -o atcs/$libid.clean.revs.bam
sambamba view -q -t $threads atcs/$libid.clean.revs.bam | \
    convert2bed --input=sam --keep-header - > atcs/$libid.clean.revs.bed
cut -f 1-4 atcs/$libid.clean.revs.bed | tr "~" $'\t' | cut -f 1,3,7,16 | gzip \
    > atcs/$libid.clean.revs.umi.txt.gz
rm atcs/$libid.clean.revs.bam atcs/$libid.clean.revs.bed

sambamba view -q -t $threads -h -f bam -F "not reverse_strand" \
    mapping/$libid.clean.bam -o atcs/$libid.clean.plus.bam
sambamba view -q -t $threads atcs/$libid.clean.plus.bam | \
    convert2bed --input=sam --keep-header - > atcs/$libid.clean.plus.bed
cut -f 1-4 atcs/$libid.clean.plus.bed | tr "~" $'\t' | cut -f 1,2,7,16 | gzip \
    > atcs/$libid.clean.plus.umi.txt.gz
rm atcs/$libid.clean.plus.bam atcs/$libid.clean.plus.bed
```

### 8. Group reads

```bash
# Group UMIs
scripts/group_umis.py \
    atcs/$libid.clean.plus.umi.txt.gz \
    atcs/$libid.clean.revs.umi.txt.gz \
    atcs/$libid.clean.umis.txt.gz \
    --compress-level 6 --len 4
rm atcs/$libid.clean.plus.umi.txt.gz atcs/$libid.clean.revs.umi.txt.gz
```

### 9. Assign read groups to sites

```bash
# Assign UMIs to cutsites
scripts/umis2cutsite.py \
    atcs/$libid.clean.umis.txt.gz $cutsite_path \
    atcs/$libid.clean.umis_at_cs.txt.gz --compress --threads $threads
rm atcs/$libid.clean.umis.txt.gz
```

### 10. De-duplicate

```bash
# Deduplicate
mkdir -p dedup
scripts/umi_dedupl.R \
    atcs/$libid.clean.umis_at_cs.txt.gz \
    dedup/$libid.clean.umis_dedupd.txt.gz \
    -c $threads -r 10000
```

### 11. Generate final BED file

```bash
# Generate final bed
mkdir -p bed
zcat dedup/$libid.clean.umis_dedupd.txt.gz | \
    awk 'BEGIN{FS=OFS="\t"}{print $1 FS $2 FS $2 FS "pos_"NR FS $4}' | \
    gzip > bed/$libid.bed.gz
``` 
