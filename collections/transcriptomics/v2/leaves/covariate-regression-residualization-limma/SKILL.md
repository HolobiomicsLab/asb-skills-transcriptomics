---
name: covariate-regression-residualization-limma
description: Use when a per-cell technical covariate (e.g. dsb's PC1 of isotype controls
  and fitted Gaussian-mixture background means µ1) must be regressed out of normalized
  protein/ADT values to return denoised residuals.
when_to_use_negative:
- Input is raw uncorrected ADT counts — the limma residualization must be applied
  to the step-I ambient-noise-corrected values, not raw UMIs
- Dataset lacks isotype control antibodies and a background-mean estimate; usage of
  the technical component in experiments without isotype controls requires the dsb
  documentation / Supplementary Note guidance
- Goal is removing batch structure across multiple batches — the article notes batch
  normalization behavior depends on the extent of batch uniformity, so this covariate
  regression is not a general batch-correction guarantee
edam_operation: http://edamontology.org/operation_3659
edam_topics:
- http://edamontology.org/topic_0091
tools:
- name: limma
  role: Provides removeBatchEffect() used to regress the dsb technical component out
    of ambient-corrected ADT values as a covariate
  repo: https://cran.r-project.org/package=dsb
- name: dsb
  role: Method/package whose DSBNormalizeProtein() wraps the technical-component computation
    and limma residualization; reproduction validated against its output
  repo: https://cran.r-project.org/package=dsb
- name: mclust
  role: Fits the 2-component Gaussian mixture per cell whose background mean µ1 feeds
    the noise matrix used to build the regressed covariate
  repo: https://cran.r-project.org/package=dsb
- name: R
  role: prcomp() computes the PC1 loading vectors of the noise matrix that define
    the technical component
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1038/s41467-022-29356-8
    title: Normalizing and denoising protein expression data from droplet-based single
      cell profiling
  source_content_hash: sha256:4dab4d568b4915fb7091bfa0ba6ed6f7d951bcd3daa4bf02a3c4774ab35de55f
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - R
  - dsb
  - limma
  - mclust
  edam_operation: http://edamontology.org/operation_3659
  edam_topics:
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 8
---

# covariate-regression-residualization-limma

## Summary

Residualize a per-cell technical noise component out of ambient-corrected ADT values using limma::removeBatchEffect(covariates = technical component), the second-step residualization used by the dsb method for CITE-seq protein data.

## When to use

Use when a per-cell technical covariate (e.g. dsb's PC1 of isotype controls and fitted Gaussian-mixture background means µ1) must be regressed out of normalized protein/ADT values to return denoised residuals.

## When NOT to use

- Input is raw uncorrected ADT counts — the limma residualization must be applied to the step-I ambient-noise-corrected values, not raw UMIs
- Dataset lacks isotype control antibodies and a background-mean estimate; usage of the technical component in experiments without isotype controls requires the dsb documentation / Supplementary Note guidance
- Goal is removing batch structure across multiple batches — the article notes batch normalization behavior depends on the extent of batch uniformity, so this covariate regression is not a general batch-correction guarantee

## Inputs

- Step-I ambient-corrected ADT matrix (cells x proteins, log-transformed with pseudocount 10 and standardized by empty-droplet mean mu_N and SD sigma_N)
- Per-cell technical component vector (PC1 of the noise matrix built from fitted mclust background means µ1 and isotype control values)

## Outputs

- Denoised, scaled ADT matrix of residuals (same dimensions as the input step-I matrix)
- Optionally written as a CSV, e.g. dsb_normalized_matrix.csv

## How to apply

First compute the per-cell technical component: bind each cell's fitted 2-component mclust background mean µ1 with the isotype control rows (e.g. Mouse IgG2bk, MouseIgG1kappa, MouseIgG2akappa, RatIgG2bk) into a noise matrix and take PC1 via prcomp(tmat, scale = TRUE). Then call limma::removeBatchEffect on the step-I ambient-corrected matrix (log(x + 10), standardized as (x - mu_N)/sigma_N against empty-droplet mean and SD) with the technical component supplied as a covariate. The returned residuals are the final dsb denoised values. Rationale: limma's removeBatchEffect performs a robust, efficient linear decomposition so the shared variance of isotype controls and background protein means — the cell-intrinsic noise source — is removed while preserving biological signal.

## Related tools

- **limma** (Provides removeBatchEffect() used to regress the dsb technical component out of ambient-corrected ADT values as a covariate) — https://cran.r-project.org/package=dsb
- **dsb** (Method/package whose DSBNormalizeProtein() wraps the technical-component computation and limma residualization; reproduction validated against its output) — https://cran.r-project.org/package=dsb
- **mclust** (Fits the 2-component Gaussian mixture per cell whose background mean µ1 feeds the noise matrix used to build the regressed covariate) — https://cran.r-project.org/package=dsb
- **R** (prcomp() computes the PC1 loading vectors of the noise matrix that define the technical component)

## Examples

```
denoised_adt <- limma::removeBatchEffect(step1_matrix, covariates = technical_component)
```

## Evaluation signals

- all.equal(denoised_adt, denoised_adt_3) confirms the manual mclust + prcomp + limma::removeBatchEffect pipeline reproduces DSBNormalizeProtein() output exactly
- Check that the technical component is the first PC score (prcomp with scale = TRUE) of the noise matrix combining µ1 with isotype rows 67:70 (Mouse IgG2bk, MouseIgG1kappa, MouseIgG2akappa, RatIgG2bk)
- Output matrix dimensions must match the step-I ambient-corrected matrix (cells x proteins), with residuals centered per protein
- Isotype-derived shared variance should no longer correlate with the residualized protein values across single cells

## Limitations

- dsb normalized values are sensitive to how background droplets are defined; defining a reasonable subset of barcodes as background requires care
- Caveats for using dsb on datasets without isotype control antibody measurements are deferred to the Supplementary Note and CRAN documentation
- In general, batch normalization behavior could be dependent on the extent of uniformity among the batches
- Software version numbers for limma/dsb are not stated in the article methods

## Evidence

- [discussion] limma removeBatchEffect covariate regression: "dsb uses a function from the limma52 package removeBatchEffect() for robust and efficient matrix decomposition"
- [discussion] residuals returned by dsb: "the dsb technical component for each cell is then regressed out of the ambient-noise-corrected values “Y” from part 1; the values returned by dsb are the residuals"
- [discussion] noise matrix construction: "A noise variable matrix is then constructed by combining all the fitted µ1 values with the isotype control values for all cells. dsb then calculates principal component 1"
- [discussion] prcomp loading vectors: "loading vectors in equation III calculated by the R function prcomp() are multiplied by the noise matrix"
