---
name: cohort-gene-exclusion-filtering
description: Use when a genome-wide shape-change analysis over a base-level pileup
  matrix must first exclude genes with many non-expressed samples in the cohort, so
  that downstream on/off artifacts do not masquerade as shape changes.
when_to_use_negative:
- 'The input is a single-sample pileup with no cohort: the off classification is defined
  relative to a gene''s mean coverage vector across all samples, so a cohort (paper
  suggests >= 30 samples) is required.'
- The goal is filtering lowly expressed genes by expression magnitude — that step
  uses percentile cutoffs on median and MAD of normalized RSEM values, not angle-based
  on/off shape statistics.
- 'Severely degraded samples have not yet been removed: on/off fractions computed
  over the 70 degraded TCGA HNSC cases would be inflated and the 5802-gene result
  would not reproduce.'
edam_operation: http://edamontology.org/operation_3695
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: SCISSOR
  role: Provides build_pileup/get_Ranges for the input matrix, angle.hy for the sample-vs-gene-mean
    angle statistic, and get_exon_offstat/get_offstat for per-gene off-sample fractions
  repo: https://github.com/hyochoi/SCISSOR
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1038/s41467-020-20593-3
    title: 'SCISSOR: a framework for identifying structural changes in RNA transcripts'
  source_content_hash: sha256:c0500aad27c9fa832f9c315e5651cfeee974444b94dd6f9c5b9b3332eb89077c
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/hyochoi/SCISSOR
  tools:
  - SCISSOR
  edam_operation: http://edamontology.org/operation_3695
  edam_topics:
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7777777777777778
n_claims: 9
---

# cohort-gene-exclusion-filtering

## Summary

SCISSOR's on/off gene filter classifies each sample-gene pair as 'off' using angle-based shape similarity (angle.hy) between the sample's base-level coverage vector and the gene's mean coverage vector, then excludes genes whose fraction of off samples exceeds 20% — removing 5802 genes (~28%) in the 522-sample TCGA HNSC cohort before shape-change analysis.

## When to use

Use when a genome-wide shape-change analysis over a base-level pileup matrix must first exclude genes with many non-expressed samples in the cohort, so that downstream on/off artifacts do not masquerade as shape changes.

## When NOT to use

- The input is a single-sample pileup with no cohort: the off classification is defined relative to a gene's mean coverage vector across all samples, so a cohort (paper suggests >= 30 samples) is required.
- The goal is filtering lowly expressed genes by expression magnitude — that step uses percentile cutoffs on median and MAD of normalized RSEM values, not angle-based on/off shape statistics.
- Severely degraded samples have not yet been removed: on/off fractions computed over the 70 degraded TCGA HNSC cases would be inflated and the 5802-gene result would not reproduce.

## Inputs

- Genome-wide base-level pileup coverage matrix (samples x bases) from build_pileup on TCGA HNSC BAMs, after excluding 70 severely degraded samples
- Genomic ranges per gene from get_Ranges (built via build_gaf against a GTF such as Homo_sapiens.GRCh37.87.gtf)

## Outputs

- Per-gene table of off-sample fractions (one fraction per gene)
- List of excluded genes with off-sample fraction > 20% (expected: 5802 genes, ~28%), written to onoff_gene_fractions.csv
- Filtered pileup matrix retaining genes with <= 20% off samples for downstream GSC/LSC analysis

## How to apply

Start from the base-level pileup matrix built with build_pileup/get_Ranges for the cohort (here 522 TCGA HNSC cases after removing the 70 severely degraded samples). For each gene, compute the mean coverage vector across all samples as the gene-shape reference, then use SCISSOR's angle computation (angle.hy) to measure the high-dimensional angle between each sample vector and this mean vector. Apply the on/off classification per sample-gene pair from the angle statistic and aggregate with get_exon_offstat/get_offstat to obtain the fraction of off samples per gene. Flag genes whose off-sample fraction exceeds 20% (the paper's cutoff) and write the off fractions plus the excluded-gene list to onoff_gene_fractions.csv. Rationale: a per-gene 'not expressed' vs 'expressed' definition is needed because genes lacking expression in many samples produce degenerate coverage shapes that would corrupt unsupervised shape-change detection.

## Related tools

- **SCISSOR** (Provides build_pileup/get_Ranges for the input matrix, angle.hy for the sample-vs-gene-mean angle statistic, and get_exon_offstat/get_offstat for per-gene off-sample fractions) — https://github.com/hyochoi/SCISSOR

## Examples

```
library(SCISSOR); offstat <- get_offstat(pileup_matrix, Ranges); excluded_genes <- names(which(offstat$off_fraction > 0.20)); write.csv(offstat, "onoff_gene_fractions.csv")
```

## Evaluation signals

- The excluded-gene count reproduces the published 5802 genes (~28% of genes analyzed) on the 522-sample TCGA HNSC cohort after removal of 70 degraded cases.
- Every entry in the per-gene off-fraction table lies in [0, 1], and flagged excluded genes all have off fractions strictly greater than 0.20.
- onoff_gene_fractions.csv contains both the per-gene off fractions and the excluded-gene list, as specified in the workflow.
- Recomputing the mean coverage vector per gene yields a vector of the same base-level dimension as the gene's genomic range from get_Ranges.

## Limitations

- The 20% off-sample cutoff is the threshold reported for the TCGA HNSC cohort; other cohorts may need a different fraction.
- SCISSOR's tuning parameters, including log normalization and the normality cutoff value, likely require optimization when a significantly different wet-lab RNA-seq protocol is used, which changes angle-based on/off classifications.
- The filter only excludes genes with many non-expressed samples; it does not address per-sample degradation, which must be handled first (the paper empirically removed 70 severely degraded cases).

## Evidence

- [results] Per gene definition of not expressed vs expressed excluding genes with many non-expressed samples: "A per gene definition of “not expressed” versus “expressed” was developed for the purposes of excluding genes with many non-expressed samples"
- [results] 5802 genes (~28%) excluded from downstream analysis: "we identified 5802 genes (~28%) and they were excluded from the downstream analysis"
- [results] Severely degraded samples removed before gene filtering: "identified 70 cases that were severely degraded in a large fraction of genes and excluded from the downstream analysis"
- [methods] SCISSOR package provides pileup matrix and genomic ranges inputs: "We first obtain pileup data matrix using `build_pileup` and genomic ranges using `get_Ranges`"
- [readme] SCISSOR targets unsupervised screening of structural RNA alterations on base-level pileups: "`SCISSOR` (shape changes in selecting sample outliers in RNA-seq) aims for unsupervised screening of a range of structural alterations in RNA-seq data."
