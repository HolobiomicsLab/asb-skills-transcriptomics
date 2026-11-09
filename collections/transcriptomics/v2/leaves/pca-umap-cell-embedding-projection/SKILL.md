---
name: pca-umap-cell-embedding-projection
description: 'Use when comparing synthetic scATAC-seq cells to real cells at the cell-embedding
  level: you have peak-by-cell count matrices for real and simulated cells (BAM/BED-derived)
  and need a quantitative mixing score such as miLISI.'
when_to_use_negative:
- When the research question is read-sequence or read-count fidelity alone and no
  cell-embedding mixing comparison is required; KS tests on the six summary statistics
  suffice
- When a simulator's output cannot be converted to counts over the same trustworthy
  peak set (BAM or BED), since real and synthetic cells must share the same feature
  space for joint PCA/UMAP projection
- When benchmarking isoform or alternative splicing tools; the article explicitly
  does not recommend the current scReadSim version for that scope
tools:
- name: scReadSim
  role: Generates the synthetic scATAC-seq reads (output BAM) whose cells are projected
    into the real-cell PCA/UMAP embedding; trained via scDesign2 on the trustworthy
    peak-by-cell count matrix
  repo: https://github.com/JSB-UCLA/scReadSim
- name: MACS3
  role: Calls trustworthy peaks (q-value 0.01) from the real sci-ATAC-seq BAM, defining
    the shared feature space for all peak-by-cell count matrices
  repo: https://github.com/macs3-project/MACS
- name: R umap package
  role: Fits the UMAP embedding on real-cell PCs and provides predict() to project
    synthetic cells into the same space
- name: scDesign2
  role: Count simulator trained on the trustworthy peak-by-cell matrix to generate
    synthetic UMI/read counts for scReadSim
  repo: https://github.com/JSB-UCLA/scDesign2
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1038/s41467-023-43162-w
    title: 'scReadSim: a single-cell RNA-seq and ATAC-seq read simulator'
  source_content_hash: sha256:5dba7114566295be66e98a2def913fc63a12dc96f5a74fa4846137cb8981818b
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/JSB-UCLA/scReadSim
  tools:
  - MACS3
  - R umap package
  - SCAN-ATAC-Sim
  - scDesign2
  - scReadSim
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 11
---

# pca-umap-cell-embedding-projection

## Summary

A cell-embedding comparison skill from the scReadSim paper (Fig. 2b): real and synthetic scATAC-seq cells are placed in a shared PCA/UMAP space so that mixing between real and synthetic cells can be quantified with the miLISI score.

## When to use

Use when comparing synthetic scATAC-seq cells to real cells at the cell-embedding level: you have peak-by-cell count matrices for real and simulated cells (BAM/BED-derived) and need a quantitative mixing score such as miLISI.

## When NOT to use

- When the research question is read-sequence or read-count fidelity alone and no cell-embedding mixing comparison is required; KS tests on the six summary statistics suffice
- When a simulator's output cannot be converted to counts over the same trustworthy peak set (BAM or BED), since real and synthetic cells must share the same feature space for joint PCA/UMAP projection
- When benchmarking isoform or alternative splicing tools; the article explicitly does not recommend the current scReadSim version for that scope

## Inputs

- Real scATAC-seq peak-by-cell read count matrix (sci-ATAC-seq, counts over MACS3 trustworthy peaks at q-value 0.01)
- scReadSim synthetic peak-by-cell count matrix (counts from output BAM over the same trustworthy peaks)
- SCAN-ATAC-Sim synthetic peak-by-cell count matrix (counts from output BED over the same trustworthy peaks)
- Per-cell-type labels used to generate equal numbers of synthetic and real cells per type

## Outputs

- Shared UMAP embedding containing real and projected synthetic cells
- miLISI mixing score per simulator (range 1 to 2, perfect mixing = 2), e.g., recorded in scan_atac_sim_comparison_metrics.csv
- KS statistics for six summary statistics of each synthetic count matrix against the real matrix (Supplementary Fig. 6a)

