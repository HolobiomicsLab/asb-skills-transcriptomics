---
name: scrna-seq-to-bulk-3prime-seq-concordance
description: Use when scUTRquant 3'UTR isoform quantification from scRNA-seq must
  be validated against bulk 3'-seq reference TPMs (e.g., ArrayExpress E-MTAB-7391
  HSC or PolyASite-annotated ESC bulk data) to establish concordance.
when_to_use_negative:
- Input is already a bulk 3'-seq or bulk RNA-seq abundance table with no single-cell
  UMI matrix to pseudobulk
- The scRNA-seq library is not 3'-end tag-based (e.g., full-length protocols), since
  scUTRquant targets 3'-end tag data like 10x Genomics and Microwell-seq
- The goal is de novo cleavage-site mapping rather than validating pre-defined UTRome
  isoform quantification
edam_operation: http://edamontology.org/operation_0570
edam_topics:
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_3512
tools:
- name: scUTRquant
  role: Produces the SingleCellExperiment '*.txs.Rds' 3'UTR isoform UMI count matrices
    that are pseudobulked and correlated against bulk 3'-seq TPMs
  repo: https://github.com/Mayrlab/scUTRquant
- name: kallisto-bustools
  role: Underlying quantification engine of scUTRquant that generates gene and 3'UTR
    isoform counts from 3'-end tag scRNA-seq libraries
  repo: https://github.com/Mayrlab/scUTRquant
- name: Snakemake
  role: Workflow manager used to run the scUTRquant pipeline that generates the input
    count objects
  repo: https://github.com/Mayrlab/scUTRquant
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1038/s41467-024-48254-9
    title: Quantifying 3′UTR length from scRNA-seq data reveals changes independent
      of gene expression
  source_content_hash: sha256:16d65be22fdb970d58b4f281520693ec7b276c021232648f1105520bfab2e1dc
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/Mayrlab/scUTRquant
  tools:
  - Snakemake
  - kallisto-bustools
  - scUTRquant
  edam_operation: http://edamontology.org/operation_0570
  edam_topics:
  - http://edamontology.org/topic_0203
  - http://edamontology.org/topic_3308
  - http://edamontology.org/topic_3512
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8
n_claims: 10
---

# scrna-seq-to-bulk-3prime-seq-concordance

## Summary

Validation skill that benchmarks scUTRquant-derived 3'UTR isoform counts from single-cell RNA-seq against bulk 3' end sequencing (the gold standard) by computing Spearman correlations of per-sample TPMs in matched cell types such as FACS-sorted HSCs and ESCs.

## When to use

Use when scUTRquant 3'UTR isoform quantification from scRNA-seq must be validated against bulk 3'-seq reference TPMs (e.g., ArrayExpress E-MTAB-7391 HSC or PolyASite-annotated ESC bulk data) to establish concordance.

## When NOT to use

- Input is already a bulk 3'-seq or bulk RNA-seq abundance table with no single-cell UMI matrix to pseudobulk
- The scRNA-seq library is not 3'-end tag-based (e.g., full-length protocols), since scUTRquant targets 3'-end tag data like 10x Genomics and Microwell-seq
- The goal is de novo cleavage-site mapping rather than validating pre-defined UTRome isoform quantification

## Inputs

- scUTRquant SingleCellExperiment Rds objects ('*.txs.Rds') with sparse UMI count matrices in the `counts` slot and sample metadata in `colData`
- Bulk 3'-seq TPM reference table from ArrayExpress E-MTAB-7391 (HSC)
- Bulk 3'-seq TPM reference table with PolyASite-annotated 3' ends (ESC)
- UTRome transcript isoform annotations for feature matching

## Outputs

- tpm_correlations.csv reporting Spearman rho values for the HSC and ESC comparisons
- Per-sample pseudobulk TPM tables per 3'UTR transcript isoform

## How to apply

Load the scUTRquant SingleCellExperiment Rds objects (e.g., '*.txs.Rds') and extract the sparse UMI count matrices from the `counts` slot. Sum counts across cells within each FACS-sorted sample to obtain pseudobulk UMI counts per transcript isoform, using sample metadata in `colData` for grouping, then convert pseudobulk UMI counts to TPM per sample. Load the bulk 3'-seq TPM reference tables (ArrayExpress E-MTAB-7391 for HSC and PolyASite-annotated ESC bulk data) and match features to the UTRome isoforms. Compute Spearman rank correlations between pseudobulk scUTRquant TPMs and bulk 3'-seq TPMs for each cell-type comparison. The rationale is that bulk 3' end sequencing directly captures mRNA 3' ends, so strong rank agreement demonstrates that scUTRquant's truncated-transcript (UTRome) quantification from 3'-tag scRNA-seq recovers true isoform abundances.

## Related tools

- **scUTRquant** (Produces the SingleCellExperiment '*.txs.Rds' 3'UTR isoform UMI count matrices that are pseudobulked and correlated against bulk 3'-seq TPMs) — https://github.com/Mayrlab/scUTRquant
- **kallisto-bustools** (Underlying quantification engine of scUTRquant that generates gene and 3'UTR isoform counts from 3'-end tag scRNA-seq libraries) — https://github.com/Mayrlab/scUTRquant
- **Snakemake** (Workflow manager used to run the scUTRquant pipeline that generates the input count objects) — https://github.com/Mayrlab/scUTRquant

## Examples

```
R: sce <- readRDS('sample.txs.Rds'); pb <- Matrix::rowSums(SummarizedExperiment::assay(sce,'counts')[,which(colData(sce)$sample=='HSC_1')]); cor.test(tpm_sc, tpm_bulk, method='spearman')
```

## Evaluation signals

- HSC Spearman rho between pseudobulk scUTRquant TPMs and bulk 3'-seq TPMs falls near the reported ~0.86 (value_in_range check with tolerance ±0.05)
- ESC Spearman rho falls near the reported ~0.70 (value_in_range check with tolerance ±0.05)
- tpm_correlations.csv is produced and contains rho entries for both the HSC and ESC comparisons
- Feature matching step confirms bulk 3'-seq cleavage sites map onto UTRome isoforms without excessive unmatched features
- Pseudobulk aggregation reproduces per-sample totals consistent with colData sample grouping (one pseudobulk column per sample)

## Limitations

- Most reads from 10x Genomics data do not span mRNA 3' end cleavage sites (CS), so exact de novo mapping of mRNA 3' ends is not possible and concordance is assessed on UTRome isoforms rather than exact sites
- Concordance depends on availability of matched bulk 3'-seq references (e.g., E-MTAB-7391 HSC, PolyASite ESC); cell types without bulk 3' end data cannot be validated this way
- scUTRquant counts a consistent set of 3'UTR isoforms from truncated transcript references, so agreement is at isoform level and sensitive to the CS catalog used

## Evidence

- [results] In FACS-sorted hematopoietic stem cells (HSCs), scUTRquant-derived 3′UTR isoform counts strongly correlate with bulk 3′-seq values (Spearman's ρ = 0.86): [evidence span withheld — anchor status: not_found]
- [methods] workflow pseudobulk step: [evidence span withheld — anchor status: not_found]
- [methods] bulk reference tables: [evidence span withheld — anchor status: not_found]
- [discussion] kallisto-bustools basis: "By building on the kallisto-bustools workflow, we provide a fast and direct means of quantification that is compatible with most 3′-end tag scRNA-seq libraries"
- [intro] 10x read limitation: "most reads from 10x Genomics data do not span mRNA 3′ end cleavage sites (CS), which prevents exact de novo mapping of mRNA 3′ ends"
