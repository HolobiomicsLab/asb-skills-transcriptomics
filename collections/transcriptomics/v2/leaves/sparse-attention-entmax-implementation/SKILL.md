---
name: sparse-attention-entmax-implementation
description: Use when building a cross-tissue single-cell RNA-seq cell type classifier
  on tabular gene expression data where interpretable, sparse per-input feature attention
  is required.
when_to_use_negative:
- The input is not tabular per-cell gene expression (e.g. raw sequencing reads or
  image data) — scTab's architecture is specific to tabular data
- A different cell type annotation model is needed, e.g. transformer-based scGPT or
  CIForm, which the paper benchmarks as alternatives rather than using this TabNet
  formulation
- Many decision steps or multi-step feature selection are required — the scTab specification
  uses a single decision step
edam_operation: http://edamontology.org/operation_3936
edam_topics:
- http://edamontology.org/topic_3474
tools:
- name: TabNet
  role: Underlying tabular-data architecture whose feature transformer and attention
    transformer blocks are adapted for scTab
- name: scTab
  role: Model whose specification (Methods, Fig. 1b) defines the 1.5-entmax attention
    and single decision step being implemented
  repo: https://github.com/theislab/scTab
- name: NumPy
  role: Used for the input normalization stack, e.g. the sf_normalize function shown
    in the paper's methods code
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1038/s41467-024-51059-5
    title: 'scTab: Scaling cross-tissue single-cell annotation models'
  source_content_hash: sha256:ab12a0e5e240f9ce56977fcb5df1a77be38f69d9af3022a7c6310298a1e35221
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/theislab/scTab
  tools:
  - NumPy
  - TabNet
  - scTab
  edam_operation: http://edamontology.org/operation_3936
  edam_topics:
  - http://edamontology.org/topic_3474
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8
n_claims: 5
---

# Sparse attention with 1.5-entmax (TabNet-adapted feature selection)

## Summary

Implements the scTab cell type classifier's TabNet-derived architecture: feature transformer blocks with batch norm, skip connections and GLU activations, plus a feature attention mask computed with 1.5-entmax (replacing sparsemax) over a 19,331-protein-coding-gene input space to sparsely select informative genes.

## When to use

Use when building a cross-tissue single-cell RNA-seq cell type classifier on tabular gene expression data where interpretable, sparse per-input feature attention is required.

## When NOT to use

- The input is not tabular per-cell gene expression (e.g. raw sequencing reads or image data) — scTab's architecture is specific to tabular data
- A different cell type annotation model is needed, e.g. transformer-based scGPT or CIForm, which the paper benchmarks as alternatives rather than using this TabNet formulation
- Many decision steps or multi-step feature selection are required — the scTab specification uses a single decision step

## Inputs

- Single-cell RNA-seq gene expression matrix restricted to the 19,331 protein-coding genes (features.parquet)
- Cell type label vector for supervised training
- Class frequency table used to derive inverse-frequency class weights

## Outputs

- TabnetClassifier-style PyTorch module emitting class-probability outputs over cell types
- Per-step sparse feature attention masks over the gene input space
- Weighted cross-entropy loss values during training

## How to apply

Fix the input space to the 19,331 protein-coding genes listed in notebooks/store_creation/features.parquet, then apply per-cell size-factor normalization to 10,000 counts followed by log1p on top of the batch norm in the model forward pass. Build feature transformer blocks with batch normalization, skip connections and GLU activations per the TabNet formulation (arXiv:1908.07442), and compute the feature attention mask using 1.5-entmax with a single decision step as specified in the scTab Methods (Fig. 1b). Train with cross-entropy weighted inversely by class frequency to handle the strong cell-type imbalance, wiring everything into a TabnetClassifier-style module mirroring cellnet/models.py. The rationale is that entmax at alpha=1.5 yields sparser attention than softmax, enabling discrete selection of relevant input features per decision step.

## Related tools

- **TabNet** (Underlying tabular-data architecture whose feature transformer and attention transformer blocks are adapted for scTab)
- **scTab** (Model whose specification (Methods, Fig. 1b) defines the 1.5-entmax attention and single decision step being implemented) — https://github.com/theislab/scTab
- **NumPy** (Used for the input normalization stack, e.g. the sf_normalize function shown in the paper's methods code)

## Examples

```
import numpy as np; X_norm = np.log1p(X / X.sum(axis=1, keepdims=True) * 10000)
```

## Evaluation signals

- A forward pass on a dummy batch produces class-probability outputs of the correct shape
- The inverse-frequency weighted cross-entropy loss computes without error on the dummy batch
- Attention masks are sparse (many genes near-zero per cell) as expected from 1.5-entmax rather than a dense softmax distribution
- Gene input dimensionality of the first layer equals 19,331, matching the features.parquet gene list
- Predictions agree with the normalized input pipeline (10000-count size-factor scaling + log1p) used to produce training data

## Limitations

- Performance of the architecture depends on scaling: the paper shows scTab performance scales with both training dataset size (22.2 million cells) and model size, so a small-data implementation may underperform linear baselines
- Cross-tissue annotation itself requires nonlinear models; the same architecture trained without adequate augmentation or data may generalize poorly across tissues
- Label quality limits the model: pre-trained scTab checkpoints still contain some fairly coarsely annotated cell types despite filtering

## Evidence

- [other] scTab first applies input feature normalization to the gene expression vector, then encodes the data via a feature transformer and selects relevant input features through feature attention: [evidence span withheld — anchor status: not_found]
- [methods] Normalize each cell to have `10000` counts (size factor normalization): "1. Normalize each cell to have `10000` counts (size factor normalization)"
- [intro] we selected a recent architecture specifically proposed for tabular data33: [evidence span withheld — anchor status: not_found]
- [readme] `cellnet`: code for models + data loading infrastructure: "* ``cellnet``: code for models + data loading infrastructure"
