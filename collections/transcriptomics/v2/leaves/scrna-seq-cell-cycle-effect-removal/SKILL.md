---
name: scrna-seq-cell-cycle-effect-removal
description: Use when scRNA-Seq expression data shows within-cell-type heterogeneity
  from the cell cycle that obscures cell-type differences, and a known set of cell-cycle
  genes (mouse or human) is available for control-gene annotation.
when_to_use_negative:
- The dataset has very low cell-cycle activity, e.g. the Patel et al. human glioblastoma
  data, where the article instead uses a smaller cutoff CΔ = 2 rather than default
  settings
- 'The dominant unwanted effect is not the cell cycle (e.g. batch or technical replicates):
  ccRemover is designed for removing the cell-cycle effect exclusively'
- The cell-cycle effect influences only a few control genes (~8% or fewer), so it
  may not be captured by principal components and ccRemover may miss it
edam_operation: http://edamontology.org/operation_2939
edam_topics:
- http://edamontology.org/topic_3170
tools:
- name: ccRemover
  role: 'Core method: identifies cell-cycle-associated principal components via a
    formal bootstrap statistical test and removes them from the expression matrix'
  repo: https://cran.r-project.org/package=ccRemover
- name: scLVM
  role: Comparator latent-factor method applied via its Python script with K = 1 or
    scree-plot-chosen K for benchmarking cell-cycle removal
- name: Biomart
  role: Used to download lists of genes annotated to the cell cycle
- name: org.Mm.eg.db
  role: Provides mouse cell-cycle gene annotations
- name: org.Hs.eg.db
  role: Provides human cell-cycle gene annotations
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1038/srep33892
    title: Identifying and removing the cell-cycle effect from single-cell RNA-Sequencing
      data
  source_content_hash: sha256:182faf51db3c687ba83ab0aa007bc452d336147ecc02b4f81996b24484273aa6
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - Biomart
  - Python
  - ccRemover
  - org.Hs.eg.db
  - org.Mm.eg.db
  - scLVM
  edam_operation: http://edamontology.org/operation_2939
  edam_topics:
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 15
---

# scrna-seq-cell-cycle-effect-removal

## Summary

A pre-processing workflow for identifying and removing the cell-cycle effect from single-cell RNA-Seq data using ccRemover (with scLVM as a comparator), which preserves other biological signals of interest while boosting downstream clustering of cell types.

## When to use

Use when scRNA-Seq expression data shows within-cell-type heterogeneity from the cell cycle that obscures cell-type differences, and a known set of cell-cycle genes (mouse or human) is available for control-gene annotation.

## When NOT to use

- The dataset has very low cell-cycle activity, e.g. the Patel et al. human glioblastoma data, where the article instead uses a smaller cutoff CΔ = 2 rather than default settings
- The dominant unwanted effect is not the cell cycle (e.g. batch or technical replicates): ccRemover is designed for removing the cell-cycle effect exclusively
- The cell-cycle effect influences only a few control genes (~8% or fewer), so it may not be captured by principal components and ccRemover may miss it

## Inputs

- scRNA-Seq expression matrix with genes as rows and cells as columns (log-transformable raw counts), e.g. the Buettner et al. mESC dataset (ArrayExpress E-MTAB-2802) or the Mahata et al. T-helper cell dataset
- Gene name annotations for the organism (mouse or human symbols)
- Cell-cycle gene annotations retrieved via Biomart and org.Mm.eg.db / org.Hs.eg.db

## Outputs

- Cell-cycle-corrected expression matrix 'xhat' from ccRemover
- scLVM-corrected data matrix (when running the comparison)
- Downstream comparison outputs, e.g. 2-means clusters, DE gene lists (t-test at FDR 0.01), and hypergeometric odds ratios; for the mESC task, the summary figure mesc_ccremover_sclvm_comparison.png

## How to apply

