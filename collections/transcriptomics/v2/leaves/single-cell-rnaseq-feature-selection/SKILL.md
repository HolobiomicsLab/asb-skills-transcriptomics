---
name: single-cell-rnaseq-feature-selection
description: Use when clustering scRNA-seq data (e.g., 10X pbmc_1k_v2 counts) and
  you need an optimal, non-redundant feature gene set to improve cluster separation
  and DE-gene detection before PCA/clustering.
when_to_use_negative:
- The input is already a curated feature/marker gene table rather than a raw or normalized
  single-cell count matrix
- The analysis goal is differential expression testing or cell type annotation of
  existing clusters, not feature selection for unsupervised clustering
- Genes have not been filtered for detectability (no min.cells cutoff applied), which
  would let poorly measured genes corrupt the GGC matrix
edam_operation: http://edamontology.org/operation_2939
edam_topics:
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_0203
tools:
- name: DUBStepR
  role: 'Primary algorithm: constructs the GGC matrix, runs stepwise regression, guilt-by-association
    expansion, and Density Index-based feature set optimization'
  repo: github.com/prabhakarlab/DUBStepR
- name: Seurat
  role: Recommended companion for loading 10X data, LogNormalize normalization, and
    holding the selected features in assays$RNA@var.features
  repo: https://satijalab.org/seurat/
- name: hdf5r
  role: Loads 10X hdf5 feature-barcode matrices into the Seurat object as input
provenance:
  source_task_ids:
  - task_007
  - task_001
  - task_008
  source_papers:
  - doi: 10.1038/s41467-021-26085-2
    title: DUBStepR is a scalable correlation-based feature selection method for accurately
      clustering single-cell data
  source_content_hash: sha256:d720fe869643f3c69e693e85acd38aa70f9a7f155b620eb686ca53d770839186
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - DUBStepR
  - GiniClust
  - M3Drop
  - R
  - Seurat
  - graphics
  - hdf5r
  edam_operation: http://edamontology.org/operation_2939
  edam_topics:
  - http://edamontology.org/topic_3308
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6923076923076923
n_claims: 13
---

# single-cell-rnaseq-feature-selection

## Summary

Selects informative genes for single-cell RNA-seq clustering using DUBStepR, a correlation-based feature selection method that ranks genes by gene-gene correlation (GGC) structure, derives a minimally redundant seed set via stepwise regression, and expands it to the Density Index-optimal feature set.

## When to use

Use when clustering scRNA-seq data (e.g., 10X pbmc_1k_v2 counts) and you need an optimal, non-redundant feature gene set to improve cluster separation and DE-gene detection before PCA/clustering.

## When NOT to use

- The input is already a curated feature/marker gene table rather than a raw or normalized single-cell count matrix
- The analysis goal is differential expression testing or cell type annotation of existing clusters, not feature selection for unsupervised clustering
- Genes have not been filtered for detectability (no min.cells cutoff applied), which would let poorly measured genes corrupt the GGC matrix

## Inputs

- Raw scRNA-seq count matrix (e.g., 10X Genomics pbmc_1k_v2 filtered feature-barcode matrix, loaded via hdf5r into a Seurat object)
- Log-normalized expression matrix (DUBStepR logNormalize or Seurat LogNormalize output)
- Independent cell type annotations for benchmarking (optional)

## Outputs

- Ranked feature gene list (ordered by guilt-by-association)
- Seed gene set (genes up to the scree-plot elbow index)
- Optimal feature gene set maximizing the Density Index (e.g., written to optimal_feature_genes.csv)
- Stepwise regression scree plot (e.g., scree_plot.png) and ordered regressed-out gene list (stepwise_regression_results.csv)

## How to apply

Preprocess counts by discarding low-quality cells, filtering mitochondrial, ribosomal, spike-in, and pseudogenes (min.cells threshold, e.g. 0.05*ncol), and log-normalizing (normalization.method = 'LogNormalize' or DUBStepR's logNormalize). Construct the gene-gene correlation (GGC) matrix with getGGC(), then bin genes by expression to compute correlation range z-scores (getCorrelationRange) that prioritize well-correlated genes. Run runStepwiseReg() on the GGC matrix, regressing out the gene explaining maximal residual variance at each step, and use findElbow() on the resulting scree plot of variance explained to define the minimally redundant seed gene set. Expand the seed set by guilt-by-association (adding the gene most correlated with the current set) and use getOptimalFeatureSet() to pick the feature set size maximizing the Density Index; assign the result to the Seurat object's var.features. Rationale: cell-type-specific marker genes tend to be well correlated with each other, so GGC structure outperforms variance/dispersion-only feature selection.

## Related tools

- **DUBStepR** (Primary algorithm: constructs the GGC matrix, runs stepwise regression, guilt-by-association expansion, and Density Index-based feature set optimization) — github.com/prabhakarlab/DUBStepR
- **Seurat** (Recommended companion for loading 10X data, LogNormalize normalization, and holding the selected features in assays$RNA@var.features) — https://satijalab.org/seurat/
- **hdf5r** (Loads 10X hdf5 feature-barcode matrices into the Seurat object as input)

## Examples

```
DUBStepR(input.data = logNormalize(raw.data), min.cells = 0.05*ncol(input.data), species = 'human', optimise.features = TRUE, k = 10, num.pcs = 20, error = 0)
```

## Evaluation signals

- The stepwise regression scree plot shows monotonically decreasing variance explained per step with a clear elbow detected by findElbow()
- The seed gene set exactly equals the ordered regressed-out gene list truncated at the detected elbow index
- Downstream PCA + clustering (k = 10, num.pcs = 20) yields higher mean scaled silhouette index against known cell type labels than baselines (HVGDisp, HVGVST, trendVar, devianceFS, HLG, M3Drop, GiniClust) at comparable feature set sizes
- DE-detection AUROC against top-500 DE vs bottom-500 non-DE genes exceeds 0.97 on benchmark datasets, matching reported results
- The optimal feature set is the gene set size at which the Density Index computed by getOptimalFeatureSet() is maximized (reported near 200-500 feature genes)

## Limitations

- Elbow point computation is only described in a supplementary note (Supp. Note 1B), so the exact findElbow() algorithm is not specified in the main text
- DUBStepR approaches its maximum silhouette index at 200-500 feature genes; performance details at other set sizes are deferred to supplementary figures (Supp. Fig. S3, S4) not in the main text
- Benchmark cell type annotations relied on an independently ascertained dataset collection identified only by reference number 28, complicating exact reproduction

## Evidence

- [intro] GGC matrix construction from filtered, binned genes: "After filtering out mitochondrial, ribosomal, spike-in, and pseudogenes, DUBStepR constructs a GGC matrix"
- [intro] Correlation range z-scores select well-correlated genes: "DUBStepR constructs a GGC matrix and bins genes by expression to compute their correlation range z-scores, which are used to select well-correlated genes."
- [intro] Stepwise regression identifies minimally redundant seed features: "DUBStepR then performs stepwise regression on the GGC matrix to identify a minimally redundant subset of seed features"
- [methods] API signature of the main function: "DUBStepR(input.data, min.cells, species, optimise.features, k, num.pcs, error)"
- [readme] Rationale: marker genes are correlated: "DUBStepR is based on the intuition that cell-type-specific marker genes tend to be well correlated with each other"
- [abstract] Benchmark superiority claim: "Despite selecting a relatively small number of genes, DUBStepR substantially outperformed existing single-cell feature selection methods across diverse clustering benchmarks."
