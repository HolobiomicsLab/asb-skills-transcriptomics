---
name: dual-genome-transcriptome-riboseq-alignment
description: Use when processing Ribo-seq FastQ data that has already passed QC and
  contaminant removal and must be mapped to a reference genome and transcriptome before
  CDS-level read counting and differential expression.
when_to_use_negative:
- Inputs are already alignment files (BAM) or a count matrix — skip re-alignment and
  start from counting or DESeq2
- The study requires ribosome P-site offset estimation on a dataset that does not
  align enough material — use TRiP instead of riboWaltz, or a dedicated tool outside
  RiboDoc
- Analysis of non-RiboSeq RNA-seq data without ribosome footprints, where RPF-specific
  length selection (25-35 nt) and periodicity QC do not apply
edam_operation: http://edamontology.org/operation_3198
edam_topics:
- http://edamontology.org/topic_0085
- http://edamontology.org/topic_0622
tools:
- name: RiboDoc
  role: Docker/Singularity-packaged Snakemake pipeline orchestrating trimming, contaminant
    removal, dual alignment, counting and DESeq2 analysis
  repo: https://github.com/equipeGST/RiboDoc
- name: HISAT2
  role: Genome alignment of contaminant-depleted ribosome-protected reads
- name: Bowtie 2
  role: Transcriptome alignment of reads conjointly with HISAT2, and removal of rRNA/tRNA/viral
    RNA contaminants
- name: samtools
  role: Sorting and indexing the BAM alignments produced by the dual alignment
- name: HTSeq
  role: htseq-count counts reads over CDS features using the 25-35 nt RPF selection
- name: Cutadapt
  role: Adapter and quality trimming plus read length filtering prior to alignment
- name: riboWaltz
  role: Optional P-site offset prediction and periodicity graph creation after alignment
    (replaced by TRiP when little material aligns)
  repo: https://github.com/LabTranslationalArchitectomics/riboWaltz
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1016/j.csbj.2021.05.014
    title: 'RiboDoc: A Docker-based package for ribosome profiling analysis'
  source_content_hash: sha256:5e1a301c7cae7a7a60ba7e589fbbcf23f23301616d2ab402e38ef5ef9f581520
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
  - riboWaltz
  - samtools
  edam_operation: http://edamontology.org/operation_3198
  edam_topics:
  - http://edamontology.org/topic_0085
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.6
n_claims: 15
---

# Dual genome-transcriptome RiboSeq alignment

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Jointly align ribosome-protected fragments (RPF) to both the genome (HISAT2) and the transcriptome (Bowtie2) within the RiboDoc pipeline, after adapter trimming and rRNA/tRNA contaminant depletion, to produce sorted, indexed BAMs that feed htseq-count CDS quantification and DESeq2 differential translation analysis.

## When to use

Use when processing Ribo-seq FastQ data that has already passed QC and contaminant removal and must be mapped to a reference genome and transcriptome before CDS-level read counting and differential expression.

## When NOT to use

- Inputs are already alignment files (BAM) or a count matrix — skip re-alignment and start from counting or DESeq2
- The study requires ribosome P-site offset estimation on a dataset that does not align enough material — use TRiP instead of riboWaltz, or a dedicated tool outside RiboDoc
- Analysis of non-RiboSeq RNA-seq data without ribosome footprints, where RPF-specific length selection (25-35 nt) and periodicity QC do not apply

## Inputs

- gzip-compressed FastQ files named [CONDITION]_[NAME].[REPLICATE].fastq.gz
- Genome FASTA (e.g. Saccharomyces_cerevisiae.R64-1-1.dna.toplevel.fa.gz from Ensembl)
- GFF3 gene annotation from Ensembl (or NCBI)
- rRNA/tRNA contaminant FASTA generated from Silva
- config.yaml with readsLength_min/readsLength_max, reference and comparison conditions, p-val and logFC thresholds
- Docker or Singularity installed on the host

## Outputs

- Sorted and indexed BAM alignments (genome and transcriptome)
- count_matrix_by_transcript_IDs.csv under RESULTS/DESeq2_CDS.25-35
- Analysis report (e.g. Yeast_RiboSeq.Analysis_Report.25-35.txt) documenting read numbers at each pipeline step
- Differential analysis HTML report (e.g. Yeast_RiboSeq.Final_report.html)

## How to apply

Organize FastQ files (gzip-compressed, named [CONDITION]_[NAME].[REPLICATE].fastq.gz) in a fastq/ subfolder with genome FASTA, GFF3 annotation and an rRNA contaminant FASTA (e.g. Silva-derived, S. cerevisiae R64-1-1 from Ensembl) in a database/ subfolder plus a config.yaml. Launch the RiboDoc Docker/Singularity container so FastQC checks raw reads, Cutadapt trims adapters and applies read-length filtering (default 25-35 nt, since RPF are usually ~30 nt), and Bowtie2 removes rRNA/tRNA/viral contaminant reads before alignment. Then HISAT2 aligns depleted reads to the genome while Bowtie2 aligns to the transcriptome conjointly; multimapping reads are removed so only uniquely mapped reads are kept. Sort and index the resulting BAM alignments with samtools, then count reads over CDS features with htseq-count using the configured 25-35 nt RPF selection. Choose TRiP rather than riboWaltz for the qualitative/offset step when the dataset aligns little material, as was the case for GEO GSE173856.

## Related tools

- **RiboDoc** (Docker/Singularity-packaged Snakemake pipeline orchestrating trimming, contaminant removal, dual alignment, counting and DESeq2 analysis) — https://github.com/equipeGST/RiboDoc
- **HISAT2** (Genome alignment of contaminant-depleted ribosome-protected reads)
- **Bowtie 2** (Transcriptome alignment of reads conjointly with HISAT2, and removal of rRNA/tRNA/viral RNA contaminants)
- **samtools** (Sorting and indexing the BAM alignments produced by the dual alignment)
- **HTSeq** (htseq-count counts reads over CDS features using the 25-35 nt RPF selection)
- **Cutadapt** (Adapter and quality trimming plus read length filtering prior to alignment)
- **riboWaltz** (Optional P-site offset prediction and periodicity graph creation after alignment (replaced by TRiP when little material aligns)) — https://github.com/LabTranslationalArchitectomics/riboWaltz

## Examples

```
docker run --rm -v /path/to/project/:/data/ equipegst/ribodoc CPU_NUMBER MEMORY_AMOUNT
```

## Evaluation signals

- RESULTS/DESeq2_CDS.25-35 contains count_matrix_by_transcript_IDs.csv and the final HTML differential analysis report
- Yeast_RiboSeq.Analysis_Report.25-35.txt documents read numbers at each step, with CDS enrichment and metagene-periodicity QC passed
- Read-length distribution after filtering shows a dominant RPF peak (~28 nt in the example dataset)
- Multimapping reads are removed and only uniquely mapped reads are retained, as shown in the Raw reads / Filtered reads / Multimapping / Unique map reporting
- Metagene analysis shows 3-nt periodicity starting ~12 nt before the start codon and ending ~15 nt before the stop codon, plus overrepresentation of footprints within CDS

## Limitations

- Datasets that align little material cannot use the riboWaltz qualitative step; TRiP must be selected instead (as for GEO GSE173856)
- RiboDoc performs only the first steps of RiboSeq analysis; downstream exploitation requires more specific tools
- The project folder architecture is mandatory: subfolder names fastq/ and database/ must be lowercase, and filenames must avoid dashes, parentheses, spaces and special characters except underscores

## Evidence
