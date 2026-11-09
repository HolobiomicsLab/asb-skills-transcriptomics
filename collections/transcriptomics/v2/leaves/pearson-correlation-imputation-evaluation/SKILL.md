---
name: pearson-correlation-imputation-evaluation
description: Use when comparing gene imputation methods on paired observed/reference
  matrices (e.g. SERM vs MAGIC, SAVER, scVI on the cellular taxonomy dataset) and
  you need a quantitative accuracy score per sampling efficiency.
when_to_use_negative:
- When no dropout-free reference matrix is available — the Pearson evaluation requires
  paired reference and imputed data and cannot be run on experimental data with unknown
  ground truth
- When the goal is evaluating downstream biological utility (e.g. clustering accuracy,
  NMI, AR, Hubert indices) rather than raw expression recovery accuracy
- When evaluating methods on very small genomic datasets, where the reference constructed
  from only high-expression cells and genes may not represent the true distribution
edam_operation: http://edamontology.org/operation_0276
edam_topics:
- http://edamontology.org/topic_0203
tools:
- name: self-consistent expression recovery machine (SERM)
  role: Primary imputation method being evaluated; run via srm.serm(dataObs, ROIsize=[6100,1250],
    percOL=0.5, randomize=False) on each observed matrix
  repo: https://github.com/xinglab-ai/self-consistent-expression-recovery-machine
- name: MAGIC
  role: Baseline imputation method compared against SERM using the Pearson-improvement
    benchmark
  repo: https://github.com/pkathail/magic
- name: mcImpute
  role: Baseline imputation method compared against SERM using the Pearson-improvement
    benchmark
  repo: https://github.com/aanchalMongia/McImpute_scRNAseq
- name: DeepImpute
  role: Baseline imputation method compared against SERM using the Pearson-improvement
    benchmark
  repo: https://github.com/lanagarmire/deepimpute
- name: SAUCIE
  role: Baseline imputation method compared against SERM using the Pearson-improvement
    benchmark
  repo: https://github.com/KrishnaswamyLab/SAUCIE
- name: SAVER
  role: Baseline imputation method referenced among prominent methods of the same
    imputation group
- name: SciPy
  role: Used with scipy.io.loadmat to load the .mat reference and observed matrices
    for evaluation
provenance:
  source_task_ids:
  - task_008
  - task_001
  source_papers:
  - doi: 10.1038/s41467-022-34595-w
    title: Leveraging data-driven self-consistency for high-fidelity gene expression
      recovery
  source_content_hash: sha256:f4c1eb98dcacebfd135e086ec109acb293160807bc9bbf7239894fe1d0ba6cde
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/xinglab-ai/self-consistent-expression-recovery-machine
  tools:
  - AutoImpute
  - DCA
  - DeepImpute
  - MAGIC
  - SAUCIE
  - SAVER
  - SERM
  - SciPy
  - mcImpute
  - scScope
  - scVI
  - self-consistent expression recovery machine (SERM)
  edam_operation: http://edamontology.org/operation_0276
  edam_topics:
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8571428571428571
n_claims: 21
---

# pearson-correlation-imputation-evaluation

## Summary

Benchmark scRNA-seq imputation methods by computing per-cell Pearson correlation coefficients between imputed and dropout-free reference expression vectors, then aggregating into percent improvement over observed data across sampling efficiencies (0.1%–10%).

## When to use

Use when comparing gene imputation methods on paired observed/reference matrices (e.g. SERM vs MAGIC, SAVER, scVI on the cellular taxonomy dataset) and you need a quantitative accuracy score per sampling efficiency.

## When NOT to use

- When no dropout-free reference matrix is available — the Pearson evaluation requires paired reference and imputed data and cannot be run on experimental data with unknown ground truth
- When the goal is evaluating downstream biological utility (e.g. clustering accuracy, NMI, AR, Hubert indices) rather than raw expression recovery accuracy
- When evaluating methods on very small genomic datasets, where the reference constructed from only high-expression cells and genes may not represent the true distribution

## Inputs

- data_reference.mat — dropout-free reference expression matrix (cells x genes, e.g. 12,162 cells x 2,422 genes for cellular taxonomy)
- data_observed_*.mat — observed matrices with simulated dropouts at 0.1%-10% sampling efficiencies (Poisson sampling with Gamma-drawn cell efficiencies, e.g. Gamma(10,100) for 10%)
- imputed expression matrices from SERM and baseline methods (MAGIC, mcImpute, DeepImpute, SAUCIE, AutoImpute, DCA, scScope, scVI, SAVER)

## Outputs

- Per-cell Pearson correlation coefficients between imputed and reference expression vectors
- Per-method, per-sampling-efficiency table of percent improvement in mean Pearson coefficient (e.g. pearson_improvement_results.csv)
- Per-gene Pearson scatter plots at each sampling efficiency

