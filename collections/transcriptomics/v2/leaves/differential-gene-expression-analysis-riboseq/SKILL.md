---
name: differential-gene-expression-analysis-riboseq
description: Use when ribosome profiling FastQ data from at least two conditions (reference
  vs comparison) must be processed end-to-end to identify genes with significant differential
  translational regulation.
when_to_use_negative:
- Input is already a gene count matrix or DESeq2 result table — the pipeline would
  redundantly repeat FastQC/Cutadapt/alignment/counting steps
- Data are not ribosome profiling (RiboSeq) footprints but bulk RNA-seq or proteomics,
  where RiboDoc's 25–35 nt RPF selection and CDS-enrichment QC do not apply
- Only one experimental condition is available, since DESeq2 differential analysis
  requires a reference and a comparison condition
edam_operation: http://edamontology.org/operation_3223
edam_topics:
- http://edamontology.org/topic_0085
- http://edamontology.org/topic_0203
tools:
- name: RiboDoc
  role: Docker/Snakemake pipeline orchestrating QC, trimming, alignment, counting,
    and DESeq2 differential expression for RiboSeq
  repo: https://github.com/equipeGST/RiboDoc
- name: FastQC
  role: Quality Control of raw, trimmed, and contaminant-depleted reads
- name: Cutadapt
  role: Adapter and quality trimming, read length filtering
- name: Bowtie 2
  role: Removal of contaminant RNA (rRNA, tRNA, viral RNA) and joint transcriptome
    alignment
- name: HISAT2
  role: Genome alignment of reads conjointly with Bowtie2
- name: samtools
  role: Sort and index BAM alignments
- name: HTSeq
  role: Reads Count over CDS features via htseq-count with 25–35 nt selection
- name: DESeq2
  role: Normalization and differential gene expression analysis between conditions
- name: riboWaltz
  role: Offset prediction and periodicity graph creation (alternative qualitative
    step)
  repo: https://github.com/LabTranslationalArchitectomics/riboWaltz
- name: TRiP
  role: Alternative qualitative step (offset/periodicity) when datasets align too
    little material for riboWaltz
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1016/j.csbj.2021.05.014
    title: 'RiboDoc: A Docker-based package for ribosome profiling analysis'
  source_content_hash: sha256:8b96620af9366a08116a1ea66e894263f26d8959d9db7a0f2fdcb0ec32650716
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-ND-4.0
    url: https://github.com/equipeGST/RiboDoc
  repo_url: https://github.com/equipeGST/RiboDoc
  tools:
  - Bowtie 2
  - Cutadapt
  - DESeq2
  - FastQC
  - HISAT2
  - HTSeq
  - RiboDoc
  - TRiP
  - riboWaltz
  - samtools
  edam_operation: http://edamontology.org/operation_3223
  edam_topics:
  - http://edamontology.org/topic_0085
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.46153846153846156
n_claims: 13
---

# differential-gene-expression-analysis-riboseq

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Perform RiboSeq differential translation analysis by counting ribosome-protected fragments over CDS features (25–35 nt selection) and applying DESeq2 normalization and statistical testing with user-configured p-value and log2FoldChange thresholds, as standardized by the RiboDoc Docker/Snakemake pipeline.

## When to use

Use when ribosome profiling FastQ data from at least two conditions (reference vs comparison) must be processed end-to-end to identify genes with significant differential translational regulation.

## When NOT to use

- Input is already a gene count matrix or DESeq2 result table — the pipeline would redundantly repeat FastQC/Cutadapt/alignment/counting steps
- Data are not ribosome profiling (RiboSeq) footprints but bulk RNA-seq or proteomics, where RiboDoc's 25–35 nt RPF selection and CDS-enrichment QC do not apply
- Only one experimental condition is available, since DESeq2 differential analysis requires a reference and a comparison condition

## Inputs

- gzipped FastQ files of ribosome profiling reads ([CONDITION]_[NAME].[REPLICATE].fastq.gz)
- Reference genome FASTA (e.g. Saccharomyces_cerevisiae.R64-1-1.dna.toplevel.fa.gz from Ensembl)
- GFF3 gene annotation from Ensembl
- rRNA/tRNA contaminant FASTA generated from Silva
- config.yaml with readsLength_min, readsLength_max, reference/comparison conditions, p-val and logFC thresholds

## Outputs

