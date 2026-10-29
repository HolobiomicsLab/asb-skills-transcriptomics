---
name: umap-embedding-projection-and-distance-computation
description: Use when benchmarking UMI deduplication or read-level count-recovery
  tools against a ground-truth gene-by-cell UMI matrix and you need a cell-position-level
  accuracy metric (mean Euclidean UMAP distance per tool) rather than only correlation
  statistics.
when_to_use_negative:
- The input is raw synthetic or real sequencing reads rather than already-deduplicated
  gene-by-cell count matrices — deduplication (UMI-tools/STARsolo/cellranger) must
  happen first
- The tool under test studies isoforms or alternative splicing — the scReadSim authors
  do not recommend their ground truths for that purpose (gene-level UMI ground truths
  only)
- You need an embedding metric where cells unseen by all tools can be compared — comparisons
  are restricted to the common cells shared across all tools
tools:
- name: umap (R package)
  role: Fits the top-30-PC ground-truth UMAP embedding and projects deduplicated matrices
    into it via predict()
- name: scReadSim
  role: Generates the synthetic scRNA-seq FASTQ reads and the gene-by-cell UMI count
    matrix used as ground truth for the embedding
  repo: https://github.com/JSB-UCLA/scReadSim
- name: UMI-tools
  role: Produces a deduplicated UMI matrix that is projected and scored by mean UMAP
    distance
- name: STARsolo
  role: Produces a deduplicated UMI matrix that is projected and scored by mean UMAP
    distance
- name: cellranger
  role: Produces a deduplicated UMI matrix (include-introns=false) that is projected
    and scored by mean UMAP distance
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1038/s41467-023-43162-w
    title: 'scReadSim: a single-cell RNA-seq and ATAC-seq read simulator'
  source_content_hash: sha256:7f5b27d866eff9367dbd44ffe3797229bdbdcd8eedb11302090cd1f2676f4986
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/JSB-UCLA/scReadSim
  tools:
  - STARsolo
  - UMI-tools
  - cellranger
  - scReadSim
  - umap (R package)
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 6
---

# umap-embedding-projection-and-distance-computation

## Summary

A benchmark-accuracy skill from the scReadSim UMI-deduplication study (Fig. 4d): fit a UMAP embedding on the ground-truth UMI count matrix (after log-transformation and top-30-PC PCA), project each tool's deduplicated matrix onto that fixed embedding via the R umap package's predict() function, and score each tool by mean Euclidean distance between projected and ground-truth cell positions.

## When to use

Use when benchmarking UMI deduplication or read-level count-recovery tools against a ground-truth gene-by-cell UMI matrix and you need a cell-position-level accuracy metric (mean Euclidean UMAP distance per tool) rather than only correlation statistics.

## When NOT to use

- The input is raw synthetic or real sequencing reads rather than already-deduplicated gene-by-cell count matrices — deduplication (UMI-tools/STARsolo/cellranger) must happen first
- The tool under test studies isoforms or alternative splicing — the scReadSim authors do not recommend their ground truths for that purpose (gene-level UMI ground truths only)
- You need an embedding metric where cells unseen by all tools can be compared — comparisons are restricted to the common cells shared across all tools

## Inputs

- Ground-truth gene-by-cell UMI count matrix from scReadSim (synthetic scRNA-seq, mouse 10x single-cell Multiome RNA modality)
- Deduplicated gene-by-cell count matrices from UMI-tools, STARsolo, and cellranger (include-introns=false) run on the synthetic FASTQ reads
- Cell-index mapping restricting all matrices to the common cells outputted by all tools

## Outputs

- Per-tool mean Euclidean UMAP distance between projected deduplicated cells and ground-truth cells (Fig. 4d statistic)
- Projected cell coordinates of each deduplicated matrix in the shared ground-truth UMAP embedding

## How to apply

First restrict all count matrices to the common cells outputted by all deduplication tools (UMI-tools, STARsolo, cellranger) and remove genes with zero expression in any matrix. Log-transform every gene-by-cell UMI matrix, run PCA on the ground-truth matrix keeping the top 30 PCs, and fit a UMAP embedding on those ground-truth PCs. Project each tool's deduplicated cells into the fixed ground-truth UMAP embedding using the predict() function of the R umap package — do not refit UMAP per tool, since the shared embedding is what makes distances comparable. Compute the mean Euclidean distance between each tool's projected cells and the corresponding ground-truth cells; the tool with the smallest mean distance is the most accurate at preserving cell-level structure (this is the metric behind Fig. 4d of the scReadSim paper, where UMI-tools attained the top accuracy).

## Related tools

- **umap (R package)** (Fits the top-30-PC ground-truth UMAP embedding and projects deduplicated matrices into it via predict())
- **scReadSim** (Generates the synthetic scRNA-seq FASTQ reads and the gene-by-cell UMI count matrix used as ground truth for the embedding) — https://github.com/JSB-UCLA/scReadSim
- **UMI-tools** (Produces a deduplicated UMI matrix that is projected and scored by mean UMAP distance)
- **STARsolo** (Produces a deduplicated UMI matrix that is projected and scored by mean UMAP distance)
- **cellranger** (Produces a deduplicated UMI matrix (include-introns=false) that is projected and scored by mean UMAP distance)

## Examples

```
R: library(umap); pcs <- prcomp(t(log1p(gt_mat)))$x[,1:30]; fit <- umap(pcs); proj <- predict(fit, pcs_dedup); mean(sqrt(rowSums((proj - fit$layout[matched_cells,])^2)))
```

## Evaluation signals

- The mean Euclidean UMAP distance for UMI-tools should be the smallest among UMI-tools, STARsolo, and cellranger, matching the paper's finding that UMI-tools achieves the top accuracy (Fig. 4d)
- All matrices must be log-transformed and restricted to common cells and non-zero genes before PCA/UMAP, otherwise distances are not comparable
- Projection must use predict() on a single UMAP fit built from the ground-truth top-30 PCs; a per-tool refit invalidates the comparison
- Distances should be computed only between matched cells (same cell in projected and ground-truth embeddings), and averaged per tool

## Limitations

- scReadSim's default implementation may generate intronic synthetic reads, which can confound benchmarks such as cellranger's default mode; hence include-introns=false is used for cellranger
- scReadSim ground truths are gene-level UMI counts only — no isoform-level ground truth — so this embedding metric is meaningless for isoform or splicing benchmarks
- The metric depends on the choice of top-30 PCs and the shared ground-truth embedding; results hold for the mouse 10x Multiome RNA-seq synthetic dataset used in the paper and may vary on other datasets

## Evidence

- [full_text] Log-transform matrices, perform PCA (top 30 PCs) on the ground-truth matrix, project deduplicated cells via the predict() function of the R umap package, and compute the mean Euclidean UMAP distance per tool: [evidence span withheld — anchor status: not_found]
- [abstract] UMI-tools achieves the top accuracy in scRNA-seq UMI deduplication: "UMI-tools achieves the top accuracy in scRNA-seq UMI deduplication"
- [full_text] restrict all comparisons to common cells and remove zero genes: [evidence span withheld — anchor status: not_found]
- [abstract] ground-truth UMI counts from scReadSim enable read-level benchmarking: "scReadSim provides ground truths, including unique molecular identifier (UMI) counts"
