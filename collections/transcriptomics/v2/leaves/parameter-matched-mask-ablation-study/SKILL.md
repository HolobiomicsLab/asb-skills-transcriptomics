---
name: parameter-matched-mask-ablation-study
description: Use when validating that a knowledge-masked Transformer annotator (e.g.,
  TOSICA on mPancreas, Supplementary Fig. 8) actually gains accuracy from curated
  .gmt masks rather than model capacity alone.
when_to_use_negative:
- The goal is to run TOSICA for prediction/annotation of a new query with an already-trained
  model, not to interrogate whether the mask itself is responsible for accuracy
- The model's masks are not a controllable design parameter (e.g., unmasked, fully
  attention-based architectures) so no knowledge-vs-random comparison is possible
- The reference has no per-epoch validation split (30% of training) available, so
  convergence behavior cannot be measured
edam_topics:
- http://edamontology.org/topic_3474
tools:
- name: TOSICA
  role: Annotated model trained via TOSICA.train under each mask condition (knowledge
    .gmt masks vs random masks)
  repo: https://github.com/JackieHanLab/TOSICA
- name: torch
  role: Backend for training with SGD, cosine learning rate decay, and cross-entropy
    loss; recording per-epoch validation accuracy
  repo: https://github.com/JackieHanLab/TOSICA
- name: scanpy
  role: Loads the mPancreas reference AnnData and enables var_names alignment between
    reference and query
  repo: https://github.com/JackieHanLab/TOSICA
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1038/s41467-023-35923-4
    title: Transformer for one stop interpretable cell type annotation
  source_content_hash: sha256:3c53561dc7d3719796335d14f60b925f489110ed950da5b5e9bee5a9ed20f896
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/JackieHanLab/TOSICA
  tools:
  - Python
  - TOSICA
  - scanpy
  - torch
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3474
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6470588235294118
n_claims: 17
---

# Parameter-matched mask ablation study

## Summary

An ablation protocol testing whether a masked-attention annotator's accuracy truly depends on expert-knowledge masks, by training TOSICA on the same reference with knowledge-based gene-set masks and with parameter-matched random masks (1% and 5% reserved gene-to-token connections) and comparing epochs-to-convergence and converged validation accuracy.

## When to use

Use when validating that a knowledge-masked Transformer annotator (e.g., TOSICA on mPancreas, Supplementary Fig. 8) actually gains accuracy from curated .gmt masks rather than model capacity alone.

## When NOT to use

- The goal is to run TOSICA for prediction/annotation of a new query with an already-trained model, not to interrogate whether the mask itself is responsible for accuracy
- The model's masks are not a controllable design parameter (e.g., unmasked, fully attention-based architectures) so no knowledge-vs-random comparison is possible
- The reference has no per-epoch validation split (30% of training) available, so convergence behavior cannot be measured

## Inputs

- Reference AnnData object (e.g., mPancreas mouse reference) loaded with scanpy, with a cell-type label column in .obs
- Knowledge-based mask .gmt files: m_GO_bp.gmt, m_reactome.gmt, m_TF.gmt
- Random mask matrices reserving 1% and 5% of gene-to-token connections, parameter-matched to the knowledge mask
- Query AnnData with var_names consistent and in the same order as ref_adata

## Outputs

- Trained model weights (model-n.pth) for each of the three mask conditions
- Per-epoch validation accuracy curves per mask condition
- Summary table convergence_comparison.csv (epochs-to-convergence and converged accuracy by mask type)

## How to apply

Load the reference AnnData (e.g., the mPancreas mouse reference) with scanpy, confirm the label column in ref_adata.obs, and align query gene ordering so var_names are identical and in the same order. Train a TOSICA model with the mouse knowledge masks (m_GO_bp.gmt, m_reactome.gmt, m_TF.gmt) using SGD with cosine learning rate decay and cross-entropy loss, holding out 30% of the training set as validation. Construct random mask matrices reserving 1% and 5% of gene-to-token connections, matched in parameter count to the knowledge-based mask, and train TOSICA with each. Record validation accuracy per epoch for all three conditions to determine epochs needed to converge and the converged accuracy. Summarize epochs-to-convergence and converged accuracy per mask condition and save as convergence_comparison.csv; interpret knowledge-mask superiority where random masks need more epochs or converge lower (as on mPancreas).

## Related tools

- **TOSICA** (Annotated model trained via TOSICA.train under each mask condition (knowledge .gmt masks vs random masks)) — https://github.com/JackieHanLab/TOSICA
- **torch** (Backend for training with SGD, cosine learning rate decay, and cross-entropy loss; recording per-epoch validation accuracy) — https://github.com/JackieHanLab/TOSICA
- **scanpy** (Loads the mPancreas reference AnnData and enables var_names alignment between reference and query) — https://github.com/JackieHanLab/TOSICA

## Examples

```
TOSICA.train(ref_adata, gmt_path='mouse_gobp', project='mPancreas_mask_ablation', label_name='celltype')
```

## Evaluation signals

- Trained model weights (model-n.pth) exist for all three mask conditions (knowledge, 1% random, 5% random) in their project folders
- Random-mask parameter count matches the knowledge-based mask parameter count (1% and 5% reserved gene-to-token connections)
- convergence_comparison.csv contains epochs-to-convergence and converged accuracy for every mask condition
- On mPancreas, random-mask models approach the knowledge-mask accuracy but require more epochs and/or reach lower converged accuracy, consistent with the supplementary finding
- ref_adata and query_adata var_names are identical and in the same order before training

## Limitations

- Random masks usually achieve accuracy similar to knowledge-based masks, so knowledge-mask benefit may only appear on specific datasets (e.g., lower converged accuracy on mPancreas)
- Learning rate value, batch size, and random seed are not reported, so exact convergence curves may not be reproducible
- Number of attention heads (H) is not specified in the methods, leaving the mask ablation architecture partly underdetermined

## Evidence

- [supplementary] Random masks usually achieve accuracy similar to knowledge-based masks, but on the mPancreas dataset the accuracy converges lower with the random mask.: [evidence span withheld — anchor status: not_found]
- [discussion] SGD with cosine learning rate decay and 30% validation split: [evidence span withheld — anchor status: not_found]
- [discussion] Masked tokens by biologically relevant pathways underpin accuracy: "The high accuracy and batch-insensitivity of TOSICA can be mainly attributed to the attention layers and tokens masked by high-level biologically relevant pathways or regulons"
- [readme] Gene ordering constraint for reference/query alignment: "the `var_names` (genes) of the `ref_adata` and `query_adata` must be consistent and in the same order."
