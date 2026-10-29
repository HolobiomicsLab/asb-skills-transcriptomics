---
name: single-cell-umi-count-matrix-simulation
description: Use when simulating synthetic scRNA-seq UMI count matrices from real
  gene-by-cell count matrices (e.g., produced by Utility.scRNA_bam2countmat with UMI_modeling=True)
  to create a deduplication benchmark ground truth.
when_to_use_negative:
- The goal is to benchmark isoform-level or alternative splicing tools — scReadSim
  provides gene-level ground truths (UMI counts) only
- Input is an scATAC-seq chromatin accessibility matrix — use GenerateSyntheticCount.scATAC_GenerateSyntheticCount
  instead
- The analysis requires spatial locations, conditions/batches, or multi-omics simulation
  — scDesign3 support is not yet the internal simulator of scReadSim
edam_operation: http://edamontology.org/operation_2426
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_3518
tools:
- name: scReadSim
  role: Host framework; its GenerateSyntheticCount.scRNA_GenerateSyntheticCount function
    invokes scDesign2 to produce the synthetic UMI count matrices
  repo: https://github.com/JSB-UCLA/scReadSim
- name: scDesign2
  role: Count simulator trained on real gene-by-cell UMI count matrices of cells in
    distinct clusters to generate the synthetic matrices
  repo: https://github.com/JSB-UCLA/scDesign2
- name: scDesign3
  role: Alternative count simulator substituted for scDesign2 when cells follow continuous
    trajectories rather than discrete clusters
- name: Seurat
  role: R package used for Louvain pre-clustering of cells to supply the discrete
    cell types scDesign2 requires when no celllabel_file is given
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1038/s41467-023-43162-w
    title: 'scReadSim: a single-cell RNA-seq and ATAC-seq read simulator'
  source_content_hash: sha256:95375641a782b08ec88b61a05980966787169acf1eb89bc2818469d84568e059
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/JSB-UCLA/scReadSim
  tools:
  - Seurat
  - scDesign2
  - scDesign3
  - scReadSim
  edam_operation: http://edamontology.org/operation_2426
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_3518
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6
n_claims: 10
---

# single-cell-umi-count-matrix-simulation

## Summary

Train the count simulator scDesign2 (or scDesign3 for continuous trajectories) on real gene-by-cell and inter-gene-by-cell UMI count matrices to generate synthetic UMI count matrices that serve as the ground truth for benchmarking scRNA-seq UMI deduplication tools within scReadSim.

## When to use

Use when simulating synthetic scRNA-seq UMI count matrices from real gene-by-cell count matrices (e.g., produced by Utility.scRNA_bam2countmat with UMI_modeling=True) to create a deduplication benchmark ground truth.

## When NOT to use

- The goal is to benchmark isoform-level or alternative splicing tools — scReadSim provides gene-level ground truths (UMI counts) only
- Input is an scATAC-seq chromatin accessibility matrix — use GenerateSyntheticCount.scATAC_GenerateSyntheticCount instead
- The analysis requires spatial locations, conditions/batches, or multi-omics simulation — scDesign3 support is not yet the internal simulator of scReadSim

## Inputs

- Real gene-by-cell UMI count matrix (e.g., from Utility.scRNA_bam2countmat with UMI_modeling=True)
- Real inter-gene-by-cell UMI count matrix with identical cells and column order
- Optional celllabel_file with predefined cell type labels (otherwise Louvain clustering via Seurat is performed)
- BAM file of scRNA-seq reads in cells (upstream preprocessing requirement)

## Outputs

- Synthetic gene-by-cell UMI count matrix named <count_mat_filename>.scDesign2Simulated.txt
- Per-feature summation files <count_mat_filename>.scDesign2Simulated.nReadRegionmargional.txt
- Synthetic inter-gene-by-cell UMI count matrix serving as the deduplication ground truth

## How to apply

