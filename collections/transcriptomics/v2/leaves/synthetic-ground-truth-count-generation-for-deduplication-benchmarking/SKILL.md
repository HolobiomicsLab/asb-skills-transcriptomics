---
name: synthetic-ground-truth-count-generation-for-deduplication-benchmarking
description: 'Use when you need a synthetic scRNA-seq UMI count matrix as deduplication
  ground truth: you have real gene/inter-gene by-cell UMI count matrices (BAM-derived)
  and cells falling in distinct clusters or continuous trajectories.'
when_to_use_negative:
- You need isoform-level ground truths (isoform abundance or alternative splicing
  events) — scReadSim currently generates gene-level UMI counts only
- Cells follow continuous trajectories or spatial locations — scDesign2 assumes discrete
  cell types; scDesign3 is the appropriate (though not yet internal) simulator
- Input is raw sequencing reads or a BAM file rather than UMI count matrices — the
  count-matrix construction step (Utility.scRNA_bam2countmat) must run first
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: scReadSim
  role: Wraps the synthetic-count generation; its GenerateSyntheticCount.scRNA_GenerateSyntheticCount
    function trains the simulator on real UMI count matrices
  repo: https://github.com/JSB-UCLA/scReadSim
- name: scDesign2
  role: Count simulator trained per cell cluster to generate the synthetic gene- and
    inter-gene-by-cell UMI count matrices
  repo: https://github.com/JSB-UCLA/scDesign2
- name: scDesign3
  role: Alternative count simulator for cells following continuous trajectories instead
    of distinct clusters
  repo: https://github.com/JSB-UCLA/scReadSim
- name: Seurat
  role: Used for Louvain pre-clustering of cells on the gene-by-cell UMI count matrix
    when no cell labels are supplied
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1038/s41467-023-43162-w
    title: 'scReadSim: a single-cell RNA-seq and ATAC-seq read simulator'
  source_content_hash: sha256:d4eba60ce4ac12554c5d8f87368481b2c60e56f3b5287752f30cbb444ceb6b52
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/JSB-UCLA/scReadSim
  tools:
  - Seurat
  - scDesign2
  - scDesign3
  - scReadSim
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 9
---

# synthetic-ground-truth-count-generation-for-deduplication-benchmarking

## Summary

Train scDesign2 (or scDesign3 for continuous cell trajectories) on real gene- and inter-gene-by-cell UMI count matrices via scReadSim to produce a synthetic UMI count matrix that serves as the ground truth for benchmarking scRNA-seq UMI deduplication tools.

## When to use

Use when you need a synthetic scRNA-seq UMI count matrix as deduplication ground truth: you have real gene/inter-gene by-cell UMI count matrices (BAM-derived) and cells falling in distinct clusters or continuous trajectories.

## When NOT to use

- You need isoform-level ground truths (isoform abundance or alternative splicing events) — scReadSim currently generates gene-level UMI counts only
- Cells follow continuous trajectories or spatial locations — scDesign2 assumes discrete cell types; scDesign3 is the appropriate (though not yet internal) simulator
- Input is raw sequencing reads or a BAM file rather than UMI count matrices — the count-matrix construction step (Utility.scRNA_bam2countmat) must run first

## Inputs

- real gene-by-cell UMI count matrix (BAM-derived, UMI_modeling=True)
- real inter-gene-by-cell UMI count matrix (same cells, same column order as gene matrix)
- optional celllabel_file with predefined discrete cell-type labels
- reference count_mat_filename and UMI_count_mat_filename arguments

## Outputs

- synthetic gene-by-cell UMI count matrix (<count_mat_filename>.scDesign2Simulated.txt)
- per-feature summation file <count_mat_filename>.scDesign2Simulated.nReadRegionmargional.txt
- synthetic inter-gene-by-cell UMI count matrix
- ground-truth UMI counts used downstream for benchmarking UMI deduplication tools

## How to apply

Load the real gene-by-cell and inter-gene-by-cell UMI count matrices from the count-matrix construction step (e.g. Utility.scRNA_bam2countmat with UMI_modeling=True), ensuring identical cells and column order in both. If no cell labels are given via celllabel_file, pre-cluster cells with Louvain clustering (Seurat) on the gene-by-cell matrix, since scDesign2 requires discrete cell types. Call GenerateSyntheticCount.scRNA_GenerateSyntheticCount on the gene-by-cell UMI matrix with UMI_modeling=True and UMI_count_mat_filename set, keeping defaults n_cell_new=None and total_count_new=None so scDesign2 uses the real cell number and sequencing depth; train it separately on the inter-gene-by-cell matrix. For cells following continuous trajectories rather than distinct clusters, substitute scDesign3 as the count simulator. Outputs are written as <count_mat_filename>.scDesign2Simulated.txt plus per-feature summation files (<count_mat_filename>.scDesign2Simulated.nReadRegionmargional.txt) for the downstream synthetic-read generation step.

## Related tools

- **scReadSim** (Wraps the synthetic-count generation; its GenerateSyntheticCount.scRNA_GenerateSyntheticCount function trains the simulator on real UMI count matrices) — https://github.com/JSB-UCLA/scReadSim
- **scDesign2** (Count simulator trained per cell cluster to generate the synthetic gene- and inter-gene-by-cell UMI count matrices) — https://github.com/JSB-UCLA/scDesign2
- **scDesign3** (Alternative count simulator for cells following continuous trajectories instead of distinct clusters) — https://github.com/JSB-UCLA/scReadSim
- **Seurat** (Used for Louvain pre-clustering of cells on the gene-by-cell UMI count matrix when no cell labels are supplied)

## Examples

```
GenerateSyntheticCount.scRNA_GenerateSyntheticCount(count_mat_filename, output_dir, UMI_modeling=True, UMI_count_mat_filename=umi_count_mat)
```

## Evaluation signals

- Synthetic gene-by-cell UMI matrix retains the same feature rows as the input matrix and contains synthetic cell columns
- Gene-wise mean, variance, coefficient of variance, and zero proportion of the synthetic matrix mimic the real UMI count matrix
- Cell-wise library size and zero proportion of synthetic cells match the real matrix's summary statistics
- Output files <count_mat_filename>.scDesign2Simulated.txt and .nReadRegionmargional.txt exist in the output directory
- Gene-by-cell and inter-gene-by-cell matrices contain the same cells in the same column order before training

## Limitations

- scDesign2 requires discrete cell types; without celllabel_file, cells must be pre-clustered (Louvain via Seurat) and clustering quality affects simulation fidelity
- Support for continuous trajectories, spatial locations, conditions/batches, and multi-omics depends on scDesign3, which is not yet scReadSim's internal simulator
- Simulator fitting via scDesign2's mclapply parallelization only runs on Unix-like operating systems, so Windows model fitting of multiple cell types is slower

## Evidence

- [methods] Synthetic UMI count matrix generation trains scDesign2: "To generate the gene- and inter-gene-by-cell UMI count matrices for synthetic cells, scReadSim trains the count simulator scDesign2"
- [methods] Function for generating the synthetic count matrix: "Use function `GenerateSyntheticCount.scRNA_GenerateSyntheticCount` to generate synthetic count matrix"
- [intro] Choice between scDesign2 and scDesign3 based on cell structure: "otherwise, scDesign322 can be used if the cells follow continuous trajectories"
- [intro] Ground-truth role of the synthetic UMI matrix: "the synthetic UMI count matrix—an intermediate output of scReadSim—serves as the ground truth for benchmarking scRNA-seq UMI deduplication tools"
- [readme] README confirms scReadSim is trained on real data: [evidence span withheld — anchor status: external_doc]
