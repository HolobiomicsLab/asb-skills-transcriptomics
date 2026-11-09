---
name: cross-batch-consistency-validation
description: 'Use when a low-dimensional embedding claimed to remove batch effects
  must be validated: compute pseudotime or another per-cell quantity per batch and
  test cross-batch consistency via regional KL divergence of batch mixing.'
when_to_use_negative:
- Technical differences across batches exceed true biological variations — DESC's
  batch removal and hence this validation rely on the assumption that technical differences
  are smaller than biological ones
- The dataset has only one processing batch, so no cross-batch KL divergence or consistency
  check is possible
- The input is raw count data not yet normalized (UMI normalization to 10,000 and
  log transform, gene standardization, and highly variable gene selection via filter_genes_dispersion
  should precede embedding)
edam_operation: http://edamontology.org/operation_2935
edam_topics:
- http://edamontology.org/topic_2229
tools:
- name: DESC
  role: Provides the batch-corrected low-dimensional embedding (X_Embeded_z in adata.obsm)
    whose cross-batch consistency is being validated
  repo: https://github.com/eleozzr/desc
- name: Scanpy
  role: Preprocessing (filter_genes_dispersion for highly variable genes, log1p, scale)
    and containerization of embedding and batch labels in AnnData
  repo: https://github.com/scverse/scanpy
- name: Monocle3
  role: Builds the trajectory and assigns pseudotime to every cell on the DESC embedding
    for cross-batch comparison
- name: Keras
  role: Backend used to train the DESC deep embedding model that produces the representation
    under validation
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1038/s41467-020-15851-3
    title: Deep learning enables accurate clustering with batch effect removal in
      single-cell RNA-seq analysis
  source_content_hash: sha256:03728bf4d73da444a1a9e1246171c51815491f343f05173395d11c05f9035d55
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/eleozzr/desc
  tools:
  - DESC
  - Keras
  - Monocle3
  - Scanpy
  edam_operation: http://edamontology.org/operation_2935
  edam_topics:
  - http://edamontology.org/topic_2229
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7777777777777778
n_claims: 9
---

# cross-batch-consistency-validation

## Summary

Validates batch effect removal in an embedding (e.g., the DESC X_Embeded_z representation) by checking that downstream quantities such as Monocle3 pseudotime have consistent distributions across processing batches, quantified with regional batch-mixing KL divergence.

## When to use

Use when a low-dimensional embedding claimed to remove batch effects must be validated: compute pseudotime or another per-cell quantity per batch and test cross-batch consistency via regional KL divergence of batch mixing.

## When NOT to use

- Technical differences across batches exceed true biological variations — DESC's batch removal and hence this validation rely on the assumption that technical differences are smaller than biological ones
- The dataset has only one processing batch, so no cross-batch KL divergence or consistency check is possible
- The input is raw count data not yet normalized (UMI normalization to 10,000 and log transform, gene standardization, and highly variable gene selection via filter_genes_dispersion should precede embedding)

## Inputs

- Low-dimensional cell embedding in adata.obsm (DESC X_Embeded_z) for 10,878 human monocyte cells across three processing batches
- Per-cell batch labels
- Raw marker gene expression values (e.g., S100A8) per cell

## Outputs

- Per-cell pseudotime values from a Monocle3 trajectory built on the DESC embedding
- Box plot of regional batch-mixing KL divergence values across 200 iterations for each pair of batches
- Smoothed S100A8 expression-versus-pseudotime curves, one per batch

## How to apply

Load the embedding stored in adata.obsm (e.g., the DESC X_Embeded_z representation for the 10,878-cell three-batch human monocyte dataset) together with per-cell batch labels. Build a Monocle3 trajectory on the embedding and assign a pseudotime to every cell. For each pair of batches, compute the regional batch-mixing KL divergence of the pseudotime distributions using K nearest neighbors with K = 5 x number of batches, repeating the procedure for 200 iterations over randomly chosen cells, then generate box plots of the KL divergence values. Additionally plot the smoothed expression of a marker gene (S100A8) versus pseudotime per batch and check for a monotone increase in each batch. Accept the validation when the pseudotime KL divergence is small across all batches and marker curves agree, confirming that DESC's iterative self-learning removed technical differences while preserving biological variation.

## Related tools

- **DESC** (Provides the batch-corrected low-dimensional embedding (X_Embeded_z in adata.obsm) whose cross-batch consistency is being validated) — https://github.com/eleozzr/desc
- **Scanpy** (Preprocessing (filter_genes_dispersion for highly variable genes, log1p, scale) and containerization of embedding and batch labels in AnnData) — https://github.com/scverse/scanpy
- **Monocle3** (Builds the trajectory and assigns pseudotime to every cell on the DESC embedding for cross-batch comparison)
- **Keras** (Backend used to train the DESC deep embedding model that produces the representation under validation)

## Examples

```
K = 5 * adata.obs['batch'].nunique(); for it in range(200): kl = regional_batch_mixing_kl(pseudotime, adata.obs['batch'], K=K)  # box plot KL across batches
```

## Evaluation signals

- Regional batch-mixing KL divergence of pseudotime distributions is small for all three pairs of batches (K = 5 x number of batches, 200 random iterations)
- Box plots of KL divergence across iterations show consistently low values with no outlier batch
- Smoothed S100A8 expression increases monotonically over pseudotime in each of the three batches
- Pseudotime distributions overlap across batches, indicating batch effect was successfully removed

## Limitations

- DESC removes batch effects only as long as technical differences across batches are smaller than true biological variations
- The Student's t distribution kernel was found more robust than a Gaussian kernel for the soft assignment underlying the embedding being validated
- Desc requires python 3.5.x/3.6.x and tensorflow < 2.0 (desc 2.0.3) to reproduce the paper's results; versions for Keras and Scanpy are not stated

## Evidence

- [discussion] Batch effect removal evaluation via regional KL divergence of batch mixing (K nearest neighbors, K = 5 × number of batches, 200 iterations): [evidence span withheld — anchor status: not_found]
- [discussion] We repeated this procedure for 200 iterations with different randomly chosen cells to generate box plots of the final KL divergence.: "We repeated this procedure for 200 iterations with different randomly chosen cells to generate box plots of the final KL divergence."
- [abstract] Through iterative self-learning, DESC gradually removes batch effects, as long as technical differences across batches are smaller than true biological variations: "Through iterative self-learning, DESC gradually removes batch effects, as long as technical differences across batches are smaller than true biological variations"
- [abstract] DESC cluster assignment probabilities are biologically interpretable and can reveal both discrete and pseudotemporal structure of cells: "As a soft clustering algorithm, cluster assignment probabilities from DESC are biologically interpretable and can reveal both discrete and pseudotemporal structure of cells"
- [readme] This iterative procedure moves each cell to its nearest cluster, balances biological and technical differences between clusters, and reduces the influence of batch effect.: "This iterative procedure moves each cell to its nearest cluster, balances biological and technical differences between clusters, and reduces the influence of batch effect."
