---
name: pseudobulk-aggregation-of-single-cell-counts
description: Use when single-cell UMI counts per transcript isoform must be compared
  against bulk 3'-seq or other sample-level references, or when per-sample (not per-cell)
  abundance estimates are required for validation or downstream statistics.
when_to_use_negative:
- The goal is single-cell resolution analysis (e.g., scUTRboot differential 3'UTR
  testing across cells) — pseudobulk collapses cell-level information
- Input is already a sample-level bulk count or TPM table with no cell barcode dimension
edam_topics:
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_0203
tools:
- name: scUTRquant
  role: Produces the '*.txs.Rds' SingleCellExperiment objects whose `counts` slot
    supplies the per-cell 3'UTR isoform UMI counts to be pseudobulked
  repo: https://github.com/Mayrlab/scUTRquant
- name: kallisto-bustools
  role: Underlying quantification workflow generating the per-cell UMI counts that
    are aggregated
  repo: https://github.com/Mayrlab/scUTRquant
- name: Snakemake
  role: Pipeline manager that can install dependencies (via Conda) and run the scUTRquant
    workflow steps
  repo: https://github.com/Mayrlab/scUTRquant
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1038/s41467-024-48254-9
    title: Quantifying 3′UTR length from scRNA-seq data reveals changes independent
      of gene expression
  source_content_hash: sha256:f2ba64441b71863ff71e24dac958148728d818887e362261b5e3de3918a71843
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/Mayrlab/scUTRquant
  tools:
  - Snakemake
  - kallisto-bustools
  - scUTRquant
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3308
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.875
n_claims: 8
---

# Pseudobulk aggregation of single-cell counts

## Summary

Aggregates sparse single-cell UMI count matrices (e.g., scUTRquant '*.txs.Rds' SingleCellExperiment counts) to sample-level pseudobulk counts by summing across cells grouped via `colData` sample metadata, then converts to TPM so single-cell 3'UTR isoform abundances can be compared to bulk 3'-seq TPM references.

## When to use

Use when single-cell UMI counts per transcript isoform must be compared against bulk 3'-seq or other sample-level references, or when per-sample (not per-cell) abundance estimates are required for validation or downstream statistics.

## When NOT to use

- The goal is single-cell resolution analysis (e.g., scUTRboot differential 3'UTR testing across cells) — pseudobulk collapses cell-level information
- Input is already a sample-level bulk count or TPM table with no cell barcode dimension

## Inputs

- scUTRquant SingleCellExperiment Rds objects ('*.txs.Rds') with sparse UMI count matrices in the `counts` slot
- Sample metadata in the SingleCellExperiment `colData` used as grouping variables for aggregation

## Outputs

- Sample-level pseudobulk UMI count matrix (transcript isoforms × samples)
- Per-sample TPM matrix for 3'UTR isoforms
- Spearman correlation values against bulk 3'-seq references, reported in 'tpm_correlations.csv'

## How to apply

Load the scUTRquant SingleCellExperiment Rds objects (e.g., '*.txs.Rds') and extract the sparse UMI count matrices from the `counts` slot. Sum counts across cells within each sample to obtain pseudobulk UMI counts per transcript isoform, using sample metadata from `colData` as the grouping key (cells below a `min_umis` threshold should already have been excluded upstream). Convert pseudobulk UMI counts to TPM per sample, then match features to the bulk reference (e.g., ArrayExpress E-MTAB-7391 HSC bulk 3'-seq or PolyASite-annotated ESC bulk data) and compute Spearman rank correlations. The rationale is that bulk 3'-seq is the gold standard, so sample-level agreement — HSC Spearman rho ~0.86 and ESC ~0.70 (tolerance ±0.05) — validates that single-cell isoform quantification recovers bulk-scale 3'UTR proportions.

## Related tools

- **scUTRquant** (Produces the '*.txs.Rds' SingleCellExperiment objects whose `counts` slot supplies the per-cell 3'UTR isoform UMI counts to be pseudobulked) — https://github.com/Mayrlab/scUTRquant
- **kallisto-bustools** (Underlying quantification workflow generating the per-cell UMI counts that are aggregated) — https://github.com/Mayrlab/scUTRquant
- **Snakemake** (Pipeline manager that can install dependencies (via Conda) and run the scUTRquant workflow steps) — https://github.com/Mayrlab/scUTRquant

## Examples

```
sce <- readRDS("sample.txs.Rds"); pseudo <- t(rowsum(t(counts(sce)), group = colData(sce)$sample)); tpm <- 1e6 * sweep(pseudo, 2, colSums(pseudo), "/")
```

## Evaluation signals

- HSC pseudobulk TPMs vs bulk 3'-seq (E-MTAB-7391) yield Spearman rho ≈ 0.86 (value_in_range check, tolerance ±0.05)
- ESC pseudobulk TPMs vs PolyASite-annotated ESC bulk data yield Spearman rho ≈ 0.70 (value_in_range check, tolerance ±0.05)
- Pseudobulk matrix dimensions are transcript isoforms × samples, with one column per unique sample key in `colData` (no cells remain as columns)
- TPM values sum to ~1e6 per sample within each sample's isoform profile
- Correlation results are written to 'tpm_correlations.csv' with the expected sample/comparison rows

## Limitations

- Most reads from 10x Genomics data do not span mRNA 3' end cleavage sites, so isoform assignment is annotation-dependent and pseudobulk quality is bounded by scUTRquant's UTRome annotation
- Aggregation is only as reliable as the cell-to-sample grouping in `colData`; misannotated cells or unannotated cells (controlled by `exclude_unannotated_cells`) bias pseudobulk profiles
- Low-expression isoforms with few pseudobulk UMIs contribute noise to the correlation and should be considered before feature matching to the bulk reference

## Evidence

- [other] Sum counts across cells within each sample to obtain pseudobulk UMI counts per transcript isoform, using the sample metadata in `colData` for grouping.: [evidence span withheld — anchor status: not_found]
- [methods] Load bulk 3'-seq references and compute Spearman correlations with ±0.05 tolerance.: [evidence span withheld — anchor status: not_found]
- [results] HSC pseudobulk validation correlates with bulk 3'-seq at Spearman rho = 0.86.: [evidence span withheld — anchor status: not_found]
- [readme] README describes pseudobulk summarization as an analysis stage of scUTRquant outputs.: [evidence span withheld — anchor status: external_doc]
