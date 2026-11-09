---
name: tfidf-log-count-matrix-preprocessing
description: Use when comparing synthetic single-cell count matrices (e.g., from scReadSim
  or SCAN-ATAC-Sim) against real scATAC-seq counts in a shared low-dimensional space
  to compute mixing metrics such as miLISI.
when_to_use_negative:
- Input is raw sequencing reads or a BAM/BED file without a peak-by-cell count matrix
  — construct count matrices first (e.g., via Utility.scATAC_bam2countmat) before
  applying this preprocessing
- The analysis target is read-sequence fidelity rather than cell-embedding mixing
  — use summary statistics (peak-wise mean, variance, CV, zero proportion; cell-wise
  library size, zero proportion) with KS tests instead
- Data are already normalized embeddings — TF-IDF + log transform would distort an
  already-reduced representation
edam_operation: http://edamontology.org/operation_2939
edam_topics:
- http://edamontology.org/topic_0091
- http://edamontology.org/topic_3308
tools:
- name: scReadSim
  role: Generates the synthetic scATAC-seq reads (output BAM) whose counts are TF-IDF/log-transformed
    and projected into the real cells' PCA/UMAP space
  repo: https://github.com/JSB-UCLA/scReadSim
- name: SCAN-ATAC-Sim
  role: Baseline read simulator whose output BED counts undergo the same TF-IDF/log
    preprocessing and projection for miLISI comparison
- name: umap (R package)
  role: Provides the UMAP embedding of real cells and the predict() function used
    to project synthetic cells into the same embedding
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1038/s41467-023-43162-w
    title: 'scReadSim: a single-cell RNA-seq and ATAC-seq read simulator'
  source_content_hash: sha256:9c1b08b466d480e77dd48d5c49e9d61acf9694b4d4dd54ff82761921b3874298
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/JSB-UCLA/scReadSim
  tools:
  - SCAN-ATAC-Sim
  - scReadSim
  - umap (R package)
  edam_operation: http://edamontology.org/operation_2939
  edam_topics:
  - http://edamontology.org/topic_0091
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 10
---

# tfidf-log-count-matrix-preprocessing

## Summary

Preprocess scATAC-seq peak-by-cell read count matrices (real and synthetic) with TF-IDF followed by logarithmic transformation, then embed them in a shared PCA/UMAP space to quantify how well synthetic cells mix with real cells (miLISI).

## When to use

Use when comparing synthetic single-cell count matrices (e.g., from scReadSim or SCAN-ATAC-Sim) against real scATAC-seq counts in a shared low-dimensional space to compute mixing metrics such as miLISI.

## When NOT to use

- Input is raw sequencing reads or a BAM/BED file without a peak-by-cell count matrix — construct count matrices first (e.g., via Utility.scATAC_bam2countmat) before applying this preprocessing
- The analysis target is read-sequence fidelity rather than cell-embedding mixing — use summary statistics (peak-wise mean, variance, CV, zero proportion; cell-wise library size, zero proportion) with KS tests instead
- Data are already normalized embeddings — TF-IDF + log transform would distort an already-reduced representation

## Inputs

- Real scATAC-seq peak-by-cell read count matrix (reads overlapping MACS3-called peaks, e.g., from a sci-ATAC-seq BAM file)
- Synthetic peak-by-cell count matrix from scReadSim (counted from its output BAM file)
- Synthetic peak-by-cell count matrix from SCAN-ATAC-Sim (counted from its output BED file)

## Outputs

- TF-IDF + log-transformed count matrices for real and synthetic cells
- Top 30 PC scores for real cells and projected PC scores for synthetic cells
- Shared UMAP embedding with real and synthetic cells
- miLISI mixing score per simulator (range 1 to 2), recorded with KS statistics in a metrics CSV (e.g., scan_atac_sim_comparison_metrics.csv)

## How to apply

Construct peak-by-cell read count matrices for the real data, the synthetic scReadSim data (from its output BAM file), and the synthetic SCAN-ATAC-Sim data (from its output BED file) by counting synthetic reads overlapping MACS3-called trustworthy peaks. Preprocess all three matrices with TF-IDF followed by logarithmic transformation. Perform PCA on the real matrix to obtain the top 30 PCs, embed real cells with UMAP, and project synthetic cells into the same PC and UMAP spaces using the 30-by-p loading matrix and the predict() function of the R umap package. Then compute the miLISI mixing score (range 1 to 2) between real and synthetic cells in the UMAP embedding; a simulator is judged superior when its miLISI is closer to the perfect-mixing value of 2.

## Related tools

- **scReadSim** (Generates the synthetic scATAC-seq reads (output BAM) whose counts are TF-IDF/log-transformed and projected into the real cells' PCA/UMAP space) — https://github.com/JSB-UCLA/scReadSim
- **SCAN-ATAC-Sim** (Baseline read simulator whose output BED counts undergo the same TF-IDF/log preprocessing and projection for miLISI comparison)
- **umap (R package)** (Provides the UMAP embedding of real cells and the predict() function used to project synthetic cells into the same embedding)

## Examples

```
library(umap); umap_fit <- umap(real_pcs[, 1:30]); syn_umap <- predict(umap_fit, syn_pcs[, 1:30])
```

## Evaluation signals

- miLISI between real and synthetic cells approaches the perfect-mixing value of 2 (scReadSim should exceed SCAN-ATAC-Sim's miLISI)
- PCA is fit only on the real matrix; synthetic cells are projected via the 30-by-p loading matrix rather than refit independently
- All three count matrices receive identical TF-IDF + log transformation before embedding
- Companion six summary statistics (peak-wise mean, variance, coefficient of variance, zero proportion; cell-wise library size, zero proportion) show smaller two-sample Kolmogorov-Smirnov statistics versus the real matrix
- Metrics table (e.g., scan_atac_sim_comparison_metrics.csv) contains KS statistics and miLISI for each simulator

## Limitations

- miLISI is bounded between 1 (no mixing) and 2 (perfect mixing); a low value may reflect poor synthetic data rather than preprocessing errors, so sanity-check the count matrices first
- Projecting synthetic cells with the real data's loading matrix assumes the synthetic reads were counted over the same MACS3-called peak space as the real data
- The comparison relies on trustworthy peaks (MACS3 q-value 0.01) and non-peaks (complement of q-value 0.1 calls); inconsistent peak thresholds across simulators would confound the embedding

## Evidence

- [full_text] Preprocessing before embedding: [evidence span withheld — anchor status: not_found]
- [full_text] Projection of synthetic cells: [evidence span withheld — anchor status: not_found]
- [full_text] Metric interpretation: [evidence span withheld — anchor status: not_found]
- [abstract] Why synthetic data should mix with real data: "At both read-sequence and read-count levels, scReadSim mimics real scRNA-seq and scATAC-seq data."
