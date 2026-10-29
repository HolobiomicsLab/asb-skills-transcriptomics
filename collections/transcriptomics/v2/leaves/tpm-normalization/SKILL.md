---
name: tpm-normalization
description: Use when scUTRquant SingleCellExperiment UMI counts must be placed on
  a transcript-per-million scale to compare 3'UTR isoform abundance against bulk 3'
  end sequencing (e.g., ArrayExpress E-MTAB-7391 HSC or PolyASite ESC) TPM values.
when_to_use_negative:
- The input is already a TPM-normalized or other normalized abundance table (e.g.,
  bulk 3'-seq TPMs), which should be matched directly rather than re-normalized
- The goal is per-cell normalization or differential expression at single-cell resolution
  — pseudobulk aggregation across cells per sample is a prerequisite here
- Counts come from a non-3'-end-tag scRNA-seq protocol that scUTRquant does not support,
  since the isoform counts would not reflect genuine 3'UTR usage
edam_operation: http://edamontology.org/operation_0256
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_0203
tools:
- name: scUTRquant
  role: Produces the '*.txs.Rds' SingleCellExperiment objects whose `counts` slot
    supplies the per-cell UMI counts aggregated to pseudobulk before TPM normalization
  repo: https://github.com/Mayrlab/scUTRquant
- name: kallisto-bustools
  role: Underlying quantification workflow that scUTRquant builds on to generate the
    gene and 3'UTR isoform count matrices being normalized
  repo: https://github.com/Mayrlab/scUTRquant
- name: Snakemake
  role: Workflow manager used to install and run the scUTRquant pipeline steps upstream
    of TPM computation
  repo: https://github.com/Mayrlab/scUTRquant
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1038/s41467-024-48254-9
    title: Quantifying 3′UTR length from scRNA-seq data reveals changes independent
      of gene expression
  source_content_hash: sha256:da97307651fa1073d04030e3019c8b034e2c12d492a45d0373093a69df15f6be
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/Mayrlab/scUTRquant
  tools:
  - Snakemake
  - kallisto-bustools
  - scUTRquant
  edam_operation: http://edamontology.org/operation_0256
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7
n_claims: 10
---

# tpm-normalization

## Summary

Convert pseudobulk UMI counts from scUTRquant 3'UTR isoform quantification into TPM per sample so that single-cell-derived isoform abundances can be directly compared against bulk 3'-seq TPM references via rank correlation.

## When to use

Use when scUTRquant SingleCellExperiment UMI counts must be placed on a transcript-per-million scale to compare 3'UTR isoform abundance against bulk 3' end sequencing (e.g., ArrayExpress E-MTAB-7391 HSC or PolyASite ESC) TPM values.

## When NOT to use

- The input is already a TPM-normalized or other normalized abundance table (e.g., bulk 3'-seq TPMs), which should be matched directly rather than re-normalized
- The goal is per-cell normalization or differential expression at single-cell resolution — pseudobulk aggregation across cells per sample is a prerequisite here
- Counts come from a non-3'-end-tag scRNA-seq protocol that scUTRquant does not support, since the isoform counts would not reflect genuine 3'UTR usage

## Inputs

- scUTRquant '*.txs.Rds' SingleCellExperiment objects with sparse UMI count matrices in the `counts` slot and sample metadata in `colData`
- Bulk 3'-seq TPM reference table for HSC (ArrayExpress E-MTAB-7391)
- PolyASite-annotated bulk 3' end sequencing TPM table for ESC

## Outputs

- Pseudobulk TPM matrix: 3'UTR isoform TPM values per sample
- 'tpm_correlations.csv' reporting Spearman rho values for HSC and ESC comparisons

## How to apply

First aggregate sparse UMI counts from the `counts` slot of the scUTRquant '*.txs.Rds' SingleCellExperiment objects into pseudobulk UMI counts per transcript isoform, summing across cells within each sample using the sample metadata in `colData` for grouping. Then normalize each pseudobulk sample by dividing isoform counts by total counts in that sample and multiplying by 10^6 to obtain TPM. Match the TPM-normalized UTRome isoform features to the bulk 3'-seq TPM reference tables (ArrayExpress E-MTAB-7391 for HSC; PolyASite-annotated ESC bulk data). Compute Spearman rank correlations between pseudobulk scUTRquant TPMs and bulk 3'-seq TPMs; the rank-based statistic makes the comparison robust to residual scale differences. Validated against the paper's gold-standard comparison where HSC Spearman rho ≈ 0.86 and ESC rho ≈ 0.70.

## Related tools

- **scUTRquant** (Produces the '*.txs.Rds' SingleCellExperiment objects whose `counts` slot supplies the per-cell UMI counts aggregated to pseudobulk before TPM normalization) — https://github.com/Mayrlab/scUTRquant
- **kallisto-bustools** (Underlying quantification workflow that scUTRquant builds on to generate the gene and 3'UTR isoform count matrices being normalized) — https://github.com/Mayrlab/scUTRquant
- **Snakemake** (Workflow manager used to install and run the scUTRquant pipeline steps upstream of TPM computation) — https://github.com/Mayrlab/scUTRquant

## Examples

```
sce <- readRDS('sample.txs.Rds'); pb <- Matrix::rowSums(counts(sce)[, colData(sce)$sample=='HSC1']); tpm <- 1e6 * pb / sum(pb); cor.test(tpm, bulk_tpm[names(tpm)], method='spearman')
```

## Evaluation signals

- HSC Spearman rho between scUTRquant pseudobulk TPMs and bulk 3'-seq TPMs falls near the reported ~0.86 (value_in_range check, tolerance ±0.05)
- ESC Spearman rho falls near the reported ~0.70 (value_in_range check, tolerance ±0.05)
- Pseudobulk aggregation correctly groups cells by sample using `colData` metadata (each sample yields exactly one TPM column)
- TPM columns sum to 10^6 per sample (scale invariant check)
- 'tpm_correlations.csv' is produced and contains rho values for both HSC and ESC comparisons

## Limitations

- Most reads from 10x Genomics data do not span mRNA 3' end cleavage sites, preventing exact de novo mapping of mRNA 3' ends; isoform TPMs reflect scUTRquant's UTRome annotation rather than exact cleavage positions
- Fidelity of the correlation depends on feature matching between the UTRome isoforms and the bulk 3'-seq reference annotations (E-MTAB-7391 / PolyASite); unmatched features are dropped
- TPM is a relative measure within a sample, so comparisons are only meaningful for isoform proportion differences, not absolute mRNA abundance

## Evidence

- [other] In FACS-sorted hematopoietic stem cells (HSCs), scUTRquant-derived 3′UTR isoform counts strongly correlate with bulk 3′-seq values (Spearman's ρ = 0.86): [evidence span withheld — anchor status: not_found]
- [other] Extract counts slot and pseudobulk workflow: [evidence span withheld — anchor status: not_found]
- [other] Sum counts across cells within each sample to obtain pseudobulk UMI counts per transcript isoform, using the sample metadata in `colData` for grouping.: [evidence span withheld — anchor status: not_found]
- [abstract] For rapid, accurate, and simultaneous quantification of gene and 3'UTR isoform expression from single-cell RNA: "We incorporate the CS annotations into a computational pipeline, called scUTRquant, for rapid, accurate, and simultaneous quantification of gene and 3'UTR isoform expression from single-cell RNA"
- [discussion] By building on the kallisto-bustools workflow, we provide a fast and direct means of quantification that is compatible with most 3′-end tag scRNA-seq libraries: "By building on the kallisto-bustools workflow, we provide a fast and direct means of quantification that is compatible with most 3′-end tag scRNA-seq libraries"
