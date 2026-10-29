---
name: cell-type-annotation-model-evaluation
description: Use when benchmarking a cell type annotation tool's accuracy on reference-to-query
  label transfer, or when assessing whether annotation performance depends on expert-knowledge
  masks vs. parameter-matched random masks.
when_to_use_negative:
- The input is a single pre-computed accuracy table without per-epoch training curves
  or dataset metadata, so neither convergence nor correlation analysis can be reproduced
- The task is to annotate a new query dataset (use TOSICA.pre for prediction, not
  this evaluation skill)
- Reference and query var_names are inconsistent or unordered differently — the mask-comparison
  evaluation would be invalid until genes are aligned
edam_operation: http://edamontology.org/operation_3920
edam_topics:
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_3474
tools:
- name: TOSICA
  role: Annotation model trained under knowledge-based and random mask conditions
    via TOSICA.train; predictions via TOSICA.pre
  repo: https://github.com/JackieHanLab/TOSICA
- name: torch
  role: Deep learning backend for training TOSICA with SGD and cosine learning rate
    decay
- name: scanpy
  role: Loading and handling reference/query AnnData single-cell objects
provenance:
  source_task_ids:
  - task_004
  - task_002
  source_papers:
  - doi: 10.1038/s41467-023-35923-4
    title: Transformer for one stop interpretable cell type annotation
  source_content_hash: sha256:9df972f9fce2ec7a158e599692a103f1e374c908b25a82608660021b70d1178a
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/JackieHanLab/TOSICA
  tools:
  - Python
  - TOSICA
  - scanpy
  - torch
  edam_operation: http://edamontology.org/operation_3920
  edam_topics:
  - http://edamontology.org/topic_3308
  - http://edamontology.org/topic_3474
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9090909090909091
n_claims: 11
---

# cell-type-annotation-model-evaluation

## Summary

A reusable evaluation protocol for cell type annotation models (e.g., TOSICA) that reproduces convergence comparisons between knowledge-based and random masks and correlates dataset characteristics (size, cell type count, entropy, DKL) with annotation accuracy.

## When to use

Use when benchmarking a cell type annotation tool's accuracy on reference-to-query label transfer, or when assessing whether annotation performance depends on expert-knowledge masks vs. parameter-matched random masks.

## When NOT to use

- The input is a single pre-computed accuracy table without per-epoch training curves or dataset metadata, so neither convergence nor correlation analysis can be reproduced
- The task is to annotate a new query dataset (use TOSICA.pre for prediction, not this evaluation skill)
- Reference and query var_names are inconsistent or unordered differently — the mask-comparison evaluation would be invalid until genes are aligned

## Inputs

- Reference dataset AnnData (e.g., mPancreas mouse reference) with a cell type label column in ref_adata.obs
- Query dataset AnnData with var_names consistent with and ordered as the reference genes (query_adata = query_adata[:, ref_adata.var_names])
- Knowledge-based mask files (.gmt): m_GO_bp.gmt, m_reactome.gmt, m_TF.gmt (mouse) or GO_bp.gmt, reactome.gmt, TF.gmt, immune.gmt (human)
- Benchmark accuracy table with per-tool accuracy joined to dataset metadata (dataset size, cell type count, label distribution)

## Outputs

- convergence_comparison.csv summarizing epochs-to-convergence and converged accuracy for knowledge-based vs. 1% and 5% random masks
- correlation_results.csv with per-dataset metrics (Log size, Types, Entropy, DKL) and Pearson correlation coefficients (PCC) of each metric vs. annotation accuracy
- Trained model artifacts per mask condition: mask.npy, pathway.csv, label_dictionary.csv, model-n.pth

## How to apply

For mask-ablation evaluation, load the reference AnnData (e.g., mPancreas mouse data with scanpy), align query genes to ref_adata.var_names in the same order, and train TOSICA via TOSICA.train with knowledge-based masks (m_GO_bp.gmt, m_reactome.gmt, m_TF.gmt) using SGD with cosine learning rate decay and cross-entropy loss, holding out 30% of the training set as validation. Construct random mask matrices reserving 1% and 5% of gene-to-token connections, matched in parameter count to the knowledge-based mask, train each condition, and record per-epoch validation accuracy to estimate epochs-to-convergence and converged accuracy. For dataset-characteristic analysis, assemble per-dataset metrics (Log size via natural-log-transformed dataset size, number of cell Types, Entropy of the cell type label distribution, and DKL of the reference label distribution) and compute Pearson correlation coefficients against per-tool annotation accuracy with scipy.stats.pearsonr, writing results to CSV.

## Related tools

- **TOSICA** (Annotation model trained under knowledge-based and random mask conditions via TOSICA.train; predictions via TOSICA.pre) — https://github.com/JackieHanLab/TOSICA
- **torch** (Deep learning backend for training TOSICA with SGD and cosine learning rate decay)
- **scanpy** (Loading and handling reference/query AnnData single-cell objects)

## Examples

```
import TOSICA, scanpy as sc; ref_adata = sc.read_h5ad('mPancreas_ref.h5ad'); TOSICA.train(ref_adata, gmt_path='m_GO_bp.gmt', project='mouse_mask_eval', label_name='celltype')
```

## Evaluation signals

- Trained model weights (model-n.pth) exist for all three mask conditions (knowledge-based, 1% random, 5% random)
- convergence_comparison.csv shows random-mask models approaching the knowledge-mask accuracy but requiring more epochs and/or reaching lower converged accuracy on mPancreas
- Reproduced Pearson correlation between annotation accuracy and DKL is within a small numerical tolerance of -0.9
- correlation_results.csv contains all four metrics (Log size, Types, Entropy, DKL) for every benchmarked dataset
- label_dictionary.csv and mask.npy artifacts confirm the label column and gene-to-token mask were constructed from the correct .gmt inputs

## Limitations

- Random masks usually achieve accuracy similar to knowledge-based masks, but on mPancreas the random mask converges lower — the result is dataset-dependent, not universal
- Number of attention heads (H), learning rate value, batch size, and random seed are not reported in the methods, limiting exact hyperparameter reproduction
- The reference and query var_names (genes) must be consistent and in the same order, or training/prediction will fail or be invalid

## Evidence

- [discussion] Training setup for mask ablation: "Stochastic gradient descent (SGD) is chosen as optimizer, and we use cosine learning rate decay to avoid too large steps in late stage of training"
- [discussion] Validation split convention: [evidence span withheld — anchor status: not_found]
- [discussion] Mask construction rationale: "we map the input genes to selected gene sets (gmt files)"
- [readme] Gene-order constraint from README: "the `var_names` (genes) of the `ref_adata` and `query_adata` must be consistent and in the same order."
- [readme] Mask file inventory from README: "+ `human_gobp` : GO_bp.gmt
+ `human_immune` : immune.gmt
+ `human_reactome` : reactome.gmt
+ `human_tf` : TF.gmt
+ `mouse_gobp` : m_GO_bp.gmt"
- [supplementary] Random vs knowledge mask finding: [evidence span withheld — anchor status: not_found]
- [results] DKL correlation finding: [evidence span withheld — anchor status: not_found]
