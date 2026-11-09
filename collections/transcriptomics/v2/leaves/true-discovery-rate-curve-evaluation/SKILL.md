---
name: true-discovery-rate-curve-evaluation
description: Use when you have simulated or spike-in single-cell count data with a
  known ground-truth differentially expressed gene set, and need to compare TDR of
  competing DE methods across a threshold grid.
when_to_use_negative:
- No ground-truth tDEG set exists (e.g., real single-timepoint data with unknown true
  DE status) — true and false discovery counts cannot be computed
- The data are cross-sectional with no temporal dimension — CASi's FindTemporalDEGs
  and its GLM-based time-effect test do not apply
- The expected time effect is nonlinear — CASi's GLM framework only detects linear
  change of gene expression by time or time-by-condition interaction
edam_operation: http://edamontology.org/operation_2945
edam_topics:
- http://edamontology.org/topic_3170
tools:
- name: CASi
  role: Method under test; its FindTemporalDEGs (generalized linear model with iterative
    feature selection, pct.cutoff = 0.3, p.cutoff = 0.05) supplies tDEG p-values for
    TDR computation
  repo: github.com/yizhuo-wang/CASi
- name: DESeq2
  role: Baseline differential expression method run on the same simulated counts for
    TDR comparison
- name: MAST
  role: Baseline differential expression method run on the same simulated counts for
    TDR comparison
- name: Seurat
  role: Provides the Wilcoxon-rank-sum FindMarkers baseline on the same simulated
    counts
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1038/s41598-024-58566-x
    title: 'CASi: A framework for cross-timepoint analysis of single-cell RNA sequencing
      data'
  source_content_hash: sha256:71487d416c0948d22a1393c3fe41860aeeb1d40b14f22f2c425096d78da633d9
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - CASi
  - DESeq2
  - MAST
  - Seurat
  edam_operation: http://edamontology.org/operation_2945
  edam_topics:
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7272727272727273
n_claims: 11
---

# true-discovery-rate-curve-evaluation

## Summary

Benchmark evaluation of temporal differential expression detection by comparing per-method true-discovery and false-discovery counts against a known planted tDEG set, then averaging over repetitions to produce true discovery rate (TDR) curves across p-value thresholds. Used in the CASi paper to show CASi's negative-binomial-GLM FindTemporalDEGs outperforms DESeq2, MAST, and Seurat Wilcoxon under weak, medium, and strong time effects.

## When to use

Use when you have simulated or spike-in single-cell count data with a known ground-truth differentially expressed gene set, and need to compare TDR of competing DE methods across a threshold grid.

## When NOT to use

- No ground-truth tDEG set exists (e.g., real single-timepoint data with unknown true DE status) — true and false discovery counts cannot be computed
- The data are cross-sectional with no temporal dimension — CASi's FindTemporalDEGs and its GLM-based time-effect test do not apply
- The expected time effect is nonlinear — CASi's GLM framework only detects linear change of gene expression by time or time-by-condition interaction

## Inputs

- Simulated longitudinal single-cell count matrix (e.g., 900 monocytes × 2000 genes from the public PBMC dataset)
- Known ground-truth set of 300 planted cell-type-specific tDEGs (one set per time-effect level: weak, medium, strong)
- Per-repetition p-values from each method being compared: CASi FindTemporalDEGs, DESeq2, MAST, Seurat Wilcoxon FindMarkers

## Outputs

- Per-method true-discovery and false-discovery counts across a grid of p-value thresholds, averaged over 200 repetitions
- TDR curves versus p-value threshold for CASi, DESeq2, MAST, and Seurat Wilcoxon at weak/medium/strong time-effect levels
- Comparison figure saved as tdr_curves.png

## How to apply

Simulate longitudinal single-cell data (e.g., 900 monocytes and 2000 genes sampled from the public PBMC dataset of over 60,000 sorted cells from eight immune cell types), injecting a known set of 300 cell-type-specific tDEGs at each of three time-effect levels (weak, medium, strong). At each of 200 repetitions, run CASi's FindTemporalDEGs with pct.cutoff = 0.3 (percentage of cells expressing a gene) and p.cutoff = 0.05, and run the baselines DESeq2, MAST, and Seurat's Wilcoxon-rank-sum FindMarkers on the same simulated counts. For each repetition, compute per-method true-discovery and false-discovery counts against the known planted tDEG set across a grid of p-value thresholds. Average over the 200 repetitions and plot TDR curves versus threshold for all four methods in a single figure (tdr_curves.png). Rationale: TDR against a known ground truth isolates detection quality from threshold selection, and sweeping the threshold grid plus the three time-effect levels reveals method robustness rather than single-point superiority.

## Related tools

- **CASi** (Method under test; its FindTemporalDEGs (generalized linear model with iterative feature selection, pct.cutoff = 0.3, p.cutoff = 0.05) supplies tDEG p-values for TDR computation) — github.com/yizhuo-wang/CASi
- **DESeq2** (Baseline differential expression method run on the same simulated counts for TDR comparison)
- **MAST** (Baseline differential expression method run on the same simulated counts for TDR comparison)
- **Seurat** (Provides the Wilcoxon-rank-sum FindMarkers baseline on the same simulated counts)

## Examples

```
results <- FindTemporalDEGs(CASiList_tDEG_demo, pct.cutoff = 0.3, p.cutoff = 0.05)
```

## Evaluation signals

- Reproduced curves must show CASi's TDR comparable to or above DESeq2, MAST, and Seurat Wilcoxon at all thresholds in all three time-effect settings
- CASi's TDR should increase with stronger time effect while competitors' TDR decreases — check this monotonic separation across weak/medium/strong settings
- True/false discovery counts must be computed against exactly the known 300 planted tDEG set, not the full gene list
- Results should be stable as an average over 200 repetitions; large run-to-run variance suggests incorrect repetition handling
- Single output figure (tdr_curves.png) contains all four methods' TDR-versus-threshold curves for direct visual comparison

## Limitations

- CASi's GLM framework can only detect linear effects — the linear change of gene expression by time or the time-by-condition interaction — so tDEG detection and hence TDR benchmarking will miss nonlinear temporal changes
- The evaluation depends entirely on simulation fidelity: planted tDEGs in simulated monocyte data (900 cells × 2000 genes from the PBMC dataset) may not reflect real multi-timepoint biology
- Results at weak time-effect levels may show reduced separation between methods, making conclusions sensitive to the injected effect sizes

## Evidence

- [methods] PBMC source dataset: "We obtain a publicly available dataset of peripheral blood mononuclear cells (PBMC)[1], containing more than 60,000 sorted cells from eight immune cell types."
- [methods] FindTemporalDEGs function parameters: "Below are the input parameters of the FindTemporalDEGs function"
- [abstract] Overall favorable performance claim: "we demonstrate the robust and favorable performance of the proposal versus existing methods serving similar purposes"
- [readme] CASi package scope from README: "CASi is a comprehensive R package for analyzing multi-timepoint scRNA-seq data"
