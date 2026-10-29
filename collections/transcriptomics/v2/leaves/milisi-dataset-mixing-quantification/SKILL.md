---
name: milisi-dataset-mixing-quantification
description: Use when comparing synthetic scATAC-seq or scRNA-seq cells against real
  cells in UMAP space and a numeric dataset-mixing score (miLISI, closer to 2 = perfect
  mixing) is needed, e.g. benchmarking read simulators.
when_to_use_negative:
- The synthetic data are count matrices from a simulator that already defines shared
  feature spaces differing from the real-data peaks, making UMAP projection onto the
  real PCA basis ill-defined
- You need read-sequence-level fidelity assessment rather than cell-level embedding
  mixing; use summary-statistic KS tests instead
- Cells lack cell-type labels, so per-cell-type matched numbers of synthetic cells
  cannot be generated for a fair mixing comparison
edam_topics:
- http://edamontology.org/topic_0102
tools:
- name: scReadSim
  role: Synthetic scATAC-seq read generator trained via scDesign2 on trustworthy peak-by-cell
    counts; its output BAM supplies the synthetic cells projected into the UMAP space
  repo: https://github.com/JSB-UCLA/scReadSim
- name: SCAN-ATAC-Sim
  role: Baseline read simulator run with default parameters on per-cell-type pseudo-bulk
    BAM files and MACS3 peak lists; its output BED supplies comparison cells
- name: MACS3
  role: Calls trustworthy peaks (q-value 0.01) and relaxed peaks (q-value 0.1) whose
    complement defines non-peaks for the shared feature space
  repo: https://github.com/macs3-project/MACS
- name: scDesign2
  role: Trains the count model on the real peak-by-cell matrix so scReadSim generates
    the same number of synthetic cells per cell type as real cells
  repo: https://github.com/JSB-UCLA/scDesign2
- name: umap (R package)
  role: Embeds real cells and, via predict(), projects synthetic cells into the same
    UMAP space before miLISI computation
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1038/s41467-023-43162-w
    title: 'scReadSim: a single-cell RNA-seq and ATAC-seq read simulator'
  source_content_hash: sha256:4bf7f02612ae03b51b28913f44409a8ded0ae4230f1f8d3c358e61c68784fa77
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/JSB-UCLA/scReadSim
  tools:
  - MACS3
  - SCAN-ATAC-Sim
  - scDesign2
  - scReadSim
  - umap (R package)
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0102
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 11
---

# milisi-dataset-mixing-quantification

## Summary

Quantify how well synthetic single-cell datasets mix with real cells in a shared UMAP embedding using the miLISI mixing score (range 1 to 2), as used to compare scReadSim against SCAN-ATAC-Sim on sci-ATAC-seq data (Fig. 2b).

## When to use

Use when comparing synthetic scATAC-seq or scRNA-seq cells against real cells in UMAP space and a numeric dataset-mixing score (miLISI, closer to 2 = perfect mixing) is needed, e.g. benchmarking read simulators.

## When NOT to use

- The synthetic data are count matrices from a simulator that already defines shared feature spaces differing from the real-data peaks, making UMAP projection onto the real PCA basis ill-defined
- You need read-sequence-level fidelity assessment rather than cell-level embedding mixing; use summary-statistic KS tests instead
- Cells lack cell-type labels, so per-cell-type matched numbers of synthetic cells cannot be generated for a fair mixing comparison

## Inputs

- BAM file of synthetic scReadSim reads overlapping trustworthy MACS3 peaks
- BED file of SCAN-ATAC-Sim synthetic reads
- BAM file of real sci-ATAC-seq reads
- MACS3-called peak lists (q-value 0.01 for peaks; complement of q-value 0.1 for trustworthy non-peaks)
- Per-cell-type labels used to generate matching numbers of synthetic cells

## Outputs

