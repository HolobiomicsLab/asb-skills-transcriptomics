---
name: knn-cell-type-classification-evaluation
description: Use when you have cell-type labels and one or more low-dimensional embeddings
  and need a quantitative metric (k-NN / logistic-regression accuracy) to compare
  embedding or batch-correction quality.
when_to_use_negative:
- When no ground-truth cell-type labels exist for the dataset — accuracy-based evaluation
  cannot be computed
- When the goal is assessing local neighborhood/crowding properties rather than global
  cell-type classification accuracy
- When data are in hyperbolic space, since the article notes 'there are not yet many
  tools to process data distributed in the hyperbolic space, such as efficient k-NN
  search tools'
edam_operation: http://edamontology.org/operation_2990
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: scPhere
  role: Produces the vMF spherical embeddings whose cell-type classification accuracy
    is evaluated, via SCPHERE model and model.encode
  repo: https://github.com/klarman-cell-observatory/scPhere
- name: t-SNE
  role: Generates baseline 2D Euclidean embeddings for accuracy comparison on RGC
    and HCL datasets
- name: UMAP
  role: Generates baseline 2D embeddings compared against scPhere global k-NN accuracies
- name: Harmony
  role: Batch-correction comparator embedding evaluated with k-NN and logistic regression
    on the UC colon dataset
- name: LIGER
  role: Batch-correction comparator embedding evaluated with k-NN and logistic regression
- name: Seurat3 CCA
  role: Batch-correction comparator embedding evaluated with k-NN and logistic regression
provenance:
  source_task_ids:
  - task_004
  - task_005
  - task_003
  source_papers:
  - doi: 10.1038/s41467-021-22851-4
    title: Deep generative model embedding of single-cell RNA-Seq profiles on hyperspheres
      and hyperbolic spaces
  source_content_hash: sha256:1bcabc2c089de4853d9c06198d2470818f5727dfaa0c8694ecd76844bd5024ea
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/klarman-cell-observatory/scPhere
  tools:
  - Harmony
  - LIGER
  - Seurat3 CCA
  - TensorFlow
  - UMAP
  - numpy
  - pandas
  - scPhere
  - scipy
  - t-SNE
  edam_operation: http://edamontology.org/operation_2990
  edam_topics:
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6363636363636364
n_claims: 11
---

# knn-cell-type-classification-evaluation

## Summary

Evaluates the biological quality of 2D single-cell embeddings (scPhere, t-SNE, UMAP, Harmony, LIGER, Seurat3 CCA) by classifying cells to cell types with k-NN (e.g., k=25) and logistic-regression classifiers in each latent space and comparing accuracies.

## When to use

Use when you have cell-type labels and one or more low-dimensional embeddings and need a quantitative metric (k-NN / logistic-regression accuracy) to compare embedding or batch-correction quality.

## When NOT to use

- When no ground-truth cell-type labels exist for the dataset — accuracy-based evaluation cannot be computed
- When the goal is assessing local neighborhood/crowding properties rather than global cell-type classification accuracy
- When data are in hyperbolic space, since the article notes 'there are not yet many tools to process data distributed in the hyperbolic space, such as efficient k-NN search tools'

## Inputs

- Cell-by-gene UMI count matrix (e.g., uc_epi.mtx loaded via scPhere read_mtx, transposed with .transpose().todense())
- Cell-type label TSV (e.g., uc_epi_celltype.tsv, header=FALSE)
- Batch annotation vectors (uc_epi_batch_patient.tsv, uc_epi_batch_health.tsv, uc_epi_batch_location.tsv, 0-based)
- Latent embeddings: scPhere z_mean (uc_epi_latent_250epoch.tsv) and comparator embeddings from t-SNE, UMAP, Harmony, LIGER, Seurat3 CCA

## Outputs

- Per-method, per-cell-type k-NN classification accuracies (e.g., knn_accuracy.csv; test-set accuracies 0.79/0.83/0.82 for stromal/epithelial/immune with batch-invariant scPhere)
- Logistic-regression classification accuracies per method and cell compartment
- Method ranking (scPhere vs t-SNE/UMAP/Harmony/LIGER/Seurat3 CCA) for validation against reported figures

