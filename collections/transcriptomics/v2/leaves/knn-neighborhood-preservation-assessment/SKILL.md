---
name: knn-neighborhood-preservation-assessment
description: Use when comparing two embedding configurations (e.g., t-SNE perplexity
  100 from scDEED vs 1000 from EMBEDR) by measuring retention of pre-embedding neighbor
  structure.
when_to_use_negative:
- When the pre-embedding space itself is under evaluation (e.g., comparing alternative
  dimensionality reductions rather than a given PCA reference)
- When the goal is hyperparameter optimization per se rather than post-hoc evaluation
  of an already-selected embedding; use scDEED's dubious-embedding minimization instead
edam_operation: http://edamontology.org/operation_3891
edam_topics:
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_2269
tools:
- name: scDEED
  role: Selects the t-SNE perplexity to evaluate by minimizing the number of dubious
    cell embeddings, defining reliable neighborhoods via pre- vs post-embedding neighbor
    similarity
  repo: github.com/JSB-UCLA/scDEED
- name: EMBEDR
  role: Competing perplexity-optimization method whose selected embedding (perplexity
    1000) is evaluated with KNN/KNC preservation
  repo: ejohnson643/EMBEDR
- name: Seurat
  role: Preprocessing (NormalizeData, ScaleData, FindVariableFeatures, RunPCA) that
    defines the pre-embedding PCA space used as the neighborhood reference
  repo: satijalab/seurat-wrappers
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1038/s41467-024-45891-y
    title: Statistical method scDEED for detecting dubious 2D single-cell embeddings
      and optimizing t-SNE and UMAP hyperparameters
  source_content_hash: sha256:922491d464e68621db575d115228b7187e14c570b51da827df03690fbbe1470b
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - EMBEDR
  - R
  - Seurat
  - scDEED
  - scDEED R package
  edam_operation: http://edamontology.org/operation_3891
  edam_topics:
  - http://edamontology.org/topic_3308
  - http://edamontology.org/topic_2269
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.75
n_claims: 12
---

# KNN neighborhood preservation assessment

## Summary

Quantifies how faithfully a 2D embedding (e.g., t-SNE at a chosen perplexity) preserves local neighborhood structure from the pre-embedding space (e.g., PCA) using KNN and KNC preservation metrics, and tests differences across datasets with paired t-tests.

## When to use

Use when comparing two embedding configurations (e.g., t-SNE perplexity 100 from scDEED vs 1000 from EMBEDR) by measuring retention of pre-embedding neighbor structure.

## When NOT to use

- When the pre-embedding space itself is under evaluation (e.g., comparing alternative dimensionality reductions rather than a given PCA reference)
- When the goal is hyperparameter optimization per se rather than post-hoc evaluation of an already-selected embedding; use scDEED's dubious-embedding minimization instead

## Inputs

- Seurat-processed scRNA-seq count matrix (e.g., Tabula Muris marrow, 4821 cells) with RunPCA applied
- scDesign3-simulated count matrices (20 datasets) preprocessed identically
- Per-dataset optimized 2D t-SNE embeddings at each method's selected perplexity (scDEED: 100; EMBEDR: 1000)

## Outputs

- Per-dataset KNN and KNC preservation values at each method's chosen perplexity
- Paired t-test results comparing methods across the 20 simulated datasets (expected KNN p=7.88e-15, KNC p=0.208)
- perplexity_optimization_comparison.csv containing per-dataset optimization tables, selected perplexities, and t-test results

## How to apply

Preprocess each dataset with Seurat (NormalizeData, ScaleData, FindVariableFeatures, RunPCA), choosing the input dimension K via ElbowPlot, so that the PCA space serves as the pre-embedding reference. Run each embedding method's optimization (e.g., scDEED minimizing dubious cell embeddings over t-SNE perplexities 20–410 step 30 and 450–800 step 50; EMBEDR over its grid including perplexity 1000) to select a perplexity per method. At each method's chosen perplexity, compute KNN preservation (fraction of k nearest neighbors in the PCA space retained among 2D-embedding neighbors) and KNC preservation (neighborhood preservation of cluster-level structure) between pre-embedding PCA distances and 2D t-SNE embedding distances. Compare the two methods by paired t-tests across replicate datasets (e.g., 20 scDesign3-simulated datasets). The rationale mirrors scDEED's core idea: a cell embedding is reliable only if its 2D-embedding neighbors resemble its pre-embedding neighbors.

## Related tools

- **scDEED** (Selects the t-SNE perplexity to evaluate by minimizing the number of dubious cell embeddings, defining reliable neighborhoods via pre- vs post-embedding neighbor similarity) — github.com/JSB-UCLA/scDEED
- **EMBEDR** (Competing perplexity-optimization method whose selected embedding (perplexity 1000) is evaluated with KNN/KNC preservation) — ejohnson643/EMBEDR
- **Seurat** (Preprocessing (NormalizeData, ScaleData, FindVariableFeatures, RunPCA) that defines the pre-embedding PCA space used as the neighborhood reference) — satijalab/seurat-wrappers

## Examples

```
result = scDEED(data, K = K, reduction.method = 'tsne', perplexity = 40, rerun = F); # then compute KNN/KNC preservation between results.PCA$pre_embedding_distances and 2D embedding distances
```

## Evaluation signals

- Output confirms EMBEDR selects the maximal candidate perplexity (1000) while scDEED selects a non-maximal perplexity (100) on the Tabula Muris marrow data
- Paired t-test p-values across the 20 simulated datasets match the reported values (KNN p=7.88e-15, KNC p=0.208)
- perplexity_optimization_comparison.csv contains per-dataset rows for all 21 datasets (1 real + 20 simulated) with both KNN and KNC columns
- scDEED's dubious-embedding counts are recorded at every grid point (20 to 410 step 30, then 450 to 800 step 50) before selecting the minimum

## Limitations

- KNC preservation may not differ significantly between methods even when KNN does (KNC p=0.208 vs KNN p=7.88e-15), so cluster-level preservation is not guaranteed to discriminate
- scDEED's classification depends on user-adjustable percentile thresholds (dubious_cutoff 0.05, trustworthy_cutoff 0.95), which propagate into the selected perplexity being assessed
- Seurat version differences (5.2.1 vs 5.1.0) can affect the pipeline, and the difference between dubious and trustworthy embeddings may reflect randomness rather than biological signals at optimized hyperparameters

## Evidence

- [readme] A cell embedding is considered 'dubious' if the cell's neighbors before and after the embedding differ drastically.: "A cell embedding is considered 'dubious' if the cell's neighbors before and after the embedding differ drastically."
- [abstract] calculating a reliability score for every cell embedding based on the similarity between the cell's 2D-embedding neighbors and pre-embedding neighbors: "By calculating a reliability score for every cell embedding based on the similarity between the cell's 2D-embedding neighbors and pre-embedding neighbors"
- [readme] By default, the pre-embedding space is the PCA space, although this can also be changed if the user wishes: "By default, the pre-embedding space is the PCA space, although this can also be changed if the user wishes"
- [readme] The default perplexity values are: 20 to 410, increasing by 30 and 450 to 800, increasing by 50: "The default perplexity values are: 20 to 410, increasing by 30 and 450 to 800, increasing by 50"