Retrieve a known set of cell-cycle genes from annotation databases (Biomart plus org.Mm.eg.db or org.Hs.eg.db), then preprocess the expression matrix so genes are rows and cells are columns, log-transformed and centered gene-by-gene. Use gene_indexer to map gene names (species = 'mouse' or 'human', name_type = 'symbols') to cell-cycle/control indices, using all non-annotated genes as control genes, then run ccRemover with cutoff = 3, max_it = 4, nboot = 200, ntop = 10 to obtain the corrected matrix 'xhat'; ccRemover performs PCA on the control genes and uses a bootstrap-based t-statistic with cutoff CΔ to decide which principal components are cell-cycle effects, iterating up to max_it times. For comparison, apply scLVM (Python script from the scLVM GitHub page) trying both the default K = 1 and the number of leading factors given by the scree plot. Assess success by applying the same downstream analysis to original and corrected data — e.g. 2-means clustering with a t-test at false discovery rate 0.01 to find differentially expressed genes and a hypergeometric-test odds ratio — expecting cell types to separate while cell-cycle stages mix after correction.

## Related tools

- **ccRemover** (Core method: identifies cell-cycle-associated principal components via a formal bootstrap statistical test and removes them from the expression matrix) — https://cran.r-project.org/package=ccRemover
- **scLVM** (Comparator latent-factor method applied via its Python script with K = 1 or scree-plot-chosen K for benchmarking cell-cycle removal)
- **Biomart** (Used to download lists of genes annotated to the cell cycle)
- **org.Mm.eg.db** (Provides mouse cell-cycle gene annotations)
- **org.Hs.eg.db** (Provides human cell-cycle gene annotations)

## Examples

```
gene_cycle_gene_indices <- gene_indexer(gene_names, species = "mouse", name_type = "symbols"); xhat <- ccRemover(dat, cutoff = 3, max_it = 4, nboot = 200, ntop = 10, bar=FALSE)
```

## Evaluation signals

- Corrected data clusters by cell type with cell-cycle stages mixed, as in the simulation where ccRemover correctly identified the 2nd and 3rd principal components as cell-cycle effects
- Downstream 2-means clustering yields a TH2 signature odds ratio greater than 1 (on the T-helper original data the odds ratio was less than 1, indicating meaningless clustering)
- The corrected matrix 'xhat' is produced with the specified parameters (cutoff = 3, max_it = 4, nboot = 200, ntop = 10) and no further cell-cycle effects are detected after removal on subsequent iterations
- For the mESC comparison, the output file mesc_ccremover_sclvm_comparison.png exists and shows both methods remove the cell-cycle effect with ccRemover slightly outperforming scLVM by Buettner et al.'s two criteria
- If scLVM with K = 1 removes the cell-type effect instead of the cell-cycle effect, or K from the scree plot removes both cell-cycle and cell-type effects, the correction has failed as in the simulation

## Limitations

- ccRemover relies on annotation databases that are always incomplete and inaccurate, and requires a known set of cell-cycle genes
- If the cell-cycle effect only influences a few of the control genes, it may not be captured by the principal components and hence missed for detection
- ccRemover removes only the cell-cycle effect, not other confounding effects that may dominate after removal
- The current scLVM approach does not inspect whether a leading factor represents the cell cycle and risks removing other biological components of interest
- No version numbers or explicit repository URLs are reported for scLVM or ccRemover in the article

## Evidence

- [abstract] We present ccRemover, a new method that reliably identifies the cell-cycle effect and removes it.: "We present ccRemover, a new method that reliably identifies the cell-cycle effect and removes it."
- [methods] We recommend transforming the data to a log scale and centering it on a gene-by-gene basis.: "We recommend transforming the data to a log scale and centering it on a gene-by-gene basis."
- [methods] xhat <- ccRemover(dat, cutoff = 3, max_it = 4, nboot = 200, ntop = 10, bar=FALSE): "xhat <- ccRemover(dat, cutoff = 3, max_it = 4, nboot = 200, ntop = 10, bar=FALSE)"
- [results] the same clustering algorithms and statistical tests are applied to each of them allowing us to compare the performance of the methods: "the same clustering algorithms and statistical tests are applied to each of them allowing us to compare the performance of the methods"
- [discussion] if the cell-cycle effect only influences a few of the control genes, it may not be captured by the principal components and hence missed for detection: "if the cell-cycle effect only influences a few of the control genes, it may not be captured by the principal components and hence missed for detection"
