---
name: covariate-regression-on-sample-parameters
description: Use when you have a fitted GEDI model for a multi-sample single-cell
  dataset (e.g., PBMCsca) and need to visualize sample-level manifold parameters or
  verify that GEDI disentangled donor from technology effects, as in Fig. 2a/2d UMAPs.
when_to_use_negative:
- You need the GEDI generative model's biological outputs (cluster-free DGE, imputation,
  pathway/TF activities) rather than post-hoc visualization of sample-level parameters
- A covariate group has only one sample (e.g., a technology represented by a single
  donor), since group-wise centering would erase the group entirely — the workflow
  explicitly restricts to rowSums(donor-by-technology table) > 1
- You want to quantify or test the covariate effect itself; regressing it out by centering
  within groups removes exactly the signal you would be testing
edam_operation: http://edamontology.org/operation_2939
edam_topics:
- http://edamontology.org/topic_3473
tools:
- name: GEDI
  role: Source of the sample-specific manifold parameters (model$params$Qi) being
    regressed and embedded; provides plot_embedding for visualization
  repo: https://github.com/csglab/GEDI/
- name: R
  role: Implementation environment for the regression, SD-based parameter selection,
    SVD/PCA, and UMAP workflow in the manuscript notebooks
- name: uwot
  role: R package used to compute the UMAP embedding (min_dist = 0.01, euclidean metric,
    n_neighbors = 3) of the rotated PCA coordinates
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1038/s41467-024-50963-0
    title: A unified model for interpretable latent embedding of multi-sample, multi-condition
      single-cell data
  source_content_hash: sha256:8a85c33de5dde6852386c892296a7fcc5076631bce1b45aa4b8e0be717eab397
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
  - http://edamontology.org/topic_3473
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7
n_claims: 10
---

# covariate-regression-on-sample-parameters

## Summary

Post-hoc procedure from the GEDI manuscript (Nat Commun 2024) that encodes each sample by its GEDI-learned sample-specific manifold parameters (Qi), regresses out a confounding covariate (technology or donor) by group-wise row centering, selects the top 20 most variable parameters, and embeds samples via SVD-based PCA followed by UMAP to test whether samples cluster by the complementary covariate.

## When to use

Use when you have a fitted GEDI model for a multi-sample single-cell dataset (e.g., PBMCsca) and need to visualize sample-level manifold parameters or verify that GEDI disentangled donor from technology effects, as in Fig. 2a/2d UMAPs.

## When NOT to use

- You need the GEDI generative model's biological outputs (cluster-free DGE, imputation, pathway/TF activities) rather than post-hoc visualization of sample-level parameters
- A covariate group has only one sample (e.g., a technology represented by a single donor), since group-wise centering would erase the group entirely — the workflow explicitly restricts to rowSums(donor-by-technology table) > 1
- You want to quantify or test the covariate effect itself; regressing it out by centering within groups removes exactly the signal you would be testing

## Inputs

- Fitted GEDI model object with per-sample manifold parameters in model$params$Qi
- Sample-level metadata matrix H mapping each sample to donor and technology
- Donor-by-technology grouping table for selecting covariate groups with >1 sample

## Outputs

- Centered, covariate-regressed parameter matrix Q (rows = genes x latent variables, columns = samples)
- Set of top-20 most variable parameter row indices
- Rotated PCA sample coordinates (svd_Q$v %*% diag(svd_Q$d))
- Sample UMAP embedding coordinates with donor/technology annotations (sample_umap_coordinates.csv)
- GEDI::plot_embedding UMAP plots colored by donor and by technology

## How to apply

Load the fitted GEDI model and build the parameter matrix Q by concatenating model$params$Qi across samples (rows = genes x latent variables, columns = samples), excluding sample-specific translation vectors Δoi, and build a metadata table mapping each sample to donor and technology. Restrict to covariate groups with more than one sample (rowSums of the donor-by-technology table > 1). Regress out the covariate by centering every row of Q within each covariate group, then rank rows by standard deviation and keep the top 20 parameters (top_parameters <- order(apply(Q_subset,1,sd),decreasing=T)[1:20]). Perform PCA via SVD and compute rotated sample coordinates as svd_Q$v %*% diag(svd_Q$d), then embed with uwot UMAP using min_dist = 0.01, euclidean metric, and n_neighbors = 3. Visualize with GEDI::plot_embedding colored by donor and by technology, and export the UMAP coordinates to sample_umap_coordinates.csv.

## Related tools

- **GEDI** (Source of the sample-specific manifold parameters (model$params$Qi) being regressed and embedded; provides plot_embedding for visualization) — https://github.com/csglab/GEDI/
- **R** (Implementation environment for the regression, SD-based parameter selection, SVD/PCA, and UMAP workflow in the manuscript notebooks)
- **uwot** (R package used to compute the UMAP embedding (min_dist = 0.01, euclidean metric, n_neighbors = 3) of the rotated PCA coordinates)

## Examples

```
top_parameters <- order( apply(Q_subset,1,sd), decreasing=T )[1:20]
```

## Evaluation signals

- After regressing out technology, UMAP samples must cluster by donor (and after regressing out donor, by technology), matching the paper's Fig. 2a/2d result
- Q matrix dimensions must equal (genes x latent variables) rows by samples columns, matching the concatenation of model$params$Qi across samples
- Exactly 20 rows of the centered Q matrix are retained, and they are the 20 highest-standard-deviation parameters under the covariate-regressed ranking
- UMAP is run with the manuscript's parameters: min_dist = 0.01, metric = euclidean, n_neighbors = 3
- sample_umap_coordinates.csv contains UMAP coordinates with donor and technology annotations for every retained sample

## Limitations

- Sample-specific translation vectors Δoi are excluded from the parameter matrix, so only manifold shape parameters are visualized
- Samples whose covariate group (donor or technology) has a single member must be dropped before regression
- This is a post-hoc visualization/diagnostic procedure, not part of GEDI's generative model; the UMAP result depends on hyperparameters (min_dist, n_neighbors) and does not itself constitute a statistical test of covariate effects

## Evidence

- [methods] Regress out effect of technology: "# Regress out effect of technology"
- [methods] Select top 20 most variable parameters: "top_parameters <- order( apply(Q_subset,1,sd), decreasing=T )[1:20]"
- [readme] Notebooks reproduce the manuscript figures, including the Fig. 2 sample-to-sample variability analyses this post-hoc embedding procedure supports: [evidence span withheld — anchor status: external_doc]
