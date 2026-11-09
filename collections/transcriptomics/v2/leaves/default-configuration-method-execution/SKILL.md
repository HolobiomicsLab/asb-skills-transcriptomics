---
name: default-configuration-method-execution
description: Use when comparing wall-clock runtime of an imputation method against
  analytical or deep-learning baselines on the same expression matrix, or when reproducing
  the reported >=10x efficiency gain of SERM.
when_to_use_negative:
- The goal is imputation accuracy (Pearson coefficient, clustering indices) rather
  than computational runtime — use the accuracy-benchmarking skill instead
- Datasets are small enough that all methods complete in comparable time, making the
  runtime contrast uninformative
- The target is SERM's internal steps (ROI selection, histogram equalization, bilinear
  resampling) rather than end-to-end runtime comparison
edam_operation: http://edamontology.org/operation_0335
edam_topics:
- http://edamontology.org/topic_2229
tools:
- name: self-consistent expression recovery machine (SERM)
  role: Primary method whose runtime is measured; installed via pip and called as
    srm.serm(dataObs, ROIsize, percOL, randomize=False)
  repo: https://github.com/xinglab-ai/self-consistent-expression-recovery-machine
- name: MAGIC
  role: Default-configuration baseline imputation method timed for runtime comparison
  repo: https://github.com/pkathail/magic
- name: mcImpute
  role: Default-configuration baseline using nuclear norm minimization, timed on the
    identical matrix
  repo: https://github.com/aanchalMongia/McImpute_scRNAseq
- name: DeepImpute
  role: Default-configuration deep-learning baseline timed for runtime comparison
  repo: https://github.com/lanagarmire/deepimpute
- name: SAUCIE
  role: Default-configuration TensorFlow autoencoder baseline timed for runtime comparison
  repo: https://github.com/KrishnaswamyLab/SAUCIE
- name: SciPy
  role: Used via scipy.io.loadmat to load the .mat expression matrices
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1038/s41467-022-34595-w
    title: Leveraging data-driven self-consistency for high-fidelity gene expression
      recovery
  source_content_hash: sha256:b783e517ed29f0f5ad2e2d7c74e573b0b23a3e0e7109c499da755d82b8c0351a
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/xinglab-ai/self-consistent-expression-recovery-machine
  tools:
  - DeepImpute
  - MAGIC
  - MATLAB
  - SAUCIE
  - SciPy
  - mcImpute
  - self-consistent expression recovery machine (SERM)
  edam_operation: http://edamontology.org/operation_0335
  edam_topics:
  - http://edamontology.org/topic_2229
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5833333333333334
n_claims: 12
---

# default-configuration-method-execution

## Summary

Benchmark computational runtime of SERM against default-configured imputation baselines (MAGIC, mcImpute, AutoImpute, DeepImpute, SAUCIE) on identical scRNA-seq matrices, timing each run to quantify SERM's >=10x speed advantage.

## When to use

Use when comparing wall-clock runtime of an imputation method against analytical or deep-learning baselines on the same expression matrix, or when reproducing the reported >=10x efficiency gain of SERM.

## When NOT to use

- The goal is imputation accuracy (Pearson coefficient, clustering indices) rather than computational runtime — use the accuracy-benchmarking skill instead
- Datasets are small enough that all methods complete in comparable time, making the runtime contrast uninformative
- The target is SERM's internal steps (ROI selection, histogram equalization, bilinear resampling) rather than end-to-end runtime comparison

## Inputs

- Observed (dropout-affected) scRNA-seq expression matrix as a .mat file, e.g. data_observed_10perc.mat (12162 cells x 2422 genes)
- Baseline imputation tool installations: MAGIC, mcImpute, DeepImpute, SAUCIE (and optionally AutoImpute)

## Outputs

- Per-method wall-clock runtimes (seconds) recorded around each imputation call
- Comparison table of runtimes with SERM speedup ratios relative to each baseline

## How to apply