Load the real gene-by-cell and inter-gene-by-cell UMI count matrices from the count-matrix construction step (Utility.scRNA_bam2countmat with UMI_modeling=True), ensuring both matrices contain the same cells in the same column order. If no predefined cell labels are supplied via celllabel_file, pre-cluster the cells with Louvain clustering (Seurat R package) on the gene-by-cell UMI count matrix, since scDesign2 requires discrete cell types. Call GenerateSyntheticCount.scRNA_GenerateSyntheticCount on the gene-by-cell UMI count matrix with UMI_modeling=True and UMI_count_mat_filename set, keeping the defaults n_cell_new=None and total_count_new=None so scDesign2 uses the real cell number and real sequencing depth; repeat the training separately on the inter-gene-by-cell count matrix. For cells following continuous trajectories rather than distinct clusters, substitute scDesign3 as the count simulator. The synthetic matrices are written as <count_mat_filename>.scDesign2Simulated.txt with per-feature summation files <count_mat_filename>.scDesign2Simulated.nReadRegionmargional.txt for downstream synthetic-read generation.

## Related tools

- **scReadSim** (Host framework; its GenerateSyntheticCount.scRNA_GenerateSyntheticCount function invokes scDesign2 to produce the synthetic UMI count matrices) — https://github.com/JSB-UCLA/scReadSim
- **scDesign2** (Count simulator trained on real gene-by-cell UMI count matrices of cells in distinct clusters to generate the synthetic matrices) — https://github.com/JSB-UCLA/scDesign2
- **scDesign3** (Alternative count simulator substituted for scDesign2 when cells follow continuous trajectories rather than discrete clusters)
- **Seurat** (R package used for Louvain pre-clustering of cells to supply the discrete cell types scDesign2 requires when no celllabel_file is given)

## Examples

```
from scReadSim.GenerateSyntheticCount import scRNA_GenerateSyntheticCount; scRNA_GenerateSyntheticCount(input_dir, output_dir, count_mat_filename='gene_count_mat.txt', UMI_modeling=True, UMI_count_mat_filename='UMI_count_mat.txt')
```

## Evaluation signals

- The synthetic gene-by-cell UMI matrix retains the same feature rows as the input matrix and contains the synthetic cell columns
- Gene-wise summary statistics (mean, variance, coefficient of variance, zero proportion) of the synthetic matrix mimic the real UMI count matrix
- Cell-wise summary statistics (library size, zero proportion) of the synthetic matrix mimic the real matrix
- Output files <count_mat_filename>.scDesign2Simulated.txt and .nReadRegionmargional.txt exist in the output directory
- With defaults n_cell_new=None and total_count_new=None, the synthetic matrix preserves the real cell number and sequencing depth

## Limitations

- The default implementation may generate intronic synthetic reads when extracting sequences from the reference genome, which can affect benchmarks such as cellranger default mode
- Multi-sample or multi-condition generation requires users to supply one BAM file per sample or condition
- Support for continuous trajectories, spatial locations, conditions/batches, and multi-omics depends on scDesign3, which is not yet the internal simulator of scReadSim
- scDesign2 model fitting of multiple cell types relies on mclapply() parallelization and is slow on Windows

## Evidence

- [intro] scDesign2 choice depends on clustering structure: "scReadSim trains the count simulator scDesign26 (if the cells belong to distinct clusters; otherwise, scDesign322 can be used"
- [methods] Synthetic UMI matrices via scDesign2: "To generate the gene- and inter-gene-by-cell UMI count matrices for synthetic cells, scReadSim trains the count simulator scDesign2"
- [intro] Synthetic UMI matrix is deduplication ground truth: "the synthetic UMI count matrix—an intermediate output of scReadSim—serves as the ground truth for benchmarking scRNA-seq UMI deduplication tools"
- [methods] Three-step pipeline context: "As an overview, scReadSim consists of three major steps: genome segmentation, synthetic count generation, and synthetic read generation."
- [methods] Current version implements scDesign2: "The current version of scReadSim implement [scDesign2](https://github.com/JSB-UCLA/scDesign2) to generate synthetic count matrix"
