---
name: learning-curve-convergence-analysis
description: Decision support for whether a knowledge-informed model component (e.g.
  pathway/TF masks) actually drives performance, by benchmarking it against parameter-matched
  random alternatives via learning-curve comparison.
when_to_use_negative:
- Input is a query dataset for prediction only — use TOSICA.pre with an already-trained
  weight file instead of running an ablation
- The goal is simply to annotate cells rather than to evaluate whether expert-knowledge
  masks causally contribute to accuracy
- The reference dataset lacks a reliable label column in ref_adata.obs, since convergence
  curves require supervised validation accuracy
edam_operation: http://edamontology.org/operation_2428
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_3474
tools:
- name: TOSICA
  role: Trains Transformer models with knowledge-based or random masks via TOSICA.train;
    tracks training under identical SGD settings for comparison
  repo: https://github.com/JackieHanLab/TOSICA
- name: scanpy
  role: Loads and inspects the mPancreas reference AnnData, including label column
    and var_names alignment
- name: torch
  role: Backend for TOSICA training with SGD optimizer, cosine learning rate decay,
    and cross-entropy loss
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1038/s41467-023-35923-4
    title: Transformer for one stop interpretable cell type annotation
  source_content_hash: sha256:b42fa65aca50366405419a6bf9aac03bf2bf7343fdcea23b629790557b166c51
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/JackieHanLab/TOSICA
  tools:
  - Python
  - TOSICA
  - scanpy
  - torch
  edam_operation: http://edamontology.org/operation_2428
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_3474
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9
n_claims: 10
---

# learning-curve-convergence-analysis

## Summary

Trains TOSICA models under multiple mask conditions (knowledge-based GMT masks vs parameter-matched random masks) and tracks per-epoch validation accuracy to determine epochs-to-convergence and converged accuracy, testing whether expert-knowledge masks are required for annotation performance.

## When to use

Decision support for whether a knowledge-informed model component (e.g. pathway/TF masks) actually drives performance, by benchmarking it against parameter-matched random alternatives via learning-curve comparison.

## When NOT to use

- Input is a query dataset for prediction only — use TOSICA.pre with an already-trained weight file instead of running an ablation
- The goal is simply to annotate cells rather than to evaluate whether expert-knowledge masks causally contribute to accuracy
- The reference dataset lacks a reliable label column in ref_adata.obs, since convergence curves require supervised validation accuracy

## Inputs

- mPancreas mouse reference AnnData (ref_adata) with a cell-type label column in ref_adata.obs
- Knowledge-based mouse GMT mask files: m_GO_bp.gmt, m_reactome.gmt, m_TF.gmt
- Random mask matrices reserving 1% and 5% of gene-to-token connections, parameter-matched to the knowledge-based mask

## Outputs

- Trained model weights (model-n.pth / .pth files) for knowledge-mask, 1% random-mask, and 5% random-mask conditions
- Per-epoch validation accuracy curves for each mask condition
- convergence_comparison.csv summarizing epochs-to-convergence and converged accuracy per mask condition

## How to apply

Load the mPancreas mouse reference AnnData with scanpy, confirm the label column in ref_adata.obs, and restrict query genes to ref_adata.var_names so gene ordering is consistent. Train a baseline TOSICA model with the knowledge-based mouse masks (m_GO_bp.gmt, m_reactome.gmt, m_TF.gmt) via TOSICA.train using SGD with cosine learning rate decay and cross-entropy loss, holding out 30% of the training set as validation. Construct two random mask matrices reserving 1% and 5% of gene-to-token connections, matched in parameter count to the knowledge-based mask, and train TOSICA with each under identical optimization settings. Record validation accuracy per training epoch for all three conditions, then summarize epochs-to-convergence and converged accuracy in a table (convergence_comparison.csv). The design is a controlled ablation: random masks usually match knowledge masks, but on mPancreas the random mask converges lower, demonstrating dataset-dependent value of the prior.

## Related tools

- **TOSICA** (Trains Transformer models with knowledge-based or random masks via TOSICA.train; tracks training under identical SGD settings for comparison) — https://github.com/JackieHanLab/TOSICA
- **scanpy** (Loads and inspects the mPancreas reference AnnData, including label column and var_names alignment)
- **torch** (Backend for TOSICA training with SGD optimizer, cosine learning rate decay, and cross-entropy loss)

## Examples

```
TOSICA.train(ref_adata, 'mouse_gobp', project='mPancreas_knowledge_mask', label_name='celltype')
```

## Evaluation signals

- Trained model weight files exist for all three mask conditions (knowledge mask, 1% random, 5% random)
- Validation accuracy is recorded per epoch so the number of epochs needed to converge is identifiable for each condition
- Random-mask models approach knowledge-mask accuracy but require more epochs and/or reach lower converged accuracy on mPancreas, consistent with Supplementary Fig. 8
- Random masks are parameter-matched to the knowledge-based mask (1% and 5% gene-to-token connection reserves), so the comparison isolates prior knowledge rather than capacity
- convergence_comparison.csv is produced with rows for each mask condition and columns for epochs-to-convergence and converged accuracy

## Limitations

- On most datasets random masks achieve accuracy similar to knowledge-based masks, so a positive ablation result is dataset-dependent and may only appear on datasets like mPancreas
- Random-mask models may need more epochs to converge, so equal-epoch comparisons would understate their achievable accuracy
- Learning rate value, batch size, and random seed are not reported in the paper, limiting exact reproducibility of the training curves
- ref_adata and query_adata var_names (genes) must be consistent and in the same order for the mask to apply correctly

## Evidence

- [supplementary] Random masks usually achieve accuracy similar to knowledge-based masks, but on the mPancreas dataset the accuracy converges lower with the random mask.: [evidence span withheld — anchor status: not_found]
- [discussion] Train with SGD and cosine decay, 30% validation split: "Stochastic gradient descent (SGD) is chosen as optimizer, and we use cosine learning rate decay"
- [discussion] 30% of training set held out as validation: [evidence span withheld — anchor status: not_found]
- [discussion] TOSICA accuracy attributed to biologically masked attention tokens: "The high accuracy and batch-insensitivity of TOSICA can be mainly attributed to the attention layers and tokens masked by high-level biologically relevant pathways or regulons"
- [readme] Gene ordering must match between reference and query: "the `var_names` (genes) of the `ref_adata` and `query_adata` must be consistent and in the same order."