## How to apply

Generate each embedding on the same cell-by-gene UMI matrix — e.g., train an SCPHERE vMF model (z_dim=2, observation_dist='nb', seed=0, max_epoch=250, mb_size=128, learning_rate=0.001) with model.encode, and compute comparator embeddings with t-SNE, UMAP, Harmony, LIGER, or Seurat3 CCA. For global-structure benchmarks, assign each cell the label of the nearest cell-type median (global k-NN over cluster centers); for batch-correction benchmarks on the UC colon dataset (30 patients, 3 health statuses, 2 locations from SCP551), train k-NN (k=25) and logistic-regression classifiers on labeled cells and predict held-out or all cells per compartment (stromal, epithelial, immune). Record per-method accuracy per cell type (e.g., knn_accuracy.csv) and rank methods. Correct application requires scPhere to match reported reference accuracies within tolerance and rank above comparators, as in Fig. 2/Fig. 3d–f and Supplementary Figs. 6–9.

## Related tools

- **scPhere** (Produces the vMF spherical embeddings whose cell-type classification accuracy is evaluated, via SCPHERE model and model.encode) — https://github.com/klarman-cell-observatory/scPhere
- **t-SNE** (Generates baseline 2D Euclidean embeddings for accuracy comparison on RGC and HCL datasets)
- **UMAP** (Generates baseline 2D embeddings compared against scPhere global k-NN accuracies)
- **Harmony** (Batch-correction comparator embedding evaluated with k-NN and logistic regression on the UC colon dataset)
- **LIGER** (Batch-correction comparator embedding evaluated with k-NN and logistic regression)
- **Seurat3 CCA** (Batch-correction comparator embedding evaluated with k-NN and logistic regression)

## Examples

```
model = SCPHERE(n_gene=x.shape[1], n_batch=[30, 3, 2], z_dim=2, latent_dist='vmf', observation_dist='nb', seed=0); z_mean = model.encode(x, batch)
```

## Evaluation signals

- scPhere global k-NN accuracies reproduce reported values (73.53% RGC, 92.21% HCL) within ±0.5 percentage points, and rank above t-SNE (47.06%, 79.22%) and UMAP (52.94%, 83.12%)
- On UC epithelial/stromal/immune cells, scPhere's k-NN and logistic-regression accuracies are at least as high as Harmony, Seurat3 CCA, and LIGER (Fig. 3d–f, Supplementary Figs. 6–9)
- Batch-invariant scPhere test-set k-NN (k=25) accuracies (0.79/0.83/0.82) closely match the all-patient model (0.80/0.87/0.80)
- Saved embeddings place held-out test-patient cells mixed with training cells on the unit sphere (batch_invariant=True model)
- All expected output files exist (latent TSV, log-likelihood TSV, training trace PNG, saved model session)

## Limitations

- Accuracy-based evaluation requires deposited cell-type labels; it does not by itself validate trajectory or local structure preservation
- Hyperbolic ('wn') embeddings lack efficient k-NN tooling per the article's discussion
- Results depend on training settings (max_epoch=250, mb_size=128, learning_rate=0.001, seed=0); different settings may shift accuracies

## Evidence

- [methods] model with n_batch and z_dim: "model = SCPHERE(n_gene=x.shape[1], n_batch=[30, 3, 2], z_dim=2, latent_dist='vmf', batch_invariant=False, observation_dist='nb', seed=0)"
- [methods] embedding cells: "z_mean = model.encode(x, batch)"
- [discussion] batch correction benchmark finding: "scPhere performed better than state-of-the-art batch-correction methods such as Harmony, Seurat3 CCA, LIGER"
- [readme] batch-invariant training option: "batch_invariant=True to train batch-invarant scPhere."
- [readme] demo outputs: "The script will produce four major files in the folder `example/demo-out/`"
