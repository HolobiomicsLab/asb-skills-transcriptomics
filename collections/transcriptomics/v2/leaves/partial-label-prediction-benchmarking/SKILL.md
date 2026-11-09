---
name: partial-label-prediction-benchmarking
description: Use when evaluating how accurately a supervised single-cell integration
  method predicts cell-type labels for unannotated cells, against a baseline like
  scANVI, under partial-annotation settings (10%, 20%, or 50% of cells missing labels).
when_to_use_negative:
- The task is fully supervised integration where all cells are annotated (use scDREAMER-Sup
  in fully supervised mode instead of a partial-label benchmark)
- The goal is unsupervised batch integration without any cell-type annotations or
  label prediction (use unsupervised scDREAMER)
- The dataset lacks ground-truth cell-type labels, so macro F1 on held-out cells cannot
  be computed
edam_operation: http://edamontology.org/operation_3891
edam_topics:
- http://edamontology.org/topic_3170
tools:
- name: scDREAMER-Sup
  role: Semi-supervised integration method whose cell-type classifier C (cross-entropy
    loss on labeled cells only) predicts annotations for unlabeled cells
  repo: https://github.com/Zafar-Lab/scDREAMER
- name: scANVI
  role: Baseline semi-supervised method trained on the same preprocessed data and
    identical partial-label split for macro F1 comparison
- name: Scanpy
  role: AnnData loading, low-quality cell removal, normalize_total, log-transform,
    and highly variable gene selection (top 2000, flavor='seurat')
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1038/s41467-023-43590-8
    title: scDREAMER for atlas-level integration of single-cell datasets using deep
      generative model paired with adversarial classifier
  source_content_hash: sha256:5fa5ed61315e348e1670ffea2b7d144785f90d7cfd0c1f68266a5c4bca3a8c9e
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
  edam_operation: http://edamontology.org/operation_3891
  edam_topics:
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 10
---

# partial-label-prediction-benchmarking

## Summary

Benchmarks semi-supervised cell-type label prediction by training scDREAMER-Sup and scANVI on the same partially annotated scRNA-seq dataset (10/20/50% labels masked) and comparing macro F1-scores on the held-out unlabeled cells.

## When to use

Use when evaluating how accurately a supervised single-cell integration method predicts cell-type labels for unannotated cells, against a baseline like scANVI, under partial-annotation settings (10%, 20%, or 50% of cells missing labels).

## When NOT to use

- The task is fully supervised integration where all cells are annotated (use scDREAMER-Sup in fully supervised mode instead of a partial-label benchmark)
- The goal is unsupervised batch integration without any cell-type annotations or label prediction (use unsupervised scDREAMER)
- The dataset lacks ground-truth cell-type labels, so macro F1 on held-out cells cannot be computed

## Inputs

- Processed single-cell dataset as a Scanpy AnnData object (e.g., human pancreas from the Figshare benchmarking deposit)
- Cell-type annotations for a subset of cells (with 10%, 20%, or 50% masked)
- scANVI baseline configured for the same partial-label split

## Outputs

- Predicted cell-type labels for unlabeled cells from scDREAMER-Sup and scANVI
- macro_f1_comparison.csv with macro F1-score entries for both methods at each missing-label fraction

## How to apply

Load the processed benchmark dataset (e.g., the human pancreas dataset of 16,382 cells from the Figshare benchmarking deposit) as a Scanpy AnnData object, remove low-quality cells based on mitochondrial gene counts, normalize with scanpy.pp.normalize_total, log-transform, and select the top 2000 highly variable genes using scanpy.pp.highly_variable_genes with flavor='seurat'. Mask cell-type labels for a fraction of cells (10%, 20%, or 50%) to create the semi-supervised setting. Train scDREAMER-Sup in semi-supervised mode so the cell-type classifier C learns with cross-entropy loss on labeled cells only, using the ADAM optimizer with batch size 128; train scANVI on the identical preprocessed data and split. Predict annotations for the unlabeled cells with each trained classifier and compute the macro F1-score (arithmetic mean of per-cell-type F1-scores), compiling results into macro_f1_comparison.csv.

## Related tools

- **scDREAMER-Sup** (Semi-supervised integration method whose cell-type classifier C (cross-entropy loss on labeled cells only) predicts annotations for unlabeled cells) — https://github.com/Zafar-Lab/scDREAMER
- **scANVI** (Baseline semi-supervised method trained on the same preprocessed data and identical partial-label split for macro F1 comparison)
- **Scanpy** (AnnData loading, low-quality cell removal, normalize_total, log-transform, and highly variable gene selection (top 2000, flavor='seurat'))

## Examples

```
python scDREAMER/main.py --task semi-supervised --dataset pancreas.h5ad --missing-frac 0.5 --batch-size 128 --optimizer adam
```

## Evaluation signals

- macro_f1_comparison.csv contains a macro F1-score entry for both scDREAMER-Sup and scANVI at each missing-label fraction (10%, 20%, 50%)
- All macro F1 values fall in the range [0, 1]
- scDREAMER-Sup macro F1 is comparable to or higher than scANVI, consistent with the reported 0.86–0.9 range across fractions
- The same train/test label split is used for both methods, confirming a fair comparison
- Preprocessing verified: mitochondrial-gene QC applied, normalize_total + log transform done, and exactly 2000 highly variable genes selected with flavor='seurat'

## Limitations

- The reported advantage holds for the benchmarked pancreas/immune-style tasks; scDREAMER, being a deep learning-based method, may require fine-tuning of some parameters for extracting the best performance
- Results depend on the chosen missing-label fractions (10/20/50%); behavior at other masking rates is not established here
- The current scDREAMER model assumes prior knowledge of the number of batches

## Evidence

- [results] Semi-supervised label prediction benchmark: [evidence span withheld — anchor status: not_found]
- [methods] Semi-supervised training regime: [evidence span withheld — anchor status: not_found]
- [methods] Preprocessing pipeline: [evidence span withheld — anchor status: not_found]
- [readme] Skill applicability from README: "scDREAMER-SUP can also be when cell type annotations are missing in the datasets i.e., 10%, 20%, 50%"
