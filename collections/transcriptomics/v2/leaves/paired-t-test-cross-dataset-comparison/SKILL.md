---
name: paired-t-test-cross-dataset-comparison
description: Use when two hyperparameter-optimization or embedding-scoring methods
  must be compared across multiple matched datasets and per-dataset embedding-quality
  metrics (KNN/KNC preservation) need statistical significance testing via paired
  t-tests.
when_to_use_negative:
- When comparing methods on a single dataset — the paired t-test design requires multiple
  matched datasets (here 20 simulated replicates) to have statistical power
- When the two methods optimize different hyperparameters that cannot be aligned to
  a shared evaluation space (a shared pre-embedding PCA space is needed for KNN/KNC
  computation)
- When datasets are not matched (e.g., independent cell sets per method) — a paired
  design and paired t-test would be inappropriate
edam_operation: http://edamontology.org/operation_3659
edam_topics:
- http://edamontology.org/topic_0091
tools:
- name: scDEED
  role: Selects optimal t-SNE perplexity by minimizing the number of dubious cell
    embeddings; one arm of the paired comparison
  repo: https://github.com/JSB-UCLA/scDEED
- name: EMBEDR
  role: Selects optimal t-SNE perplexity via embedding-reliability metrics; the competing
    arm of the paired comparison (preferred perplexity 1000)
  repo: https://github.com/ejohnson643/EMBEDR
- name: Seurat
  role: Preprocessing pipeline (CreateSeuratObject, NormalizeData, ScaleData, FindVariableFeatures,
    RunPCA, RunTSNE) producing the Seurat objects scDEED consumes and the PCA pre-embedding
    space for KNN/KNC
  repo: https://github.com/satijalab/seurat-wrappers
- name: R
  role: Implementation environment for scDEED, Seurat, and the paired t-tests (t.test
    across simulated datasets)
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1038/s41467-024-45891-y
    title: Statistical method scDEED for detecting dubious 2D single-cell embeddings
      and optimizing t-SNE and UMAP hyperparameters
  source_content_hash: sha256:7a484e76cf3203d798309a4407be83d1a34d65844bcb8ac0a048177d43c45d32
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/JSB-UCLA/scDEED
  tools:
  - EMBEDR
  - R
  - Seurat
  - scDEED
  - scDEED R package
  edam_operation: http://edamontology.org/operation_3659
  edam_topics:
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.46153846153846156
n_claims: 13
---

# paired-t-test-cross-dataset-comparison

## Summary

Compares two embedding-quality assessment methods (scDEED vs EMBEDR) by running each method's perplexity optimization on the same real and scDesign3-simulated datasets, measuring KNN/KNC neighborhood preservation between PCA space and 2D t-SNE embeddings, and testing per-dataset metric differences with paired t-tests.

## When to use

Use when two hyperparameter-optimization or embedding-scoring methods must be compared across multiple matched datasets and per-dataset embedding-quality metrics (KNN/KNC preservation) need statistical significance testing via paired t-tests.

## When NOT to use

- When comparing methods on a single dataset — the paired t-test design requires multiple matched datasets (here 20 simulated replicates) to have statistical power
- When the two methods optimize different hyperparameters that cannot be aligned to a shared evaluation space (a shared pre-embedding PCA space is needed for KNN/KNC computation)
- When datasets are not matched (e.g., independent cell sets per method) — a paired design and paired t-test would be inappropriate

## Inputs

- Seurat-normalized single-cell count matrices (e.g., Tabula Muris marrow, 4821 cells) with PCA embedding and elbow-plot-chosen dimension K
- scDesign3-simulated single-cell datasets (20 replicates) processed identically
- Per-method optimal t-SNE perplexity selections (e.g., scDEED: perplexity 100; EMBEDR: perplexity 1000)
- Pre-embedding PCA distance matrices and 2D t-SNE embedding distance matrices per dataset

## Outputs

- Per-dataset perplexity optimization tables (dubious-embedding counts / reliability metrics per perplexity setting)
- KNN and KNC preservation scores per dataset at each method's selected perplexity
- Paired t-test p-values for KNN and KNC across the 20 simulated datasets
- Consolidated results file, e.g., perplexity_optimization_comparison.csv

## How to apply

