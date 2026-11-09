---
name: speedup-ratio-computation
description: Use when comparing imputation-method runtimes on the same expression
  matrix to quantify a speedup ratio, e.g. verifying that SERM is >=10x faster than
  MAGIC, mcImpute, AutoImpute, or DeepImpute.
when_to_use_negative:
- When the goal is imputation accuracy (Pearson coefficient, clustering metrics) rather
  than runtime comparison — accuracy benchmarks need reference data and do not measure
  speedup.
- When methods cannot run on the same hardware or dataset size, since the ratio is
  only meaningful for identical inputs and machine configuration.
edam_operation: http://edamontology.org/operation_3438
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_0091
tools:
- name: self-consistent expression recovery machine (SERM)
  role: Method whose runtime is timed; invoked via srm.serm(dataObs, ROIsize, percOL,
    randomize=False) with seed 0
  repo: https://github.com/xinglab-ai/self-consistent-expression-recovery-machine
- name: MAGIC
  role: Baseline imputation method timed for runtime comparison against SERM
  repo: https://github.com/pkathail/magic
- name: mcImpute
  role: Baseline matrix-completion imputation method timed for runtime comparison
    against SERM
  repo: https://github.com/aanchalMongia/McImpute_scRNAseq
- name: DeepImpute
  role: Baseline deep-learning imputation method timed for runtime comparison against
    SERM
  repo: https://github.com/lanagarmire/deepimpute
- name: SAUCIE
  role: Baseline autoencoder imputation method timed for runtime comparison against
    SERM
  repo: https://github.com/KrishnaswamyLab/SAUCIE
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1038/s41467-022-34595-w
    title: Leveraging data-driven self-consistency for high-fidelity gene expression
      recovery
  source_content_hash: sha256:3cf34c7c8d450a1c651eb0646eac4c6f0ef7c620f172c1e588ff0f0d13283afe
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
  edam_operation: http://edamontology.org/operation_3438
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7272727272727273
n_claims: 11
---

# speedup-ratio-computation

## Summary

Benchmark the wall-clock runtime of SERM imputation against analytical baselines (MAGIC, mcImpute, DeepImpute, SAUCIE) on an identical scRNA-seq matrix, then compute SERM's speedup ratio per baseline to verify its reported order-of-magnitude computational efficiency gain.

## When to use

Use when comparing imputation-method runtimes on the same expression matrix to quantify a speedup ratio, e.g. verifying that SERM is >=10x faster than MAGIC, mcImpute, AutoImpute, or DeepImpute.

## When NOT to use

- When the goal is imputation accuracy (Pearson coefficient, clustering metrics) rather than runtime comparison — accuracy benchmarks need reference data and do not measure speedup.
- When methods cannot run on the same hardware or dataset size, since the ratio is only meaningful for identical inputs and machine configuration.

## Inputs

- Observed scRNA-seq gene expression matrix in .mat format (cells x genes), e.g. data_observed_10perc.mat (12162 cells x 2422 genes)
- Installed imputation methods: SERM (pip install serm), MAGIC, mcImpute, DeepImpute, SAUCIE

## Outputs

- Per-method wall-clock runtime measurements (seconds) on the identical input matrix
- Comparison table of runtimes across SERM, MAGIC, mcImpute, DeepImpute, and SAUCIE
- SERM speedup ratio relative to each baseline method

## How to apply

Load the observed gene expression matrix (e.g. data_observed_10perc.mat with 12162 cells x 2422 genes for the 10% sampling-efficiency benchmark) via scipy.io.loadmat. Run SERM with ROI size [6100, 1250] (a 2x2 ROI partition), percent overlap 0.5, and random seed 0, timing the srm.serm() call with time.time(). Run each baseline (MAGIC, mcImpute, DeepImpute, SAUCIE) with its default configuration on the identical matrix, timing each the same way. Aggregate per-method wall-clock runtimes into a comparison table and compute the speedup ratio of SERM relative to each baseline. The reported scaling target is at least an order of magnitude (>=10x); in the paper SERM imputed 0.6 million cells and 30,000 genes in under 18 minutes on an Intel Core i9 PC with 64GB RAM while the baselines could not finish within 3 days.

## Related tools

- **self-consistent expression recovery machine (SERM)** (Method whose runtime is timed; invoked via srm.serm(dataObs, ROIsize, percOL, randomize=False) with seed 0) — https://github.com/xinglab-ai/self-consistent-expression-recovery-machine
- **MAGIC** (Baseline imputation method timed for runtime comparison against SERM) — https://github.com/pkathail/magic
- **mcImpute** (Baseline matrix-completion imputation method timed for runtime comparison against SERM) — https://github.com/aanchalMongia/McImpute_scRNAseq
- **DeepImpute** (Baseline deep-learning imputation method timed for runtime comparison against SERM) — https://github.com/lanagarmire/deepimpute
- **SAUCIE** (Baseline autoencoder imputation method timed for runtime comparison against SERM) — https://github.com/KrishnaswamyLab/SAUCIE

## Examples

```
import scipy.io as sio, serm as srm, random, time; a=sio.loadmat('data_observed_10perc.mat'); t=time.time(); random.seed(0); out_SERM=srm.serm(a['data_observed'],[6100,1250],0.5,randomize=False); print(time.time()-t)
```

## Evaluation signals

- SERM's wall-clock runtime on the 12162 x 2422 benchmark is recorded around a single srm.serm() call with time.time(), with random.seed(0) set for reproducibility.
- Speedup ratio of SERM vs. each analytical baseline is at least 10x (order-of-magnitude enhancement) on identical hardware and data.
- All methods are run on the same input matrix and same machine (e.g. Intel Core i9, 64GB RAM), so ratios are computed from comparable measurements.
- Large-scale scaling check: SERM completes 0.6 million cells x 30,000 genes in under 18 minutes, while MAGIC, mcImpute, AutoImpute, and DeepImpute do not finish within 3 days.

## Limitations

- Runtime ratios are hardware- and configuration-dependent; the reference result is an Intel Core i9 PC with 64GB RAM and may not transfer to other machines.
- For very large datasets, baselines may not complete within a practical timeout (e.g. 3 days), so exact ratios must be bounded rather than measured.
- SERM's speed relies on its divide-and-conquer ROI strategy; timing includes ROI size [6100, 1250] and 50% overlap settings and may change with different parameterizations.

## Evidence

- [results] SERM analyzed a dataset of 0.6 million cells and 30,000 genes in under 18 minutes: [evidence span withheld — anchor status: not_found]
- [readme] computationally at least 10 times faster than other analytical techniques: "is computationally at least 10 times faster than other analytical techniques"
- [readme] timing with time.time around srm.serm call: "t=time.time() # For keeping record of computational time"
- [intro] Because of this divide-and-conquer strategy, SERM is scalable to large datasets: "Because of this divide-and-conquer strategy, SERM is scalable to large datasets and can impute the datasets much faster than other analytical or deep learning-based methods."
