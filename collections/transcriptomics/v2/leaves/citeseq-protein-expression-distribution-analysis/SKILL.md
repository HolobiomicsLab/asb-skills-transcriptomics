---
name: citeseq-protein-expression-distribution-analysis
description: Use when CITE-seq/ADT counts have been dsb step-I normalized (log10 pseudocount
  10, standardized by empty-droplet background mean/SD) and you need to determine
  the number of Gaussian components per cell or extract the fitted background mean
  µ1.
when_to_use_negative:
- Input is raw, un-normalized ADT counts that have not undergone dsb step-I ambient
  correction (log transform with pseudocount 10 plus empty-droplet standardization)
  — mixture fits would not reflect the paper's model
- Analysis goal is mRNA expression modeling rather than antibody-derived-tag (ADT)
  protein distributions from droplet-based assays
edam_operation: http://edamontology.org/operation_3659
edam_topics:
- http://edamontology.org/topic_2229
tools:
- name: dsb
  role: Provides example CITE-seq matrices and implements step-I ambient background
    correction via DSBNormalizeProtein()
  repo: https://cran.r-project.org/package=dsb
- name: mclust
  role: Fits Gaussian mixture models (G = 1..6) per cell via the EM algorithm and
    returns BIC used for model selection
- name: R
  role: Analysis environment in which dsb and mclust are run
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1038/s41467-022-29356-8
    title: Normalizing and denoising protein expression data from droplet-based single
      cell profiling
  source_content_hash: sha256:88859f2d13e3d16edf9505ac85cfb120ebc64d94f5b75eb7d7c83fc79c641fd0
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - R
  - dsb
  - mclust
  edam_operation: http://edamontology.org/operation_3659
  edam_topics:
  - http://edamontology.org/topic_2229
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8
n_claims: 5
---

# citeseq-protein-expression-distribution-analysis

## Summary

Model the within-cell distribution of dsb step-I ambient-corrected CITE-seq protein counts with Gaussian mixture models (k=1–6, mclust EM) and select the best fit per cell by BIC, revealing a predominantly bimodal (k=2) protein distribution.

## When to use

Use when CITE-seq/ADT counts have been dsb step-I normalized (log10 pseudocount 10, standardized by empty-droplet background mean/SD) and you need to determine the number of Gaussian components per cell or extract the fitted background mean µ1.

## When NOT to use

- Input is raw, un-normalized ADT counts that have not undergone dsb step-I ambient correction (log transform with pseudocount 10 plus empty-droplet standardization) — mixture fits would not reflect the paper's model
- Analysis goal is mRNA expression modeling rather than antibody-derived-tag (ADT) protein distributions from droplet-based assays

## Inputs

- dsb::cells_citeseq_mtx (cell-containing droplet ADT count matrix)
- dsb::empty_drop_citeseq_mtx (empty droplet ADT count matrix for background mean/SD)
- Raw and filtered Cell Ranger ADT UMI matrices from external CITE-seq datasets (10X Genomics, Mission Bio, ASAP-seq, TEA-seq)

## Outputs

- Per-cell BIC values for Gaussian mixture models with k=1..6 components
- Per-cell optimal k (highest BIC)
- Fitted two-component mixture background mean µ1 per cell (used in dsb step II)
- Summary table of best-fit-k proportions written to best_fit_k_proportions.csv

## How to apply

First compute step-I corrected values: log-transform protein counts with pseudocount 10 (log(cell + 10)), then subtract the empty-droplet background mean and divide by the background standard deviation for each antibody. For each single cell, fit Gaussian mixture models with mclust using G = 1 through 6 mixing components via the EM algorithm, and extract the BIC per k (mclust convention: higher BIC is better). Select the optimal k per cell as the model with the highest BIC, then compute the proportion of cells where k=2 is optimal. The paper's dsb step II uses the fitted two-component mixture's background mean µ1 as a noise variable; validation is that k=2 is optimal in >85% of cells (81% in batch-1 cells and an average of 89% across four external CITE-seq datasets).

## Related tools

- **dsb** (Provides example CITE-seq matrices and implements step-I ambient background correction via DSBNormalizeProtein()) — https://cran.r-project.org/package=dsb
- **mclust** (Fits Gaussian mixture models (G = 1..6) per cell via the EM algorithm and returns BIC used for model selection)
- **R** (Analysis environment in which dsb and mclust are run)

## Examples

```
dlog = log(cell + 10); dsb_rescaled <- DSBNormalizeProtein(cell_protein_matrix = cells_citeseq_mtx, empty_drop_matrix = empty_drop_citeseq_mtx)
```

## Evaluation signals

- Proportion of cells with optimal k=2 should exceed 85% (paper reports 81% in batch-1 cells and 89% average across four external CITE-seq datasets)
- mclust BIC comparison uses the convention that higher BIC is better; the per-cell chosen model must have the maximum BIC across G=1..6
- best_fit_k_proportions.csv must be produced and contain per-dataset proportions of best-fit k
- Step-I corrected values must equal standardization of log(cell + 10) by empty-droplet background mean and standard deviation for each protein

## Limitations

- dsb normalized values are sensitive to how background droplets are defined; 'defining a reasonable subset of barcodes as background droplets still requires care'
- Assessment details on modeling assumptions, caveats and limitations are deferred to the Supplementary Note rather than the main text
- Proteins with very low raw signal (maximum UMI count < 5 across all cells) should be removed prior to normalization (e.g., CD34 removed from two datasets)

## Evidence

- [discussion] dsb fits a Gaussian mixture model through the expectation-maximization algorithm implemented with the mclust51 R package: "dsb fits a Gaussian mixture model through the expectation-maximization algorithm implemented with the mclust51 R package"
- [methods] dsb fits a Gaussian mixture with 2 mixing components to model the ADT levels in each cell: "dsb fits a Gaussian mixture with 2 mixing components to model the ADT levels in each cell"
- [methods] log transformation of ADT counts with pseudocount 10: "# log transformation
dlog = log(cell + 10)"
- [methods] Standardization by background mean and standard deviation: "Subtract background mean $\mu_{N}$ and divide by the background standard deviation $\sigma_{N}$"
- [discussion] Proteins with very low raw data signal removed prior to normalization: "Proteins with very low raw data signal (a maximum UMI count <5 across all cells) were removed prior to normalization"
- [discussion] dsb fits GMM to transformed counts of each cell from step 1 with k = 2 mixture components: "to the transformed count of each cell from step 1 with k = 2 mixture components"
