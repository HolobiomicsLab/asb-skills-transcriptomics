---
name: small-rna-reference-annotation
description: Use when you have STAR-aligned BAM files from human small RNA-seq data
  (e.g. hg38/hg19) and need to classify reads into miRNA, piRNA, tRNA, snRNA, snoRNA
  and circRNA classes with per-molecule counts.
when_to_use_negative:
- Input reads are not yet mapped to a reference genome — COMPSRA's Annotation module
  expects the STAR Alignment module's BAM output, not raw FASTQ
- You need single-assignment (one RNA type per read) classification — COMPSRA deliberately
  provides all possible annotations and assigns counts to each overlapping small RNA
- Target organism is non-human and non-rat without pre-built databases — prebuilt
  small RNA databases cover human hg19/hg38 (and miRNA for Rattus norvegicus rno5/rno6
  as of v1.0.1)
edam_operation: http://edamontology.org/operation_0362
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_0099
tools:
- name: COMPSRA
  role: Hosts the Annotation module (-ann) that performs the multi-database small
    RNA annotation with LiftOver-based genome harmonization
  repo: https://github.com/cougarlj/COMPSRA
- name: STAR
  role: Default aligner whose mapped-read BAM output is the input to the Annotation
    module
- name: miRBase
  role: Reference database used first for miRNA annotation
- name: gtRNAdb
  role: Reference database for tRNA annotation
- name: GENCODE release 27
  role: Reference database for snRNA and snoRNA annotation
- name: circBase
  role: Reference database for circRNA annotation
provenance:
  source_task_ids:
  - task_006
  - task_003
  source_papers:
  - doi: 10.1038/s41598-020-61495-0
    title: 'COMPSRA: a COMprehensive Platform for Small RNA-Seq data Analysis'
  source_content_hash: sha256:856a7228d7f596a308c12ca4e0eaad853c4f6a155ebb615b99a6d98f30771a3d
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/cougarlj/COMPSRA
  tools:
  - COMPSRA
  - GENCODE release 27
  - Java
  - STAR
  - circBase
  - exceRpt
  - gtRNAdb
  - miRBase
  edam_operation: http://edamontology.org/operation_0362
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_0099
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9230769230769231
n_claims: 13
---

# small-rna-reference-annotation

## Summary

Annotate human genome-mapped small RNA-seq reads against multiple pre-built small RNA reference databases (miRBase, piRNABank/piRBase/piRNACluster, gtRNAdb, GENCODE, circBase) within COMPSRA's Annotation module, producing per-RNA-type read count tables rather than forcing a single RNA-type assignment per read.

## When to use

Use when you have STAR-aligned BAM files from human small RNA-seq data (e.g. hg38/hg19) and need to classify reads into miRNA, piRNA, tRNA, snRNA, snoRNA and circRNA classes with per-molecule counts.

## When NOT to use

- Input reads are not yet mapped to a reference genome — COMPSRA's Annotation module expects the STAR Alignment module's BAM output, not raw FASTQ
- You need single-assignment (one RNA type per read) classification — COMPSRA deliberately provides all possible annotations and assigns counts to each overlapping small RNA
- Target organism is non-human and non-rat without pre-built databases — prebuilt small RNA databases cover human hg19/hg38 (and miRNA for Rattus norvegicus rno5/rno6 as of v1.0.1)

## Inputs

- STAR alignment output BAM from hg38/hg19-aligned qualified small RNA-seq reads (e.g. sample01_17to50_FitRead_STAR_Aligned.out.bam)
- Pre-built COMPSRA annotation databases (miRNA_hg38, piRNA_hg38, tRNA_hg38, snRNA_hg38, snoRNA_hg38, circRNA_hg38 or hg19 equivalents)
- Annotation class selection flags (-ac 1,2,3,4,5,6)

## Outputs

- Tab-delimited per-RNA-type read count text files (miRNA, piRNA, tRNA, snoRNA, snRNA, circRNA) per sample
- Read counts per annotated RNA molecule, including the database items supporting each annotation

## How to apply

