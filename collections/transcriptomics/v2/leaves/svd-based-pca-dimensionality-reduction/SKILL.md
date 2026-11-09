---
name: svd-based-pca-dimensionality-reduction
description: Use when you have a fitted GEDI model with sample-specific manifold parameter
  matrices (model$params$Qi) and need a low-dimensional sample-level embedding to
  visualize how samples cluster by donor or technology.
when_to_use_negative:
- Input is cell-level expression data rather than the sample-level GEDI manifold parameter
  matrix Q (this skill operates post-fitting on Qi, not on the raw counts)
- No fitted GEDI model or fewer than 20 sample-specific parameters exist, since the
  procedure explicitly keeps the top 20 parameters
- The goal is linear manifold learning itself or nonlinear parametric manifold approximation
  — the paper notes GEDI currently supports only linear manifold learning with optional
  hyperellipsoid restriction
edam_operation: http://edamontology.org/operation_2939
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: GEDI
  role: Source of the sample-specific manifold parameters (model$params$Qi) being
    reduced, and provider of plot_embedding for visualizing the UMAP colored by donor/technology
  repo: https://github.com/csglab/GEDI/
- name: R
  role: Implementation environment; SVD-based PCA and covariate regression (with QR-based
    singularity removal) are done in R using scuttle/scran/scater ecosystem libraries
- name: uwot
  role: R package used for the final UMAP embedding of the rotated PCA coordinates
    (min_dist = 0.01, n_neighbors = 3)
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1038/s41467-024-50963-0
    title: A unified model for interpretable latent embedding of multi-sample, multi-condition
      single-cell data
  source_content_hash: sha256:bd592548a43a97ecc993e79df3f0b4fd7cae9602259e3d9c70766006941903ec
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/csglab/GEDI
  tools:
  - GEDI
  - R
  - uwot
  edam_operation: http://edamontology.org/operation_2939
  edam_topics:
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 12
---

# svd-based-pca-dimensionality-reduction

## Summary

A post-hoc procedure from the GEDI manuscript for encoding and visualizing sample-specific manifold parameters: regress out a covariate, keep the top-20 most variable parameters by row standard deviation, and embed them with SVD-based PCA followed by UMAP (e.g., Fig. 2a and Fig. 2d of the GEDI paper).

## When to use

Use when you have a fitted GEDI model with sample-specific manifold parameter matrices (model$params$Qi) and need a low-dimensional sample-level embedding to visualize how samples cluster by donor or technology.

## When NOT to use

- Input is cell-level expression data rather than the sample-level GEDI manifold parameter matrix Q (this skill operates post-fitting on Qi, not on the raw counts)
- No fitted GEDI model or fewer than 20 sample-specific parameters exist, since the procedure explicitly keeps the top 20 parameters
- The goal is linear manifold learning itself or nonlinear parametric manifold approximation — the paper notes GEDI currently supports only linear manifold learning with optional hyperellipsoid restriction

## Inputs

- Fitted GEDI model object with sample-specific manifold parameters model$params$Qi (one matrix per sample)
- Sample-level metadata matrix H mapping each sample to donor and technology
- Donor-by-technology annotation table used to restrict to technologies with more than one sample

## Outputs

- Top-20 most-variable parameter subset of the centered Q matrix
- SVD-based PCA sample coordinates (svd_Q$v %*% diag(svd_Q$d))
- UMAP embedding of sample coordinates (min_dist = 0.01, euclidean metric, n_neighbors = 3)
- UMAP plots colored by donor and by technology (via GEDI::plot_embedding) and a UMAP coordinate table annotated with donor/technology (e.g., sample_umap_coordinates.csv)

## How to apply

Assemble the sample-specific manifold parameter matrix Q by concatenating model$params$Qi across samples (rows = genes x latent variables, columns = samples), excluding the sample-specific translation vectors Δoi, and build a sample-level metadata matrix H (donor, technology). Restrict to technologies with more than one sample (rowSums of the donor-by-technology table > 1). Regress out the covariate by centering every row of Q within each covariate group (e.g., subtract the technology-group row mean). Select the top 20 most variable parameters by ordering rows of the centered matrix by apply(Q, 1, sd) decreasingly, keeping [1:20]. Perform PCA via SVD and compute sample coordinates as svd_Q$v %*% diag(svd_Q$d); if building a covariate design matrix C, remove any columns causing singularity via QR pivoting (QR$pivot[seq_len(QR$rank)]). Finally embed the rotated coordinates with UMAP (min_dist = 0.01, metric = euclidean, n_neighbors = 3) and visualize with GEDI::plot_embedding colored by donor and by technology.

## Related tools

- **GEDI** (Source of the sample-specific manifold parameters (model$params$Qi) being reduced, and provider of plot_embedding for visualizing the UMAP colored by donor/technology) — https://github.com/csglab/GEDI/
- **R** (Implementation environment; SVD-based PCA and covariate regression (with QR-based singularity removal) are done in R using scuttle/scran/scater ecosystem libraries)
- **uwot** (R package used for the final UMAP embedding of the rotated PCA coordinates (min_dist = 0.01, n_neighbors = 3))

## Examples

```
svd_Q <- svd(Q_centered[top_parameters, ]); emb <- umap(t(svd_Q$v %*% diag(svd_Q$d)), min_dist = 0.01, n_neighbors = 3, metric = "euclidean")
```

## Evaluation signals

- Exactly 20 parameters are retained after ranking rows by standard deviation (order(apply(Q_subset,1,sd), decreasing=T)[1:20])
- After regressing out technology, samples must cluster by donor in the UMAP; after regressing out donor, they must cluster by technology, matching the paper's Fig. 2 result
- Technologies with only one sample are excluded (rowSums of the donor-by-technology table must be > 1 per kept technology)
- Any columns of the covariate design matrix C that yield singularity are removed via QR pivoting (C <- C[, QR$pivot[seq_len(QR$rank)]]) before regression
- The exported UMAP coordinate table carries donor and technology annotations for every sample

## Limitations

- GEDI currently supports linear manifold learning (with optional hyperellipsoid restriction); other parametric manifold approximations are future extensions, so this PCA-based post-hoc view inherits that linearity
- The embedding operates on a small fixed number of parameters (top 20), so it is a visualization-oriented summary rather than a full-resolution manifold representation
- The UMAP parameters used in the paper (min_dist = 0.01, n_neighbors = 3) were tuned for these PBMC/COVID sample embeddings and may need retuning for other datasets

## Evidence

- [methods] top-20 variable selection, PCA, and UMAP embedding: [evidence span withheld — anchor status: not_found]
- [methods] covariate regression step: [evidence span withheld — anchor status: not_found]
- [methods] singularity removal via QR: "C <- C[ , QR$pivot[seq_len(QR$rank)] ] # remove any columns that results in singularity"
- [results] full post-hoc procedure description: [evidence span withheld — anchor status: not_found]
- [readme] UMAP and PCA tooling context: [evidence span withheld — anchor status: external_doc]
