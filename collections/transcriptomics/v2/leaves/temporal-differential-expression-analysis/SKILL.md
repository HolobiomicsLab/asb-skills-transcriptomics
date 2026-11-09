---
name: temporal-differential-expression-analysis
description: Use when scRNA-seq data are collected at multiple timepoints (e.g., t0/t1
  CASiList with Group and Time metadata) and the goal is genes whose expression changes
  over time or by a time-by-condition interaction, or when comparing tDEG calls with
  DESeq2/MAST/Seurat.
when_to_use_negative:
- Data are from a single cross-sectional timepoint — ordinary differential expression
  tools (DESeq2, MAST, Seurat) suffice and CASi's temporal model adds no signal
- The expected temporal effect is nonlinear — the article's GLM framework only detects
  linear change of expression by time or the time-by-condition interaction
- Cell labels for t0 are absent and the goal is annotation rather than DE — use CrossTimeAnnotation
  instead
edam_operation: http://edamontology.org/operation_3565
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_0203
tools:
- name: CASi
  role: Primary framework; FindTemporalDEGs fits a GLM with iterative feature selection
    for time and time-by-group interaction effects on a CASiList
  repo: github.com/yizhuo-wang/CASi
- name: Seurat
  role: Baseline DE method (Wilcoxon-rank-sum FindMarkers) and real-data comparison
    on the MCL dataset
- name: DESeq2
  role: Baseline differential expression method in the TDR benchmark on simulated
    PBMC data
- name: MAST
  role: Baseline differential expression method in the TDR benchmark on simulated
    PBMC data
- name: ggplot2
  role: Plots mean expression trajectories of tDEGs over time split by responders
    vs non-responders
provenance:
  source_task_ids:
  - task_005
  - task_003
  source_papers:
  - doi: 10.1038/s41598-024-58566-x
    title: 'CASi: A framework for cross-timepoint analysis of single-cell RNA sequencing
      data'
  source_content_hash: sha256:0120ef24b2f5f78fee34f9f6b6e1398be03a98551e3edbc3e0ea7710a7361b49
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - CASi
  - DESeq2
  - MAST
  - Seurat
  - dplyr
  - ggplot2
  edam_operation: http://edamontology.org/operation_3565
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6153846153846154
n_claims: 13
---

# temporal-differential-expression-analysis

## Summary

Detect temporal differentially expressed genes (tDEGs) in multi-timepoint scRNA-seq data using CASi's FindTemporalDEGs, which fits a negative-binomial GLM with iterative feature selection for time and time-by-group interaction effects, and benchmark/interpret results against standard DE methods.

## When to use

Use when scRNA-seq data are collected at multiple timepoints (e.g., t0/t1 CASiList with Group and Time metadata) and the goal is genes whose expression changes over time or by a time-by-condition interaction, or when comparing tDEG calls with DESeq2/MAST/Seurat.

## When NOT to use

- Data are from a single cross-sectional timepoint — ordinary differential expression tools (DESeq2, MAST, Seurat) suffice and CASi's temporal model adds no signal
- The expected temporal effect is nonlinear — the article's GLM framework only detects linear change of expression by time or the time-by-condition interaction
- Cell labels for t0 are absent and the goal is annotation rather than DE — use CrossTimeAnnotation instead

## Inputs

- Annotated CASiList (gene expression matrix) with metadata containing binary factor Group and numeric/categorical Time, covering t0 and t1 timepoints
- metadata_tDEG_demo / CASiList_tDEG_demo-style demonstration objects lazy-loaded with the CASi R package
- Simulated or real count matrices (e.g., PBMC monocyte-derived longitudinal counts) with a known planted tDEG set for benchmarking

## Outputs

- tDEG_results.csv: genes passing t-test p < 0.05 with time/time-by-group interaction terms
- TDR comparison curves (e.g., tdr_curves.png) of CASi vs DESeq2, MAST, Seurat Wilcoxon across p-value thresholds
- Trajectory plots (ggplot2) of example tDEGs (e.g., IFITM2, IFITM3) split by responders vs non-responders, and Hallmark/Reactome enrichment tables

