---
name: wall-clock-timing-measurement
description: Use when benchmarking the computational cost of imputation pipelines
  on scRNA-seq matrices, e.g. timing SERM against baselines on the 12162x2422 observed
  matrix (data_observed_10perc.mat) or large HCL/MCA datasets.
when_to_use_negative:
- When the goal is imputation accuracy rather than runtime — use Pearson coefficient
  or clustering metrics instead
- When methods are run on different hardware or data subsets, since the timing comparison
  requires identical hardware and the identical expression matrix to be meaningful
edam_operation: http://edamontology.org/operation_0335
edam_topics:
- http://edamontology.org/topic_3170
tools:
- name: self-consistent expression recovery machine (SERM)
  role: Method being timed; invoked via srm.serm(dataObs, ROIsize, percOL, randomize=False)
    inside time.time() brackets
  repo: https://github.com/xinglab-ai/self-consistent-expression-recovery-machine
- name: MAGIC
  role: Baseline imputation method timed on the same matrix with default configurations
  repo: https://github.com/pkathail/magic
- name: mcImpute
  role: Baseline imputation method timed on the same matrix
  repo: https://github.com/aanchalMongia/McImpute_scRNAseq
- name: DeepImpute
  role: Baseline deep-learning imputation method timed on the same matrix
  repo: https://github.com/lanagarmire/deepimpute
- name: SAUCIE
  role: Baseline autoencoder imputation method timed on the same matrix
  repo: https://github.com/KrishnaswamyLab/SAUCIE
- name: SciPy
  role: Used via scipy.io.loadmat to load the .mat expression matrices that are passed
    to each timed method
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1038/s41467-022-34595-w
    title: Leveraging data-driven self-consistency for high-fidelity gene expression
      recovery
  source_content_hash: sha256:c5cce130e5b1817d7396a4cc436677d775f89e3f876e4e6830fd5d26f9295dca
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
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 15
---

# wall-clock-timing-measurement

## Summary

Measures the wall-clock runtime of scRNA-seq imputation methods (SERM vs. MAGIC, mcImpute, AutoImpute, DeepImpute) on identical hardware and data to quantify computational efficiency gains.

## When to use

Use when benchmarking the computational cost of imputation pipelines on scRNA-seq matrices, e.g. timing SERM against baselines on the 12162x2422 observed matrix (data_observed_10perc.mat) or large HCL/MCA datasets.

## When NOT to use

- When the goal is imputation accuracy rather than runtime — use Pearson coefficient or clustering metrics instead
- When methods are run on different hardware or data subsets, since the timing comparison requires identical hardware and the identical expression matrix to be meaningful

## Inputs

- Observed gene expression matrix in .mat format (e.g. data_observed_10perc.mat, 12162 cells x 2422 genes)
- Large-scale benchmark matrices (e.g. human cell landscape / mouse cell atlas, 0.6 million cells x 30,000 genes)
- Baseline imputation tool installations: MAGIC, mcImpute, DeepImpute, SAUCIE

## Outputs

- Per-method wall-clock runtime in seconds
- Comparison table of runtimes across methods
- SERM speedup ratio relative to each baseline (target: >=10x)

## How to apply

Load the expression matrix (e.g. data_observed_10perc.mat, 12162 cells x 2422 genes for the 10% sampling-efficiency benchmark) with scipy.io.loadmat, then bracket the imputation call with time.time() before and after execution, per the SERM README example. Run SERM with ROI size [6100, 1250] (a 2x2 ROI partition), percent overlap 0.5, and random.seed(0), and run MAGIC, mcImpute, DeepImpute, and SAUCIE with default configurations on the identical matrix, timing each the same way. Aggregate per-method runtimes into a comparison table and compute SERM's speedup ratio relative to each baseline. Success is judged by reproducing the paper's result: SERM processed 0.6 million cells and 30,000 genes in under 18 minutes on an Intel Core i9 PC with 64GB RAM while baselines could not finish within 3 days (>=10x speedup).

## Related tools

- **self-consistent expression recovery machine (SERM)** (Method being timed; invoked via srm.serm(dataObs, ROIsize, percOL, randomize=False) inside time.time() brackets) — https://github.com/xinglab-ai/self-consistent-expression-recovery-machine
- **MAGIC** (Baseline imputation method timed on the same matrix with default configurations) — https://github.com/pkathail/magic
- **mcImpute** (Baseline imputation method timed on the same matrix) — https://github.com/aanchalMongia/McImpute_scRNAseq
- **DeepImpute** (Baseline deep-learning imputation method timed on the same matrix) — https://github.com/lanagarmire/deepimpute
- **SAUCIE** (Baseline autoencoder imputation method timed on the same matrix) — https://github.com/KrishnaswamyLab/SAUCIE
- **SciPy** (Used via scipy.io.loadmat to load the .mat expression matrices that are passed to each timed method)

## Examples

```
import scipy.io as sio, serm as srm, random, time; dataObs=sio.loadmat('data_observed_10perc.mat')['data_observed']; t=time.time(); random.seed(0); out=srm.serm(dataObs,[6100,1250],0.5,randomize=False); print('Required time:',str(time.time()-t),'second')
```

## Evaluation signals

- Recorded SERM runtime on the 12162x2422 matrix completes in seconds-to-minutes range, consistent with the README example that prints 'Required time:' in seconds
- SERM's speedup ratio vs. each baseline is at least an order of magnitude (>=10x)
- Large-scale run: SERM completes 0.6 million cells x 30,000 genes in under 18 minutes on an Intel Core i9 PC with 64GB RAM
- All methods are timed on the identical expression matrix with the same timing pattern (time.time() before and after the imputation call)
- Random seed 0 is set before the SERM run so timing is reproducible for the fixed ROI size [6100, 1250] and percOL 0.5

## Limitations

- Baselines (MAGIC, mcImpute, AutoImpute, DeepImpute) could not complete the 0.6M-cell analysis within 3 days, so censored runtimes (">3 days") cap the measurable speedup ratio
- Timing is hardware-dependent; the reported 18-minute result applies to an Intel Core i9 PC with 64GB RAM
- Larger percent overlap between ROIs increases computation time, so runtime figures depend on the chosen ROI size and overlap parameters

## Evidence

- [results] SERM analyzed a dataset of 0.6 million cells and 30,000 genes in under 18 minutes on an Intel Core i9 PC with 64GB RAM: [evidence span withheld — anchor status: not_found]
- [readme] README timing pattern around SERM call: "t=time.time() # For keeping record of computational time"
- [readme] ROI size and overlap parameters used in the timed run: "ROIsize = [6100,1250]"
- [abstract] Claims of computational efficiency advantage: "We show that SERM improves the accuracy of gene imputation with orders of magnitude enhancement in computational efficiency in comparison to the state-of-the-art imputation techniques."
- [results] Baseline methods fail to complete the large-scale run: [evidence span withheld — anchor status: not_found]