- RESULTS/DESeq2_CDS.25-35/count_matrix_by_transcript_IDs.csv (read count matrix)
- Differential analysis HTML report (e.g. Yeast_RiboSeq.Final_report.html) with DESeq2 results filtered by p-val and logFC thresholds
- Yeast_RiboSeq.Analysis_Report.25-35.txt documenting read numbers at each pipeline step
- FastQC QC reports for raw, trimmed, and contaminant-depleted reads
- Sorted and indexed BAM alignments
- Periodicity graphs and P-site offsets from riboWaltz or TRiP

## How to apply

Organize the project as RiboDoc requires: fastq/ subfolder with gzipped FastQ files named [CONDITION]_[NAME].[REPLICATE].fastq.gz, a database/ subfolder holding the genome FASTA, GFF3 annotation and rRNA contaminant FASTA, plus a config.yaml setting readsLength_min=25 and readsLength_max=35, reference/comparison conditions, p-val and logFC thresholds. Launch the equipegst/ribodoc container (Docker or Singularity) so FastQC performs QC, Cutadapt trims adapters and filters read length, Bowtie2 removes rRNA/tRNA contaminants, HISAT2 and Bowtie2 align reads to genome and transcriptome, samtools sorts/indexes BAMs, and htseq-count counts reads over CDS features for the 25–35 nt RPF selection. RiboDoc then runs DESeq2 to normalize counts and compare conditions against the reference, applying the configured p-value and log2FoldChange cutoffs. RiboDoc also provides CDS enrichment and metagene-periodicity QC plus PCA and Spearman's correlation to validate that data correspond to active ribosome footprints and that replicates are suitable. Because GSE173856 aligns little material, choose TRiP rather than riboWaltz for the qualitative (P-site/periodicity) step.

## Related tools

- **RiboDoc** (Docker/Snakemake pipeline orchestrating QC, trimming, alignment, counting, and DESeq2 differential expression for RiboSeq) — https://github.com/equipeGST/RiboDoc
- **FastQC** (Quality Control of raw, trimmed, and contaminant-depleted reads)
- **Cutadapt** (Adapter and quality trimming, read length filtering)
- **Bowtie 2** (Removal of contaminant RNA (rRNA, tRNA, viral RNA) and joint transcriptome alignment)
- **HISAT2** (Genome alignment of reads conjointly with Bowtie2)
- **samtools** (Sort and index BAM alignments)
- **HTSeq** (Reads Count over CDS features via htseq-count with 25–35 nt selection)
- **DESeq2** (Normalization and differential gene expression analysis between conditions)
- **riboWaltz** (Offset prediction and periodicity graph creation (alternative qualitative step)) — https://github.com/LabTranslationalArchitectomics/riboWaltz
- **TRiP** (Alternative qualitative step (offset/periodicity) when datasets align too little material for riboWaltz)

## Examples

```
docker run --rm -v /path/to/project/:/data/ equipegst/ribodoc CPU_NUMBER MEMORY_AMOUNT
```

## Evaluation signals

- RESULTS/DESeq2_CDS.25-35 exists and contains count_matrix_by_transcript_IDs.csv and the differential analysis HTML report
- Yeast_RiboSeq.Analysis_Report.25-35.txt documents read numbers at each step; CDS enrichment and metagene-periodicity QC passed
- Read-length distribution shows a majority of ~28 nt reads, and 28 nt RPF show 3-nt periodicity starting ~12 nt before the start codon and ending ~15 nt before the stop codon
- PCA and pairwise sample-by-sample Spearman correlation confirm replicate reproducibility; low-quality samples removed and RiboDoc rerun for remaining steps
- Differential hits respect the configured p-val and log2FoldChange thresholds (e.g. in the OAZ1 example, PGM2/YMR105c was the only gene strongly downregulated in OAZ-if)

## Limitations

- RiboSeq bioinformatics suffers from a lack of standardization; RiboDoc addresses this but no version number is stated for RiboDoc or the Docker package
- Datasets that do not align enough material cannot use the riboWaltz qualitative step (TRiP must be selected instead)
- File naming is strict: no dashes, parentheses, blank spaces, or special characters except underscores, and subfolder names must not contain uppercase letters
- RiboDoc performs the first/initial steps of RiboSeq analysis; downstream more specific tools are needed for further exploitation

## Evidence
