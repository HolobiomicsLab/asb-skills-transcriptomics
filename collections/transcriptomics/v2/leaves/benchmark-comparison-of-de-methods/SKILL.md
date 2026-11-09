---
name: benchmark-comparison-of-de-methods
description: Use when you must validate a temporal differential expression method's
  detection performance against established DE tools on simulated single-cell counts
  with a known ground-truth gene set.
when_to_use_negative:
- The goal is differential expression on a single time point / cross-sectional scRNA-seq
  experiment — plain DESeq2, MAST, or Seurat FindMarkers alone suffices without the
  temporal simulation benchmark
- Input lacks a known ground-truth tDEG set (e.g. real data without planted signals)
  — TDR cannot be computed, so use alternative validation (supplementary ARI comparison,
  clustering) instead
- Evaluating cell annotation or novel-cell detection rather than temporal differential
  expression — those CASi modules (CrossTimeAnnotation, DetectNovelCells) are benchmarked
  separately (ARI in supplementary Fig 11)
edam_operation: http://edamontology.org/operation_3802
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_0204
tools:
- name: CASi
  role: Method under evaluation; FindTemporalDEGs combines a generalized linear model
    with iterative feature selection using pct.cutoff = 0.3 and p.cutoff = 0.05
  repo: github.com/yizhuo-wang/CASi
- name: DESeq2
  role: Baseline differential expression method run on the same simulated counts for
    TDR comparison
- name: MAST
  role: Baseline differential expression method run on the same simulated counts for
    TDR comparison
- name: Seurat
  role: Provides the Wilcoxon-rank-sum FindMarkers baseline run on the same simulated
    counts for TDR comparison
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1038/s41598-024-58566-x
    title: 'CASi: A framework for cross-timepoint analysis of single-cell RNA sequencing
      data'
  source_content_hash: sha256:ad417cd3d74c0bad0f156e5e8640694772cceccde5743310e05e8552c7a9e68b
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - CASi
  - DESeq2
  - MAST
  - Seurat
  edam_operation: http://edamontology.org/operation_3802
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_0204
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.625
n_claims: 8
---

# benchmark-comparison-of-de-methods

## Summary

A benchmarking skill that evaluates CASi's FindTemporalDEGs against conventional differential expression methods (DESeq2, MAST, Seurat Wilcoxon) on simulated longitudinal scRNA-seq data with planted tDEGs, measuring true discovery rate (TDR) across p-value thresholds. It grounds claims of favorable performance in a reproducible simulation-plus-baseline comparison rather than a single method run.

## When to use

Use when you must validate a temporal differential expression method's detection performance against established DE tools on simulated single-cell counts with a known ground-truth gene set.

## When NOT to use

- The goal is differential expression on a single time point / cross-sectional scRNA-seq experiment — plain DESeq2, MAST, or Seurat FindMarkers alone suffices without the temporal simulation benchmark
- Input lacks a known ground-truth tDEG set (e.g. real data without planted signals) — TDR cannot be computed, so use alternative validation (supplementary ARI comparison, clustering) instead
- Evaluating cell annotation or novel-cell detection rather than temporal differential expression — those CASi modules (CrossTimeAnnotation, DetectNovelCells) are benchmarked separately (ARI in supplementary Fig 11)

## Inputs

- Simulated longitudinal scRNA-seq count matrix (900 monocytes x 2000 genes from the public PBMC reference dataset) with 300 planted cell-type-specific tDEGs
- Time-effect level specification (weak / medium / strong settings)
- Gene expression matrix and cell labels for CASiList construction (e.g. pbmc_demo, CASiList_tDEG_demo)
- Baseline method software: DESeq2, MAST, Seurat (R/Bioconductor packages)

## Outputs

- Per-method true-discovery and false-discovery counts at each p-value threshold per repetition
- TDR curves versus threshold averaged over 200 repetitions for CASi, DESeq2, MAST, and Seurat Wilcoxon
- Comparison figure (tdr_curves.png) showing CASi's TDR at or above baseline methods across thresholds

