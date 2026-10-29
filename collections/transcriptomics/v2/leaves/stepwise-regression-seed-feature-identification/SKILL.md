---
name: stepwise-regression-seed-feature-identification
description: Use when clustering scRNA-seq data and you need a non-redundant seed
  set of well-correlated, cell-type-specific marker genes to drive feature selection
  (e.g., on a log-normalized 10X PBMC matrix in Seurat).
when_to_use_negative:
- Input is already a curated feature/marker table — the stepwise regression assumes
  a full gene-by-cell expression matrix from which correlations are computed
- Genes have not been pre-filtered for mitochondrial, ribosomal, spike-in, and pseudogene
  content, which DUBStepR expects removed before GGC construction
edam_operation: http://edamontology.org/operation_3935
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_0203
tools:
- name: DUBStepR
  role: Implements GGC construction, correlation range filtering, stepwise regression
    seed feature identification, and getOptimalFeatureSet Density Index optimization
  repo: https://github.com/prabhakarlab/DUBStepR
- name: Seurat
  role: Hosts the Seurat object with log-normalized counts from which DUBStepR is
    run and receives the final var.features assignment
  repo: https://satijalab.org/seurat/
- name: R
  role: Runtime environment; DUBStepR requires R >= 3.5.0
- name: hdf5r
  role: Loads the 10X hdf5 feature-barcode data into the Seurat object before feature
    selection
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1038/s41467-021-26085-2
    title: DUBStepR is a scalable correlation-based feature selection method for accurately
      clustering single-cell data
  source_content_hash: sha256:cff5d84175f8f5495f73ec6ef6d4cf6a72bd5ab1850629ddc08affc640fe76a0
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/prabhakarlab/DUBStepR
  tools:
  - DUBStepR
  - R
  - Seurat
  - hdf5r
  edam_operation: http://edamontology.org/operation_3935
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7058823529411765
n_claims: 17
---

# stepwise-regression-seed-feature-identification

## Summary

DUBStepR's stepwise regression identifies a minimally redundant core seed feature set from a gene-gene correlation (GGC) matrix, after filtering genes by correlation range z-score; the seed set is then expanded by guilt-by-association and sized optimally via the Density Index.

## When to use

Use when clustering scRNA-seq data and you need a non-redundant seed set of well-correlated, cell-type-specific marker genes to drive feature selection (e.g., on a log-normalized 10X PBMC matrix in Seurat).

## When NOT to use

- Input is already a curated feature/marker table — the stepwise regression assumes a full gene-by-cell expression matrix from which correlations are computed
- Genes have not been pre-filtered for mitochondrial, ribosomal, spike-in, and pseudogene content, which DUBStepR expects removed before GGC construction

## Inputs

- Log-normalized scRNA-seq count matrix (e.g., 10X pbmc_1k_v2 filtered feature-barcode matrix loaded into a Seurat object via hdf5r)
- Parameters: min.cells (e.g., 0.05*ncol), species, optimise.features = TRUE, k = 10, num.pcs = 20, error = 0

## Outputs

- Minimally redundant seed feature gene set (stepwise-regression core features)
- Guilt-by-association ordered gene list
- Optimal feature gene set (maximizing Density Index), assignable to seuratObj@assays$RNA@var.features (e.g., written to optimal_feature_genes.csv)

## How to apply

After QC, log-normalization (normalization.method = 'LogNormalize'), and filtering out mitochondrial, ribosomal, spike-in, and pseudogenes, construct the gene-gene correlation (GGC) matrix from the normalized data (min.cells = 0.05*ncol(seuratObj)). Bin genes by expression to compute correlation range z-scores and select well-correlated genes. Perform stepwise regression on the GGC matrix to find a minimally redundant subset of seed features that most strongly represent coherent expression variation. Then expand the seed features by guilt-by-association, adding the candidate gene most strongly correlated with the current set, and call getOptimalFeatureSet (with k = 10, num.pcs = 20, error = 0) to pick the feature set size maximizing the Density Index; assign the optimal set to seuratObj@assays$RNA@var.features.

## Related tools

- **DUBStepR** (Implements GGC construction, correlation range filtering, stepwise regression seed feature identification, and getOptimalFeatureSet Density Index optimization) — https://github.com/prabhakarlab/DUBStepR
- **Seurat** (Hosts the Seurat object with log-normalized counts from which DUBStepR is run and receives the final var.features assignment) — https://satijalab.org/seurat/
- **R** (Runtime environment; DUBStepR requires R >= 3.5.0)
- **hdf5r** (Loads the 10X hdf5 feature-barcode data into the Seurat object before feature selection)

## Examples

```
library(DUBStepR); res <- DUBStepR(input.data = lognorm_data, min.cells = 0.05*ncol(seuratObj), species = 'human', optimise.features = TRUE, k = 10, num.pcs = 20, error = 0)
```

## Evaluation signals

- The seed feature set returned by stepwise regression is minimally redundant: no two seed genes carry largely overlapping correlation profiles in the GGC matrix
- Running with optimise.features = TRUE yields a Density Index curve whose maximum identifies the reported optimal feature set size (the article notes a maximum silhouette at 200–500 feature genes)
- getOptimalFeatureSet executes without error on the filtered data, ordered.genes, and elbow point, and returns a non-empty ordered gene list
- Downstream PCA (num.pcs = 20) + k = 10 clustering on the selected features produces silhouette/cluster separation consistent with or better than using all genes or alternative feature selection methods
- The final feature set is assigned to seuratObj@assays$RNA@var.features and matches the genes written to optimal_feature_genes.csv

## Limitations

- Existing feature selection methods, including the baseline DUBStepR is compared against, perform inconsistently across datasets and can yield worse clustering accuracy than no feature selection — evaluate on your own data
- Elbow point computation (findElbow) used by getOptimalFeatureSet is not described in the main text; it is deferred to a supplementary note
- DUBStepR's quantitative scalability thresholds (e.g., for >1 million cells) are only reported in supplementary figures without effect sizes

## Evidence

- [intro] Stepwise regression identifies minimally redundant seed features: "DUBStepR then performs stepwise regression on the GGC matrix to identify a minimally redundant subset of seed features"
- [intro] Guilty-by-association expansion: "which are then expanded by adding correlated features (guilt-by-association)"
- [intro] Stepwise approach rationale: "A key feature of DUBStepR is the use of a stepwise approach to identify an initial core set of genes that most strongly represent coherent expression variation in the dataset."
- [methods] getOptimalFeatureSet signature: "`getOptimalFeatureSet(filt.data, ordered.genes, elbow.pt, k, num.pcs, error)`"
- [methods] DUBStepR call signature: "`DUBStepR(input.data, min.cells, species, optimise.features, k, num.pcs, error)`"
- [intro] Gene pre-filtering before GGC: "After filtering out mitochondrial, ribosomal, spike-in, and pseudogenes, DUBStepR constructs a GGC matrix"
- [readme] Correlation-based intuition (README): "DUBStepR is based on the intuition that cell-type-specific marker genes tend to be well correlated with each other"
