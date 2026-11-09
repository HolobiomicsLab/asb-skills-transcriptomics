---
name: spearman-rank-correlation-analysis
description: Use when pseudobulk 3'UTR isoform counts from scRNA-seq samples (e.g.,
  FACS-sorted HSCs or ESCs) need to be benchmarked against bulk 3' end sequencing
  TPMs via rank correlation.
when_to_use_negative:
- Input is already a bulk 3'-seq feature table with no single-cell UMI matrix — there
  is nothing to pseudobulk or validate
- The goal is differential 3'UTR usage testing across perturbations rather than benchmarking
  against a bulk gold standard (use scUTRboot instead)
- Raw FASTQ data that still needs kallisto-bustools quantification — run the scUTRquant
  pipeline first to obtain '*.txs.Rds' outputs
edam_operation: http://edamontology.org/operation_3891
edam_topics:
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_3308
tools:
- name: scUTRquant
  role: Produces the SingleCellExperiment '*.txs.Rds' objects with per-isoform UMI
    counts that are pseudobulked and correlated
  repo: https://github.com/Mayrlab/scUTRquant
- name: kallisto-bustools
  role: Underlying quantification workflow on which scUTRquant builds to generate
    3'UTR isoform counts from 3'-end tag scRNA-seq libraries
  repo: https://github.com/Mayrlab/scUTRquant
- name: Snakemake
  role: Workflow manager for running the scUTRquant pipeline that generates the input
    counts
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1038/s41467-024-48254-9
    title: Quantifying 3′UTR length from scRNA-seq data reveals changes independent
      of gene expression
  source_content_hash: sha256:41c280d61028a4b2a1bd11130f14e78486295a878ec1fc7caeafd8da0fd23116
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/Mayrlab/scUTRquant
  tools:
  - Snakemake
  - kallisto-bustools
  - scUTRquant
  edam_operation: http://edamontology.org/operation_3891
  edam_topics:
  - http://edamontology.org/topic_0203
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6923076923076923
n_claims: 13
---

# spearman-rank-correlation-analysis

## Summary

Validates scUTRquant-derived 3'UTR isoform quantification from scRNA-seq by computing Spearman rank correlations between pseudobulk UMI-derived TPMs and gold-standard bulk 3'-seq TPMs (HSC: ArrayExpress E-MTAB-7391; ESC: PolyASite-annotated bulk data).

## When to use

Use when pseudobulk 3'UTR isoform counts from scRNA-seq samples (e.g., FACS-sorted HSCs or ESCs) need to be benchmarked against bulk 3' end sequencing TPMs via rank correlation.

## When NOT to use

- Input is already a bulk 3'-seq feature table with no single-cell UMI matrix — there is nothing to pseudobulk or validate
- The goal is differential 3'UTR usage testing across perturbations rather than benchmarking against a bulk gold standard (use scUTRboot instead)
- Raw FASTQ data that still needs kallisto-bustools quantification — run the scUTRquant pipeline first to obtain '*.txs.Rds' outputs

## Inputs

- scUTRquant SingleCellExperiment Rds objects ('*.txs.Rds') with sparse UMI count matrices in the `counts` slot
- colData sample metadata for grouping cells into samples
- Bulk 3'-seq TPM reference tables (ArrayExpress E-MTAB-7391 for HSC; PolyASite-annotated ESC bulk data)

## Outputs

- 'tpm_correlations.csv' with Spearman rho values for HSC and ESC comparisons
- Pseudobulk TPM matrices per sample per 3'UTR transcript isoform

## How to apply

Load the scUTRquant SingleCellExperiment '*.txs.Rds' objects and extract sparse UMI count matrices from the `counts` slot. Sum counts across cells within each sample to pseudobulk per transcript isoform, grouping with `colData` sample metadata, then convert pseudobulk UMI counts to TPM per sample. Load bulk 3'-seq TPM references (ArrayExpress E-MTAB-7391 for HSC, PolyASite-annotated ESC bulk data) and match features to the UTRome isoforms. Compute Spearman rank correlations between pseudobulk scUTRquant TPMs and bulk 3'-seq TPMs separately for the HSC and ESC comparisons, and report rho values in 'tpm_correlations.csv'.

## Related tools

- **scUTRquant** (Produces the SingleCellExperiment '*.txs.Rds' objects with per-isoform UMI counts that are pseudobulked and correlated) — https://github.com/Mayrlab/scUTRquant
- **kallisto-bustools** (Underlying quantification workflow on which scUTRquant builds to generate 3'UTR isoform counts from 3'-end tag scRNA-seq libraries) — https://github.com/Mayrlab/scUTRquant
- **Snakemake** (Workflow manager for running the scUTRquant pipeline that generates the input counts)

## Examples

```
cor.test(hsc_sc_tpm, hsc_bulk_tpm, method = "spearman")
```

## Evaluation signals

- HSC Spearman rho falls near the reported ~0.86 (value_in_range check, tolerance ±0.05)
- ESC Spearman rho falls near the reported ~0.70 (value_in_range check, tolerance ±0.05)
- 'tpm_correlations.csv' exists and contains rho entries for both HSC and ESC comparisons
- Pseudobulk step yields one TPM vector per sample over matched UTRome isoform features shared with the bulk 3'-seq reference

## Limitations

- Most reads from 10x Genomics data do not span mRNA 3' end cleavage sites, so scUTRquant isoform calls are approximate relative to exact de novo 3' end mapping
- Correlations are expected lower for ESC (~0.70) than HSC (~0.86), so a single rho threshold is not appropriate across cell types
- Feature matching between bulk 3'-seq references and UTRome isoforms may leave unmatched features that must be dropped before correlation

## Evidence

- [methods] Sum counts across cells within each sample to obtain pseudobulk UMI counts per transcript isoform, using the sample metadata in `colData` for grouping.: [evidence span withheld — anchor status: not_found]
- [results] scUTRquant-derived 3'UTR isoform counts strongly correlate with bulk 3'-seq values (Spearman's ρ = 0.86): [evidence span withheld — anchor status: not_found]
- [abstract] We incorporate the CS annotations into a computational pipeline, called scUTRquant, for rapid, accurate, and simultaneous quantification of gene and 3'UTR isoform expression from single-cell RNA: "We incorporate the CS annotations into a computational pipeline, called scUTRquant, for rapid, accurate, and simultaneous quantification of gene and 3'UTR isoform expression from single-cell RNA"
- [other] check that HSC rho falls near the reported ~0.86 and ESC rho near the reported ~0.70 (value_in_range check with a tolerance of ±0.05): [evidence span withheld — anchor status: not_found]
