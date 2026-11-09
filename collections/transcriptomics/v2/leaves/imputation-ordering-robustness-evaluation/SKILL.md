---
name: imputation-ordering-robustness-evaluation
description: Use when an imputation method claims arrangement-invariant recovery of
  scRNA-seq expression values and you must verify that permuted cell/gene orderings
  yield near-identical imputation correlations with minimal accuracy loss.
when_to_use_negative:
- The imputer is deterministic in value but you only care about a single run's accuracy
  (e.g. downsampled-recovery benchmarks) — permutation testing adds no information
  there
- The reference/observed pair is not established (dropouts were not simulated at a
  controlled sampling efficiency from a filtered reference), so correlations cannot
  be meaningfully compared to a baseline
- The method explicitly reorders rows/columns internally and does not claim ordering
  invariance — evaluate on its stated metrics instead
edam_operation: http://edamontology.org/operation_2423
edam_topics:
- http://edamontology.org/topic_2229
- http://edamontology.org/topic_3170
tools:
- name: self-consistent expression recovery machine (SERM)
  role: 'The imputer under test: applied to each permuted matrix with ROIsize=[6100,1250],
    percOL=0.5, randomize=True/False'
  repo: https://github.com/xinglab-ai/self-consistent-expression-recovery-machine
- name: SciPy
  role: scipy.io.loadmat to read data_reference.mat and data_observed_10perc.mat;
    Pearson correlation computation
- name: random
  role: random.seed(0) for reproducible row/column permutations over 1000 iterations
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1038/s41467-022-34595-w
    title: Leveraging data-driven self-consistency for high-fidelity gene expression
      recovery
  source_content_hash: sha256:db4810298b65b491bdb1d52c19016cfddcfdbe8a60e2b1fa6646798ce45e97bd
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/xinglab-ai/self-consistent-expression-recovery-machine
  tools:
  - SciPy
  - random
  - self-consistent expression recovery machine (SERM)
  edam_operation: http://edamontology.org/operation_2423
  edam_topics:
  - http://edamontology.org/topic_2229
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 12
---

# imputation-ordering-robustness-evaluation

## Summary

A validation protocol that tests whether an imputation method's results depend on the arrangement of cells and genes in the expression matrix, by repeatedly permuting rows/columns, re-imputing, restoring order, and comparing per-cell Pearson correlations against a non-permuted baseline. It was used to verify SERM's bilinear-resampling-based global consistency.

## When to use

Use when an imputation method claims arrangement-invariant recovery of scRNA-seq expression values and you must verify that permuted cell/gene orderings yield near-identical imputation correlations with minimal accuracy loss.

## When NOT to use

- The imputer is deterministic in value but you only care about a single run's accuracy (e.g. downsampled-recovery benchmarks) — permutation testing adds no information there
- The reference/observed pair is not established (dropouts were not simulated at a controlled sampling efficiency from a filtered reference), so correlations cannot be meaningfully compared to a baseline
- The method explicitly reorders rows/columns internally and does not claim ordering invariance — evaluate on its stated metrics instead

## Inputs

- Reference (dropout-free) expression matrix, cells x genes, as .mat (data_reference.mat, e.g. 12,162 cells x 2,422 genes bone marrow stroma)
- Dropout-affected observed expression matrix at a given sampling efficiency as .mat (data_observed_10perc.mat, 10% efficiency)
- Imputation function (e.g. srm.serm) with a randomize option and ROI size / percent overlap parameters

## Outputs

- Per-permutation summary table of mean Pearson correlations over 1000 permutations (serm_randomization_correlations.csv)
- Baseline mean Pearson correlation from the non-permuted imputation run
- Comparison statistics showing permutation invariance (correlations near-identical to baseline; <2% accuracy drop per Supplementary Figs. S25-S29)

## How to apply

Load the dropout-free reference matrix (data_reference.mat) and the dropout-affected observed matrix (e.g. data_observed_10perc.mat at 10% sampling efficiency) with scipy.io.loadmat. Set random.seed(0), then for each of 1000 iterations randomly permute cell rows and gene columns of the observed matrix (equivalent to running srm.serm with randomize=True) and impute with the default 2x2 ROI partitioning — ROIsize=[6100,1250] and percOL=0.5 for the 12,162-cell x 2,422-gene bone marrow dataset. Restore each imputed matrix to its original ordering and compute the per-cell Pearson correlation between the imputed expression vector and the reference expression vector. Aggregate the mean Pearson correlation per permutation and compare against the non-permuted SERM run; near-identical correlations across permutations confirm invariance. Rationale: SERM interpolates all ROIs with bilinear resampling, so expression levels are similarly distributed regardless of matrix arrangement; if an imputer is not order-robust, correlations will vary substantially across permutations.

## Related tools

- **self-consistent expression recovery machine (SERM)** (The imputer under test: applied to each permuted matrix with ROIsize=[6100,1250], percOL=0.5, randomize=True/False) — https://github.com/xinglab-ai/self-consistent-expression-recovery-machine
- **SciPy** (scipy.io.loadmat to read data_reference.mat and data_observed_10perc.mat; Pearson correlation computation)
- **random** (random.seed(0) for reproducible row/column permutations over 1000 iterations)

## Examples

```
random.seed(0); out = srm.serm(permuted_obs, [6100,1250], 0.5, randomize=True)
```

## Evaluation signals

- Correlations across all 1000 permutations are near-identical to the non-permuted baseline SERM run
- Accuracy loss for permuted/sorted/single-cell-type arrangements stays under the paper's reported ~2% threshold (Supplementary Figs. S25-S29)
- Per-permutation summary table serm_randomization_correlations.csv is produced with mean Pearson correlation per permutation
- random.seed(0) reproducibility: rerunning the 1000-iteration loop reproduces the same correlation distribution
- Per-cell Pearson correlations are computed only after restoring each imputed matrix to its original row/column ordering

## Limitations

- SERM learns the gene expression model from a small fraction of the data, so on small genomic datasets the estimated distribution may diverge from the actual distribution, leading to inaccurate imputation values
- The validation relies on simulated dropouts against a reference built by choosing only cells and genes with high expressions — not experimentally observed dropouts
- SERM assumes ROI coverage is sufficiently large that the captured distribution in the ROI is representative of the entire matrix; batch effects are not considered explicitly in the formulation

## Evidence

- [readme] ROI partitioning default: "We choose ROI size in such a way that the expression matrix is divided into 4 (i.e. 2 by 2) ROIs. However, other ROI sizes are also fine as SERM is generally very robust against the ROI size."
- [intro] bilinear resampling consistency: "In the last step, all the ROIs are interpolated using bilinear resampling to achieve the global consistency of the expression values"
- [readme] randomize option: "out_SERM = srm.serm(dataObs,ROIsize,percOL,randomize=False) # If you want to randomize the rows and columns"
- [discussion] small dataset limitation: "This poses a challenge in dealing with small genomic datasets, as the estimated distribution in SERM may diverge from the"
- [discussion] ROI representativeness assumption: "An implicit assumption in our formulation is that the coverage of ROI is sufficiently large so that the captured data distribution in the ROI is representative of the entire matrix"
