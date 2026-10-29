---
name: pearson-correlation-matrix-computation
description: Use when validating that a per-cell background/noise estimate (µ1) captures
  shared variance with isotype controls in CITE-seq ADT data, or when quantifying
  concordance of fitted parameters between alternative mixture-component definitions
  (k=2 vs k=3).
when_to_use_negative:
- Datasets without isotype control antibody measurements — caveats for such datasets
  are deferred to the dsb Supplementary Note
- Analyzing normalized/denoised dsb output values — correlations must be computed
  on step-I ambient-corrected values before technical-component removal
- Background droplets are not well-defined — dsb normalized values are sensitive to
  how background barcodes are defined and a reasonable subset requires care
edam_operation: http://edamontology.org/operation_3891
edam_topics:
- http://edamontology.org/topic_2229
- http://edamontology.org/topic_3360
tools:
- name: dsb
  role: Provides the step-I ambient-noise correction (log(x+10), background mean/std
    rescaling) and the source data matrices cells_citeseq_mtx and empty_drop_citeseq_mtx
  repo: https://cran.r-project.org/package=dsb
- name: mclust
  role: Fits the per-cell 2-component (and 3-component) Gaussian mixture model via
    EM from which the background mean µ1 is extracted
- name: R
  role: Environment in which the per-cell mixture fits and Pearson correlations across
    single cells are computed
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1038/s41467-022-29356-8
    title: Normalizing and denoising protein expression data from droplet-based single
      cell profiling
  source_content_hash: sha256:b729c15de6e8d26744e58bca40e21d8ba059a93d2428729c788c4a3d2a65e6fd
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - R
  - dsb
  - mclust
  edam_operation: http://edamontology.org/operation_3891
  edam_topics:
  - http://edamontology.org/topic_2229
  - http://edamontology.org/topic_3360
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7272727272727273
n_claims: 11
---

# pearson-correlation-matrix-computation

## Summary

Compute a Pearson correlation matrix across single cells between the per-cell fitted background protein mean (µ1 from a k=2 Gaussian mixture), isotype control antibodies, their mean, and protein library size, to validate the shared-variance technical noise estimate underlying the dsb technical component (λ).

## When to use

Use when validating that a per-cell background/noise estimate (µ1) captures shared variance with isotype controls in CITE-seq ADT data, or when quantifying concordance of fitted parameters between alternative mixture-component definitions (k=2 vs k=3).

## When NOT to use

- Datasets without isotype control antibody measurements — caveats for such datasets are deferred to the dsb Supplementary Note
- Analyzing normalized/denoised dsb output values — correlations must be computed on step-I ambient-corrected values before technical-component removal
- Background droplets are not well-defined — dsb normalized values are sensitive to how background barcodes are defined and a reasonable subset requires care

## Inputs

- Cell protein/ADT count matrix (dsb::cells_citeseq_mtx)
- Background empty-droplet protein/ADT count matrix (dsb::empty_drop_citeseq_mtx) with isotype control antibodies at rows 67:70
- Per-cell protein library size values
- Fitted background means µ1 from k=2 and k=3 mclust Gaussian mixture fits per cell

## Outputs

- Full Pearson correlation matrix (written to correlation_matrix.csv) covering µ1, individual isotype controls, mean isotype value, and library size
- Pearson concordance value between k=2 and k=3 µ1 estimates

## How to apply

First apply dsb step I: log(x+10) transform both the cell protein matrix (dsb::cells_citeseq_mtx) and the background droplet matrix (dsb::empty_drop_citeseq_mtx, isotype controls at rows 67:70), then per protein subtract the empty-droplet mean µ_N and divide by the standard deviation σ_N. Fit a k=2 Gaussian mixture (mclust::Mclust, G=2) to each cell's step-I-corrected values and extract the background mean µ1 for every cell; repeat with G=3 to obtain k=3 µ1 values. Then compute Pearson correlations across single cells: µ1 vs the mean of the four isotype controls (expected r≈0.59), µ1 vs each individual isotype control (expected r≈0.33), among the isotype controls themselves (expected r≈0.11), and mean isotype value vs protein library size (expected r≈0.45). Write the full correlation matrix to correlation_matrix.csv and record the k=2 vs k=3 µ1 Pearson concordance, which should be near 1, confirming that shared variance between µ1 and isotype controls is a robust technical noise measure captured by PC1 (λ).

## Related tools

- **dsb** (Provides the step-I ambient-noise correction (log(x+10), background mean/std rescaling) and the source data matrices cells_citeseq_mtx and empty_drop_citeseq_mtx) — https://cran.r-project.org/package=dsb
- **mclust** (Fits the per-cell 2-component (and 3-component) Gaussian mixture model via EM from which the background mean µ1 is extracted)
- **R** (Environment in which the per-cell mixture fits and Pearson correlations across single cells are computed)

## Examples

```
dlog <- log(cell + 10); dlog <- t(apply(dlog, 1, function(x) (x - mu_N) / sigma_N)); fit <- mclust::Mclust(dlog, G = 2); r <- cor(mu1_k2, rowMeans(isotype_vals), method = 'pearson')
```

## Evaluation signals

- Pearson r between µ1 and the mean of the four isotype controls should be approximately 0.59
- Pearson r between µ1 and each individual isotype control should be approximately 0.33, and among isotype controls themselves approximately 0.11
- Pearson r between the mean isotype value and protein library size should be approximately 0.45
- µ1 concordance between k=2 and k=3 mixture fits should be high (Pearson near 1)
- correlation_matrix.csv is produced and contains all pairwise Pearson values from step 5 of the workflow

## Limitations

- Isotype controls correlate weakly with each other (r≈0.11), so individual isotypes are noisier noise proxies than their shared variance / µ1
- Defining a reasonable subset of barcodes as background droplets still requires care; dsb results are sensitive to this definition
- Caveats and robustness assessments (empty droplet definition sensitivity, isotype-free usage) are documented in the Supplementary Note, not the main text

## Evidence

- [abstract] Isotype control antibodies and the background protein population average in each cell exhibit significant correlations across single cells: [evidence span withheld — anchor status: not_found]
- [discussion] dsb fits a Gaussian mixture model with mclust and k=2 components to estimate the background mean: "dsb fits a Gaussian mixture model through the expectation-maximization algorithm implemented with the mclust51 R package to the transformed count of each cell from step 1 with k = 2 mixture components"
- [methods] Step I uses empty droplets to estimate ambient background per antibody: "protein counts in empty droplets are used to estimate the expected ambient background noise for each antibody"
- [methods] Per-cell two-component Gaussian mixture to extract background mean µ1 (dsb step II part I): "dsb fits a Gaussian mixture with 2 mixing components to model the ADT levels in each cell"
- [discussion] Caveats for datasets without isotype control measurements deferred to Supplementary Note: "caveats for using dsb on datasets without isotype control antibody measurements."
