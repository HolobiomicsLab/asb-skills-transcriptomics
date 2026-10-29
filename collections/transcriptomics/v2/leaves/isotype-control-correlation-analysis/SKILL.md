---
name: isotype-control-correlation-analysis
description: Use when you need to verify that a cell-intrinsic technical noise estimate
  for droplet-based protein (ADT) data is robust — e.g. correlating fitted background
  means µ1 with isotype controls or checking k=2 vs k=3 mixture-model concordance
  before building the dsb technical component.
when_to_use_negative:
- The dataset has no isotype control antibody measurements — caveats for dsb in this
  case are deferred to the Supplementary Note and the technical component definition
  differs.
- The goal is ambient, protein-specific noise correction (dsb step I); that uses empty-droplet
  statistics, not the cell-to-cell shared variance this skill quantifies.
edam_operation: http://edamontology.org/operation_2238
edam_topics:
- http://edamontology.org/topic_3577
tools:
- name: dsb
  role: provides the packaged cell and empty-droplet CITE-seq matrices and the step-I
    log(x+10) transform with µ_N/σ_N background standardization
  repo: https://cran.r-project.org/package=dsb
- name: mclust
  role: fits the 2- and 3-component Gaussian mixture models via EM to each cell's
    step-I values to extract the background mean µ1
  repo: https://cran.r-project.org/package=dsb
- name: R
  role: implementation environment for the mixture fits, Pearson correlations, and
    prcomp-based technical component computation
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1038/s41467-022-29356-8
    title: Normalizing and denoising protein expression data from droplet-based single
      cell profiling
  source_content_hash: sha256:522099371b48ab46b3e4fcfecd0cc4ee07d6a7df7aea541ce9474654133d70f3
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - R
  - dsb
  - mclust
  edam_operation: http://edamontology.org/operation_2238
  edam_topics:
  - http://edamontology.org/topic_3577
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9166666666666666
n_claims: 12
---

# isotype-control-correlation-analysis

## Summary

Quantifies how strongly the per-cell background protein mean (µ1, fitted via per-cell Gaussian mixture models) shares variance with isotype control antibodies and protein library size, validating the shared-variance technical component (λ, PC1) used by dsb to correct cell-to-cell technical noise in CITE-seq ADT data.

## When to use

Use when you need to verify that a cell-intrinsic technical noise estimate for droplet-based protein (ADT) data is robust — e.g. correlating fitted background means µ1 with isotype controls or checking k=2 vs k=3 mixture-model concordance before building the dsb technical component.

## When NOT to use

- The dataset has no isotype control antibody measurements — caveats for dsb in this case are deferred to the Supplementary Note and the technical component definition differs.
- The goal is ambient, protein-specific noise correction (dsb step I); that uses empty-droplet statistics, not the cell-to-cell shared variance this skill quantifies.

## Inputs

- cell protein/ADT count matrix (dsb::cells_citeseq_mtx)
- empty droplet background matrix (dsb::empty_drop_citeseq_mtx)
- isotype control antibody rows (rows 67:70 in the packaged matrices)
- per-cell protein library size values

## Outputs

- full Pearson correlation matrix across single cells (correlation_matrix.csv) covering µ1 vs mean isotype (r≈0.59), µ1 vs individual isotypes (r≈0.33), isotype-vs-isotype (r≈0.11), and mean isotype vs library size (r≈0.45)
- Pearson concordance of µ1 between k=2 and k=3 mixture fits (expected near 1)
- per-cell background mean µ1 vector from k=2 and k=3 mclust fits

## How to apply

Load the cell protein matrix (dsb::cells_citeseq_mtx) and background droplet matrix (dsb::empty_drop_citeseq_mtx), with isotype controls at rows 67:70. Perform dsb step I: log(x+10) transform both matrices, then per protein subtract the empty-droplet mean µ_N and divide by the empty-droplet standard deviation σ_N. Fit a 2-component Gaussian mixture (mclust, G=2) to each cell's step-I-corrected values via EM to extract the background mean µ1 per cell, then repeat with G=3 to check µ1 concordance between k=2 and k=3 fits. Compute Pearson correlations across single cells between µ1, the mean of the four isotype controls, each individual isotype, the isotypes themselves, and protein library size. Rationale: the paper shows isotype controls and the per-cell background protein average share variance across single cells, so their shared variance (PC1 of a noise matrix combining µ1 and isotype values) is a robust estimate of droplet/cell-specific technical noise.

## Related tools

- **dsb** (provides the packaged cell and empty-droplet CITE-seq matrices and the step-I log(x+10) transform with µ_N/σ_N background standardization) — https://cran.r-project.org/package=dsb
- **mclust** (fits the 2- and 3-component Gaussian mixture models via EM to each cell's step-I values to extract the background mean µ1) — https://cran.r-project.org/package=dsb
- **R** (implementation environment for the mixture fits, Pearson correlations, and prcomp-based technical component computation)

## Examples

```
library(dsb); library(mclust); cells <- dsb::cells_citeseq_mtx; iso <- cells[67:70, ]; mu1 <- apply(log(cells + 10), 2, function(x) mclust::Mclust(x, G = 2)$parameters$mean[1])
```

## Evaluation signals

- µ1 vs mean isotype control Pearson r ≈ 0.59 across single cells
- µ1 vs each individual isotype control Pearson r ≈ 0.33
- inter-isotype control Pearson r ≈ 0.11 (weak correlation among isotypes)
- mean isotype value vs protein library size Pearson r ≈ 0.45
- µ1 concordance between k=2 and k=3 mixture fits is high (Pearson near 1)

## Limitations

- Isotype control correlations are weak individually (r ≈ 0.11), so the shared-variance estimate relies on the combined background mean and mean isotype signal rather than any single isotype.
- Assessment details (robustness of noise estimation, empty droplet definition sensitivity, isotype caveats) are in the Supplementary Note, not the main text; usage of the technical component in experiments without isotype controls requires the dsb documentation/Supplementary Note.

## Evidence

- [abstract] isotype control antibodies and the background protein population average in each cell exhibit significant correlations across single cells, we thus use their shared variance to correct for: "isotype control antibodies and the background protein population average in each cell exhibit significant correlations across single cells, we thus use their shared variance to correct for"
- [discussion] dsb fits a Gaussian mixture model through the expectation-maximization algorithm implemented with the mclust51 R package to the transformed count of each cell from step 1 with k = 2 mixture components: "dsb fits a Gaussian mixture model through the expectation-maximization algorithm implemented with the mclust51 R package to the transformed count of each cell from step 1 with k = 2 mixture components"
- [methods] Subtract background mean $\mu_{N}$ and divide by the background standard deviation $\sigma_{N}$: "Subtract background mean $\mu_{N}$ and divide by the background standard deviation $\sigma_{N}$"
- [discussion] A noise variable matrix is then constructed by combining all the fitted µ1 values with the isotype control values for all cells. dsb then calculates principal component 1: "A noise variable matrix is then constructed by combining all the fitted µ1 values with the isotype control values for all cells. dsb then calculates principal component 1"
- [methods] # log transformation
dlog = log(cell + 10): "# log transformation
dlog = log(cell + 10)"