Load the observed gene expression matrix (e.g. data_observed_10perc.mat, 12162 cells x 2422 genes) with scipy.io.loadmat, then run SERM via srm.serm(dataObs, ROIsize, percOL, randomize=False) with ROI size [6100, 1250] (a 2x2 ROI partition), percent overlap 0.5, and random seed 0, wrapping the call with time.time() to record wall-clock time. Run MAGIC, mcImpute, DeepImpute, and SAUCIE on the identical matrix using each tool's default configuration from its published repository, timing each the same way. Aggregate the per-method runtimes into a comparison table and compute the speedup ratio of SERM over each baseline. The rationale is that baselines in the paper could not finish 0.6 million cells x 30,000 genes within 3 days while SERM finished in under 18 minutes, so timing must be measured on the same hardware (Intel Core i9, 64GB RAM) for a fair comparison.

## Related tools

- **self-consistent expression recovery machine (SERM)** (Primary method whose runtime is measured; installed via pip and called as srm.serm(dataObs, ROIsize, percOL, randomize=False)) — https://github.com/xinglab-ai/self-consistent-expression-recovery-machine
- **MAGIC** (Default-configuration baseline imputation method timed for runtime comparison) — https://github.com/pkathail/magic
- **mcImpute** (Default-configuration baseline using nuclear norm minimization, timed on the identical matrix) — https://github.com/aanchalMongia/McImpute_scRNAseq
- **DeepImpute** (Default-configuration deep-learning baseline timed for runtime comparison) — https://github.com/lanagarmire/deepimpute
- **SAUCIE** (Default-configuration TensorFlow autoencoder baseline timed for runtime comparison) — https://github.com/KrishnaswamyLab/SAUCIE
- **SciPy** (Used via scipy.io.loadmat to load the .mat expression matrices)

## Examples

```
import scipy.io as sio, serm as srm, random, time; dataObs = sio.loadmat('data_observed_10perc.mat')['data_observed']; ROIsize=[6100,1250]; percOL=0.5; t=time.time(); random.seed(0); out_SERM = srm.serm(dataObs, ROIsize, percOL, randomize=False); print('Required time:', str(time.time()-t), 'second')
```

## Evaluation signals

- SERM's recorded runtime is at least an order of magnitude (>=10x) smaller than each baseline's runtime on the same matrix and hardware
- On the 0.6M cell x 30,000 gene dataset, SERM completes in under 18 minutes while MAGIC, mcImpute, AutoImpute, and DeepImpute do not finish within 3 days
- Wall-clock timing wraps the imputation call itself (time.time() before and after srm.serm) with random seed 0 set for reproducibility
- All methods receive the byte-identical observed matrix (e.g. data_observed_10perc.mat) so runtime differences are attributable to the algorithms

## Limitations

- Baseline runtimes are hardware-dependent; the reported contrast (under 18 minutes vs. >3 days) was obtained on an Intel Core i9 PC with 64GB RAM and may differ elsewhere
- Baselines run with default configurations may not be optimally tuned, which could exaggerate or alter the runtime gap
- DeepImpute's README notes it was implemented in Python2 and Python3 (Python3 recommended), so environment mismatches can confound cross-method timing

## Evidence

- [other] SERM analyzed a dataset of 0.6 million cells and 30,000 genes in under 18 minutes on an Intel Core i9 PC with 64GB RAM, whereas MAGIC, mcImpute, AutoImpute, and DeepImpute could not complete the same analysis within 3 days.: [evidence span withheld — anchor status: not_found]
- [abstract] We show that SERM improves the accuracy of gene imputation with orders of magnitude enhancement in computational efficiency in comparison to the state-of-the-art imputation techniques.: "We show that SERM improves the accuracy of gene imputation with orders of magnitude enhancement in computational efficiency in comparison to the state-of-the-art imputation techniques."
- [methods] MAGIC was downloaded from https://github.com/pkathail/magic.: "MAGIC was downloaded from https://github.com/pkathail/magic."
- [methods] mcImpute, DeepImpute, and SAUCIE were downloaded from https://github.com/aanchalMongia/McImpute_scRNAseq, https://github.com/lanagarmire/deepimpute, https://github.com/KrishnaswamyLab/SAUCIE.: "mcImpute, DeepImpute, and SAUCIE were downloaded from https://github.com/aanchalMongia/McImpute_scRNAseq, https://github.com/lanagarmire/deepimpute, https://github.com/KrishnaswamyLab/SAUCIE."
- [readme] t=time.time() # For keeping record of computational time: "t=time.time() # For keeping record of computational time"