Preprocess each dataset (e.g., Tabula Muris marrow, 4821 cells, and 20 scDesign3-simulated datasets) with Seurat (NormalizeData, ScaleData, FindVariableFeatures, RunPCA), choosing the input dimension K via ElbowPlot. Run each competing method over its own perplexity grid — scDEED over 20 to 410 (step 30) then 450 to 800 (step 50), EMBEDR including its maximal candidate value 1000 — and select each method's optimal perplexity by minimizing the number of dubious cell embeddings (scDEED) or maximizing embedding-reliability metrics (EMBEDR). At each method's chosen perplexity, compute KNN (k-nearest-neighbor preservation) and KNC (k-neighbor-class preservation) between the pre-embedding PCA space and the 2D t-SNE embedding. Because the same datasets are used by both methods, apply paired t-tests across the 20 simulated datasets for each metric; in the reference benchmark this yielded KNN p=7.88e-15 (scDEED significantly better) and KNC p=0.208 (no significant difference). Record per-dataset optimization tables, selected perplexities, and t-test results in a single output CSV such as perplexity_optimization_comparison.csv.

## Related tools

- **scDEED** (Selects optimal t-SNE perplexity by minimizing the number of dubious cell embeddings; one arm of the paired comparison) — https://github.com/JSB-UCLA/scDEED
- **EMBEDR** (Selects optimal t-SNE perplexity via embedding-reliability metrics; the competing arm of the paired comparison (preferred perplexity 1000)) — https://github.com/ejohnson643/EMBEDR
- **Seurat** (Preprocessing pipeline (CreateSeuratObject, NormalizeData, ScaleData, FindVariableFeatures, RunPCA, RunTSNE) producing the Seurat objects scDEED consumes and the PCA pre-embedding space for KNN/KNC) — https://github.com/satijalab/seurat-wrappers
- **R** (Implementation environment for scDEED, Seurat, and the paired t-tests (t.test across simulated datasets))

## Examples

```
result = scDEED(data, K = K, reduction.method = 'tsne', perplexity = 40, rerun = F); t.test(knn_scDEED, knn_EMBEDR, paired = TRUE)
```

## Evaluation signals

- EMBEDR selects the maximal candidate perplexity (1000) while scDEED selects a non-maximal perplexity (100) on the Tabula Muris marrow dataset
- Paired t-test across the 20 simulated datasets yields KNN p=7.88e-15 for the KNN preservation metric
- Paired t-test across the 20 simulated datasets yields KNC p=0.208 for the KNC metric, i.e., no significant difference
- Each method's optimal perplexity is confirmed by a minimum in dubious-embedding counts (scDEED) across its full grid (20–410 step 30, then 450–800 step 50)
- Output CSV (perplexity_optimization_comparison.csv) contains per-dataset rows, both selected perplexities, and both t-test p-values

## Limitations

- KNC results may show no significant difference (p=0.208) even when KNN strongly favors one method, so a single metric can mask the comparison outcome
- EMBEDR's tendency to prefer high perplexity values (1000) means grid endpoints must include the maximal candidate to expose this bias
- The number of dubious embeddings can exhibit complex trends as hyperparameter value increases, indicating complex topology where interpretation of the selected optimum should be cautious
- Seurat version differences (5.2.1 vs 5.1.0) reportedly affect installation and may affect reproducibility of preprocessing

## Evidence

- [other] EMBEDR preference for high perplexity and paired comparison outcome: [evidence span withheld — anchor status: not_found]
- [discussion] scDEED outperforms EMBEDR as the comparison rationale: "Our results indicate that scDEED outperforms a competing method, EMBEDR, in terms of dubious embedding detection, hyperparameter optimization, and computational time."
- [abstract] Perplexity optimization by minimizing dubious embeddings: "by minimizing the number of dubious cell embeddings, scDEED provides intuitive guidance for optimizing the hyperparameters of an embedding method"
- [readme] Default t-SNE perplexity grid searched by scDEED: "perplexity: The default perplexity values are: 20 to 410, increasing by 30 and 450 to 800, increasing by 50"
- [methods] Seurat preprocessing pipeline feeding the comparison: "data = CreateSeuratObject(input_counts) data = NormalizeData(data) data = ScaleData(data) data = FindVariableFeatures(data) data = RunPCA(data)"
