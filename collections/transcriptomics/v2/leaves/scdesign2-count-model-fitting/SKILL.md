---
name: scdesign2-count-model-fitting
description: Use when a synthetic scRNA-seq benchmark needs realistic UMI count matrices
  as ground truth and cells belong to distinct clusters, so a cluster-aware count
  simulator can be trained on real UMI count data.
when_to_use_negative:
- Cells follow continuous trajectories or have spatial locations rather than discrete
  clusters — scDesign3 (not yet scReadSim's internal simulator) is the appropriate
  count model
- The goal is to benchmark isoform-level or alternative-splicing tools — scReadSim
  generates only gene-level ground truths (UMI counts), not isoform abundance or alternative
  splicing events
- Input is raw sequencing reads or a BAM file without a prior count-matrix construction
  step (Utility.scRNA_bam2countmat) — the simulator requires UMI count matrices for
  training
edam_operation: http://edamontology.org/operation_3562
edam_topics:
- http://edamontology.org/topic_2229
- http://edamontology.org/topic_3170
tools:
- name: scReadSim
  role: Wraps scDesign2 via GenerateSyntheticCount.scRNA_GenerateSyntheticCount to
    train the count simulator on the real UMI count matrices and write the synthetic
    matrices
  repo: https://github.com/JSB-UCLA/scReadSim
- name: scDesign2
  role: The count simulator trained on the real gene-by-cell and inter-gene-by-cell
    UMI count matrices when cells belong to distinct clusters
  repo: https://github.com/JSB-UCLA/scDesign2
- name: scDesign3
  role: Alternative count simulator for cells following continuous trajectories; scReadSim
    is adaptive to it but it is not yet the internal simulator
- name: Seurat
  role: R package used for Louvain pre-clustering of cells to supply the discrete
    cell types scDesign2 requires when no celllabel_file is given
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1038/s41467-023-43162-w
    title: 'scReadSim: a single-cell RNA-seq and ATAC-seq read simulator'
  source_content_hash: sha256:885b1fcdd1f045f9275075c7af318860b24906d93ee6e080a535f0a4c123418b
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/JSB-UCLA/scReadSim
  tools:
  - Seurat
  - scDesign2
  - scDesign3
  - scReadSim
  edam_operation: http://edamontology.org/operation_3562
  edam_topics:
  - http://edamontology.org/topic_2229
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7
n_claims: 10
---

# scdesign2-count-model-fitting

## Summary

Fit the scDesign2 count simulator to real gene-by-cell and inter-gene-by-cell UMI count matrices (via scReadSim's GenerateSyntheticCount.scRNA_GenerateSyntheticCount) to produce synthetic count matrices that serve as the ground truth for benchmarking scRNA-seq UMI deduplication tools.

## When to use

Use when a synthetic scRNA-seq benchmark needs realistic UMI count matrices as ground truth and cells belong to distinct clusters, so a cluster-aware count simulator can be trained on real UMI count data.

## When NOT to use

- Cells follow continuous trajectories or have spatial locations rather than discrete clusters — scDesign3 (not yet scReadSim's internal simulator) is the appropriate count model
- The goal is to benchmark isoform-level or alternative-splicing tools — scReadSim generates only gene-level ground truths (UMI counts), not isoform abundance or alternative splicing events
- Input is raw sequencing reads or a BAM file without a prior count-matrix construction step (Utility.scRNA_bam2countmat) — the simulator requires UMI count matrices for training

## Inputs

- Real gene-by-cell UMI count matrix (from Utility.scRNA_bam2countmat with UMI_modeling=True)
- Real inter-gene-by-cell UMI count matrix with the same cells in the same column order
- Optional celllabel_file with predefined discrete cell type labels (otherwise Louvain/Seurat clustering is performed)
- Input BAM file of real scRNA-seq reads with reference annotations (GTF, e.g. gencode.vM10.annotation.gtf) from the upstream count-matrix construction step

## Outputs

- Synthetic gene-by-cell UMI count matrix named <count_mat_filename>.scDesign2Simulated.txt
- Per-feature summation files <count_mat_filename>.scDesign2Simulated.nReadRegionmargional.txt written to the output directory
- Synthetic inter-gene-by-cell UMI count matrix for downstream synthetic-read generation

## How to apply

Load the real gene-by-cell and inter-gene-by-cell UMI count matrices produced by Utility.scRNA_bam2countmat with UMI_modeling=True, ensuring both matrices contain the same cells in the same column order. If no cell labels are supplied via celllabel_file, pre-cluster cells with Louvain clustering (Seurat R package) on the gene-by-cell UMI count matrix, since scDesign2 requires discrete cell types. Call GenerateSyntheticCount.scRNA_GenerateSyntheticCount separately on the gene-by-cell and inter-gene-by-cell UMI matrices with UMI_modeling=True and the defaults n_cell_new=None and total_count_new=None so scDesign2 retains the real cell number and real sequencing depth. The rationale is that the synthetic UMI count matrix is the intermediate output of scReadSim against which deduplicated counts are compared, so its statistics must faithfully mimic the real matrix. scDesign3 can be substituted when cells follow continuous trajectories rather than discrete clusters, though it is not yet the internal simulator of scReadSim.

## Related tools

- **scReadSim** (Wraps scDesign2 via GenerateSyntheticCount.scRNA_GenerateSyntheticCount to train the count simulator on the real UMI count matrices and write the synthetic matrices) — https://github.com/JSB-UCLA/scReadSim
- **scDesign2** (The count simulator trained on the real gene-by-cell and inter-gene-by-cell UMI count matrices when cells belong to distinct clusters) — https://github.com/JSB-UCLA/scDesign2
- **scDesign3** (Alternative count simulator for cells following continuous trajectories; scReadSim is adaptive to it but it is not yet the internal simulator)
- **Seurat** (R package used for Louvain pre-clustering of cells to supply the discrete cell types scDesign2 requires when no celllabel_file is given)

## Examples

```
GenerateSyntheticCount.scRNA_GenerateSyntheticCount(UMI_modeling=True, UMI_count_mat_filename=UMI_count_mat_filename, celllabel_file='', n_cell_new=None, total_count_new=None)
```

## Evaluation signals

- The synthetic gene-by-cell UMI matrix retains the same feature rows as the input matrix and contains the synthetic cell columns
- Gene-wise summary statistics (mean, variance, coefficient of variance, zero proportion) of the synthetic matrix mimic the real UMI count matrix
- Cell-wise summary statistics (library size, zero proportion) of the synthetic matrix mimic the real UMI count matrix
- Output files follow the expected naming: <count_mat_filename>.scDesign2Simulated.txt and <count_mat_filename>.scDesign2Simulated.nReadRegionmargional.txt in the output directory
- The gene-by-cell and inter-gene-by-cell input matrices were verified to contain the same cells in the same column order before training

## Limitations

- scDesign2 requires discrete cell types, so cells must be pre-clustered (e.g. Louvain via Seurat) when no labels are provided; multi-cell-type model fitting via mclapply is Unix-only and slower on Windows
- Support for continuous trajectories, spatial locations, conditions/batches, and multi-omics depends on scDesign3, which is not yet scReadSim's internal simulator
- Defaults n_cell_new=None and total_count_new=None fix the synthetic data to the real cell number and sequencing depth; varying throughput/library size must be set explicitly for experimental-design guidance

## Evidence

- [methods] In the second step—synthetic count generation, scReadSim trains the count simulator scDesign2: "In the second step—synthetic count generation, scReadSim trains the count simulator scDesign2"
- [methods] To generate the gene- and inter-gene-by-cell UMI count matrices for synthetic cells, scReadSim trains the count simulator scDesign2: "To generate the gene- and inter-gene-by-cell UMI count matrices for synthetic cells, scReadSim trains the count simulator scDesign2"
- [intro] scReadSim trains the count simulator scDesign26 (if the cells belong to distinct clusters; otherwise, scDesign322 can be used: "scReadSim trains the count simulator scDesign26 (if the cells belong to distinct clusters; otherwise, scDesign322 can be used"
- [methods] The current version of scReadSim implement [scDesign2](https://github.com/JSB-UCLA/scDesign2) to generate synthetic count matrix: "The current version of scReadSim implement [scDesign2](https://github.com/JSB-UCLA/scDesign2) to generate synthetic count matrix"
- [readme] A transparent simulator that generates high-fidelity single-cell gene expression count data with gene correlations captured: [evidence span withheld — anchor status: external_doc]
