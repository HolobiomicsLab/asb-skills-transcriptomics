---
name: scrnaseq-confounder-corrected-pca
description: Use when preprocessing 10X scRNA-seq matrices (matrix.mtx/gene.tsv/barcode.tsv)
  prior to clustering, to correct confounders like cell cycle or mitochondrial load
  and to choose the number of PCs via shuffled-data simulation.
when_to_use_negative:
- Input is raw unfiltered 10X data — read.data quality filtering must precede this
  skill, and PCA figures assume post-QC mitochondrial-score-vs-UMI relationships
- Bulk RNA-seq or non-single-cell expression matrices — the shuffled-data PC selection
  and confounder scoring are designed for single-cell 10X-format data
- Pooled replicate analysis where existing per-dataset cluster assignments must be
  retained and PCA is only re-run for integration
edam_operation: http://edamontology.org/operation_2939
edam_topics:
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_3308
tools:
- name: robustSingleCell
  role: Provides the add.confounder.variables, controlled.mean.score, cell.cycle.score,
    mitochondrial.score, ribosomal.score and PCA functions that implement confounder
    correction and shuffled-data PC selection
  repo: github.com/asmagen/robustSingleCell
- name: R
  role: Language in which the robustSingleCell pipeline and its PCA/confounder functions
    are executed
- name: slurm
  role: Parallel batch backend for running the shuffled-data PCA simulations off the
    interactive session (local = F)
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1016/j.celrep.2019.10.131
    title: Single-Cell Profiling Defines Transcriptomic Signatures Specific to Tumor-Reactive
      versus Virus-Responsive CD4+ T Cells
  source_content_hash: sha256:d82771262e09ab779bbafd64916f77fd80575d0ebe84013907f205f3d12356fe
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - R
  - robustSingleCell
  - slurm
  edam_operation: http://edamontology.org/operation_2939
  edam_topics:
  - http://edamontology.org/topic_0203
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 4
---

# scrnaseq-confounder-corrected-pca

## Summary

A scRNA-seq dimensionality-reduction skill from the robustSingleCell pipeline (Magen et al. 2019) that adds confounder covariates (ribosomal, mitochondrial, cell-cycle, exhaustion scores) to quality-filtered 10X data and runs PCA with shuffled-data simulations to pick the appropriate number of PCs before clustering.

## When to use

Use when preprocessing 10X scRNA-seq matrices (matrix.mtx/gene.tsv/barcode.tsv) prior to clustering, to correct confounders like cell cycle or mitochondrial load and to choose the number of PCs via shuffled-data simulation.

## When NOT to use

- Input is raw unfiltered 10X data — read.data quality filtering must precede this skill, and PCA figures assume post-QC mitochondrial-score-vs-UMI relationships
- Bulk RNA-seq or non-single-cell expression matrices — the shuffled-data PC selection and confounder scoring are designed for single-cell 10X-format data
- Pooled replicate analysis where existing per-dataset cluster assignments must be retained and PCA is only re-run for integration

## Inputs

- Quality-filtered 10X scRNA-seq data (matrix.mtx, gene.tsv, barcode.tsv) loaded via read.data into a robustSingleCell project environment (e.g., LCMV1, LCMV2)
- Highly variable gene list from get.variable.genes
- Custom gene sets for confounder scores (e.g., exhaustion markers Pdcd1, Cd244, Havcr2, Ctla4, Cd160, Lag3, Tigit, Cd96)

## Outputs

- Project environment annotated with confounder score columns (ribosomal.score, mitochondrial.score, cell.cycle.score, Exhaustion)
- PCA embedding and a data-driven selection of the appropriate number of PCs from shuffled-data simulations
- PC scores feeding downstream cluster.analysis and summarize outputs (tSNE, PCA loadings/rotation figures)

## How to apply

After read.data quality filtering (as in Magen et al. 2019), call get.variable.genes to select highly variable genes that drive PCA and clustering. Then use add.confounder.variables to compute ribosomal.score, mitochondrial.score, cell.cycle.score, and exhaustion as controlled.mean.score over the markers Pdcd1, Cd244, Havcr2, Ctla4, Cd160, Lag3, Tigit, Cd96, so technical and activation confounders can be corrected. Run the PCA function, which performs multiple simulations of shuffled data to determine the appropriate number of PCs (set local=T to run locally or local=F for parallel slurm jobs). Pass the resulting selected PCs into cluster.analysis, which uses them for KNN ratio-based (Phenograph) clustering with false-discovery control via modularity fold change (mdlrty/mean.shfl > 2). For pooled replicate analysis, re-run PCA on the pooled environment with clear.previously.calculated.clustering = F so per-dataset clustering is preserved.

## Related tools

- **robustSingleCell** (Provides the add.confounder.variables, controlled.mean.score, cell.cycle.score, mitochondrial.score, ribosomal.score and PCA functions that implement confounder correction and shuffled-data PC selection) — github.com/asmagen/robustSingleCell
- **R** (Language in which the robustSingleCell pipeline and its PCA/confounder functions are executed)
- **slurm** (Parallel batch backend for running the shuffled-data PCA simulations off the interactive session (local = F))

## Examples

```
LCMV1 <- add.confounder.variables(LCMV1, ribosomal.score=ribosomal.score(LCMV1), mitochondrial.score=mitochondrial.score(LCMV1), cell.cycle.score=cell.cycle.score(LCMV1), Exhaustion=controlled.mean.score(LCMV1, exhaustion_markers)); LCMV1 <- PCA(LCMV1, local=T)
```

## Evaluation signals

- Check that all four confounder scores (ribosomal, mitochondrial, cell cycle, Exhaustion) were computed and attached before PCA is invoked
- Confirm PCA was computed on highly variable genes only, as identified by get.variable.genes
- Verify the number of PCs was chosen by the shuffled-data simulation procedure embedded in the PCA function, not set arbitrarily
- Inspect pre/post QC figures: mitochondrial score vs. number of UMIs should improve after read.data filtering and before PCA
- Downstream check: cluster.analysis on the selected PCs should reach a clustering resolution with original-vs-shuffled modularity fold change mdlrty/mean.shfl > 2 (e.g., KNN ratio 0.05 in the LCMV tutorial)

## Limitations

- The robustSingleCell package is not currently maintained, per the README
- Requires at least 8G of memory on slurm; the tutorial subsamples to 500 cells, so full datasets will be more demanding
- PC selection depends on multiple shuffled-data simulations, which are computationally costly when run locally rather than in parallel

## Evidence

- [methods] PCA function uses shuffled simulations: "The `PCA` function performs multiple simulation analyses of shuffled data to determine the appropriate number of PCs."
- [readme] Confounder correction rationale: "We also compute the activation of gene sets of interest, such as cell cycle genes, for confounder correction."
- [readme] Variable genes precede PCA and clustering: "Next, we identify highly variable genes for the following PCA and clustering analyses."
