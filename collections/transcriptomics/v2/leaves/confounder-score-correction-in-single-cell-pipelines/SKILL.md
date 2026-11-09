---
name: confounder-score-correction-in-single-cell-pipelines
description: Use when preprocessing scRNAseq data (10X matrix.mtx/gene.tsv/barcode.tsv)
  with robustSingleCell, after read.data quality filtering and get.variable.genes,
  to attach covariate scores before PCA and clustering.
when_to_use_negative:
- Input data has not yet passed read.data quality filtering or had highly variable
  genes identified with get.variable.genes — the scores are meant to be added after
  these steps
- Running the pooled/integrated analysis via read.preclustered.datasets without recomputing
  variable genes, where confounder variables are added directly to the pooled environment
  instead
- Pipelines outside robustSingleCell that use alternative correction frameworks (e.g.
  regression-based batch removal) rather than score-based confounder variables
edam_operation: http://edamontology.org/operation_2939
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: robustSingleCell
  role: Provides add.confounder.variables and the ribosomal.score, mitochondrial.score,
    cell.cycle.score, and controlled.mean.score functions that compute the per-cell
    covariate scores prior to PCA
  repo: github.com/asmagen/robustSingleCell
- name: R
  role: Execution environment in which the robustSingleCell project environments and
    score functions are invoked
- name: slurm
  role: High performance computing workload manager supporting parallel batch jobs
    (e.g. srun --pty -p <partition> --mem=8G) for running the pipeline, requiring
    at least 8G of memory
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1016/j.celrep.2019.10.131
    title: Single-Cell Profiling Defines Transcriptomic Signatures Specific to Tumor-Reactive
      versus Virus-Responsive CD4+ T Cells
  source_content_hash: sha256:c2e62a9bbf9433253c5b0a452124425f530161c8924b19333d6d8aa8a0f02023
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - R
  - robustSingleCell
  - slurm
  edam_operation: http://edamontology.org/operation_2939
  edam_topics:
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 8
---

# confounder-score-correction-in-single-cell-pipelines

## Summary

Computes per-cell confounder scores (ribosomal, mitochondrial, cell cycle, and exhaustion marker scores) with robustSingleCell functions on the quality-filtered LCMV1/LCMV2 10X environments so these technical/biological covariates can be accounted for in downstream PCA and clustering.

## When to use

Use when preprocessing scRNAseq data (10X matrix.mtx/gene.tsv/barcode.tsv) with robustSingleCell, after read.data quality filtering and get.variable.genes, to attach covariate scores before PCA and clustering.

## When NOT to use

- Input data has not yet passed read.data quality filtering or had highly variable genes identified with get.variable.genes — the scores are meant to be added after these steps
- Running the pooled/integrated analysis via read.preclustered.datasets without recomputing variable genes, where confounder variables are added directly to the pooled environment instead
- Pipelines outside robustSingleCell that use alternative correction frameworks (e.g. regression-based batch removal) rather than score-based confounder variables

## Inputs

- robustSingleCell project environment initialized via initialize.project with 10X data (matrix.mtx, gene.tsv, barcode.tsv) in data.path
- Quality-filtered and 500-cell-subsampled expression matrix from read.data(subsample = 500)
- Named marker gene vectors, e.g. exhaustion_markers = c('Pdcd1','Cd244','Havcr2','Ctla4','Cd160','Lag3','Tigit','Cd96')

## Outputs

- Project environment enriched with per-cell ribosomal.score, mitochondrial.score, cell.cycle.score, and Exhaustion score columns
- Confounded variables available to the downstream PCA function for PC number selection and the summarize diagnostic figures (PCA.png, violin.png of cell cycle score)

## How to apply

After initialize.project and read.data (with subsample = 500 for the LCMV tutorial), run get.variable.genes to identify highly variable genes for PCA and clustering, then call add.confounder.variables on the project environment, supplying ribosomal.score(LCMV1), mitochondrial.score(LCMV1), cell.cycle.score(LCMV1), and controlled.mean.score over exhaustion markers (Pdcd1, Cd244, Havcr2, Ctla4, Cd160, Lag3, Tigit, Cd96). The rationale is to compute activation of gene sets of interest, such as cell cycle genes, for confounder correction so that these signatures do not dominate PCA — the subsequent PCA function performs multiple simulation analyses of shuffled data to determine the appropriate number of PCs. Inspect pre/post quality-control figures (mitochondrial score vs. number of UMIs) and record the attached scores before running PCA with local = T or parallel slurm execution.

## Related tools

- **robustSingleCell** (Provides add.confounder.variables and the ribosomal.score, mitochondrial.score, cell.cycle.score, and controlled.mean.score functions that compute the per-cell covariate scores prior to PCA) — github.com/asmagen/robustSingleCell
- **R** (Execution environment in which the robustSingleCell project environments and score functions are invoked)
- **slurm** (High performance computing workload manager supporting parallel batch jobs (e.g. srun --pty -p <partition> --mem=8G) for running the pipeline, requiring at least 8G of memory)

## Examples

```
LCMV1 <- add.confounder.variables(LCMV1, ribosomal.score=ribosomal.score(LCMV1), mitochondrial.score=mitochondrial.score(LCMV1), cell.cycle.score=cell.cycle.score(LCMV1))
```

## Evaluation signals

- All four expected score columns (ribosomal.score, mitochondrial.score, cell.cycle.score, Exhaustion) are present on the project environment before PCA(LCMV1, local = T) is called
- Pre/post quality-control figures (pre_stats.png, post_stats.png) show the mitochondrial score vs. number of UMIs, with post-filtering cells restricted to acceptable mitochondrial scores
- The subsequent PCA run's permutation test (shuffled-data simulations) selects a number of PCs that is consistent with simulated-null comparison output in pca_results.txt
- The violin plot of cell cycle score (violin.png) generated by summarize reflects the computed cell.cycle.score per cluster
- The same add.confounder.variables call pattern is reproduced for LCMV2 and for the pooled_env integration step

## Limitations

- The robustSingleCell package is not currently maintained, so fixing issues requires contacting the authors
- Score computation is demonstrated on a 500-cell random subsample of the CD44+ T cell datasets; behavior on full-size datasets or other tissues is not shown in the tutorial
- Confounder correction here is score-based (added variables) rather than explicit regression removal, so scores must be recomputed with dataset-appropriate marker sets for other systems

## Evidence

- [methods] add.confounder.variables with cell cycle and gene-set scores: "We also compute the activation of gene sets of interest, such as cell cycle genes, for confounder correction."
- [readme] read.data quality filtering and subsampling: "`read.data` function reads the data in 10X genomics format and performs
quality filtering as described in *Magen et al 2019*."
- [readme] variable genes before PCA and clustering: "Next, we identify highly variable genes for the following PCA and
clustering analyses."
- [readme] PCA permutation-based PC selection: "The `PCA` function performs multiple simulation analyses of shuffled
data to determine the appropriate number of PCs."
- [readme] package maintenance status: "**This package is not maintained at the moment -- please contact if you're interested in collaborating and improving this project.**"
