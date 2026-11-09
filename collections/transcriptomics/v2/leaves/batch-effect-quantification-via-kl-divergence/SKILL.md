---
name: batch-effect-quantification-via-kl-divergence
description: Use when per-cell batch labels and a low-dimensional representation (e.g.,
  DESC's X_Embeded_z in adata.obsm) or derived quantities such as Monocle3 pseudotime
  are available, and you need a numeric, distribution-level measure of whether batches
  are well mixed.
when_to_use_negative:
- Technical differences across batches are larger than true biological variation —
  DESC's iterative self-learning only removes batch effects 'as long as technical
  differences across batches are smaller than true biological variations'
- No per-cell batch labels are available, since the regional batch-mixing KL divergence
  is computed between batch-specific distributions
- The corrected embedding order-dependence matters and a non-pairwise evaluation is
  required on data where pairwise-based correction methods (CCA, MNN, Seurat 3.0,
  Scanorama) were applied
edam_operation: http://edamontology.org/operation_3432
edam_topics:
- http://edamontology.org/topic_0091
- http://edamontology.org/topic_3308
tools:
- name: DESC
  role: Produces the low-dimensional X_Embeded_z representation in adata.obsm whose
    batch mixing is quantified by the regional KL divergence criterion
  repo: https://github.com/eleozzr/desc
- name: Scanpy
  role: Preprocessing framework holding the annotated data (adata) with embedding,
    batch labels, and expression values used in the batch-mixing evaluation
  repo: https://github.com/scverse/scanpy
- name: Monocle3
  role: Builds the trajectory on the DESC embedding whose pseudotime distributions
    across batches are compared via KL divergence
- name: Keras
  role: Backend used to train the DESC model that yields the embedding evaluated for
    batch mixing
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1038/s41467-020-15851-3
    title: Deep learning enables accurate clustering with batch effect removal in
      single-cell RNA-seq analysis
  source_content_hash: sha256:2a8a8599da9de70617d6e6250b5c9e654365a696772de96fecf0687b86f9005f
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/eleozzr/desc
  tools:
  - DESC
  - Keras
  - Monocle3
  - Scanpy
  edam_operation: http://edamontology.org/operation_3432
  edam_topics:
  - http://edamontology.org/topic_0091
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8
n_claims: 10
---

# batch-effect-quantification-via-kl-divergence

## Summary

Quantifies batch effect removal by repeatedly computing regional KL divergence between batch-specific distributions (e.g., pseudotime or embedding neighborhoods) over randomly chosen cells, using K = 5 × number of batches nearest neighbors, and summarizing across 200 iterations as box plots. This is the criterion used in the DESC paper to confirm that the DESC representation removed batch effect in the three-batch 10,878-cell human monocyte dataset.

## When to use

Use when per-cell batch labels and a low-dimensional representation (e.g., DESC's X_Embeded_z in adata.obsm) or derived quantities such as Monocle3 pseudotime are available, and you need a numeric, distribution-level measure of whether batches are well mixed.

## When NOT to use

- Technical differences across batches are larger than true biological variation — DESC's iterative self-learning only removes batch effects 'as long as technical differences across batches are smaller than true biological variations'
- No per-cell batch labels are available, since the regional batch-mixing KL divergence is computed between batch-specific distributions
- The corrected embedding order-dependence matters and a non-pairwise evaluation is required on data where pairwise-based correction methods (CCA, MNN, Seurat 3.0, Scanorama) were applied

## Inputs

- Per-cell low-dimensional embedding (DESC X_Embeded_z stored in adata.obsm) for 10,878 human monocyte cells
- Per-cell processing batch labels (three batches)
- Per-cell pseudotime values from a Monocle3 trajectory built on the DESC embedding
- Raw per-cell gene expression values (e.g., S100A8) for downstream per-batch validation curves

## Outputs

- Pairwise KL divergence values between batch pseudotime distributions, repeated over 200 random iterations
- Box plot of batch-mixing KL divergence values across iterations
- Smoothed per-batch expression-versus-pseudotime curves (e.g., S100A8) used to confirm consistency

## How to apply

Start from a per-cell low-dimensional embedding (the DESC X_Embeded_z representation stored in adata.obsm) plus per-cell batch labels; optionally derive a trajectory quantity such as Monocle3 pseudotime built on the DESC embedding. Apply the regional batch-mixing criterion: for each of 200 iterations, randomly choose cells and find K nearest neighbors with K = 5 × number of batches, then compute the KL divergence between the pseudotime (or embedding) distributions of each pair of processing batches within the region. Aggregate the per-iteration KL divergence values and generate a box plot, as done for the three-batch 10,878-cell human monocyte data. Accept the correction as successful if the batch-mixing KL divergence is small across all batches and a downstream check (e.g., smoothed S100A8 expression versus pseudotime per batch) is consistent across batches. The rationale is that DESC's iterative self-learning gradually removes batch effects as long as technical differences are smaller than true biological variation, so residual batch effect should appear as inflated KL divergence between batch distributions.

## Related tools

- **DESC** (Produces the low-dimensional X_Embeded_z representation in adata.obsm whose batch mixing is quantified by the regional KL divergence criterion) — https://github.com/eleozzr/desc
- **Scanpy** (Preprocessing framework holding the annotated data (adata) with embedding, batch labels, and expression values used in the batch-mixing evaluation) — https://github.com/scverse/scanpy
- **Monocle3** (Builds the trajectory on the DESC embedding whose pseudotime distributions across batches are compared via KL divergence)
- **Keras** (Backend used to train the DESC model that yields the embedding evaluated for batch mixing)

## Examples

```
for it in range(200): idx = np.random.choice(n_cells, size=n_cells//10, replace=False); neighbors = NearestNeighbors(n_neighbors=5*n_batches).fit(adata.obsm['X_Embeded_z']); kl = compute_batch_mixing_kl(pseudotime, batch_labels, neighbors, idx)
```

## Evaluation signals

- Batch-mixing KL divergence of pseudotime distributions is small across all three batches (box plot across 200 iterations shows low values)
- Smoothed S100A8 expression increases monotonically over pseudotime in each of the three batches, matching reported DESC behavior
- The procedure uses K = 5 × number of batches nearest neighbors and 200 iterations with different randomly chosen cells, per the paper's protocol
- Pseudotime distributions across the three processing batches are visually similar when overlaid

## Limitations

- Batch removal only works while technical differences across batches are smaller than true biological variations
- Evaluation is stochastic (200 iterations with random cell choices), so results vary between runs and are summarized as box plots rather than single values
- The software environment matters for reproducibility: DESC requires Python 3.5.x/3.6.x and, to reproduce paper results, tensorflow below 2.0 (desc 2.0.3)

## Evidence

- [discussion] Batch effect removal evaluation via regional KL divergence of batch mixing (K nearest neighbors, K = 5 × number of batches, 200 iterations): [evidence span withheld — anchor status: not_found]
- [discussion] 200 iterations / box plots: "We repeated this procedure for 200 iterations with different randomly chosen cells to generate box plots of the final KL divergence."
- [abstract] Condition for batch removal: "Through iterative self-learning, DESC gradually removes batch effects, as long as technical differences across batches are smaller than true biological variations"
- [readme] DESC moves cells to balance biological and technical differences: "This iterative procedure moves each cell to its nearest cluster, balances biological and technical differences between clusters, and reduces the influence of batch effect."
