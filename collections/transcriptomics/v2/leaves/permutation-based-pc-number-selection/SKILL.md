---
name: permutation-based-pc-number-selection
description: Use when a scRNAseq pipeline needs to decide how many PCs to retain before
  clustering, and the choice must be justified by a shuffled-data null instead of
  an arbitrary elbow/scree cutoff.
when_to_use_negative:
- Input is raw 10X data that has not yet passed read.data quality filtering and variable-gene
  selection — run those steps first.
- The analysis requires a predetermined, fixed number of PCs (e.g., for strict reproducibility
  across pipelines) rather than a data-driven, permutation-justified choice.
- Working outside R/robustSingleCell with a feature table already reduced by another
  PCA implementation — the shuffled-null selection is specific to this function.
edam_operation: http://edamontology.org/operation_2939
edam_topics:
- http://edamontology.org/topic_3170
tools:
- name: robustSingleCell
  role: Provides the PCA function that performs multiple simulation analyses of shuffled
    data to determine the appropriate number of PCs
  repo: github.com/asmagen/robustSingleCell
- name: R
  role: Runtime environment in which the robustSingleCell PCA step is executed
- name: slurm
  role: Parallel batch job manager used when running shuffled simulations non-locally
    (local=F)
  repo: https://slurm.schedmd.com
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1016/j.celrep.2019.10.131
    title: Single-Cell Profiling Defines Transcriptomic Signatures Specific to Tumor-Reactive
      versus Virus-Responsive CD4+ T Cells
  source_content_hash: sha256:995a1925588cd04af250fff67197c63ad0de237e2c5b3edf4784e1fe5d724575
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - R
  - robustSingleCell
  - slurm
  edam_operation: http://edamontology.org/operation_2939
  edam_topics:
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.42857142857142855
n_claims: 7
---

# permutation-based-pc-number-selection

## Summary

robustSingleCell's PCA step determines the appropriate number of principal components by running multiple simulation analyses on shuffled (permuted) data, so PC retention is chosen against a simulated null rather than fixed heuristics.

## When to use

Use when a scRNAseq pipeline needs to decide how many PCs to retain before clustering, and the choice must be justified by a shuffled-data null instead of an arbitrary elbow/scree cutoff.

## When NOT to use

- Input is raw 10X data that has not yet passed read.data quality filtering and variable-gene selection — run those steps first.
- The analysis requires a predetermined, fixed number of PCs (e.g., for strict reproducibility across pipelines) rather than a data-driven, permutation-justified choice.
- Working outside R/robustSingleCell with a feature table already reduced by another PCA implementation — the shuffled-null selection is specific to this function.

## Inputs

- quality-filtered robustSingleCell project environment (e.g., LCMV1) after read.data(subsample=500)
- highly variable genes matrix from get.variable.genes
- confounder variables (ribosomal.score, mitochondrial.score, cell.cycle.score, Exhaustion)

## Outputs

- PCA-reduced environment with the permutation-determined number of retained PCs
- pca_results.txt recording the retained principal component count
- PCA diagnostic figures (PCA.png, rotation.png, PC1_heatmap.png) in the analysis folder

## How to apply

After loading the quality-filtered environment (e.g., the 500-cell LCMV1 subsample from read.data with subsample=500) in which highly variable genes were identified via get.variable.genes and confounder scores (ribosomal, mitochondrial, cell cycle, Exhaustion via controlled.mean.score) were added, run the robustSingleCell PCA function with local=T so that multiple simulations of shuffled data are performed to determine the appropriate number of PCs. Use local=F to run each shuffled simulation in parallel on slurm for larger workloads. Inspect the permutation-based PC selection output (e.g., pca_results.txt) and record the retained PC count; downstream diagnostics (PCA.png, rotation.png, PC1_heatmap.png) are generated later by the summarize function. The rationale is that PCs with variance no greater than the simulated null carry little reproducible signal, so only PCs exceeding the shuffled-data comparison should feed clustering.

## Related tools

- **robustSingleCell** (Provides the PCA function that performs multiple simulation analyses of shuffled data to determine the appropriate number of PCs) — github.com/asmagen/robustSingleCell
- **R** (Runtime environment in which the robustSingleCell PCA step is executed)
- **slurm** (Parallel batch job manager used when running shuffled simulations non-locally (local=F)) — https://slurm.schedmd.com

## Examples

```
LCMV1 <- PCA(LCMV1, local = T)
```

## Evaluation signals

- The retained PC count in pca_results.txt is consistent with the simulated-null comparison produced by the PCA run (permutation test supports the choice).
- The environment entering PCA already contains get.variable.genes output and confounder variables (ribosomal, mitochondrial, cell cycle, Exhaustion scores).
- PCA diagnostic figures (PCA.png, rotation.png, PC1_heatmap.png) are generated in the analysis folder by the summarize routine.
- For large datasets, simulations were distributed via local=F on slurm rather than run locally.

## Limitations

- The robustSingleCell package is not currently maintained, per the README.
- The pipeline currently supports slurm for parallel batch jobs; other schedulers are not covered.
- The tutorial analysis requires at least 8G of memory on the slurm workload manager.
- PCA figures are only produced in a later summarize step, not by PCA itself.

## Evidence

- [methods] PCA function performs permutation-based selection: "The `PCA` function performs multiple simulation analyses of shuffled data to determine the appropriate number of PCs."
- [readme] PCA invocation from README: "The `PCA` function performs multiple simulation analyses of shuffled
  data to determine the appropriate number of PCs. You can also run each
  simulation in parallel using the option `local = F`."
- [readme] slurm support: "This pipeline currently supports [slurm](https://slurm.schedmd.com) for
parallel batch jobs."
- [methods] variable genes precede PCA: "Next, we identify highly variable genes for the following PCA and
clustering analyses."
- [readme] package maintenance status: "**This package is not maintained at the moment -- please contact if you're interested in collaborating and improving this project.**"
