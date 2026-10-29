---
name: single-cell-batch-effect-correction
description: Use when raw read-count matrices from multiple scRNA-seq batches show
  batch separation (e.g., in t-SNE colored by batch) and the goal is corrected counts
  plus cell-type clustering without a priori subtype labels.
when_to_use_negative:
- The experimental design is completely confounded (e.g., batch 1 measures cell types
  1 and 2, batch 2 measures cell types 3 and 4) — the article states no method is
  applicable in that case
- The data require bulk-level batch correction with known subtype labels a priori,
  where ComBat or SVA are designed instead
- Input is not raw read-count data (e.g., already-normalized or corrected matrices),
  since BUSseq models counts, overdispersion, dropout, and cell-specific sequencing
  depth directly
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: BUSseq
  role: Fits the Bayesian hierarchical model (BUSseq_MCMC) and provides corrected_read_counts()
    for quantile-matching-based batch correction of unknown cell types
  repo: github.com/songfd2018/BUSseq
- name: BUSseq-Rpackage
  role: R package installation path providing the BUSseqfits model objects and accessor
    functions (corrected_read_counts, celltypes, intrinsic_genes_BUSseq)
  repo: songfd2018/BUSseq-Rpackage
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1038/s41467-020-16905-2
    title: Flexible experimental designs for valid single-cell RNA-sequencing experiments
      allowing batch effects correction
  source_content_hash: sha256:d07d0dfd47ac2fc8ecc78c7d484500eca1906d0eb517d353fd7ac59dd27d3689
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - BUSseq
  - BUSseq-Rpackage
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 12
---

# single-cell-batch-effect-correction

## Summary

Generate batch-effect-corrected scRNA-seq count data by fitting the BUSseq Bayesian hierarchical model and applying its quantile-matching procedure, so corrected values can be analyzed as if all cells were sequenced in a single batch.

## When to use

Use when raw read-count matrices from multiple scRNA-seq batches show batch separation (e.g., in t-SNE colored by batch) and the goal is corrected counts plus cell-type clustering without a priori subtype labels.

## When NOT to use

- The experimental design is completely confounded (e.g., batch 1 measures cell types 1 and 2, batch 2 measures cell types 3 and 4) — the article states no method is applicable in that case
- The data require bulk-level batch correction with known subtype labels a priori, where ComBat or SVA are designed instead
- Input is not raw read-count data (e.g., already-normalized or corrected matrices), since BUSseq models counts, overdispersion, dropout, and cell-specific sequencing depth directly

## Inputs

- G-by-N raw read-count matrix of scRNA-seq UMI counts across multiple batches (e.g., count_data_[study]_v[version].txt)
- Dimension file listing number of cells, genes, batches, and per-batch cell counts (dim_[study]_v[version].txt)
- Fitted BUSseq model object (e.g., sce_BUSseqfit from BUSseqfits_example)
- Number of cell types K for BUSseq_MCMC (or enumerated K range for BIC selection)

## Outputs

- Batch-effect-free corrected count matrix (corrected_read_counts) written to corrected_read_counts.csv
- Inferred cell-type labels via celltypes(sce_BUSseqfit)
- t-SNE embedding of corrected counts colored by cell type and batch (tsne_corrected_counts.png)

## How to apply

Load the fitted BUSseq model object (e.g., BUSseqfits_example) and the raw read-count matrices in R. For each gene-cell pair, compute the negative-binomial CDF of the observed count under the full fitted model, incorporating the fitted cell-type means, batch location effects, cell-specific sequencing depth, overdispersion, and dropout terms. Draw u uniformly between those CDF values and map u to the quantile of the batch-effect-free negative-binomial distribution whose mean depends only on cell type, yielding corrected counts via corrected_read_counts(). Retrieve inferred cell labels with celltypes(sce_BUSseqfit) and run t-SNE on the corrected matrix, coloring by cell type and by batch. If the true number of cell types K is unknown, run BUSseq_MCMC for a range of K values and select the K with the minimum BIC before correction.

## Related tools

- **BUSseq** (Fits the Bayesian hierarchical model (BUSseq_MCMC) and provides corrected_read_counts() for quantile-matching-based batch correction of unknown cell types) — github.com/songfd2018/BUSseq
- **BUSseq-Rpackage** (R package installation path providing the BUSseqfits model objects and accessor functions (corrected_read_counts, celltypes, intrinsic_genes_BUSseq)) — songfd2018/BUSseq-Rpackage

## Examples

```
R: corrected <- corrected_read_counts(sce_BUSseqfit); write.csv(corrected, "corrected_read_counts.csv"); tsne <- Rtsne(t(corrected)) colored by celltypes(sce_BUSseqfit)
```

## Evaluation signals

- In the t-SNE of corrected counts (tsne_corrected_counts.png), cells group by cell type and show no separation by batch
- Corrected counts require no further cell-specific normalization before downstream analysis
- Output matrix matches the input G-by-N dimensions and is written successfully to corrected_read_counts.csv
- BIC values for the enumerated K range (e.g., K = 3 to 6) identify a stable optimal number of cell types
- Dropout-caused missing values are imputed as part of the correction (imputed_read_counts consistent with corrected values)

## Limitations

- MCMC algorithms carry a heavy computation load; most MCMC steps are parallelizable but runtime can be substantial (e.g., ~12 min for the demo across 4 K values on 4 cores)
- Multi-stage pipelines that ignore uncertainties in previous stages (e.g., clustering errors) can lead to false positives and false negatives
- Cell types must be measured in at least some overlapping batches — the design must fulfill the article's identifiability conditions (reference panel or chain-type designs)

## Evidence

- [methods] corrected value usable downstream: "the corrected value can be used for downstream analysis as if all cells were sequenced in a single batch"
- [methods] corrected_read_counts accessor: "- `corrected_read_counts(sce_BUSseqfit)`"
- [readme] BUSseq model scope: "BUSseq is able to simultaneously correct batch effects, clusters cell types, and takes care of the count data nature, the overdispersion, the dropout events"
- [readme] platform integration and missing cell types: "BUSseq can integrate read count matrices obtained from different scRNA-seq platforms and allow cell types to be measured in some but not all of the batches"
