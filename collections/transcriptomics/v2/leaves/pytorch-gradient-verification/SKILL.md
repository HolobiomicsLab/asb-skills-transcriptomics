---
name: pytorch-gradient-verification
description: 'Use when reproducing or extending the scDCC loss module: you need to
  reconstruct the soft pairwise constraint terms (must-link/cannot-link) added to
  a ZINB deep embedded clustering objective and verify their gradient behavior numerically.'
when_to_use_negative:
- When the goal is hard-constrained clustering (e.g. COP K-means) that forbids violations
  rather than penalizing them softly
- When the input is not a soft cluster assignment distribution q — the constraint
  losses require pairwise joint assignment probabilities sum_j q_aj*q_bj
- When prior knowledge is limited to marker genes and predefined cell types (CellAssign/SCINA-style),
  which scDCC's pairwise constraint mechanism was designed to go beyond
edam_operation: http://edamontology.org/operation_3432
edam_topics:
- http://edamontology.org/topic_3474
- http://edamontology.org/topic_3308
tools:
- name: scDCC
  role: Reference implementation whose constraint loss module and joint objective
    (ZINB + clustering + constraint) are being reconstructed and verified
  repo: https://github.com/ttgump/scDCC
- name: PyTorch 1.5.1+cu101
  role: Framework in which the autoencoder, soft assignments q, and constraint losses
    are instantiated and differentiated via gradient descent
  repo: https://pytorch.org
- name: Scanpy 1.0.4
  role: Normalization of the scRNA-seq count batch to produce autoencoder inputs
  repo: https://scanpy.readthedocs.io/en/stable
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1038/s41467-021-22008-3
    title: Model-based deep embedding for constrained clustering analysis of single
      cell RNA-seq data
  source_content_hash: sha256:e2e1a0b84cada624bed61e089a7a89fd37724ce0a41b8dc363bcc60dc3d73cae
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/ttgump/scDCC
  tools:
  - PyTorch
  - PyTorch 1.5.1+cu101
  - Python
  - Scanpy
  - Scanpy 1.0.4
  - scDCC
  edam_operation: http://edamontology.org/operation_3432
  edam_topics:
  - http://edamontology.org/topic_3474
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 6
---

# pytorch-gradient-verification

## Summary

Implement and verify the soft must-link/cannot-link constraint loss module of scDCC in PyTorch 1.5.1, composing L_ML = -sum log(sum_j q_aj*q_bj) and L_CL = -sum log(1 - sum_j q_aj*q_bj) with the ZINB reconstruction and clustering losses, then confirm via gradient descent that violated pairs incur higher penalties and the joint objective decreases.

## When to use

Use when reproducing or extending the scDCC loss module: you need to reconstruct the soft pairwise constraint terms (must-link/cannot-link) added to a ZINB deep embedded clustering objective and verify their gradient behavior numerically.

## When NOT to use

- When the goal is hard-constrained clustering (e.g. COP K-means) that forbids violations rather than penalizing them softly
- When the input is not a soft cluster assignment distribution q — the constraint losses require pairwise joint assignment probabilities sum_j q_aj*q_bj
- When prior knowledge is limited to marker genes and predefined cell types (CellAssign/SCINA-style), which scDCC's pairwise constraint mechanism was designed to go beyond

## Inputs

- Small batch of raw scRNA-seq count data from the scDCC_data figshare deposit (e.g. 10X CITE PBMC, human liver)
- Synthetic must-link and cannot-link cell-pair indices
- Soft cluster assignment distributions q from the denoising ZINB autoencoder (hidden sizes 256-64-32-64-256)
- Constraint weights (ml_weight, cl_weight) and clustering-loss weight gamma

## Outputs

- Per-pair constraint loss values (L_ML, L_CL) for must-link and cannot-link pairs
- Combined loss trajectory over gradient-descent iterations (L_ZINB + gamma*L_clustering + gamma'*L_constraint)
- test_constraint_losses.py pass/fail output with printed loss values

## How to apply

Load a small batch of scRNA-seq counts from the figshare scDCC_data deposit and normalize it with Scanpy 1.0.4 to feed a denoising ZINB autoencoder (hidden sizes 256-64-32-64-256) in PyTorch 1.5.1, computing soft cluster assignment distributions q. Implement the must-link loss L_ML = -sum log(sum_j q_aj*q_bj) and cannot-link loss L_CL = -sum log(1 - sum_j q_aj*q_bj), weighted by ml_weight and cl_weight as exposed by the --ml_weight and --cl_weight flags of the reference wrappers. Compose the joint objective L = L_ZINB + gamma*L_clustering + gamma'*L_constraint with gamma = gamma' = 1, mirroring the --gamma clustering-loss weight. Rationale: scDCC converts partial prior knowledge into soft pairwise constraints added as additional terms to the loss, unlike hard-constrained clustering such as COP K-means; during the clustering stage the ZINB and clustering losses are optimized per batch of data points while the constraint loss is optimized per batch of constraints.

## Related tools

- **scDCC** (Reference implementation whose constraint loss module and joint objective (ZINB + clustering + constraint) are being reconstructed and verified) — https://github.com/ttgump/scDCC
- **PyTorch 1.5.1+cu101** (Framework in which the autoencoder, soft assignments q, and constraint losses are instantiated and differentiated via gradient descent) — https://pytorch.org
- **Scanpy 1.0.4** (Normalization of the scRNA-seq count batch to produce autoencoder inputs) — https://scanpy.readthedocs.io/en/stable

## Examples

```
import torch; p = torch.sum(q[a]*q[b], dim=1); L_ml = -torch.log(p).sum(); L_cl = -torch.log(1-p).sum(); L = L_zinb + L_cluster + L_ml + L_cl; L.backward(); optimizer.step()
```

## Evaluation signals

- test_constraint_losses.py runs without error (script_runs)
- The combined loss value monotonically decreases during gradient-descent optimization
- The constraint term for a violated pair exceeds that of a satisfied pair (value_in_range / contains_substring checks on printed loss values)
- Constraint losses are optimized per batch of constraints while ZINB and clustering losses are optimized per batch of data points
- Composition matches the reference flags: gamma = gamma' = 1 with weights mirroring --ml_weight and --cl_weight

## Limitations

- The verification uses small synthetic batches of must-link/cannot-link pairs, so gradient behavior may not reflect full-dataset optimization dynamics
- The mechanism only integrates pairwise prior knowledge as soft penalties; it cannot enforce hard constraints and requires tuning ml_weight/cl_weight
- Hidden sizes 256-64-32-64-256 and gamma = gamma' = 1 mirror the reference wrappers; other architectures or weights are untested by this verification

## Evidence

- [intro] soft constraint loss terms: "Unlike traditional hard-constrained clustering algorithms (e.g., COP K-means36), we convert (partial) prior knowledge into soft pairwise constraints and add them as additional terms into the loss"
- [methods] PyTorch version used: "pytorch -- 1.5.1+cu101 (https://pytorch.org)"
- [methods] Scanpy version used: "Scanpy --- 1.0.4 (https://scanpy.readthedocs.io/en/stable)"
- [readme] scDCC README context: "Clustering is a critical step in single cell-based studies. Most existing methods support unsupervised clustering without the a priori exploitation of any domain knowledge."
- [intro] Constraint information integrated via loss: "Here, we name the proposed model-based deep embedding clustering method as scDCC (Single Cell Deep Constrained Clustering)."