## How to apply

Construct peak-by-cell read count matrices for real cells (sci-ATAC-seq), scReadSim synthetic cells (from its output BAM), and SCAN-ATAC-Sim synthetic cells (from its output BED), counting reads overlapping MACS3-called trustworthy peaks (q-value 0.01 peaks; non-peaks as complement of q-value 0.1 peaks). Preprocess all three matrices identically with TF-IDF followed by logarithmic transformation. Fit PCA on the real count matrix only, retaining the top 30 PCs, embed real cells with UMAP, and project synthetic cells into the same PC and UMAP spaces using the 30-by-p loading matrix and the predict() function of the R umap package. Compute the miLISI mixing score (range 1 to 2) between real and synthetic cells in the shared UMAP embedding; a simulator is judged better when its cells mix with real cells at higher miLISI, closer to the perfect-mixing value of 2, complemented by smaller two-sample Kolmogorov-Smirnov statistics on six summary statistics (peak-wise mean, variance, coefficient of variance, zero proportion; cell-wise library size, zero proportion).

## Related tools

- **scReadSim** (Generates the synthetic scATAC-seq reads (output BAM) whose cells are projected into the real-cell PCA/UMAP embedding; trained via scDesign2 on the trustworthy peak-by-cell count matrix) — https://github.com/JSB-UCLA/scReadSim
- **MACS3** (Calls trustworthy peaks (q-value 0.01) from the real sci-ATAC-seq BAM, defining the shared feature space for all peak-by-cell count matrices) — https://github.com/macs3-project/MACS
- **R umap package** (Fits the UMAP embedding on real-cell PCs and provides predict() to project synthetic cells into the same space)
- **scDesign2** (Count simulator trained on the trustworthy peak-by-cell matrix to generate synthetic UMI/read counts for scReadSim) — https://github.com/JSB-UCLA/scDesign2

## Examples

```
library(umap); um_fit <- umap(real_pcs[, 1:30]); synthetic_umap <- predict(um_fit, synthetic_pcs[, 1:30])
```

## Evaluation signals

- miLISI between real and synthetic cells should be closer to 2 (perfect mixing) for the better simulator; scReadSim outperformed SCAN-ATAC-Sim in the UMAP embedding (Fig. 2b)
- All three count matrices must be preprocessed identically (TF-IDF then logarithmic transformation) before PCA; PCA is fit on the real matrix only and synthetic cells are projected via the 30-by-p loading matrix
- The PC and UMAP spaces must retain exactly the top 30 PCs for both real embedding and synthetic projection
- Each simulator should produce the same number of synthetic cells as real cells per cell type
- Supporting check: two-sample Kolmogorov-Smirnov tests of the six summary statistics should show smaller KS statistics for scReadSim than SCAN-ATAC-Sim versus the real matrix

## Limitations

- miLISI ranges only from 1 to 2; values must be interpreted relative to the perfect-mixing value of 2 rather than as absolute accuracy
- PCA and UMAP are fit exclusively on the real matrix, so synthetic-cell placement depends on the trustworthiness of the MACS3 peak set (stringent q-value 0.01 peaks vs. non-peaks from the complement of q-value 0.1 peaks)
- SCAN-ATAC-Sim does not learn from real scATAC-seq data, so its lower miLISI reflects a fundamental limitation of that simulator rather than a projection artifact
- The default implementation of scReadSim may generate intronic synthetic reads, which can affect downstream count-based comparisons

## Evidence

- [methods] PCA/UMAP shared-space projection workflow: [evidence span withheld — anchor status: not_found]
- [methods] miLISI mixing metric definition: [evidence span withheld — anchor status: not_found]
- [methods] Preprocessing applied identically to all matrices: [evidence span withheld — anchor status: not_found]
- [intro] Why scReadSim is expected to mix better: [evidence span withheld — anchor status: not_found]
- [readme] Synthetic data source for the embedding comparison: [evidence span withheld — anchor status: external_doc]
