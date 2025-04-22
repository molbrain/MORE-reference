# MORE-reference

## Summary

The reference files and data are prepared for the MORE (Mobile-elements Originated Reads Enrichment) methods for sequencing data analysis. These files are available on bulk RNA-seq, WGS/WES, ChIP-seq, scRNA-seq, long-read sequencing analysis, etc. Of course, any other kind of analysis other than sequencing is also OK.

## Usage

For one of the typical examples, please see the GitHub site for [MORE-RNAseq](https://github.com/molbrain/MORE-RNAseq).
For other tools and pipelines, some examples are shared as follows:

### Example 1: [STAR](https://github.com/alexdobin/STAR) and [RSEM](https://github.com/deweylab/RSEM) 

```bash
## make reference files
wget https://f/ftp.ensembl.org/pub/release-102/fasta/homo_sapiens/dna/Homo_sapiens.GRCh38.dna.primary_assembly.fa.gz
wget https://ftp.ensembl.org/pub/release-102/gtf/homo_sapiens/Homo_sapiens.GRCh38.102.gtf.gz

zcat Homo_sapiens.GRCh38.102.gtf.gz L1fli_Utr5toUtr3.GRCh38.gtf.gz > reference.gtf

rsem-prepare-reference
    --star --star-path STAR \
    --gtf reference.gtf \
    --star-sjdboverhang 149 \
    Homo_sapiens.GRCh38.dna.primary_assembly.fa \
    ref

## Mapping and count (same as MORE-RNAseq)

STAR --runMode alignReads
    --runMode alignReads \
    --genomeDir ref \
    --readFilesCommand gunzip -c \
    --quantMode TranscriptomeSAM \
    --readFilesIn YOUR_DATA/data.R1.fastq.gz YOUR_DATA/data.R2.fastq.gz\
    --outSAMtype BAM SortedByCoordinate \
    --outSAMprimaryFlag AllBestScore \
    --outSAMmultNmax -1 \
    --outFilterMultimapNmax 100000

rsem-calculate-expression \
		--alignments \
		--paired-end \
		--append-names \
		YOUR_DATA/Aligned.sortedByCoord.out.bam \
		./ref \
		YOUR_DATA/

```

### Example 2: [TEtranscripts](https://github.com/mhammell-laboratory/TEtranscripts)

```bash

## After mapping by STAR as above

TEtranscripts --sortByPos --format BAM \
   --GTF Homo_sapiens.GRCh38.102.gtf.gz \
   --TE L1fli_Utr5toUtr3.GRCh38.gtf.gz \
   -t YOUR_DATA/T1/Aligned.sortedByCoord.out.bam YOUR_DATA/T2/Aligned.sortedByCoord.out.bam \
   -c YOUR_DATA/C1/Aligned.sortedByCoord.out.bam YOUR_DATA/C2/Aligned.sortedByCoord.out.bam
```

### Example 3: HISAT2

```

## HISAT2 is also OK to refer with MORE-reference

## underconstruction

hisat2-build \
    Homo_sapiens.GRCh38.dna.primary_assembly.fa \
    HISAT2_ref/GRCh38.102
```

### Example 4: single-cell RNA-seq (10X)

CellRanger also uses the STAR as the mapping tool inside. So, like Example 1, this rc-L1 information is available to create the reference for the CellRanger pipeline.


## Description

These reference files have been prepared to estimate and calculate the LINE-1 (L1) expression and novel insertion analysis. In this version (v1), the retrotransposition capable L1 (rc-L1) of humans and mice are listed in the GTF and BED format files, whose sequences correspond to the FASTA files. Additional annotations about the relationships of the rc-L1s and surrounding and flanking genes are also prepared as CSV files.

Most L1 analysis tools have used RepeatMasker's results as is. However, unlike the actual L1 transcripts, their entries are often annotated not as a single entry but as some divided entries. This causes problems such as the expression level of one rc-L1 being calculated as the expression level of two or more separate L1s. For this reason, we created a reference, MORE-reference, that has been curated in detail for these separated entries and for rc-L1s that were not detected by RepeatMasker but were identified in L1Base.


### "L1fli_Utr5toUtr3" files


The MORE-reference (v1) uses the following regions of rc-L1.

-The regions from the 5-term UTR to the 3-term UTR, including full-length intact ORF1 and ORF2.
-The 3-term of the regions are the same as the conserved poly-A signal sequences closest to ORF2.
-In the case of the mouse, the monomer regions are excluded (as these are repetitive regions and will become an artifact factor of multi-mapping for analyses requiring mapping such as NGS).

(図)

Following the above, we prepared GTF, BED, and FASTA-format files. `L1fli_Utr5toUtr3.GRCh38` is for human, and `L1fli_Utr5toUtr3.GRCm38` is for mouse. They correspond to the genomes of GRCh38 and GRCm38, respectively.
The IDs assigned are based on the same-named fli (full-length intact) L1 entries in L1Base2.
GTF also includes subfamily information: HS or PA2 for humans and Md-A, Md-Tf, Md-Gf, and Md-F for mice. If multiple subfamilies are noted, that entry has the chimeric subtypes.
These contain information on rc-L1, so in not only bulk RNA-seq but also TEtranscript, etc., rc-L1 can be added to each analysis by adding or replacing it with the GTF file of conventional genes or other reference information.


### "LIST_INTER_INTRA_GENE" files

These CSV files describe information about the INTRAGENIC/intergenic relations of each L1 and the surrounding/flanking genes. You can use this information to analyze intergenic-rc-L1s and intragenic-rc-L1s.

As the typical example, ID905 of mouse rc-L1 showed as follows:
```plain text
ID905	[ ID905 4659 <{ ENSMUSG00000115030 408 } 837 ]>	gene:ENSMUSG00000115030|location:COVERED|coverlength:408|flankdistance:0
ID905	{ ENSMUSG00000115329 }> 747 [ ID905 ]>	gene:ENSMUSG00000115329|location:flanking|coverlength:0|flankdistance:747
```
`[` and `]` are the terminal ends of L1s, and `{` and `}` are shown as terminal-ends of the genes. `<` and `>` are indicated as arrowheads in the direction of each transcription (5' to 3'). Each number other than ensemble IDs and L1 IDs is the distance between the terminal ends.


### "mart_export.102" files

These files were originally created by Biomart on ENSEMBL release 102. These data files are not important for the reference directory, but the above "LIST_INTER_INTRA_GENE" files depend on them.


## Future Plan

In the next version, we plan to support species other than humans and mice. We also plan to expand the reference to include expressible L1s other than non-rc-L1s. Furthermore, we are considering adding annotations for rc-L1, which has been observed to be polymorphic in human populations and specific lineages.
