---
name: attention-head-selection-with-ridge-classifier
description: Use when a labeled ground-truth network (e.g. ChIP-seq or perturb-seq
  edges) is available and you want to bias scPRINT's averaged attention matrix toward
  the heads that recover known regulatory edges.
when_to_use_negative:
- No ground-truth network edges are available to train the classifier — fall back
  to uniform averaging of attention heads (the default zero-shot scPRINT network).
- The goal is a fair comparison of un-supervised foundation-model methods on cell
  type-specific benchmarks; head selection with labeled edges introduces supervised
  information other baselines (scGPT, DeepSEM, Geneformer, GENIE3) do not use.
edam_operation: http://edamontology.org/operation_1781
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_0605
tools:
- name: scPRINT
  role: Foundation model whose per-head Keys/Queries attention embeddings over cells
    provide the candidate head matrices to be weighted
  repo: https://github.com/cantinilab/scPRINT
- name: scikit-learn
  role: Provides the RidgeClassifier (L2 penalty 1, positivity constraint, no intercept)
    fit on ground-truth edges to select heads
- name: BenGRN
  role: Benchmarking package used to compute EPR and AUPRC of the head-selected network
    against gold standards
  repo: https://github.com/jkobject/benGRN
- name: GRnnData
  role: Gene regulatory network data structure used with BenGRN for network evaluation
  repo: https://github.com/cantinilab/GRnnData
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1038/s41467-025-58699-1
    title: 'scPRINT: pre-training on 50 million cells allows robust gene network predictions'
  source_content_hash: sha256:a133fbd121b655509a4cf21e07de9a4752f3c046432be1e6373c464bba6b2eae
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/cantinilab/scPRINT
  tools:
  - BenGRN
  - DeepSEM
  - GRnnData
  - Geneformer
  - Python
  - scGPT
  - scPRINT
  - scikit-learn
  edam_operation: http://edamontology.org/operation_1781
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_0605
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 13
---

# Attention-head selection with RidgeClassifier

## Summary

A supervised variant of scPRINT gene network extraction in which attention heads are weighted (rather than uniformly averaged) using a linear classifier trained on ground-truth regulatory edges, improving recovery of cell-type-specific ChIP-seq/perturb-seq networks.

## When to use

Use when a labeled ground-truth network (e.g. ChIP-seq or perturb-seq edges) is available and you want to bias scPRINT's averaged attention matrix toward the heads that best recover known regulatory edges.

## When NOT to use

- No ground-truth network edges are available to train the classifier — fall back to uniform averaging of attention heads (the default zero-shot scPRINT network).
- The goal is a fair comparison of un-supervised foundation-model methods on cell type-specific benchmarks; head selection with labeled edges introduces supervised information other baselines (scGPT, DeepSEM, Geneformer, GENIE3) do not use.

## Inputs

- scPRINT model attention embeddings (Keys and Queries) computed over ~1024 cells and the 5000 most variable genes
- Gold-standard directed binary gene regulatory network (ChIP-seq or perturb-seq) as source-to-target lists converted to a gene-by-gene binary graph
- Gene expression dataset (e.g. MCalla et al. or Yan / E-GEOD-36552 ESC datasets)

## Outputs

- Per-head coefficient vector from the RidgeClassifier used to weight attention heads
- Weighted genome-wide attention matrix interpreted as an inferred gene network (rows = targets, columns = regulators)
- AUPRC and EPR scores against the gold-standard network

## How to apply

After computing scPRINT's Keys and Queries embeddings averaged over a set of cells (e.g. 1024 cells over the 5000 most variable genes) to obtain per-head attention matrices, fit a scikit-learn RidgeClassifier on the ground-truth edges with an L2 penalty set to 1, a positivity constraint on the coefficients, and no intercept; the learned per-head coefficients act as weights that replace uniform averaging of attention heads. This lets cells' evidence from ChIP-seq or perturb-seq ground truths steer head contribution, then the weighted matrix is evaluated against the gold standard. Evaluate with EPR (diagnostic odds ratio at K = number of ground-truth positives) and AUPRC (area floored at positive-class prevalence, no interpolation to perfect recall) using the BenGRN/GRnnData packages.

## Related tools

- **scPRINT** (Foundation model whose per-head Keys/Queries attention embeddings over cells provide the candidate head matrices to be weighted) — https://github.com/cantinilab/scPRINT
- **scikit-learn** (Provides the RidgeClassifier (L2 penalty 1, positivity constraint, no intercept) fit on ground-truth edges to select heads)
- **BenGRN** (Benchmarking package used to compute EPR and AUPRC of the head-selected network against gold standards) — https://github.com/jkobject/benGRN
- **GRnnData** (Gene regulatory network data structure used with BenGRN for network evaluation) — https://github.com/cantinilab/GRnnData

## Examples

```
from sklearn.linear_model import RidgeClassifier; clf = RidgeClassifier(alpha=1, positive=True, fit_intercept=False); clf.fit(head_features, edge_labels); weights = clf.coef_
```

## Evaluation signals

- EPR near 1 indicates random recovery; values above 1 indicate better-than-random recovery of gold-standard edges.
- The weighted network should outperform the uniformly averaged scPRINT attention matrix on AUPRC and EPR against the MCalla et al. ChIP-seq/perturb-seq ground truths.
- Orientation must respect the convention that row elements are targets and column elements are regulators (non-transposed orientation for ChIP-seq/perturb-seq ground truths).
- RidgeClassifier is fit with L2 penalty = 1, positive=True, fit_intercept=False — coefficients must be non-negative.

## Limitations

- Requires labeled ground-truth edges, which remain scarce and ambiguous (ChIP-seq, perturb-seq, and literature-based ground truths).
- No perfect ground truths exist for gene network inference benchmarking, so head selection optimizes toward an imperfect proxy of true regulation.
- Head selection injects supervision not available to zero-shot baselines, so results are not directly comparable to unsupervised attention averaging.

## Evidence

- [methods] heads selection via RidgeClassifier on ground truth edges: "we use a linear classifier RidgeClassifier from scikit-learn130 (with an L2 penalty set to 1, a positivity constraint on the coefficients"
- [methods] gene network extraction from averaged attention matrices: [evidence span withheld — anchor status: not_found]
- [abstract] scPRINT outperforms state of the art on cell type-specific benchmarks: "scPRINT demonstrates superior performance in gene network inference to the state of the art"
- [discussion] ground truths scarce: "ChIP-seq, perturb-seq, and literature-based ground truths remain scarce and ambiguous"
