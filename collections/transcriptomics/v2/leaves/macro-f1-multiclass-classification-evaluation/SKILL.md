---
name: macro-f1-multiclass-classification-evaluation
description: Use when a semi-supervised single-cell label-prediction experiment has
  trained cell-type classifiers on partially annotated data and you need a balanced
  metric that weights rare and abundant cell types equally.
when_to_use_negative:
- Task requires batch-mixing or trajectory-conservation integration metrics (e.g.,
  kBET, NMI/Louvain clustering) rather than label-prediction accuracy — this skill
  only scores annotation prediction
- Class imbalance should be weighted by population size rather than treating all cell
  types equally (macro F1 weights rare and abundant classes identically, unlike micro-averaged
  F1)
- No held-out ground-truth labels exist for the unlabeled cells, so F1 per cell type
  cannot be computed
edam_operation: http://edamontology.org/operation_2938
edam_topics:
- http://edamontology.org/topic_3170
tools:
- name: scDREAMER-Sup
  role: Supervised/semi-supervised variant whose cell type classifier C (trained with
    cross-entropy loss on labeled cells) produces the predictions being scored with
    macro F1
  repo: https://github.com/Zafar-Lab/scDREAMER
- name: scANVI
  role: Supervised baseline classifier whose predictions are scored with the same
    macro F1 computation on the identical partial-label split
- name: Scanpy
  role: Provides AnnData loading and preprocessing (normalize_total, log-transform,
    scanpy.pp.highly_variable_genes flavor='seurat') of the data on which the compared
    classifiers operate
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1038/s41467-023-43590-8
    title: scDREAMER for atlas-level integration of single-cell datasets using deep
      generative model paired with adversarial classifier
  source_content_hash: sha256:b894f8ef95001a2e2d76b4f2db78546c307cb41b95e318082136eb135a88c109
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
  edam_operation: http://edamontology.org/operation_2938
  edam_topics:
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.875
n_claims: 8
---

# macro-f1-multiclass-classification-evaluation

## Summary

Evaluate multi-class cell-type label prediction by computing the macro F1-score (arithmetic mean of per-cell-type F1-scores) for each classifier on held-out unlabeled cells, as used to show scDREAMER-Sup outperforms scANVI in semi-supervised settings with 10%, 20% and 50% missing labels.

## When to use

Use when a semi-supervised single-cell label-prediction experiment has trained cell-type classifiers on partially annotated data and you need a balanced metric that weights rare and abundant cell types equally.

## When NOT to use

- Task requires batch-mixing or trajectory-conservation integration metrics (e.g., kBET, NMI/Louvain clustering) rather than label-prediction accuracy — this skill only scores annotation prediction
- Class imbalance should be weighted by population size rather than treating all cell types equally (macro F1 weights rare and abundant classes identically, unlike micro-averaged F1)
- No held-out ground-truth labels exist for the unlabeled cells, so F1 per cell type cannot be computed

## Inputs

- Predicted cell type labels for unlabeled/held-out cells from each trained classifier (e.g., scDREAMER-Sup classifier C and scANVI)
- Ground-truth cell type annotations for the held-out cells
- Set of cell type categories present in the labeled training split

## Outputs

- Macro F1-score per method per missing-label fraction (10%, 20%, 50%)
- Comparison table saved as macro_f1_comparison.csv

## How to apply

After training each cell type classifier (e.g., scDREAMER-Sup's classifier C trained with cross-entropy loss on labeled cells only, and the scANVI baseline on the identical partial-label split), run the classifiers on the unlabeled cells to obtain predicted annotations. For each method and each missing-label fraction (10%, 20%, or 50%), compute the F1-score per cell type and then take the arithmetic mean over cell types — the macro F1 — so that performance on rare populations is not dominated by abundant ones. Compile per-method, per-fraction macro F1 values into a comparison table (macro_f1_comparison.csv). Valid scores lie in [0, 1], with scDREAMER-Sup reported in the 0.86–0.9 range and above scANVI across fractions. In this paper the classifiers were trained on Scanpy-AnnData inputs preprocessed by mitochondrial-gene QC, scanpy.pp.normalize_total, log-transformation, and top-2000 highly variable gene selection (flavor='seurat').

## Related tools

- **scDREAMER-Sup** (Supervised/semi-supervised variant whose cell type classifier C (trained with cross-entropy loss on labeled cells) produces the predictions being scored with macro F1) — https://github.com/Zafar-Lab/scDREAMER
- **scANVI** (Supervised baseline classifier whose predictions are scored with the same macro F1 computation on the identical partial-label split)
- **Scanpy** (Provides AnnData loading and preprocessing (normalize_total, log-transform, scanpy.pp.highly_variable_genes flavor='seurat') of the data on which the compared classifiers operate)

## Examples

```
from sklearn.metrics import f1_score; macro_f1 = f1_score(y_true_unlabeled, y_pred_scDREAMERSup, average='macro')
```

## Evaluation signals

- macro_f1_comparison.csv contains a macro F1 entry for both scDREAMER-Sup and scANVI at each missing-label fraction (10%, 20%, 50%)
- All macro F1 values fall within [0, 1]
- Per-fraction macro F1 for scDREAMER-Sup is comparable to or higher than scANVI (paper reports 0.86–0.9 across fractions)
- Macro F1 is computed as the arithmetic mean of per-cell-type F1-scores over the full set of annotated cell types, not accuracy or micro-average

## Limitations

- Macro F1 assumes every cell type contributes equally; performance on very rare cell types can shift the mean substantially
- Scores depend on the fraction of masked labels; results at 10% missing labels are not directly comparable to the 50% setting
- scDREAMER, being deep-learning-based, may require fine-tuning of some parameters for extracting the best performance, which affects reproducibility of classifier scores

## Evidence

- [results] Across semi-supervised settings with 10, 20 and 50% of cells missing labels, scDREAMER-Sup predicted cell type labels with high accuracy (Macro F1-score 0.86–0.9) and outperformed scANVI.: [evidence span withheld — anchor status: not_found]
- [methods] Compute the macro F1-score (arithmetic mean of per-cell-type F1-scores) for each method and fraction, and compile the results into a comparison table saved as macro_f1_comparison.csv.: [evidence span withheld — anchor status: not_found]
- [intro] Using semi-supervised integration settings, we also demonstrate scDREAMER-Sup’s superiority over: "Using semi-supervised integration settings, we also demonstrate scDREAMER-Sup’s superiority over"
- [readme] scDREAMER-SUP can also be when cell type annotations are missing in the datasets i.e., 10%, 20%, 50%: "scDREAMER-SUP can also be when cell type annotations are missing in the datasets i.e., 10%, 20%, 50%"
- [discussion] scDREAMER being a deep learning-based method may require fine-tuning of some parameters for extracting the best performance: "scDREAMER being a deep learning-based method may require fine-tuning of some parameters for extracting the best performance"