- Real, scReadSim, and SCAN-ATAC-Sim peak-by-cell read count matrices
- Joint UMAP embedding of real and projected synthetic cells
- miLISI mixing scores per simulator (range 1 to 2), recorded in scan_atac_sim_comparison_metrics.csv
- Six summary statistics with two-sample Kolmogorov-Smirnov test statistics vs. the real matrix

## How to apply

Build a peak-by-cell count matrix for real, scReadSim, and SCAN-ATAC-Sim cells by counting reads/BED intervals overlapping MACS3-called trustworthy peaks (q-value 0.01 peaks; non-peaks from the complement of relaxed q-value 0.1 calls). Preprocess all three matrices with TF-IDF followed by logarithmic transformation, run PCA on the real matrix only to get the top 30 PCs, embed real cells with UMAP, then project synthetic cells into the same PC and UMAP spaces using the 30-by-p loading matrix and the predict() function of the R umap package. Compute the miLISI mixing score between real and synthetic cells in the shared UMAP embedding; a higher miLISI, closer to the perfect-mixing value of 2, indicates the simulator mimics the real data better (scReadSim outperforms SCAN-ATAC-Sim, which does not learn from real scATAC-seq data).

## Related tools

- **scReadSim** (Synthetic scATAC-seq read generator trained via scDesign2 on trustworthy peak-by-cell counts; its output BAM supplies the synthetic cells projected into the UMAP space) — https://github.com/JSB-UCLA/scReadSim
- **SCAN-ATAC-Sim** (Baseline read simulator run with default parameters on per-cell-type pseudo-bulk BAM files and MACS3 peak lists; its output BED supplies comparison cells)
- **MACS3** (Calls trustworthy peaks (q-value 0.01) and relaxed peaks (q-value 0.1) whose complement defines non-peaks for the shared feature space) — https://github.com/macs3-project/MACS
- **scDesign2** (Trains the count model on the real peak-by-cell matrix so scReadSim generates the same number of synthetic cells per cell type as real cells) — https://github.com/JSB-UCLA/scDesign2
- **umap (R package)** (Embeds real cells and, via predict(), projects synthetic cells into the same UMAP space before miLISI computation)

## Examples

```
syn_umap <- predict(umap(real_pca30, n_components=2)$embedding_fit, syn_pca30)
```

## Evaluation signals

- miLISI for scReadSim cells is higher than for SCAN-ATAC-Sim cells and closer to the perfect-mixing value of 2 (Fig. 2b)
- Synthetic cells land within the same PC/UMAP space as real cells via the 30-by-p loading matrix and predict(), without refitting UMAP on synthetic data
- All three count matrices are computed over the same trustworthy peak set (q-value 0.01 peaks) so the comparison is feature-matched
- KS statistics of the six summary statistics (peak-wise mean, variance, coefficient of variance, zero proportion; cell-wise library size, zero proportion) are smaller for scReadSim than SCAN-ATAC-Sim (Supplementary Fig. 6a)
- Metrics are written to scan_atac_sim_comparison_metrics.csv with one KS statistic and miLISI per simulator

## Limitations

- SCAN-ATAC-Sim does not learn from real scATAC-seq data, so a lower miLISI is expected and the comparison favors learning-based simulators
- miLISI depends on the UMAP embedding parameters and the TF-IDF + log preprocessing of all matrices; changes to the pipeline shift the score
- Only trustworthy peaks are used; regions outside MACS3 q-value 0.01 peaks and the complement of q-value 0.1 peaks are excluded from the mixing assessment

## Evidence

- [full_text] UMAP projection step: [evidence span withheld — anchor status: not_found]
- [full_text] miLISI computation: [evidence span withheld — anchor status: not_found]
- [intro] Rationale for simulator comparison: "For scATAC-seq, SCAN-ATAC-Sim20 is the only read simulator to our knowledge."
- [readme] Synthetic read generation context: [evidence span withheld — anchor status: external_doc]