## How to apply

Build a CASiList (annotated cell-expression list with metadata where Group is a binary factor and Time is numeric/categorical, row count equal to total t0+t1 cells), then run CASi::FindTemporalDEGs with p.cutoff = 0.05 and pct.cutoff = 0.3 (minimum fraction of cells expressing a gene). The function fits a generalized linear model with iterative feature selection for time effects and time-by-group interaction effects; save genes with t-test p < 0.05 as tDEG_results.csv. Benchmark by running DESeq2, MAST, and Seurat Wilcoxon FindMarkers on the same counts and comparing true discovery rate across p-value threshold grids (e.g., 300 planted tDEGs in simulated PBMC monocytes over 200 repetitions). Interpret biologically by plotting mean expression trajectories over time split by condition (ggplot2) and running Hallmark/Reactome enrichment on the tDEG set.

## Related tools

- **CASi** (Primary framework; FindTemporalDEGs fits a GLM with iterative feature selection for time and time-by-group interaction effects on a CASiList) — github.com/yizhuo-wang/CASi
- **Seurat** (Baseline DE method (Wilcoxon-rank-sum FindMarkers) and real-data comparison on the MCL dataset)
- **DESeq2** (Baseline differential expression method in the TDR benchmark on simulated PBMC data)
- **MAST** (Baseline differential expression method in the TDR benchmark on simulated PBMC data)
- **ggplot2** (Plots mean expression trajectories of tDEGs over time split by responders vs non-responders)

## Examples

```
require(keras); require(CASi); tdegs <- FindTemporalDEGs(CASiList_tDEG_demo, metadata_tDEG_demo, p.cutoff = 0.05, pct.cutoff = 0.3)
```

## Evaluation signals

- Reproduction of the MCL counts: at p = 0.05 among 1469 total genes, CASi identifies 268 tDEGs vs Seurat's 75 DEGs with 21 overlapping
- Benchmark TDR curves show CASi's true discovery rate at or above DESeq2/MAST/Seurat Wilcoxon across thresholds, increasing with stronger time effect while competitors' TDR decreases
- Gene count in the CASiList equals total cells (t0 + t1); Group is a binary factor and Time is numeric or categorical
- All reported tDEGs satisfy p < 0.05 and pct.cutoff = 0.30 expression-frequency filtering
- Enrichment tables recover expected biology, e.g., MYC and oxidative phosphorylation among Hallmark/Reactome terms

## Limitations

- The current GLM framework only detects linear effects — linear change of expression by time or time-by-condition interaction — not nonlinear changes
- Power depends on the expression-frequency filter: pct.cutoff = 0.3 excludes genes expressed in fewer than 30% of cells
- Competitor method versions (Seurat, BPSC, MAST, Monocle) are not stated, complicating exact benchmark reproduction
- Benchmark ARI/comparison details for some scenarios appear only in supplementary Figure 11

## Evidence

- [results] Run CASi's FindTemporalDEGs with p.cutoff = 0.05 and pct.cutoff = 0.3, fitting a generalized linear model with iterative feature selection for time effects and time-by-group interaction effects.: [evidence span withheld — anchor status: not_found]
- [results] At p = 0.05, among a total of 1469 genes, CASi identifies 268 tDEGs and Seurat identifies 75 DEGs, with 21 genes overlapping between the two methods' results.: [evidence span withheld — anchor status: not_found]
- [results] Across weak, medium, and strong time-effect settings, CASi achieved the highest true discovery rate (TDR) at all thresholds compared with existing methods: [evidence span withheld — anchor status: not_found]
- [discussion] the current GLM framework can only detect linear effect, i.e., the linear change of gene expression by time or the interaction of time and condition: "the current GLM framework can only detect linear effect, i.e., the linear change of gene expression by time or the interaction of time and condition"
- [abstract] identification of temporal differentially expressed genes (tDEGs): "identification of temporal differentially expressed genes (tDEGs)"
