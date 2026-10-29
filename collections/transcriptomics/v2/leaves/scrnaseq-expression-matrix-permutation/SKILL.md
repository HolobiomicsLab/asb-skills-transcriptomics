---
name: scrnaseq-expression-matrix-permutation
description: Use when you must verify that a matrix-imputation pipeline (e.g., SERM)
  is invariant to the arrangement of cells and genes — i.e., row/column permutations
  of a scRNA-seq count matrix yield near-identical Pearson correlations to the reference.
when_to_use_negative:
- The dataset is a small genomic dataset — SERM learns its distribution from a small
  fraction of the data, so the estimated distribution may diverge and imputed values
  may be inaccurate before any permutation test is meaningful
- The goal is accuracy benchmarking against competing imputers (SAVER, MAGIC, scImpute,
  DeepImpute, etc.) rather than testing arrangement/ordering invariance
- The input is not a cells x genes expression matrix with a matched dropout-free reference,
  since per-cell Pearson validation against the reference is required
edam_operation: http://edamontology.org/operation_2945
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: self-consistent expression recovery machine (SERM)
  role: Imputation method under test; called as srm.serm(dataObs, ROIsize, percOL,
    randomize=True) on each permuted matrix with 2x2 ROI partitioning and 50% overlap
  repo: https://github.com/xinglab-ai/self-consistent-expression-recovery-machine
- name: SciPy
  role: scipy.io.loadmat reads the .mat reference and observed matrices and provides
    numerical handling of the cells-by-genes arrays
- name: random
  role: random.seed(0) plus random permutations of cell rows and gene columns across
    1000 reproducible iterations
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1038/s41467-022-34595-w
    title: Leveraging data-driven self-consistency for high-fidelity gene expression
      recovery
  source_content_hash: sha256:89630878ba0972cd0d2017f015dc8a2347d003e90023f09bbb814fd2e7767448
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/xinglab-ai/self-consistent-expression-recovery-machine
  tools:
  - SciPy
  - random
  - self-consistent expression recovery machine (SERM)
  edam_operation: http://edamontology.org/operation_2945
  edam_topics:
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7
n_claims: 10
---

# scrnaseq-expression-matrix-permutation

## Summary

A robustness-validation skill for the SERM gene expression recovery framework: randomly permute cell rows and gene columns of a dropout-affected cells-by-genes matrix, re-impute each permutation, and confirm imputed expression levels are similarly distributed regardless of matrix arrangement.

## When to use

Use when you must verify that a matrix-imputation pipeline (e.g., SERM) is invariant to the arrangement of cells and genes — i.e., row/column permutations of a scRNA-seq count matrix yield near-identical Pearson correlations to the reference.

## When NOT to use

- The dataset is a small genomic dataset — SERM learns its distribution from a small fraction of the data, so the estimated distribution may diverge and imputed values may be inaccurate before any permutation test is meaningful
- The goal is accuracy benchmarking against competing imputers (SAVER, MAGIC, scImpute, DeepImpute, etc.) rather than testing arrangement/ordering invariance
- The input is not a cells x genes expression matrix with a matched dropout-free reference, since per-cell Pearson validation against the reference is required

## Inputs

- data_reference.mat: dropout-free reference expression matrix (cells x genes; 12,162 cells x 2,422 genes for the bone marrow stroma dataset)
- data_observed_10perc.mat: dropout-affected observed matrix generated at 10% sampling efficiency via gamma-scaled Poisson sampling
- serm Python package (pip install serm) and Python's random module for seeded permutations

## Outputs

- serm_randomization_correlations.csv: per-permutation (n=1000) mean Pearson correlation summary table
- Per-cell Pearson correlation vectors between each permuted-then-restored imputed matrix and the reference matrix
- Baseline mean Pearson correlation from the non-permuted SERM run (randomize=False) for comparison

## How to apply

Load the reference matrix (data_reference.mat) and a dropout-affected observed matrix (e.g., data_observed_10perc.mat at 10% sampling efficiency) with scipy.io.loadmat; the bone-marrow benchmark contains 12,162 cells x 2,422 genes. Set random.seed(0) and, over ~1000 iterations, randomly permute the cell rows and gene columns of the observed matrix (equivalent to calling srm.serm with randomize=True). Impute each permuted matrix with SERM using the default 2x2 ROI partitioning (ROIsize = [6100, 1250]) and 50% ROI overlap (percOL = 0.5); SERM achieves global consistency by interpolating all ROIs with bilinear resampling, which is the mechanistic reason arrangement should not matter. Restore each imputed matrix to its original row/column ordering and compute the per-cell Pearson correlation between the imputed expression vector and the corresponding reference expression vector. Aggregate the mean Pearson correlation per permutation, compare against the non-permuted (randomize=False) SERM baseline, and write the per-permutation summary table to serm_randomization_correlations.csv.

## Related tools

- **self-consistent expression recovery machine (SERM)** (Imputation method under test; called as srm.serm(dataObs, ROIsize, percOL, randomize=True) on each permuted matrix with 2x2 ROI partitioning and 50% overlap) — https://github.com/xinglab-ai/self-consistent-expression-recovery-machine
- **SciPy** (scipy.io.loadmat reads the .mat reference and observed matrices and provides numerical handling of the cells-by-genes arrays)
- **random** (random.seed(0) plus random permutations of cell rows and gene columns across 1000 reproducible iterations)

## Examples

```
random.seed(0); out_SERM = srm.serm(dataObs, [6100,1250], 0.5, randomize=True)
```

## Evaluation signals

- Mean Pearson correlations across all 1000 permutations are near-identical to the non-permuted SERM baseline correlation
- Accuracy drop stays below the ~2% reported for sorted/single-cell-type arrangements (Supplementary Figs. S25–S29)
- Each imputed matrix, after inverse row/column permutation, aligns cell-for-cell and gene-for-gene with the original ordering of data_reference.mat
- Re-running with random.seed(0) reproduces the same per-permutation correlation table in serm_randomization_correlations.csv
- Same ROI parameters (ROIsize=[6100,1250], percOL=0.5) are used across permutations so only the matrix arrangement varies

## Limitations

- SERM learns the gene expression model from a small fraction of the data, so on small genomic datasets the estimated distribution may diverge from the actual data distribution, leading to inaccurate imputation
- The formulation implicitly assumes ROI coverage is sufficiently large that the captured data distribution in each ROI is representative of the entire matrix
- The reference data were created by filtering to only cells and genes with high expressions, and dropouts were simulated (gamma-Poisson at 10% efficiency) rather than observed experimentally

## Evidence

- [readme] Randomize rows/columns via srm.serm: "If you want to randomize the rows and columns of the data before applying SERM, please choose randomize=True"
- [intro] Bilinear resampling gives global consistency: "In the last step, all the ROIs are interpolated using bilinear resampling to achieve the global consistency of the expression values"
- [readme] Self-consistency ensures similar distribution across the matrix: "SERM then recovers the overall gene expression data by analytically imposing a self- consistency on the gene expression matrix"
- [discussion] ROI coverage representativeness assumption: "An implicit assumption in our formulation is that the coverage of ROI is sufficiently large so that the captured data distribution in the ROI is representative of the entire matrix"
- [discussion] Small-dataset distribution divergence risk: "This poses a challenge in dealing with small genomic datasets, as the estimated distribution in SERM may diverge from the"
