---
name: semi-supervised-cell-type-annotation
description: Use when a preprocessed scRNA-seq AnnData has cell type annotations for
  only a fraction of cells (e.g. 10/20/50% missing labels) and labels must be predicted
  for the unlabeled remainder.
when_to_use_negative:
- All cells already have cell type annotations — the fully supervised scDREAMER-Sup
  mode is the appropriate choice instead
- No cell type annotations exist at all — only the unsupervised scDREAMER integration
  can be run, since classifier C requires labeled cells for cross-entropy training
- The goal is batch correction only without label prediction — unsupervised scDREAMER
  suffices and avoids classifier training
edam_operation: http://edamontology.org/operation_3800
edam_topics:
- http://edamontology.org/topic_3170
tools:
- name: scDREAMER-Sup
  role: Supervised extension whose feed-forward classifier C predicts cell type labels
    from latent embeddings zi in semi-supervised mode
  repo: https://github.com/Zafar-Lab/scDREAMER
- name: scANVI
  role: Baseline semi-supervised label-prediction method trained on the identical
    partial-label split for comparison
- name: Scanpy
  role: AnnData loading, normalize_total, log-transform, and highly_variable_genes
    (flavor='seurat') preprocessing
provenance:
  source_task_ids:
  - task_007
  - task_002
  source_papers:
  - doi: 10.1038/s41467-023-43590-8
    title: scDREAMER for atlas-level integration of single-cell datasets using deep
      generative model paired with adversarial classifier
  source_content_hash: sha256:7cb1041f9487238ab0fc2c8c28c609e1e9f1d439e294a6dc7eded2830c44aaa4
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/Zafar-Lab/scDREAMER
  tools:
  - ADAM optimizer
  - Scanpy
  - scANVI
  - scDREAMER
  - scDREAMER-Sup
  edam_operation: http://edamontology.org/operation_3800
  edam_topics:
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7857142857142857
n_claims: 14
---

# semi-supervised-cell-type-annotation

## Summary

Predict cell type labels for unannotated cells in single-cell integration settings using scDREAMER-Sup, whose feed-forward classifier C learns labels from the latent embedding, and benchmark against scANVI with macro F1-score.

## When to use

Use when a preprocessed scRNA-seq AnnData has cell type annotations for only a fraction of cells (e.g. 10/20/50% missing labels) and labels must be predicted for the unlabeled remainder.

## When NOT to use

- All cells already have cell type annotations — the fully supervised scDREAMER-Sup mode is the appropriate choice instead
- No cell type annotations exist at all — only the unsupervised scDREAMER integration can be run, since classifier C requires labeled cells for cross-entropy training
- The goal is batch correction only without label prediction — unsupervised scDREAMER suffices and avoids classifier training

## Inputs

- Processed human pancreas AnnData (16,382 cells, top 2000 HVGs) from the Figshare benchmarking deposit
- Batch labels and partial cell type annotations with 10/20/50% of labels masked

## Outputs

- Predicted cell type labels for unlabeled cells
- Informative latent embeddings z and predicted labels saved as a single h5ad file
- macro_f1_comparison.csv with macro F1 for scDREAMER-Sup and scANVI at each missing-label fraction

## How to apply

Load the processed dataset (e.g. human pancreas, 16,382 cells) as a Scanpy AnnData, remove low-quality cells by mitochondrial gene counts, normalize with scanpy.pp.normalize_total, log-transform, and select the top 2000 highly variable genes (flavor='seurat'). Mask cell type labels on a held-out fraction (10%, 20%, or 50%) of cells, then train scDREAMER-Sup: the adversarial VAE backbone plus the additional VAE AY and feed-forward classifier C, with C trained via cross-entropy loss on labeled cells only (ADAM optimizer, batch size 128). Predict labels for the masked cells with classifier C and, in parallel, train scANVI on the identical split as the baseline. Compute macro F1-score (arithmetic mean of per-cell-type F1) for both methods and compile a comparison table (macro_f1_comparison.csv).

## Related tools

- **scDREAMER-Sup** (Supervised extension whose feed-forward classifier C predicts cell type labels from latent embeddings zi in semi-supervised mode) — https://github.com/Zafar-Lab/scDREAMER
- **scANVI** (Baseline semi-supervised label-prediction method trained on the identical partial-label split for comparison)
- **Scanpy** (AnnData loading, normalize_total, log-transform, and highly_variable_genes (flavor='seurat') preprocessing)

## Examples

```
model = scDREAMERSup(batch_size=128); model.fit(adata, celltype_name='Celltypes', mask=masked_fraction); preds = model.predict_labels(adata_unlabeled)
```

## Evaluation signals

- macro_f1_comparison.csv contains a macro F1 entry for both scDREAMER-Sup and scANVI at each of the 10%, 20% and 50% missing-label fractions
- All macro F1 values fall in [0, 1]
- scDREAMER-Sup macro F1 is comparable to or higher than scANVI, with reported benchmark values in the 0.86–0.9 range
- Predicted labels for masked cells match held-out annotations when scored with macro F1
- Latent embeddings support both batch mixing and biological conservation

## Limitations

- Macro F1 was only reported for semi-supervised settings with 10, 20 and 50% of cells missing labels; other masking fractions are untested here
- scDREAMER, being deep-learning-based, may require fine-tuning of some parameters for extracting the best performance
- The model assumes prior knowledge of the number of batches

## Evidence

- [full_text] Across semi-supervised settings with 10, 20 and 50% of cells missing labels, scDREAMER-Sup predicted cell type labels with high accuracy (Macro F1-score 0.86–0.9) and outperformed scANVI.: [evidence span withheld — anchor status: not_found]
- [results] classifier trained with cross-entropy loss: [evidence span withheld — anchor status: not_found]
- [methods] top 2000 HVGs selection: "we selected the top 2000 highly variable genes using the function “scanpy.pp.highly_variable_genes” (with flavor parameter as “seurat”)"
- [methods] ADAM batch size 128: [evidence span withheld — anchor status: not_found]
- [readme] README missing-labels support: "scDREAMER-SUP can also be when cell type annotations are missing in the datasets i.e., 10%, 20%, 50%"