Run the COMPSRA Annotation module on the STAR alignment output BAM (e.g. sample01_17to50_FitRead_STAR_Aligned.out.bam) using java -jar COMPSRA.jar -ann -ac 1,2,3,4,5,6, which annotates against pre-built databases for miRNA (miRBase), piRNA (piRNABank, piRBase, piRNACluster), tRNA (gtRNAdb), snRNA/snoRNA (GENCODE release 27) and circRNA (circBase). Apply the read-support overlap filter with -aol 1.0 so each read must lie completely within the annotated gene region; the databases are expandable and can be augmented with additional references. Genome-version conflicts between databases (hg18/hg19/hg38) are resolved internally with an automatic UCSC LiftOver. Counting is multi-assign: a read mapping to multiple loci of one RNA is counted once, while a read shared by distinct RNAs (e.g. let-7a-5p vs piR_008113) contributes one count to each overlapping molecule. Each RNA type's counts are written as a separate tab-delimited text file, and prebuilt databases can be fetched via the ToolKit (-tk -dr -ck miRNA_hg38, etc.).

## Related tools

- **COMPSRA** (Hosts the Annotation module (-ann) that performs the multi-database small RNA annotation with LiftOver-based genome harmonization) — https://github.com/cougarlj/COMPSRA
- **STAR** (Default aligner whose mapped-read BAM output is the input to the Annotation module)
- **miRBase** (Reference database used first for miRNA annotation)
- **gtRNAdb** (Reference database for tRNA annotation)
- **GENCODE release 27** (Reference database for snRNA and snoRNA annotation)
- **circBase** (Reference database for circRNA annotation)

## Examples

```
java -jar COMPSRA.jar -ref hg38 -ann -ac 1,2,3,4,5,6 -in ./example_out/sample01/sample01_17to50_FitRead_STAR_Aligned.out.bam -out ./example_out/sample01/sample01_17to50_FitRead_STAR_Aligned
```

## Evaluation signals

- Tab-delimited annotation output exists for all six RNA classes (miRNA, piRNA, tRNA, snoRNA, snRNA, circRNA) with database items supporting each annotation
- Reads mapping to multiple genomic loci of one RNA are counted once, not per-locus
- Ambiguous reads shared between distinct RNAs (e.g. let-7a-5p vs piR_008113) yield one count in each overlapping RNA's file
- All databases are harmonized to the chosen reference genome version via automatic LiftOver before counting
- On the 12 serum samples, plausible outputs include hundreds of miRNAs/piRNAs/tRNAs and thousands of circRNAs at a read count threshold of 5 (e.g. 375 miRNAs, 7285 circRNAs)

## Limitations

- Annotation is restricted to human genome-mapped reads; unmapped reads go to the Microbe module instead
- The default overlap ratio Rolp = 1 (-aol 1.0) requires each read to lie completely within the annotated gene region, which excludes partially overlapping reads unless the parameter is relaxed
- Reference databases are pre-built and version-locked (miRBase20, GENCODE release 27, etc.); updating them depends on the databases being expandable and on LiftOver handling genome-version differences

## Evidence

- [methods] COMPSRA currently uses several different (and expandable) small RNA reference databases for annotating human genome mapped reads: miRBase20 for miRNA: "COMPSRA currently uses several different (and expandable) small RNA reference databases for annotating human genome mapped reads: miRBase20 for miRNA"
- [readme] To conform the different reference human genome versions in these databases, we use an automatic LiftOver created by the UCSC Genome Browser Group.: "To conform the different reference human genome versions in these databases, we use an automatic LiftOver created by the UCSC Genome Browser Group."
- [readme] The read count of each RNA molecule that is identified in the Annotation Module is outputted as a tab-delimited text file according to RNA type.: "The read count of each RNA molecule that is identified in the Annotation Module is outputted as a tab-delimited text file according to RNA type."
- [results] COMPSRA detected 375 miRNAs, 280 piRNAs, 167 snRNAs, 88 snoRNAs, 401 tRNAs and 7285 circRNAs: "COMPSRA detected 375 miRNAs, 280 piRNAs, 167 snRNAs, 88 snoRNAs, 401 tRNAs and 7285 circRNAs"
