---
name: single-cell-count-aggregation-by-cell-type
description: Use when scUTRquant SingleCellExperiment objects contain sparse gene
  and 3'UTR isoform count matrices at single-cell resolution and counts must be summed
  per cell type before pairwise differential testing.
when_to_use_negative:
- Cells lack cell-type annotations and exclude_unannotated_cells cannot resolve them,
  so no valid pseudobulk groups can be formed
- The analysis target is single-cell-level variability rather than cell-type-level
  comparisons, where pseudobulk aggregation would obscure per-cell signal
- Genes of interest are single-UTR genes, since the pipeline restricts to multi-UTR
  genes co-expressed in both members of the cell-type pair
edam_operation: http://edamontology.org/operation_3800
edam_topics:
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_3170
tools:
- name: scUTRquant
  role: Produces the SingleCellExperiment objects with gene and 3'UTR isoform count
    matrices that are aggregated per cell type
  repo: https://github.com/Mayrlab/scUTRquant
- name: scUTRboot
  role: Consumes the aggregated counts to run weighted UTR isoform (WUI) tests calling
    differentially used 3' ends
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1038/s41467-024-48254-9
    title: Quantifying 3′UTR length from scRNA-seq data reveals changes independent
      of gene expression
  source_content_hash: sha256:fa6f43e02aa42efedb0cbd0e342ee589a7e6a2eff65bf284db84662c790a2867
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/Mayrlab/scUTRquant
  tools:
  - scUTRboot
  - scUTRquant
  edam_operation: http://edamontology.org/operation_3800
  edam_topics:
  - http://edamontology.org/topic_0203
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9230769230769231
n_claims: 13
---

# Single-cell count aggregation by cell type

## Summary

Aggregates per-cell gene-level and 3'UTR isoform (multi-UTR) counts from scUTRquant-generated SingleCellExperiment objects into per-cell-type pseudobulk matrices so that pairwise cell-type statistical tests (Welch t-test DGE, scUTRboot WUI tests) can be run.

## When to use

Use when scUTRquant SingleCellExperiment objects contain sparse gene and 3'UTR isoform count matrices at single-cell resolution and counts must be summed per cell type before pairwise differential testing.

## When NOT to use

- Cells lack cell-type annotations and exclude_unannotated_cells cannot resolve them, so no valid pseudobulk groups can be formed
- The analysis target is single-cell-level variability rather than cell-type-level comparisons, where pseudobulk aggregation would obscure per-cell signal
- Genes of interest are single-UTR genes, since the pipeline restricts to multi-UTR genes co-expressed in both members of the cell-type pair

## Inputs

- scUTRquant-generated SingleCellExperiment objects
- Sparse gene-level count matrix
- Sparse 3'UTR isoform count matrix
- Cell metadata / cell-type annotations from colData

## Outputs

- Per-cell-type aggregated gene count matrix (pseudobulk)
- Per-cell-type aggregated multi-UTR isoform count matrix restricted to co-expressed multi-UTR genes
- dge_dul_overlap_fractions.tsv with per-pair overlap fractions and chi-square results

## How to apply

Load the scUTRquant-generated SingleCellExperiment objects and extract the sparse gene-level and 3'UTR isoform count matrices together with cell metadata from colData. Sum isoform counts within each cell type to form pseudobulk profiles, restricting to multi-UTR genes that are co-expressed in both members of each cell-type pair being compared. This aggregation mirrors scUTRquant's design of quantifying a consistent set of 3'UTR isoforms so datasets can be integrated, and it provides the per-cell-type count inputs required by Welch t-tests (DGE calls at |fold change| > 1.5, q < 0.05) and scUTRboot weighted UTR isoform (WUI) tests (|dWUI| > 0.10, q < 0.05). Downstream validation uses the resulting aggregated matrices to compute DGE-DUL overlap fractions, which are tested against chance with chi-square tests and expected to fall in the 0-7.4% range.

## Related tools

- **scUTRquant** (Produces the SingleCellExperiment objects with gene and 3'UTR isoform count matrices that are aggregated per cell type) — https://github.com/Mayrlab/scUTRquant
- **scUTRboot** (Consumes the aggregated counts to run weighted UTR isoform (WUI) tests calling differentially used 3' ends)

## Examples

```
sce <- readRDS("scutrquant_out/sce.rds"); agg <- rowsum(as.matrix(assay(sce, "isoform")), sce$cell_type)
```

## Evaluation signals

- Aggregated matrices should only contain multi-UTR genes co-expressed in both members of each cell-type pair
- Each pseudobulk profile should equal the sum of per-cell isoform counts within a cell type from the sparse matrix
- Reported DGE-DUL overlap fractions must fall within the expected 0-7.4% range across 17 cell-type pairs
- Chi-square tests of independence should show independence of DGE and DUL sets in 14/17 sample pairs
- DGE calls retain |fold change| > 1.5 and q < 0.05; DUL calls retain |dWUI| > 0.10 and q < 0.05 after multiple-testing correction

## Limitations

- Aggregation is restricted to multi-UTR genes co-expressed in both members of a pair; single-isoform genes and non-co-expressed genes are excluded from 3'UTR comparisons
- Most reads from 10x Genomics data do not span mRNA 3' end cleavage sites, preventing exact de novo mapping of 3' ends and relying on the pre-built CS catalog
- Isoform resolution depends on the curated cleavage-site annotations (UTRome) used by scUTRquant to quantify a consistent set of 3'UTR isoforms

## Evidence

- [results] Aggregation of isoform counts per cell type from SingleCellExperiment objects: [evidence span withheld — anchor status: not_found]
- [results] Restriction to multi-UTR genes co-expressed in both members of each cell-type pair: [evidence span withheld — anchor status: not_found]
- [discussion] Consistent isoform quantification enabling dataset integration: "scUTRquant quantifies a consistent set of 3′UTR isoforms, making it easier to integrate datasets"
- [abstract] Simultaneous gene and isoform quantification from scRNA-seq: "simultaneous quantification of gene and 3'UTR isoform expression from single-cell RNA sequencing (scRNA-seq) data"
- [readme] Outputs SingleCellExperiment and AnnData objects from raw data: [evidence span withheld — anchor status: external_doc]
