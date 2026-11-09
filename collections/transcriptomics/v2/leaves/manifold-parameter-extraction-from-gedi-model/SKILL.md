---
name: manifold-parameter-extraction-from-gedi-model
description: Use when a fitted GEDI model (RDS file, e.g. from Zenodo accessions 8222040/8222698)
  must be converted into per-sample feature matrices for UMAP embedding of sample
  variability or cross-cohort SVM prediction of sample labels.
when_to_use_negative:
- Input is not a fitted GEDI model (e.g. an integrated latent space from Harmony or
  a LEMUR fit) — the Qi parameter structure is specific to GEDI
- The goal is cell-level embedding or cluster-free DGE rather than sample-level analysis;
  GEDI addresses those via its denoised profiles and transcriptomic vector field,
  not via Qi extraction
- The covariate group has only one sample (the paper restricts the donor/technology
  UMAP to technologies with more than one sample)
edam_operation: http://edamontology.org/operation_2928
edam_topics:
- http://edamontology.org/topic_0203
tools:
- name: GEDI
  role: Source of the fitted model whose params$Qi entries are the sample-specific
    manifold parameters being extracted
  repo: https://github.com/csglab/GEDI
- name: R
  role: Implementation environment (scuttle/scran/scater QC, apply/sd top-20 selection,
    SVD-based PCA, SVM training)
- name: uwot
  role: R package used to embed the rotated PCA coordinates of the top-20 parameters
    into UMAP space
  repo: https://cran.r-project.org/web/packages/uwot/index.html
provenance:
  source_task_ids:
  - task_001
  - task_003
  source_papers:
  - doi: 10.1038/s41467-024-50963-0
    title: A unified model for interpretable latent embedding of multi-sample, multi-condition
      single-cell data
  source_content_hash: sha256:472d3629a5cc0b7bf0e9ca65481f43dfd0972ad16eecf42cf336ae60c5a1a25f
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/csglab/GEDI
  tools:
  - GEDI
  - R
  - uwot
  edam_operation: http://edamontology.org/operation_2928
  edam_topics:
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7
n_claims: 10
---

# manifold-parameter-extraction-from-gedi-model

## Summary

Extracts the sample-specific manifold parameters (distortions) learned by a fitted GEDI model — the Qi entries in model$params, excluding translation vectors Δoi — into a sample-level feature matrix Q (rows = genes × latent variables, columns = samples). These parameters feed downstream tasks such as UMAP visualization of sample-to-sample variability (Fig. 2a/2d) and SVM prediction of sample characteristics across cohorts.

## When to use

Use when a fitted GEDI model (RDS file, e.g. from Zenodo accessions 8222040/8222698) must be converted into per-sample feature matrices for UMAP embedding of sample variability or cross-cohort SVM prediction of sample labels.

## When NOT to use

- Input is not a fitted GEDI model (e.g. an integrated latent space from Harmony or a LEMUR fit) — the Qi parameter structure is specific to GEDI
- The goal is cell-level embedding or cluster-free DGE rather than sample-level analysis; GEDI addresses those via its denoised profiles and transcriptomic vector field, not via Qi extraction
- The covariate group has only one sample (the paper restricts the donor/technology UMAP to technologies with more than one sample)

## Inputs

- Fitted GEDI model object (RDS file, e.g. from https://zenodo.org/record/8222040/ or https://zenodo.org/record/8222698/)
- Sample-level metadata matrix H mapping each sample to donor, technology, cohort, and/or disease status (COVID vs. healthy)
- Count matrix preprocessed in R with scuttle/scran/scater QC (log10_total_counts > 2.5, log10_total_features_by_counts > 2, subsets_Mito_percent <= 20; genes with sum_genes > 3)

## Outputs

- Sample-specific manifold parameter matrix Q (rows = genes × latent variables, columns = samples) from concatenated model$params$Qi
- Top-20 most variable parameter subset (top_parameters indices by row SD)
- UMAP coordinate table annotated with donor/technology (e.g. sample_umap_coordinates.csv)
- Cross-cohort AUROC values from SVM on top-20 parameters (e.g. covid_svm_auroc.csv)

## How to apply

Load the trained GEDI model and assemble Q by concatenating model$params$Qi across all samples, keeping rows = genes × latent variables and columns = samples, while excluding the sample-specific translation vectors Δoi. If visualizing, restrict to technologies with more than one sample (rowSums of the donor-by-technology table > 1), optionally regress out a covariate (technology or donor) by centering every row of Q within each covariate group, then select the top 20 most variable parameters via order(apply(Q,1,sd),decreasing=T)[1:20]. For embedding, run PCA via SVD on the selected rows, compute rotated coordinates as svd_Q$v %*% diag(svd_Q$d), and pass them to UMAP with min_dist = 0.01, euclidean metric, n_neighbors = 3 (GEDI::plot_embedding colors by donor/technology). For prediction instead, use the top-20 parameter values as SVM features with disease status as labels, training on one cohort and testing on the other.

## Related tools

- **GEDI** (Source of the fitted model whose params$Qi entries are the sample-specific manifold parameters being extracted) — https://github.com/csglab/GEDI
- **R** (Implementation environment (scuttle/scran/scater QC, apply/sd top-20 selection, SVD-based PCA, SVM training))
- **uwot** (R package used to embed the rotated PCA coordinates of the top-20 parameters into UMAP space) — https://cran.r-project.org/web/packages/uwot/index.html

## Examples

```
Q <- do.call(cbind, model$params$Qi); top_parameters <- order( apply(Q,1,sd), decreasing=T )[1:20]
```

## Evaluation signals

- After regressing out technology, UMAP samples must cluster by donor; after regressing out donor, they must cluster by technology (the paper's reported Fig. 2 result)
- Q dimensions must be (genes × latent variables) rows × samples columns, with sample-specific translation vectors Δoi excluded
- Top-20 selection must match row-SD ordering: order(apply(Q,1,sd),decreasing=T)[1:20]
- On the COVID-19 cross-cohort task, SVM AUROC should reach 1.0 (train cohort 2 → test cohort 1) and 0.97 (train cohort 1 → test cohort 2)

## Limitations

- GEDI currently supports only linear manifold learning, with an option to restrict the manifold to a hyperellipsoid; other parametric manifold approximations are untested future extensions
- The C matrix may become singular for some covariate designs — remove columns causing singularity (C <- C[ , QR$pivot[seq_len(QR$rank)] ])
- Effects of factors influencing performance (number of DE genes, sample/cell counts, sequencing depth) require more extensive analysis per the discussion

## Evidence

- [methods] top_parameters <- order( apply(Q_subset,1,sd), decreasing=T )[1:20]: "top_parameters <- order( apply(Q_subset,1,sd), decreasing=T )[1:20]"
- [methods] Regress out effect of technology: "Regress out effect of technology"
- [abstract] We introduce GEDI, a generative model that identifies latent space variations in multi-sample, multi-condition single-cell datasets: "We introduce GEDI, a generative model that identifies latent space variations in multi-sample, multi-condition single-cell datasets"
- [readme] Models and other data needed to run the notebooks can be found in the following Zenodo accessions: [evidence span withheld — anchor status: external_doc]
- [discussion] GEDI currently supports linear manifold learning, with the option to further restrict the manifold to a hyperellipsoid: "GEDI currently supports linear manifold learning, with the option to further restrict the manifold to a hyperellipsoid"
