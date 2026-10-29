---
name: bulk-rnaseq-highly-variable-gene-selection
description: Use when a raw bulk expression matrix (e.g., TCGA log2(TPM+1) from GDC)
  must be reduced to a highly variable gene set before fitting a PhenoPath covariate-pseudotime
  model.
when_to_use_negative:
- The input is already a reduced or feature-filtered expression matrix, or a single-cell
  count object where per-cell HVG selection conventions differ from bulk log2(TPM+1)
  variance filtering
- The goal is the downstream ranking of covariate-pathway interactions (interaction_effects
  / significant_interactions) rather than defining the input feature space for trajectory
  fitting
- Data are not bulk gene expression on a log2(TPM+1)-like scale (e.g., already-normalised
  microarray or proteomics matrices), where TPM-based variance ranking is inappropriate
edam_topics:
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_0203
tools:
- name: PhenoPath
  role: Downstream consumer of the HVG-filtered matrix; phenopath() is fitted with
    the ER covariate and z_init = 1 after gene selection
  repo: https://github.com/kieranrcampbell/phenopath
- name: R
  role: Implementation environment in which the variance ranking, gene filtering,
    and phenopath() call are performed
- name: dplyr
  role: Used to assemble and filter the sample/gene tables when restricting the matrix
    to HVGs
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1038/s41467-018-04696-6
    title: Uncovering pseudotemporal trajectories with covariates from single cell
      and bulk expression data
  source_content_hash: sha256:49f8fcb2cf42e252d0cb7e9ffdbcd65c2cc940c084e1a5f2ee5e08f62171f7c7
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/kieranrcampbell/phenopath
  tools:
  - PhenoPath
  - R
  - dplyr
  - ggplot2
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3308
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5555555555555556
n_claims: 9
---

# bulk-rnaseq-highly-variable-gene-selection

## Summary

Prepares a bulk RNA-seq expression matrix for PhenoPath covariate-pseudotime modelling by restricting it to the top highly variable genes (e.g., 4579 genes from the TCGA-BRCA log2(TPM+1) matrix across 1135 samples), controlling noise while keeping the variational fit tractable.

## When to use

Use when a raw bulk expression matrix (e.g., TCGA log2(TPM+1) from GDC) must be reduced to a highly variable gene set before fitting a PhenoPath covariate-pseudotime model.

## When NOT to use

- The input is already a reduced or feature-filtered expression matrix, or a single-cell count object where per-cell HVG selection conventions differ from bulk log2(TPM+1) variance filtering
- The goal is the downstream ranking of covariate-pathway interactions (interaction_effects / significant_interactions) rather than defining the input feature space for trajectory fitting
- Data are not bulk gene expression on a log2(TPM+1)-like scale (e.g., already-normalised microarray or proteomics matrices), where TPM-based variance ranking is inappropriate

## Inputs

- Bulk RNA-seq expression matrix, log2(TPM+1), genes x samples (e.g., TCGA-BRCA, 1135 samples from GDC)
- Sample-level covariate vector (e.g., ER receptor status) keyed by sample ID
- Target number of highly variable genes to retain (e.g., 4579)

## Outputs

- Expression matrix restricted to the 4579 highly variable genes (same 1135 samples, log2(TPM+1) scale)
- Gene list / row index of retained HVGs used as the feature space for phenopath()

## How to apply

Load the bulk expression matrix as log2(TPM+1) (for the TCGA-BRCA cohort, 1135 samples from GDC) and rank genes by variance across samples, retaining the top 4579 highly variable genes while keeping the sample-covariate vector (ER status) aligned by sample ID. The rationale is that PhenoPath's coordinate ascent variational inference is scalable to 'thousands of samples and genes', but pre-filtering to HVGs concentrates signal in the genes most likely to drive the trajectory and its covariate interactions. Confirm that known positive controls (e.g., ESR1) survive the filter, since downstream significance is computed as |posterior mean| / posterior sd > k on the retained genes only. Feed the reduced matrix plus covariate into phenopath() (z_init = 1, PC1 initialisation) and check convergence with plot_elbo(fit); the ranked interaction table is then assembled only over the filtered gene set.

## Related tools

- **PhenoPath** (Downstream consumer of the HVG-filtered matrix; phenopath() is fitted with the ER covariate and z_init = 1 after gene selection) — https://github.com/kieranrcampbell/phenopath
- **R** (Implementation environment in which the variance ranking, gene filtering, and phenopath() call are performed)
- **dplyr** (Used to assemble and filter the sample/gene tables when restricting the matrix to HVGs)

## Examples

```
hvg <- names(sort(apply(brca_lcpm, 1, var), decreasing = TRUE))[1:4579]; fit <- phenopath(t(brca_lcpm[hvg, ]), er_status, z_init = 1)
```

## Evaluation signals

- Filtered matrix contains exactly the target number of genes (4579) across all 1135 samples with no samples dropped; note 1932 significant interactions = 42% of 4579, so the filter size propagates to the downstream table
- ESR1 (positive control) is present in the retained gene set and is recovered among the significant ER-pseudotime interactions
- Expression values remain non-negative and on the log2(TPM+1) scale after filtering
- Downstream phenopath() fit shows an approximately flat ELBO trace via plot_elbo(fit), indicating convergence on the filtered feature space

## Limitations

- PhenoPath assumes features evolve linearly with respect to pseudotime, so HVG selection cannot compensate for nonlinear trajectory effects; the article recommends checks that model fits are consistent with the data
- Low-variance genes are excluded before interaction testing, so genes whose covariate-pseudotime interaction is real but whose overall variance is small will never appear in the ranked interaction table
- The current PhenoPath framework cannot handle complex mixtures of continuous and discrete covariates in high-dimensional settings, limiting which covariates can be paired with the filtered matrix

## Evidence

- [readme] PhenoPath models the observed expression *y* in terms of a latent pathway score (pseudotime) *z*.: "PhenoPath models the observed expression *y* in terms of a latent pathway score (pseudotime) *z*."
- [discussion] We also demonstrated applications to The Cancer Genome Atlas using PhenoPath to model disease trajectories in colorectal and breast cancer.: "We also demonstrated applications to The Cancer Genome Atlas using PhenoPath to model disease trajectories in colorectal and breast cancer."
- [readme] Scalable variational Bayesian inference allows the trajectory and interactions to be inferred for thousands of samples and genes quickly.: "Scalable variational Bayesian inference allows the trajectory and interactions to be inferred for thousands of samples and genes quickly."
- [methods] We provide an R implementation of our method PhenoPath at https://bioconductor.org/packages/release/bioc/html/phenopath.html: "We provide an R implementation of our method PhenoPath at https://bioconductor.org/packages/release/bioc/html/phenopath.html"
