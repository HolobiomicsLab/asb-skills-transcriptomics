---
name: single-cell-pca-dimensionality-reduction
description: Use when a scRNA-seq pipeline needs PCA on highly variable genes with
  a data-driven, permutation-based choice of PC count, after read.data quality filtering
  and get.variable.genes.
when_to_use_negative:
- Input is raw 10X data that has not passed read.data quality filtering and get.variable.genes
  — PCA on all genes without variable-gene selection is out of scope
- The goal is selecting clustering resolution rather than PC number — use cluster.analysis,
  which compares modularity of original vs shuffled data across resolutions (e.g.
  choosing 0.05 where mdlrty/mean.shfl > 2)
- Non-R or non-slurm environments requiring a maintained tool — the package is explicitly
  not currently maintained
edam_operation: http://edamontology.org/operation_2939
edam_topics:
- http://edamontology.org/topic_3170
tools:
- name: robustSingleCell
  role: Provides the PCA function that runs PCA on highly variable genes and performs
    multiple simulations of shuffled data to determine the appropriate number of PCs
  repo: https://github.com/asmagen/robustSingleCell
- name: R
  role: Runtime environment in which the robustSingleCell PCA function is executed
- name: slurm
  role: Parallel batch-job backend used when running shuffled-data simulations with
    local = F instead of local = T
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1016/j.celrep.2019.10.131
    title: Single-Cell Profiling Defines Transcriptomic Signatures Specific to Tumor-Reactive
      versus Virus-Responsive CD4+ T Cells
  source_content_hash: sha256:1314130d45661e35a671ad929ecc1d4caaaeb4e3b8d322e3d0f25501e98f0f5c
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/asmagen/robustSingleCell
  tools:
  - R
  - robustSingleCell
  - slurm
  edam_operation: http://edamontology.org/operation_2939
  edam_topics:
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.75
n_claims: 4
---

# single-cell-pca-dimensionality-reduction

## Summary

Performs the dimensionality-reduction stage of the robustSingleCell scRNA-seq pipeline, running PCA on highly variable genes and using permutation-based simulations of shuffled data to select the number of retained principal components.

## When to use

Use when a scRNA-seq pipeline needs PCA on highly variable genes with a data-driven, permutation-based choice of PC count, after read.data quality filtering and get.variable.genes.

## When NOT to use

- Input is raw 10X data that has not passed read.data quality filtering and get.variable.genes — PCA on all genes without variable-gene selection is out of scope
- The goal is selecting clustering resolution rather than PC number — use cluster.analysis, which compares modularity of original vs shuffled data across resolutions (e.g. choosing 0.05 where mdlrty/mean.shfl > 2)
- Non-R or non-slurm environments requiring a maintained tool — the package is explicitly not currently maintained

## Inputs

- Quality-filtered, optionally downsampled (e.g. 500-cell) scRNA-seq environment from read.data (10X genomics matrix.mtx, gene.tsv, barcode.tsv inputs)
- Highly variable gene set from get.variable.genes
- Confounder variables: ribosomal.score, mitochondrial.score, cell.cycle.score, Exhaustion score (controlled.mean.score of Pdcd1, Cd244, Havcr2, Ctla4, Cd160, Lag3, Tigit, Cd96)

## Outputs

- Retained principal components and the number of PCs selected (recorded in pca_results.txt)
- PCA diagnostic figures in the analysis folder: PCA.png (cells on 2D PC1/PC2 grid), rotation.png (top gene loadings for PC1/PC2), PC1_heatmap.png (PC1 loadings heatmap)
- PCA embedding used downstream by cluster.analysis

## How to apply

After quality filtering via read.data (e.g. subsample = 500 cells) and identifying highly variable genes with get.variable.genes, add confounder variables (ribosomal.score, mitochondrial.score, cell.cycle.score, Exhaustion via controlled.mean.score) for confounder correction. Then run the robustSingleCell PCA function, e.g. with local = T so simulations run on the local machine, or local = F to run each shuffled-data simulation in parallel via slurm. The function performs multiple simulation analyses of shuffled data to determine the appropriate number of PCs to retain. Inspect the permutation-based PC selection output (pca_results.txt) and the diagnostic figures (PCA.png, rotation.png, PC1_heatmap.png) produced downstream by the summarize plotting routines, confirming the retained PC count is consistent with the simulated-null comparison.

## Related tools

- **robustSingleCell** (Provides the PCA function that runs PCA on highly variable genes and performs multiple simulations of shuffled data to determine the appropriate number of PCs) — https://github.com/asmagen/robustSingleCell
- **R** (Runtime environment in which the robustSingleCell PCA function is executed)
- **slurm** (Parallel batch-job backend used when running shuffled-data simulations with local = F instead of local = T)

## Examples

```
LCMV1 <- get.variable.genes(LCMV1); LCMV1 <- add.confounder.variables(LCMV1, ribosomal.score = ribosomal.score(LCMV1), mitochondrial.score = mitochondrial.score(LCMV1)); LCMV1 <- PCA(LCMV1, local = T)
```

## Evaluation signals

- The reported number of retained PCs is supported by the shuffled-data simulations (permutation test) — i.e. it is consistent with the simulated-null comparison produced by the PCA run
- pca_results.txt exists in the analysis folder and records the retained PC count
- PCA.png, rotation.png, and PC1_heatmap.png are generated in the analysis folder via the summarize/PCA plotting routines
- PCA was run on the highly variable gene set (from get.variable.genes), not the full gene matrix, after confounder correction variables were added
- Downstream cluster.analysis can consume the PCA output without errors

## Limitations

- The PCA function runs multiple simulations of shuffled data, which is computationally costly; parallel execution requires slurm and the tutorial recommends at least 8G of memory (srun --mem=8G)
- The robustSingleCell package is not currently maintained, per the README
- Downstream steps like summarize create the PCA figures (PCA.png, rotation.png, PC1_heatmap.png), so diagnostics are not complete until summarize is run

## Evidence

- [methods] PCA function uses permutation-based PC selection: "The `PCA` function performs multiple simulation analyses of shuffled data to determine the appropriate number of PCs."
- [methods] Variable genes precede PCA: "Next, we identify highly variable genes for the following PCA and clustering analyses."
- [methods] Confounder scores computed before PCA: "We also compute the activation of gene sets of interest, such as cell cycle genes, for confounder correction."
- [readme] Local vs slurm-parallel execution option: "You can also run each simulation in parallel using the option `local = F`."
- [readme] Figures visualize PC loadings: "The genes driving the PCs are visualized in figure 5 according to the PCA loadings after removing the lowly ranked genes."
