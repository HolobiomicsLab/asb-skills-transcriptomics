---
name: zinb-autoencoder-loss-composition
description: Use when clustering scRNA-seq count data with partial pairwise prior
  knowledge (must-link/cannot-link pairs) that should softly shape the latent embedding
  instead of hard-constrained clustering like COP K-means.
when_to_use_negative:
- Prior knowledge is limited to marker genes and predefined cell types, which is the
  setting of CellAssign/SCINA rather than soft pairwise constraints
- A hard-constrained clustering algorithm is explicitly required to satisfy every
  constraint exactly (scDCC treats constraints as soft loss terms, not hard rules)
- Input is not raw scRNA-seq counts requiring ZINB dropout modeling (the ZINB loss
  term presupposes a count structure with overdispersion and sparsity)
edam_operation: http://edamontology.org/operation_3432
edam_topics:
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_0203
tools:
- name: scDCC
  role: Reference framework implementing the ZINB autoencoder, soft cluster assignment
    q, and the composed loss with --gamma, --ml_weight, --cl_weight flags
  repo: https://github.com/ttgump/scDCC
- name: PyTorch
  role: Framework in which the denoising ZINB autoencoder and the gradient-based optimization
    of the composed loss are implemented (version 1.5.1+cu101)
- name: Scanpy
  role: Preprocessing library (version 1.0.4) used to normalize scRNA-seq counts into
    autoencoder inputs
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1038/s41467-021-22008-3
    title: Model-based deep embedding for constrained clustering analysis of single
      cell RNA-seq data
  source_content_hash: sha256:18fd4e616831f48f4e3bc60457a9e3225513e1e642f6cd82e8dc4f5939701cee
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/ttgump/scDCC
  tools:
  - PyTorch
  - Python
  - Scanpy
  - scDCC
  edam_operation: http://edamontology.org/operation_3432
  edam_topics:
  - http://edamontology.org/topic_3308
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.75
n_claims: 12
---

# zinb-autoencoder-loss-composition

## Summary

Composes the joint scDCC objective for single-cell RNA-seq constrained clustering: a ZINB reconstruction loss from a denoising autoencoder (hidden sizes 256-64-32-64-256), a DEC-style clustering loss, and a soft pairwise constraint loss (must-link/cannot-link), combined as L = L_ZINB + gamma*L_clustering + gamma'*L_constraint with gamma = gamma' = 1. This lets partial prior knowledge guide deep clustering without hard-constrained assignments.

## When to use

Use when clustering scRNA-seq count data with partial pairwise prior knowledge (must-link/cannot-link pairs) that should softly shape the latent embedding instead of hard-constrained clustering like COP K-means.

## When NOT to use

- Prior knowledge is limited to marker genes and predefined cell types, which is the setting of CellAssign/SCINA rather than soft pairwise constraints
- A hard-constrained clustering algorithm is explicitly required to satisfy every constraint exactly (scDCC treats constraints as soft loss terms, not hard rules)
- Input is not raw scRNA-seq counts requiring ZINB dropout modeling (the ZINB loss term presupposes a count structure with overdispersion and sparsity)

## Inputs

- Raw scRNA-seq count matrix (e.g., 10X CITE PBMC dataset or human liver dataset) from the figshare scDCC_data deposit
- Scanpy-normalized expression inputs for the autoencoder
- Soft cluster assignment distributions q from the ZINB autoencoder's latent space
- Must-link (a, b) and cannot-link (a, b) cell pairs derived from prior knowledge

## Outputs

- Composed total loss L = L_ZINB + gamma*L_clustering + gamma'*L_constraint per training batch
- Per-pair constraint penalty values (L_ML and L_CL terms) for monitoring optimization
- Trained ZINB autoencoder embedding with clusters shaped by soft pairwise constraints

## How to apply

Load scRNA-seq counts (e.g., from the figshare scDCC_data deposit) and normalize with Scanpy 1.0.4 before feeding a denoising ZINB autoencoder (encoder/decoder hidden sizes 256-64-32-64-256) in PyTorch 1.5.1. Compute soft cluster assignment distributions q from the embedded latent space. Implement the must-link loss L_ML = -sum log(sum_j q_aj*q_bj) and the cannot-link loss L_CL = -sum log(1 - sum_j q_aj*q_bj), weighted by ml_weight and cl_weight as exposed by the reference wrappers' --ml_weight and --cl_weight flags. Compose the total loss L = L_ZINB + gamma*L_clustering + gamma'*L_constraint with gamma = gamma' = 1 (mirroring the --gamma flag), then optimize ZINB and clustering losses per batch of data points while the constraint loss is optimized per batch of constraints. Verify that the combined loss decreases monotonically over gradient-descent steps and that violated pairs incur higher constraint penalty than satisfied pairs.

## Related tools

- **scDCC** (Reference framework implementing the ZINB autoencoder, soft cluster assignment q, and the composed loss with --gamma, --ml_weight, --cl_weight flags) — https://github.com/ttgump/scDCC
- **PyTorch** (Framework in which the denoising ZINB autoencoder and the gradient-based optimization of the composed loss are implemented (version 1.5.1+cu101))
- **Scanpy** (Preprocessing library (version 1.0.4) used to normalize scRNA-seq counts into autoencoder inputs)

## Examples

```
python scDCC_pairwise_CITE_PBMC.py --n_pairwise <N> --gamma 1 --ml_weight <w_ml> --cl_weight <w_cl>
```

## Evaluation signals

- test_constraint_losses.py runs without error (script_runs)
- The combined loss value monotonically decreases during gradient-descent optimization
- The constraint term for a violated pair exceeds that of a satisfied pair (value_in_range / contains_substring checks on printed loss values)
- Gamma weights match the paper setting gamma = gamma' = 1, mirroring the --gamma flag of the clustering loss
- Soft assignment distributions q are produced from the 256-64-32-64-256 hidden-size autoencoder before computing constraint losses

## Limitations

- Prior-knowledge alternatives like CellAssign and SCINA are limited to marker genes and lack flexibility for other kinds of prior information; scDCC's pairwise constraints are the alternative, but partial prior knowledge is assumed
- Purely unsupervised clustering (the loss without the constraint term) may not produce biologically interpretable clusters on scRNA-Seq data with pervasive dropout
- Constraints are soft penalties weighted by ml_weight/cl_weight; pairs can still end up violating constraints if the weights are set too low

## Evidence

- [intro] soft pairwise constraints added as loss terms: "Unlike traditional hard-constrained clustering algorithms (e.g., COP K-means36), we convert (partial) prior knowledge into soft pairwise constraints and add them as additional terms into the loss"
- [intro] constraint loss combined into a joint deep learning loss: "Here, we name the proposed model-based deep embedding clustering method as scDCC (Single Cell Deep Constrained Clustering)."
- [methods] wrapper flags controlling constraint and clustering loss weights: "--n_pairwise: number of pairwise constraints want to generate"
- [readme] scDCC README framing the clustering step scDCC addresses: "Clustering is a critical step in single cell-based studies. Most existing methods support unsupervised clustering without the a priori exploitation of any domain knowledge."
- [abstract] improvement claim motivating the loss composition: "Experiments on various scRNA-seq datasets from thousands to tens of thousands of cells show that scDCC can significantly improve clustering performance"
