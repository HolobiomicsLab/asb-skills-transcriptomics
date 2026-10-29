---
name: alternating-dictionary-embedding-minimization
description: Use when factorizing paired multi-omics count matrices (e.g., 10X Multiome
  RNA/ATAC, CITE-seq ADT) into simplex-constrained dictionaries H(p) and a shared
  low-dimensional cell embedding W with entropic OT coupling.
when_to_use_negative:
- Data are unpaired (unmatched) multi-omics samples — Mowgli is designed for paired
  multi-omics integration with any type and number of omics
- Batch correction across integrated datasets is required — the paper notes Mowgli
  does not handle batch correction when integrating paired multi-omics data
- GPU resources are unavailable and the dataset is large — OT is inherently expensive
  to compute compared to Euclidean distance, and GPU availability is required for
  Mowgli to scale to the larger datasets presented
edam_operation: http://edamontology.org/operation_3935
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_3673
tools:
- name: Mowgli
  role: Provides the model class and train() routine implementing the alternating
    H(p)/W minimization with dual L-BFGS steps and softmax recovery
  repo: https://github.com/cantinilab/mowgli
- name: PyTorch
  role: Backend for the fast, GPU-enabled Sinkhorn algorithm computing the entropic-regularized
    OT objective and the differentiable loss
- name: Muon
  role: Holds the paired multi-omics MuData object whose modalities are decomposed;
    results stored in mdata.obsm['W_OT'] and mdata.uns
- name: Scanpy
  role: scverse preprocessing/downstream analysis companion for the Mowgli embedding
    within the scverse ecosystem
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1038/s41467-023-43019-2
    title: Paired single-cell multi-omics data integration with Mowgli
  source_content_hash: sha256:001d2ae3df103fd990ee5e8b6af172ac639124e04235802c24872cbc71530747
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/cantinilab/mowgli
  tools:
  - Mowgli
  - Muon
  - PyTorch
  - Python
  - Scanpy
  edam_operation: http://edamontology.org/operation_3935
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_3673
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8181818181818182
n_claims: 11
---

# alternating-dictionary-embedding-minimization

## Summary

Mowgli's core optimization: alternating minimization of the entropic-regularized OT loss (Eq. 5) between modality-specific dictionary matrices H(p) and the shared cell embedding W, solved via L-BFGS on smooth dual problems with softmax recovery of the primal variables.

## When to use

Use when factorizing paired multi-omics count matrices (e.g., 10X Multiome RNA/ATAC, CITE-seq ADT) into simplex-constrained dictionaries H(p) and a shared low-dimensional cell embedding W with entropic OT coupling.

## When NOT to use

- Data are unpaired (unmatched) multi-omics samples — Mowgli is designed for paired multi-omics integration with any type and number of omics
- Batch correction across integrated datasets is required — the paper notes Mowgli does not handle batch correction when integrating paired multi-omics data
- GPU resources are unavailable and the dataset is large — OT is inherently expensive to compute compared to Euclidean distance, and GPU availability is required for Mowgli to scale to the larger datasets presented

## Inputs

- Muon MuData object with paired modality-specific count matrices (e.g., .h5mu from 10X Multiome, CITE-seq, TEA-seq)
- Data-driven cost matrices per modality (pairwise cosine distances between feature rows)
- Hyperparameters: number of latent dimensions (15 for demo), ρ̃_rna = ρ̃_adt = 0.01, ρ̃_atac = 0.1, μ̃ = 0.001, entropic regularization ε

## Outputs

- Shared cell embedding W (n_cells × k, k=15 for demo) in mdata.obsm['W_OT'] with rows on the simplex
- Modality-specific dictionary matrices H(p) with columns summing to 1, stored in mdata.uns
- Model checkpoint file mowgli_model.pt

## How to apply

For each modality, build the ground cost matrix from pairwise cosine distances between feature rows (C_kl = 1 - u_k·u_l/(||u_k|| ||u_l||), Eq. 3), and precompute the entropic-regularized OT distance OT_eps(a,b) = min_P Σ P_kl C_kl - ε E(P) (Eq. 4) with the GPU-enabled Sinkhorn algorithm in PyTorch. Then alternate two L-BFGS minimizations of the loss (Eq. 5): for H(p), solve the smooth dual problem (Eq. 7) and recover H(p) by column-wise softmax of -G(p)W/ρ_p (Eqs. 8-9); for W, minimize the dual (Eq. 10) and recover W by softmax of (1/(dμ)) Σ_p H(p)' G(p) (Eq. 11). Enforce simplex constraints (columns of H(p) and W sum to 1) with entropic sparsity penalties ρ̃_rna = ρ̃_adt = 0.01, ρ̃_atac = 0.1, and μ̃ = 0.001; use 15 latent dimensions for the demo dataset. Train with model.train(mdata), store W in mdata.obsm['W_OT'] and dictionaries in mdata.uns, then save the checkpoint as mowgli_model.pt.

## Related tools

- **Mowgli** (Provides the model class and train() routine implementing the alternating H(p)/W minimization with dual L-BFGS steps and softmax recovery) — https://github.com/cantinilab/mowgli
- **PyTorch** (Backend for the fast, GPU-enabled Sinkhorn algorithm computing the entropic-regularized OT objective and the differentiable loss)
- **Muon** (Holds the paired multi-omics MuData object whose modalities are decomposed; results stored in mdata.obsm['W_OT'] and mdata.uns)
- **Scanpy** (scverse preprocessing/downstream analysis companion for the Mowgli embedding within the scverse ecosystem)

## Examples

```
from mowgli.model import MOWGLI; model = MOWGLI(n_dimensions=15, rho_mrna=0.01, rho_adt=0.01, rho_atac=0.1, mu=0.001); model.train(mdata); mdata.obsm['W_OT']
```

## Evaluation signals

- mdata.obsm['W_OT'] has shape n_cells × 15 for the demo dataset and each row sums to 1 (simplex constraint respected)
- Columns of each H(p) sum to 1 and the entropic sparsity penalties (ρ̃_rna = ρ̃_adt = 0.01, ρ̃_atac = 0.1, μ̃ = 0.001) are applied
- Alternating loss (Eq. 5) decreases monotonically across L-BFGS iterations of H(p) and W updates
- Resulting embedding achieves competitive silhouette score, ARI, and purity score as used in the paper's benchmark against Seurat v4, MOFA+, integrative NMF, Cobolt, and Multigrate

## Limitations

- Mowgli does not offer a straightforward approach to defining the number of latent dimensions (k); several values of k should be tested
- Mowgli does not handle batch correction when integrating paired multi-omics data
- Optimal transport is inherently expensive to compute compared to the Euclidean distance; GPU availability is required for Mowgli to scale to larger datasets

## Evidence

- [methods] Alternating optimization of H and W with L-BFGS: "we alternate between minimizing (Eq. 5) on H p ð Þ and W"
- [methods] Entropic regularization via Sinkhorn: [evidence span withheld — anchor status: not_found]
- [methods] Model decomposition definition: [evidence span withheld — anchor status: not_found]
- [readme] scverse integration: "Mowgli is implemented as a Python package seamlessly integrated within the scverse ecosystem, in particular Muon and Scanpy."