## How to apply

Simulate longitudinal single-cell counts from a real reference dataset — the paper samples 900 monocytes and 2000 genes from a public PBMC dataset (60,000+ sorted cells from eight immune cell types) — injecting 300 cell-type-specific tDEGs at weak, medium, and strong time-effect levels. In each of 200 repetitions, run CASi's FindTemporalDEGs (a negative-binomial GLM with iterative feature selection) using pct.cutoff = 0.3 (percentage of cells expressing a gene) and p.cutoff = 0.05, and run the same simulated counts through DESeq2, MAST, and Seurat's Wilcoxon-rank-sum FindMarkers as baselines. At each repetition, compute per-method true-discovery and false-discovery counts against the known simulated tDEG set across a grid of p-value thresholds. Average over repetitions and plot TDR curves versus threshold for all four methods (saved as tdr_curves.png). Judge success by whether CASi's TDR is comparable to or above the baselines at every threshold, and check the expected qualitative pattern: CASi's TDR rises with stronger time effect while competitors' TDR falls.

## Related tools

- **CASi** (Method under evaluation; FindTemporalDEGs combines a generalized linear model with iterative feature selection using pct.cutoff = 0.3 and p.cutoff = 0.05) — github.com/yizhuo-wang/CASi
- **DESeq2** (Baseline differential expression method run on the same simulated counts for TDR comparison)
- **MAST** (Baseline differential expression method run on the same simulated counts for TDR comparison)
- **Seurat** (Provides the Wilcoxon-rank-sum FindMarkers baseline run on the same simulated counts for TDR comparison)

## Examples

```
require(keras); require(CASi); res <- FindTemporalDEGs(CASiList_tDEG_demo, pct.cutoff = 0.3, p.cutoff = 0.05)
```

## Evaluation signals

- CASi's TDR curve is comparable to or above DESeq2, MAST, and Seurat Wilcoxon at all p-value thresholds in the averaged plot (tdr_curves.png)
- CASi's TDR increases monotonically with stronger time effect (weak < medium < strong), while competitor methods' TDR decreases across the same settings
- True-discovery counts match the 300 planted tDEG set: TDR = true discoveries / (true + false discoveries) per method per threshold, averaged over 200 repetitions
- Simulation is anchored to real data: 900 monocytes and 2000 genes sampled from the public PBMC dataset with >60,000 sorted cells from eight immune cell types
- FindTemporalDEGs parameters recorded per repetition: pct.cutoff = 0.3 and p.cutoff = 0.05, matching the paper's reported configuration

## Limitations

- CASi's GLM framework can only detect linear effects — the linear change of gene expression by time or the time-by-condition interaction — not nonlinear temporal changes, so the benchmark understates differences on nonlinear signals
- Simulation scenario settings and full ARI comparison results are only in supplementary Figure 11, not the main text, complicating direct reproduction of annotation-side comparisons
- Version numbers of comparison tools (Seurat, MAST, Monocle, BPSC) are not stated in the paper, so exact baseline reproduction may vary across package versions
- The PBMC source dataset citation [1] is not resolved to an accession in the methods section, so re-sampling the 900 monocytes / 2000 genes requires locating the original reference independently

## Evidence

- [abstract] Simulation studies and real multi-timepoint single cell dataset applications showed robust and favorable performance versus existing methods with similar purposes: [evidence span withheld — anchor status: not_found]
- [discussion] GLM linear-only limitation: "the current GLM framework can only detect linear effect, i.e., the linear change of gene expression by time or the interaction of time and condition"
- [methods] PBMC reference dataset description: "We obtain a publicly available dataset of peripheral blood mononuclear cells (PBMC)[1], containing more than 60,000 sorted cells from eight immune cell types."
- [intro] Comparison methods named: "e.g., DESeq27 and MAST8."
- [readme] Package installation from GitHub: [evidence span withheld — anchor status: external_doc]
