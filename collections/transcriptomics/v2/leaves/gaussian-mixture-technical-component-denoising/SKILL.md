---
name: gaussian-mixture-technical-component-denoising
description: Use when ADT/protein counts have already been ambient-corrected against
  empty droplets and residual cell-to-cell technical noise (correlated with isotype
  controls and background mean) must be regressed out before clustering or UMAP.
when_to_use_negative:
- The dataset has no isotype control antibody measurements — caveats for this case
  are deferred to the dsb Supplementary Note and software documentation, and the noise
  matrix cannot be built as described
- ADT counts have not yet been background-corrected with empty droplets (step I) —
  the Gaussian mixture must be fit to the step-I transformed values, not raw counts
- Input is already dsb-normalized (denoise.counts = FALSE exists for cases where step
  II should be skipped)
edam_operation: http://edamontology.org/operation_2939
edam_topics:
- http://edamontology.org/topic_2229
tools:
- name: dsb
  role: R package implementing the Gaussian mixture background fit, noise matrix PC1,
    and residual computation via DSBNormalizeProtein
  repo: https://cran.r-project.org/package=dsb
- name: mclust
  role: Fits the 2-component Gaussian mixture model via expectation-maximization to
    each cell's transformed ADT counts
- name: limma
  role: removeBatchEffect() performs the robust matrix decomposition/regression of
    the technical component out of ambient-corrected values
- name: R
  role: prcomp() computes loading vectors / PC1 of the noise matrix (µ1 + isotype
    controls)
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1038/s41467-022-29356-8
    title: Normalizing and denoising protein expression data from droplet-based single
      cell profiling
  source_content_hash: sha256:e6181eca25fb0e395c00e3d3c0f692b4525ef66a173574a615da67cea7c038e9
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - R
  - Seurat
  - dsb
  - limma
  - mclust
  edam_operation: http://edamontology.org/operation_2939
  edam_topics:
  - http://edamontology.org/topic_2229
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.75
n_claims: 8
---

# gaussian-mixture-technical-component-denoising

## Summary

The dsb step II denoising procedure: fit a 2-component Gaussian mixture per cell to log-transformed ADT values to estimate the background mean µ1, combine µ1 with isotype controls into a noise matrix, take PC1 (via prcomp) as the technical component, and remove it with limma::removeBatchEffect to return denoised residuals.

## When to use

Use when ADT/protein counts have already been ambient-corrected against empty droplets and residual cell-to-cell technical noise (correlated with isotype controls and background mean) must be regressed out before clustering or UMAP.

## When NOT to use

- The dataset has no isotype control antibody measurements — caveats for this case are deferred to the dsb Supplementary Note and software documentation, and the noise matrix cannot be built as described
- ADT counts have not yet been background-corrected with empty droplets (step I) — the Gaussian mixture must be fit to the step-I transformed values, not raw counts
- Input is already dsb-normalized (denoise.counts = FALSE exists for cases where step II should be skipped)

## Inputs

- Ambient-corrected, log-transformed ADT count matrix (cells x proteins) from dsb step I
- Isotype control antibody measurements per cell
- Empty-droplet (background) ADT matrix from raw Cell Ranger barcodes
- Per-cell protein library size (lambda) for validation

## Outputs

- Denoised dsb-normalized protein value matrix (residuals after technical component removal)
- Per-cell fitted background mean cellwise_background_mean (µ1)
- Per-cell technical component (PC1 of the noise matrix)
- Cor 1 (µ1 vs isotype) and Cor 2 (µ1 vs lambda) correlation statistics

## How to apply

After dsb step I (log(cell + 10) transformation and subtraction of empty-droplet background mean mu_N divided by sigma_N), fit a 2-component Gaussian mixture with mclust to the transformed counts of each cell and extract the background subpopulation mean µ1 as cellwise_background_mean. Build a noise variable matrix by combining all fitted µ1 values with isotype control antibody values, then compute principal component 1 of this matrix with prcomp() as the per-cell technical component. Regress this technical component out of the ambient-corrected values using limma::removeBatchEffect; the returned residuals are the dsb-normalized protein values. Validate that µ1 correlates positively and significantly with isotype controls (Cor 1) and with protein library size lambda (Cor 2) across single cells, as in Fig. 2i.

## Related tools

- **dsb** (R package implementing the Gaussian mixture background fit, noise matrix PC1, and residual computation via DSBNormalizeProtein) — https://cran.r-project.org/package=dsb
- **mclust** (Fits the 2-component Gaussian mixture model via expectation-maximization to each cell's transformed ADT counts)
- **limma** (removeBatchEffect() performs the robust matrix decomposition/regression of the technical component out of ambient-corrected values)
- **R** (prcomp() computes loading vectors / PC1 of the noise matrix (µ1 + isotype controls))

## Examples

```
dsb_rescaled = DSBNormalizeProtein(cell_protein_matrix = cells_citeseq_mtx, empty_drop_matrix = empty_drop_citeseq_mtx, denoise.counts = TRUE, use.isotype.control = TRUE)
```

## Evaluation signals

- The fitted background mean µ1 is positively and significantly correlated with isotype control levels across single cells (Cor 1)
- µ1 is positively and significantly correlated with protein library size lambda across single cells (Cor 2)
- The k=2 Gaussian mixture assigns the majority of cells consistently (e.g., ~89% in one component on the 10X NextGem dataset)
- Downstream clustering/UMAP on the denoised values recovers biologically interpretable protein-based populations and biaxial plots (e.g., CD14 vs CD4) matching expected structure
- Residual values are returned only after removeBatchEffect regression of the technical component; skipping regression leaves correlated technical noise

## Limitations

- The method's caveats and modeling assumptions for datasets without isotype controls are detailed in the Supplementary Note, not the main text
- Usage guidance for defining the technical component in experiments without isotype control antibodies is deferred to the CRAN documentation and Supplementary Note
- Denoising behavior can depend on how background droplets are defined and on batch uniformity

## Evidence

- [methods] Gaussian mixture per-cell µ1 fit: "dsb fits a Gaussian mixture with 2 mixing components to model the ADT levels in each cell"
- [discussion] Noise matrix PC1 construction: "A noise variable matrix is then constructed by combining all the fitted µ1 values with the isotype control values for all cells. dsb then calculates principal component 1"
- [discussion] mclust EM implementation: "dsb fits a Gaussian mixture model through the expectation-maximization algorithm implemented with the mclust51 R package"
- [discussion] Regression step: "the dsb technical component for each cell is then regressed out of the ambient-noise-corrected values “Y” from part 1; the values returned by dsb are the residuals"
- [discussion] prcomp in R: "loading vectors in equation III calculated by the R function prcomp() are multiplied by the noise matrix"
- [abstract] Isotype/background shared variance rationale: "isotype control antibodies and the background protein population average in each cell exhibit significant correlations across single cells, we thus use their shared variance to correct for"
