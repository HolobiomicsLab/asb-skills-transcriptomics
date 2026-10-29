---
name: per-gene-off-fraction-quantification
description: Use when preparing base-level RNA-seq pileup data for cohort-wide shape-change
  analysis and you must define which genes are often 'not expressed' so they can be
  excluded from downstream SCISSOR analysis.
when_to_use_negative:
- Input is gene-level expression (e.g. RSEM counts) rather than base-level pileup
  coverage — the on/off statistic requires per-base sample vectors per gene
- The cohort is small or a single sample — the fraction-of-off-samples statistic needs
  a multi-sample cohort (SCISSOR is intended for unsupervised cohort screening, ~30
  samples minimum)
- You only need to exclude lowly expressed genes by abundance — a simpler percentile
  cutoff on median/MAD of normalized RSEM values suffices; this skill is about shape-based
  non-expression across samples
edam_operation: http://edamontology.org/operation_3695
edam_topics:
- http://edamontology.org/topic_0203
tools:
- name: SCISSOR
  role: Provides build_pileup/get_Ranges for the pileup matrix, angle.hy for sample-vs-gene-mean
    angle computation, and get_exon_offstat/get_offstat for per-gene off-fraction
    aggregation
  repo: https://github.com/hyochoi/SCISSOR
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1038/s41467-020-20593-3
    title: 'SCISSOR: a framework for identifying structural changes in RNA transcripts'
  source_content_hash: sha256:51d2b0a0fc4d117d79a312fba13a3ae93edbd0f52ff2ecce147dadc44e2c9f8c
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/hyochoi/SCISSOR
  tools:
  - SCISSOR
  edam_operation: http://edamontology.org/operation_3695
  edam_topics:
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8888888888888888
n_claims: 9
---

# per-gene off-fraction quantification

## Summary

Quantifies, per gene, the fraction of RNA-seq samples classified as 'off' (non-expressed) using SCISSOR's angle-based shape similarity of each sample coverage vector to the gene mean vector, so that genes with many off samples can be excluded before genome-wide shape-change analysis. In the 522-sample TCGA HNSC cohort this flags 5802 genes (~28%) with >20% off samples.

## When to use

Use when preparing base-level RNA-seq pileup data for cohort-wide shape-change analysis and you must define which genes are often 'not expressed' so they can be excluded from downstream SCISSOR analysis.

## When NOT to use

- Input is gene-level expression (e.g. RSEM counts) rather than base-level pileup coverage — the on/off statistic requires per-base sample vectors per gene
- The cohort is small or a single sample — the fraction-of-off-samples statistic needs a multi-sample cohort (SCISSOR is intended for unsupervised cohort screening, ~30 samples minimum)
- You only need to exclude lowly expressed genes by abundance — a simpler percentile cutoff on median/MAD of normalized RSEM values suffices; this skill is about shape-based non-expression across samples

## Inputs

- Genome-wide base-level RNA-seq pileup matrix (from SCISSOR build_pileup) for the cohort, after removing degraded samples
- Genomic ranges of genes/exons (SCISSOR get_Ranges, e.g. from Homo_sapiens.GRCh37.87.gtf via build_gaf)
- Cohort of RNA-seq samples with per-base coverage (e.g. 452 retained TCGA HNSC tumors after excluding 70 degraded cases)

## Outputs

- Per-gene table of off-sample fractions (offstat) for every gene in the cohort
- List of excluded genes (off-fraction > 20%), e.g. 5802 genes (~28%) in TCGA HNSC, written to onoff_gene_fractions.csv

## How to apply

Load the genome-wide base-level pileup matrix (built with SCISSOR's build_pileup over genomic ranges from get_Ranges) for the cohort, e.g. 522 TCGA HNSC tumors after removing 70 severely degraded cases. For each gene, compute the mean coverage vector across all samples as the gene-shape reference, then use SCISSOR's angle computation (angle.hy) to measure the high-dimensional angle between each individual sample vector and that mean vector as the shape-similarity statistic. Apply SCISSOR's on/off classification per sample-gene pair from the angle statistic and aggregate with get_exon_offstat/get_offstat to obtain the fraction of off samples per gene. Flag genes whose off-sample fraction exceeds 20% as excluded from downstream analysis. The rationale is that genes with many non-expressed samples produce spurious shape-change signals, so a per-gene definition of 'not expressed' versus 'expressed' is needed before the global/local shape change detection steps.

## Related tools

- **SCISSOR** (Provides build_pileup/get_Ranges for the pileup matrix, angle.hy for sample-vs-gene-mean angle computation, and get_exon_offstat/get_offstat for per-gene off-fraction aggregation) — https://github.com/hyochoi/SCISSOR

## Examples

```
install_github("hyochoi/SCISSOR"); library(SCISSOR); ranges <- get_Ranges(Gene=Gene, GTF.file="Homo_sapiens.GRCh37.87.gtf")
```

## Evaluation signals

- The number of flagged genes with off-fraction > 20% reproduces the reported 5802 genes (~28%) for the 522-sample TCGA HNSC cohort (452 after removing the 70 degraded cases)
- Every gene in the output table has an off-fraction in [0, 1]; excluded genes all exceed the 0.20 threshold
- The output file onoff_gene_fractions.csv contains both off fractions and the excluded-gene list with one row per analyzed gene
- Genes excluded here are no longer present in the downstream global/local shape change (GSC/LSC) analysis input

## Limitations

- SCISSOR tuning parameters, including log normalization and a normality cutoff value, will likely require optimization when a significantly different wet lab RNA-seq protocol is used
- The on/off definition depends on angle-based shape similarity, so degraded or low-quality samples can distort per-gene fractions; severely degraded samples must be removed first
- The downstream GSC detection algorithm loses information about the order of base pairs, limiting interpretation of what 'off' means structurally

## Evidence

- [results] Per gene definition of 'not expressed' versus 'expressed': "A per gene definition of “not expressed” versus “expressed” was developed for the purposes of excluding genes with many non-expressed samples"
- [results] 5802 genes excluded: "we identified 5802 genes (~28%) and they were excluded from the downstream analysis"
- [results] 70 degraded samples removed first: "identified 70 cases that were severely degraded in a large fraction of genes and excluded from the downstream analysis"
- [methods] pileup and ranges inputs: "We first obtain pileup data matrix using `build_pileup` and genomic ranges using `get_Ranges`"
- [readme] package installation: "install_github("hyochoi/SCISSOR")
library(SCISSOR)"
