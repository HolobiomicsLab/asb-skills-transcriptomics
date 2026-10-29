---
name: strand-specific-feature-counting
description: Use when generating a per-cell count matrix over groHMM-defined TARs
  from strand-specific paired-end scRNA-seq alignments (e.g., Smart-seq2), where read
  orientation must be preserved for correct quantification.
when_to_use_negative:
- Input is already a gene- or feature-level count table — counting is unnecessary;
  proceed directly to Seurat analysis
- Alignments are single-end or non-strand-specific — the -s 1 strandedness setting
  would misassign fragments
- The goal is annotation-dependent gene-level quantification from Cell Ranger output
  rather than TAR discovery beyond known annotations
edam_topics:
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_0203
tools:
- name: featureCounts
  role: Counts TAR features per cell with strandedness=1 (-s 1) and paired-end option
    (-p) to build the TAR expression matrix
  repo: github.com/fw262/TAR-scRNA-seq
- name: bedtools
  role: Extracts read fragments from paired-end BAMs via bamtobed with '-bedpe' before
    groHMM TAR calling
  repo: github.com/fw262/TAR-scRNA-seq
- name: groHMM
  role: Runs a hidden Markov model on read fragments to call strand-specific transcriptionally
    active regions (TARs) labeled aTAR or uTAR
  repo: github.com/fw262/TAR-scRNA-seq
- name: Seurat
  role: Consumes the TAR count matrix and applies the Wilcoxon rank-sum test to find
    differentially expressed uTARs per cell cluster
  repo: github.com/fw262/TAR-scRNA-seq
- name: Snakemake
  role: Orchestrates the end-to-end workflow from alignments to counted TAR expression
    matrix
  repo: github.com/fw262/TAR-scRNA-seq
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1038/s41467-021-22496-3
    title: Uncovering transcriptional dark matter via gene annotation independent
      single-cell RNA sequencing analysis
  source_content_hash: sha256:cdc346574c92dfada5252fd4b243041a050b008ed167b9df132a42e46fff0450
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - STAR
  - Seurat
  - Snakemake
  - bedtools
  - featureCounts
  - groHMM
  - rtracklayer
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3308
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8571428571428571
n_claims: 14
---

# Strand-specific feature counting

## Summary

Counts sequenced fragments per feature (groHMM-defined TARs) while preserving read strandedness, converting strand-specific single-cell alignments into a TAR expression matrix for downstream Seurat analysis.

## When to use

Use when generating a per-cell count matrix over groHMM-defined TARs from strand-specific paired-end scRNA-seq alignments (e.g., Smart-seq2), where read orientation must be preserved for correct quantification.

## When NOT to use

- Input is already a gene- or feature-level count table — counting is unnecessary; proceed directly to Seurat analysis
- Alignments are single-end or non-strand-specific — the -s 1 strandedness setting would misassign fragments
- The goal is annotation-dependent gene-level quantification from Cell Ranger output rather than TAR discovery beyond known annotations

## Inputs

- STAR-aligned paired-end BAM files from strand-specific Smart-seq2 scRNA-seq libraries
- groHMM-generated TAR annotations in refFlat and GTF formats (aTAR/uTAR-labeled)
- BEDPE read-fragment files from bedtools bamtobed -bedpe

## Outputs

- Per-cell TAR feature count vectors from featureCounts
- Aggregated TAR expression matrix (cells x TARs)
- Differential uTAR table (e.g., ss2_differential_utars.csv) after Seurat Wilcoxon testing

## How to apply

First convert strand-specific paired-end alignments (STAR-produced BAMs, e.g., Smart-seq2 gray mouse lemur testes cells) to read fragments with bedtools bamtobed using the '-bedpe' option, then run groHMM in a strand-specific manner to define TARs, exporting annotation files in refFlat and gtf formats labeled aTAR or uTAR by overlap with existing gene annotations. Count TAR features per cell with featureCounts setting strandedness to 1 (-s 1) and supplying the paired-end option (-p), then aggregate the per-cell count vectors into a TAR expression matrix. Strandedness=1 is essential because groHMM and the TAR assignments are strand-specific; ignoring orientation would miscount reads relative to TARs. The resulting matrix feeds Seurat differential expression (Wilcoxon rank-sum), filtering uTARs expressed in at least 25% of cluster cells with >0.25 log-e fold change versus other cells.

## Related tools

- **featureCounts** (Counts TAR features per cell with strandedness=1 (-s 1) and paired-end option (-p) to build the TAR expression matrix) — github.com/fw262/TAR-scRNA-seq
- **bedtools** (Extracts read fragments from paired-end BAMs via bamtobed with '-bedpe' before groHMM TAR calling) — github.com/fw262/TAR-scRNA-seq
- **groHMM** (Runs a hidden Markov model on read fragments to call strand-specific transcriptionally active regions (TARs) labeled aTAR or uTAR) — github.com/fw262/TAR-scRNA-seq
- **Seurat** (Consumes the TAR count matrix and applies the Wilcoxon rank-sum test to find differentially expressed uTARs per cell cluster) — github.com/fw262/TAR-scRNA-seq
- **Snakemake** (Orchestrates the end-to-end workflow from alignments to counted TAR expression matrix) — github.com/fw262/TAR-scRNA-seq

## Examples

```
featureCounts -s 1 -p -a tars.gtf -o tar_counts.txt cell1.bam cell2.bam
```

## Evaluation signals

- featureCounts is invoked with strandedness 1 (-s 1) and the paired-end flag (-p); missing either flag means strand or pairing was not honored
- Per-cell count vectors aggregate into a TAR expression matrix whose columns/rows cover all groHMM TARs from the refFlat/GTF annotation files
- Differential uTAR table matches reported assignments: CCER1 in spermatocytes, TBXAS1 in spermatogonium, ncRNA LOC105867359 in spermatids (Supplementary Fig. 2b)
- Differential uTARs pass filters of expression in at least 25% of cluster cells and >0.25 log-e fold change versus the rest of the cells
- A 'SingleCellHMM_Run_combined_bam_HMM_features.log' with no 'chr*_HMM.bed: No such file or directory' errors confirms groHMM bed files were produced before counting

## Limitations

- groHMM is not installable through standard channels — it must be manually installed from Bioconductor along with rtracklayer, or the HMM bed files will not be generated and the count step cannot proceed
- This counting scheme targets gene-annotation-independent TAR quantification; it assumes strand-specific paired-end libraries and does not handle non-stranded or single-end data
- The degree of biologically relevant uTAR information is strongly dependent on organism, tissue, cell type, and stage of development, so downstream value varies by dataset

## Evidence

- [methods] Stranded feature counting with featureCounts on groHMM TARs: [evidence span withheld — anchor status: not_found]
- [methods] Upstream fragment extraction and TAR calling: [evidence span withheld — anchor status: not_found]
- [intro] Rationale for groHMM-based TAR definition: "groHMM uses a hidden Markov model to divide the genome into two states (transcribed and untranscribed) based on read coverage in each bin"
- [readme] Tool installation prerequisite before counting: "This error indicates that the groHMM R script did not finish running to generate groHMM bed files."
- [methods] Downstream differential analysis filters: [evidence span withheld — anchor status: not_found]
