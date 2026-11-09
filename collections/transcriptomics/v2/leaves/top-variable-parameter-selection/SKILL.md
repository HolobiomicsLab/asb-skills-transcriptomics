---
name: top-variable-parameter-selection
description: Use when you have a fitted GEDI model's sample-specific manifold parameter
  matrix (Q) and need a sample-level 2D embedding showing that samples cluster by
  one covariate after regressing out another.
when_to_use_negative:
- When the analysis target is cell-level latent embedding rather than sample-level
  manifold parameters — GEDI's cell embeddings come from the model itself, not this
  post-hoc procedure
- When a covariate group has only one sample (the workflow restricts to technologies
  with rowSums > 1)
- When sample-specific translation vectors Δoi have not been excluded — including
  them confounds the manifold-parameter encoding
edam_operation: http://edamontology.org/operation_2939
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: GEDI
  role: Supplies the fitted model's sample-specific manifold parameters (params$Qi)
    and the plot_embedding function for visualizing the UMAP
  repo: https://github.com/csglab/GEDI
- name: R
  role: Implementation environment (library(GEDI), scuttle, scran, scater) for assembling
    Q, row-centering, SVD-based PCA, and UMAP
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1038/s41467-024-50963-0
    title: A unified model for interpretable latent embedding of multi-sample, multi-condition
      single-cell data
  source_content_hash: sha256:035ffa5bd2446110d1f4815c9cbf571d969829375a0a4d49c4f506ed13ddd2a3
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/csglab/GEDI
  tools:
  - GEDI
  - R
  edam_operation: http://edamontology.org/operation_2939
  edam_topics:
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6363636363636364
n_claims: 11
---

# top-variable-parameter-selection

## Summary

A post-hoc procedure for encoding and visualizing GEDI-learned sample-specific manifold parameters: regress out a covariate (technology or donor), select the top 20 most variable parameters, run PCA via SVD, and embed with UMAP to reproduce Fig. 2a/2d-style sample embeddings.

## When to use

Use when you have a fitted GEDI model's sample-specific manifold parameter matrix (Q) and need a sample-level 2D embedding showing that samples cluster by one covariate after regressing out another.

## When NOT to use

- When the analysis target is cell-level latent embedding rather than sample-level manifold parameters — GEDI's cell embeddings come from the model itself, not this post-hoc procedure
- When a covariate group has only one sample (the workflow restricts to technologies with rowSums > 1)
- When sample-specific translation vectors Δoi have not been excluded — including them confounds the manifold-parameter encoding

## Inputs

- Fitted GEDI model (sample-specific manifold parameters Qi, excluding translation vectors Δoi)
- Sample-level metadata matrix H mapping each sample to donor and technology
- Donor-by-technology count table for restricting to technologies with >1 sample

## Outputs

- UMAP coordinate table annotated with donor and technology (sample_umap_coordinates.csv)
- UMAP plots colored by donor and by technology (GEDI::plot_embedding)

## How to apply

Assemble Q by concatenating model$params$Qi across all samples (rows = genes x latent variables, columns = samples) and build a sample-level metadata matrix H mapping each sample to donor and technology. Restrict to technologies with more than one sample (rowSums of the donor-by-technology table > 1), then regress out the covariate by centering every row of Q within each covariate group. Rank rows of the centered matrix by standard deviation and keep the top 20 parameters (top_parameters <- order(apply(Q_subset,1,sd), decreasing=T)[1:20]), perform PCA via SVD rotating coordinates as svd_Q$v %*% diag(svd_Q$d), and embed with UMAP (min_dist = 0.01, euclidean metric, n_neighbors = 3). Visualize with GEDI::plot_embedding colored by donor and technology and write the UMAP coordinates with annotations to sample_umap_coordinates.csv.

## Related tools

- **GEDI** (Supplies the fitted model's sample-specific manifold parameters (params$Qi) and the plot_embedding function for visualizing the UMAP) — https://github.com/csglab/GEDI
- **R** (Implementation environment (library(GEDI), scuttle, scran, scater) for assembling Q, row-centering, SVD-based PCA, and UMAP)

## Examples

```
top_parameters <- order( apply(Q_subset,1,sd), decreasing=T )[1:20]
```

## Evaluation signals

- After regressing out technology, samples must cluster by donor in the UMAP; after regressing out donor, samples must cluster by technology
- Exactly 20 parameters are retained, ranked by decreasing row standard deviation of the centered Q matrix
- Technologies represented by a single sample are excluded before regression (rowSums of donor-by-technology table > 1)
- PCA coordinates are computed as svd_Q$v %*% diag(svd_Q$d) before UMAP embedding with min_dist = 0.01, metric = euclidean, n_neighbors = 3
- Output CSV contains UMAP coordinates with donor/technology annotations matching the figure reproducibility notebooks

## Limitations

- GEDI currently supports only linear manifold learning (optionally restricted to a hyperellipsoid), so this encoding reflects linear sample-specific distortions
- Singular covariate design columns must be removed (C <- C[ , QR$pivot[seq_len(QR$rank)] ]) or regression fails; the procedure was tested with R 4.0.0 on CentOS Linux 7
- Code tested with R 4.0.0 on CentOS Linux 7; dependencies include Rcpp, RcppEigen (needs gfortran), uwot, rsvd

## Evidence

- [methods] Filter: top-20 variable selection: "top_parameters <- order( apply(Q_subset,1,sd), decreasing=T )[1:20]"
- [methods] Covariate regression step: "Regress out effect of technology"
- [methods] Singularity guard during regression: "C <- C[ , QR$pivot[seq_len(QR$rank)] ] # remove any columns that results in singularity"
- [other] Post-hoc encoding of manifold parameters: [evidence span withheld — anchor status: not_found]
- [readme] GEDI package scope: [evidence span withheld — anchor status: external_doc]
