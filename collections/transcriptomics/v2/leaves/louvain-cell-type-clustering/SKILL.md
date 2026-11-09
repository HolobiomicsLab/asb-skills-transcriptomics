---
name: louvain-cell-type-clustering
description: Use when training scDesign2 in scReadSim on a gene-by-cell UMI count
  matrix whose cells lack predefined cell labels (no celllabel_file), since scDesign2
  requires discrete cell types.
when_to_use_negative:
- Cells follow continuous trajectories or spatial locations rather than discrete clusters
  — use scDesign3 instead of scDesign2 with Louvain labels
- A predefined celllabel_file is already available, making re-clustering redundant
edam_operation: http://edamontology.org/operation_3432
edam_topics:
- http://edamontology.org/topic_3170
tools:
- name: Seurat
  role: R package used to perform Louvain clustering on the gene-by-cell UMI count
    matrix to obtain discrete cell-type labels for scDesign2 training
- name: scDesign2
  role: Count simulator trained per Louvain cluster to generate the synthetic UMI
    count matrix; requires discrete cell types
  repo: https://github.com/JSB-UCLA/scDesign2
- name: scReadSim
  role: Wrapper pipeline in which the clustering and synthetic count generation (GenerateSyntheticCount.scRNA_GenerateSyntheticCount)
    are executed
  repo: https://github.com/JSB-UCLA/scReadSim
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1038/s41467-023-43162-w
    title: 'scReadSim: a single-cell RNA-seq and ATAC-seq read simulator'
  source_content_hash: sha256:34a85fe52487e6b703eca0e27c0bc2dcccaf244159ea2d7dfb3c9c9d94d486a0
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/JSB-UCLA/scDesign2
  tools:
  - Seurat
  - scDesign2
  - scDesign3
  - scReadSim
  edam_operation: http://edamontology.org/operation_3432
  edam_topics:
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7777777777777778
n_claims: 9
---

# Louvain cell-type clustering

## Summary

Pre-clusters cells of a real scRNA-seq UMI count matrix with Louvain clustering (Seurat R package) so that discrete cell-type labels are available to train scDesign2 inside scReadSim's synthetic count generation step.

## When to use

Use when training scDesign2 in scReadSim on a gene-by-cell UMI count matrix whose cells lack predefined cell labels (no celllabel_file), since scDesign2 requires discrete cell types.

## When NOT to use

- Cells follow continuous trajectories or spatial locations rather than discrete clusters — use scDesign3 instead of scDesign2 with Louvain labels
- A predefined celllabel_file is already available, making re-clustering redundant

## Inputs

- Real gene-by-cell UMI count matrix from Utility.scRNA_bam2countmat (UMI_modeling=True)
- Optional user-supplied cell label file (celllabel_file) that bypasses clustering

## Outputs

- Discrete per-cell cluster/type labels usable as celllabel_file input to GenerateSyntheticCount.scRNA_GenerateSyntheticCount

## How to apply

Start from the real gene-by-cell UMI count matrix produced by scReadSim's Utility.scRNA_bam2countmat with UMI_modeling=True, ensuring all cells are present. If no celllabel_file with predefined cell types is supplied, run Louvain clustering via the Seurat R package on this gene-by-cell UMI count matrix to assign each cell to a discrete cluster. scDesign2 requires discrete cell types to fit its count model per cell type (it uses mclapply parallel model fitting per type, so labels must be complete and non-empty), so feed the resulting cluster labels into GenerateSyntheticCount.scRNA_GenerateSyntheticCount with UMI_modeling=True and defaults n_cell_new=None and total_count_new=None so the synthetic matrix keeps the real cell number and sequencing depth. The same labeling applies to training on the inter-gene-by-cell UMI count matrix, which must keep the same cells in the same column order.

## Related tools

- **Seurat** (R package used to perform Louvain clustering on the gene-by-cell UMI count matrix to obtain discrete cell-type labels for scDesign2 training)
- **scDesign2** (Count simulator trained per Louvain cluster to generate the synthetic UMI count matrix; requires discrete cell types) — https://github.com/JSB-UCLA/scDesign2
- **scReadSim** (Wrapper pipeline in which the clustering and synthetic count generation (GenerateSyntheticCount.scRNA_GenerateSyntheticCount) are executed) — https://github.com/JSB-UCLA/scReadSim

## Examples

```
GenerateSyntheticCount.scRNA_GenerateSyntheticCount(real_count_mat_filename, output_dir, UMI_modeling=True, UMI_count_mat_filename=umi_mat)
```

## Evaluation signals

- Every cell in the UMI count matrix receives exactly one discrete cluster label (no unassigned cells), since scDesign2 fits models per cell type
- Both the gene-by-cell and inter-gene-by-cell matrices share the same cells in the same column order before labeling
- Resulting synthetic gene-by-cell UMI matrix retains the same feature rows as the input matrix and contains synthetic cell columns
- Synthetic matrix summary statistics (gene-wise mean, variance, coefficient of variance, zero proportion; cell-wise library size and zero proportion) mimic the real UMI count matrix

## Limitations

- Louvain clusters are statistical groupings, not curated cell types; mislabeling propagates into scDesign2 training and the synthetic ground truth
- If cells actually follow continuous trajectories, discrete Louvain labels are the wrong model and scDesign3 should be substituted
- scDesign2's mclapply-based per-cell-type model fitting runs only on Unix-like systems, so clustering-label-dependent training is slower on Windows

## Evidence

- [methods] Louvain clustering is required before scDesign2 training: [evidence span withheld — anchor status: not_found]
- [intro] Article confirms scDesign2 trains on cells in distinct clusters: "scReadSim trains the count simulator scDesign26 (if the cells belong to distinct clusters; otherwise, scDesign322 can be used"
- [readme] scDesign2 README on clustering before use when labels unavailable: [evidence span withheld — anchor status: external_doc]
- [methods] Synthetic matrix validation against real UMI counts: [evidence span withheld — anchor status: not_found]