## How to apply

Load the dropout-free reference matrix and dropout-affected observed matrices (e.g. data_reference.mat and data_observed_*.mat via scipy.io.loadmat), then run each imputation method (SERM with ROIsize=[6100,1250], percOL=0.5, randomize=False; plus MAGIC, mcImpute, DeepImpute, SAUCIE, AutoImpute, DCA, scScope, scVI, SAVER with default configurations). For every cell, standardize the imputed and reference expression vectors and compute their Pearson correlation coefficient; average the per-cell coefficients per method and sampling efficiency. Convert each method's mean rho into percent improvement over observed data, 100 x (rho_method - rho_observed)/rho_observed, to quantify gain relative to the un-imputed baseline. SERM's performance is judged by >20% improvement in most cases; per-gene Pearson scatter plots (e.g. >0.9 for most genes at 5% efficiency) serve as a secondary check.

## Related tools

- **self-consistent expression recovery machine (SERM)** (Primary imputation method being evaluated; run via srm.serm(dataObs, ROIsize=[6100,1250], percOL=0.5, randomize=False) on each observed matrix) — https://github.com/xinglab-ai/self-consistent-expression-recovery-machine
- **MAGIC** (Baseline imputation method compared against SERM using the Pearson-improvement benchmark) — https://github.com/pkathail/magic
- **mcImpute** (Baseline imputation method compared against SERM using the Pearson-improvement benchmark) — https://github.com/aanchalMongia/McImpute_scRNAseq
- **DeepImpute** (Baseline imputation method compared against SERM using the Pearson-improvement benchmark) — https://github.com/lanagarmire/deepimpute
- **SAUCIE** (Baseline imputation method compared against SERM using the Pearson-improvement benchmark) — https://github.com/KrishnaswamyLab/SAUCIE
- **SAVER** (Baseline imputation method referenced among prominent methods of the same imputation group)
- **SciPy** (Used with scipy.io.loadmat to load the .mat reference and observed matrices for evaluation)

## Examples

```
import scipy.io as sio, serm as srm, random; dataRef = sio.loadmat('data_reference.mat')['data_reference']; dataObs = sio.loadmat('data_observed_10perc.mat')['data_observed']; random.seed(0); out_SERM = srm.serm(dataObs, [6100,1250], 0.5, randomize=False)
```

## Evaluation signals

- The percent-improvement table contains an entry for every method x sampling-efficiency combination (0.1%-10%) on the cellular taxonomy dataset
- Percent improvement is computed as 100 x (rho_method - rho_observed)/rho_observed and is positive for SERM, exceeding 20% in most cases
- Per-gene Pearson coefficients for SERM at 5% sampling efficiency are above 0.9 for most genes, versus <=0.4 for most other methods
- At 0.1% downsampling, SERM Pearson coefficients fall within the reported 0.15-0.65 range, exceeding all baseline methods
- SERM-computed computational time (recorded around the srm.serm call) is at least 10x faster than other analytical techniques, alongside its accuracy advantage

## Limitations

- Reference data were created by choosing only cells and genes with high expressions, and dropouts were simulated (Poisson sampling with Gamma-drawn efficiencies) rather than observed experimentally
- The evaluation inherits SERM's implicit assumption that ROI coverage is sufficiently large so the captured data distribution is representative of the entire matrix
- Observed-data generation parameters depend on the sampling-efficiency regime (e.g. alpha <- rgamma(ncol(data.filt), 10, 100) for 10%), so results are only comparable at matched efficiencies

## Evidence

- [results] Percent improvement in mean Pearson coefficient of the imputed data (in comparison to the observed data) by eight different techniques for cellular taxonomy dataset.: "Percent improvement in mean Pearson coefficient of the imputed data (in comparison to the observed data) by eight different techniques for cellular taxonomy  dataset."
- [readme] Percent improvement computation formula: "SERM performs much better (in most cases >20% improvement than State-of-the-art) and is computationally at least 10 times faster than other analytical techniques."
- [readme] Reference data filtering rule: "we choose 12,162 cells and 2,422 genes with high expression to create a reference dataset."
- [readme] Observed data generation via Poisson sampling with gamma alpha: "alpha <- rgamma(ncol(data.filt), 10, 100)"
- [results] Per-gene Pearson check at 5% efficiency: "For the case of 5% sampling efficiency, SERM provides a Pearson coefficient better than 0.9 for most of the genes"
- [results] Pearson range at 0.1% downsampling: "For the case of 0.1% downsampling, SERM recovers the original expression values with Pearson coefficients of 0.15–0.65"
- [discussion] Simulation caveat from discussion: "the reference data created by choosing only the cells and genes with high expressions provides a dropout-free representation. The observed data is created from the reference data by simulating"
