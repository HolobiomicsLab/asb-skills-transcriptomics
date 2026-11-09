---
name: lbfgs-dual-variable-optimization
description: 'Use when implementing or reproducing Mowgli''s alternating minimization:
  nonnegative factors H(p) and shared embedding W must satisfy simplex constraints
  while minimizing the entropic OT-regularized objective (Eqs. 5-11).'
when_to_use_negative:
- The input is unpaired (unmatched) multi-omics data — Mowgli is designed for paired
  single-cell multi-omics integration
- GPU resources are unavailable and the dataset is large — optimal transport is inherently
  expensive to compute and GPU availability is required for Mowgli to scale
- The goal requires batch correction across batches — Mowgli does not handle batch
  correction when integrating paired multi-omics data
edam_operation: http://edamontology.org/operation_3935
edam_topics:
- http://edamontology.org/topic_3316
- http://edamontology.org/topic_2229
tools:
- name: Mowgli
  role: Provides the model whose alternating L-BFGS dual optimization over G(p) and
    W is implemented; trained via model.train(mdata)
  repo: github.com/cantinilab/mowgli
- name: PyTorch
  role: Framework for the GPU-enabled Sinkhorn computation of the entropic-regularized
    OT term and differentiable L-BFGS dual minimization
- name: Muon
  role: Loads the paired Multiome data (mu.load_h5mu) into a MuData object whose obsm['W_OT']
    and uns fields receive the optimization outputs
- name: Scanpy
  role: scverse tool used alongside Muon for preprocessing and downstream analysis
    of the learned embedding
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1038/s41467-023-43019-2
    title: Paired single-cell multi-omics data integration with Mowgli
  source_content_hash: sha256:c8bd830410d0a63112d80578c60a299575f73367a56785cdd10aa515da5cab5e
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - Mowgli
  - Muon
  - PyTorch
  - Python
  - Scanpy
  edam_operation: http://edamontology.org/operation_3935
  edam_topics:
  - http://edamontology.org/topic_3316
  - http://edamontology.org/topic_2229
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8181818181818182
n_claims: 11
---

# lbfgs-dual-variable-optimization

## Summary

Solves the smooth dual problems of Mowgli's entropic-regularized integrative NMF by minimizing over dual variables G(p) and dual W with L-BFGS, then recovering simplex-constrained H(p) and the shared cell embedding W via column-wise softmax transformations.

## When to use

Use when implementing or reproducing Mowgli's alternating minimization: nonnegative factors H(p) and shared embedding W must satisfy simplex constraints while minimizing the entropic OT-regularized objective (Eqs. 5-11).

## When NOT to use

- The input is unpaired (unmatched) multi-omics data — Mowgli is designed for paired single-cell multi-omics integration
- GPU resources are unavailable and the dataset is large — optimal transport is inherently expensive to compute and GPU availability is required for Mowgli to scale
- The goal requires batch correction across batches — Mowgli does not handle batch correction when integrating paired multi-omics data

## Inputs

- Paired multi-omics count matrices in a Muon MuData object (e.g., RNA, ADT, ATAC modalities loaded from a .h5mu file)
- Data-driven ground cost matrices C_kl = 1 - u_k.u_l/(||u_k|| ||u_l||) built from pairwise cosine distances between feature rows (Eq. 3)
- Hyperparameters: entropic OT regularization epsilon, sparsity penalties rho_tilde per modality (RNA/ADT 0.01, ATAC 0.1), mu_tilde = 0.001, number of latent dimensions k (15 for the demo dataset)

## Outputs

- Shared cell embedding W (n_cells x k), satisfying simplex constraints, stored in mdata.obsm['W_OT']
- Modality-specific dictionaries H(p) with columns summing to 1, stored in mdata.uns
- Saved model checkpoint (e.g., mowgli_model.pt)

## How to apply

Alternate between two L-BFGS minimizations of the loss (Eq. 5). For each modality's dictionary H(p), minimize the smooth dual problem (Eq. 7) over dual variables, then recover H(p) by column-wise softmax of -G(p)W/rho_p (Eqs. 8-9). For the shared embedding W, minimize the dual problem (Eq. 10) and recover W by softmax of (1/(d*mu)) * sum_p H(p)' G(p) (Eq. 11). Enforce simplex constraints (columns of H(p) and W sum to 1) and entropic sparsity penalties with rho_tilde_rna = rho_tilde_adt = 0.01, rho_tilde_atac = 0.1, mu_tilde = 0.001. The rationale is that optimizing the dual variables yields unconstrained smooth objectives amenable to L-BFGS, while the softmax maps back to the probability simplex, guaranteeing nonnegativity without projection. The entropic OT term inside the objective is computed with the fast, GPU-enabled Sinkhorn algorithm in PyTorch. For the 10X Multiome demo dataset, use 15 latent dimensions and train via model.train(mdata), storing W in mdata.obsm['W_OT'].

## Related tools

- **Mowgli** (Provides the model whose alternating L-BFGS dual optimization over G(p) and W is implemented; trained via model.train(mdata)) — github.com/cantinilab/mowgli
- **PyTorch** (Framework for the GPU-enabled Sinkhorn computation of the entropic-regularized OT term and differentiable L-BFGS dual minimization)
- **Muon** (Loads the paired Multiome data (mu.load_h5mu) into a MuData object whose obsm['W_OT'] and uns fields receive the optimization outputs)
- **Scanpy** (scverse tool used alongside Muon for preprocessing and downstream analysis of the learned embedding)

## Examples

```
import mowgli; model = mowgli.model.MowgliModel(d=15, rho_meth=0.01, rho_adt=0.01, rho_atac=0.1, mu=0.001); model.train(mdata); W = mdata.obsm['W_OT']
```

## Evaluation signals

- mdata.obsm['W_OT'] has shape n_cells x 15 for the demo dataset
- Rows of W_OT and columns of each H(p) respect the simplex constraint (sum to 1) after the softmax recovery steps
- The learned embedding yields competitive silhouette score, ARI, and purity score relative to the paper's benchmark against Seurat v4, MOFA+, integrative NMF, Cobolt, and Multigrate
- The alternating loop monotonically decreases the objective (Eq. 5), with the entropic OT term correctly computed via Sinkhorn
- Recovered factors support interpretable downstream steps (GSEA on top 150 genes per factor, motif enrichment on top 100 peaks per factor), indicating non-degenerate dictionaries

## Limitations

- The number of latent dimensions k is not chosen automatically — several values of k should be tested since Mowgli offers no straightforward way to define it
- Optimal transport is inherently expensive to compute compared to Euclidean distance; GPU availability is required for Mowgli to scale to larger datasets like BM CITE (29,803 cells)
- Mowgli does not handle batch correction when integrating paired multi-omics data
- The alternating H(p)/W minimization procedure itself is a description of the algorithm rather than a validated benchmark result

## Evidence

- [methods] Alternating optimization of H and W with L-BFGS: "we alternate between minimizing (Eq. 5) on H p ð Þ and W"
- [methods] Entropic regularization via Sinkhorn: [evidence span withheld — anchor status: not_found]
- [methods] Model decomposition structure: [evidence span withheld — anchor status: not_found]
- [other] Sparsity penalty values for demo: [evidence span withheld — anchor status: not_found]
- [readme] scverse integration: "Mowgli is implemented as a Python package seamlessly integrated within the scverse ecosystem, in particular Muon and Scanpy."
- [discussion] Latent dimension selection limitation: "it does not offer a straightforward approach to de fi ning the number of latent dimensions"
