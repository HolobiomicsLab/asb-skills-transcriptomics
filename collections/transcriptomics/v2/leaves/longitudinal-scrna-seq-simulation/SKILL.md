---
name: longitudinal-scrna-seq-simulation
description: Use when a method for multi-timepoint scRNA-seq (cross-timepoint annotation,
  novel-cell detection, or tDEG calling) needs a ground-truth benchmark with controlled
  temporal effects, since no real dataset has fully known labels/DEG truth.
when_to_use_negative:
- The real dataset already carries validated ground truth (e.g., MCL pre/post labels,
  accuracy, and ARI) — benchmark directly on it instead of simulating
- The question is about nonlinear time effects or time×condition interactions — the
  GLM-based tDEG framework underpinning this benchmark can only detect linear effects,
  so a linear-effect simulation would not exercise the failure mode
- The goal is a one-off descriptive analysis of an existing longitudinal dataset rather
  than method comparison against planted ground truth
edam_topics:
- http://edamontology.org/topic_0203
tools:
- name: CASi
  role: Package containing FindTemporalDEGs (GLM + iterative feature selection with
    pct.cutoff=0.3, p.cutoff=0.05) and the pbmc_demo / CASiList_tDEG_demo objects
    used to exercise the simulation benchmark
  repo: github.com/yizhuo-wang/CASi
- name: DESeq2
  role: Baseline differential-expression method run on the same simulated counts for
    TDR comparison
- name: MAST
  role: Baseline differential-expression method run on the same simulated counts for
    TDR comparison
- name: Seurat
  role: Provides Wilcoxon-rank-sum FindMarkers as a baseline and UMAP-based visualization
    of simulated/annotated populations
- name: keras
  role: Backend for CASi's CrossTimeAnnotation neural-network classifier evaluated
    in the novel-cell and annotation simulation scenarios
provenance:
  source_task_ids:
  - task_003
  - task_002
  - task_004
  source_papers:
  - doi: 10.1038/s41598-024-58566-x
    title: 'CASi: A framework for cross-timepoint analysis of single-cell RNA sequencing
      data'
  source_content_hash: sha256:f0375422bf9fc57848d466d4187cfbcbfdb2399b2ef3111478ed357e47bb2ca7
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - CAMLU
  - CASi
  - CHETAH
  - DESeq2
  - MAST
  - Seurat
  - ggplot2
  - keras
  - scmap
  - singleR
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5294117647058824
n_claims: 17
---

# longitudinal-scrna-seq-simulation

## Summary

Simulates multi-timepoint single-cell RNA-seq data by sampling real reference cells (e.g., public PBMC monocytes) and planting controlled temporal effects (novel cell types, cell-type-specific tDEGs at graded strengths) so that cross-timepoint methods like CASi can be benchmarked against DESeq2, MAST, Seurat Wilcoxon, CHETAH, scPred, and scmap with known ground truth.

## When to use

Use when a method for multi-timepoint scRNA-seq (cross-timepoint annotation, novel-cell detection, or tDEG calling) needs a ground-truth benchmark with controlled temporal effects, since no real dataset has fully known labels/DEG truth.

## When NOT to use

- The real dataset already carries validated ground truth (e.g., MCL pre/post labels, accuracy, and ARI) — benchmark directly on it instead of simulating
- The question is about nonlinear time effects or time×condition interactions — the GLM-based tDEG framework underpinning this benchmark can only detect linear effects, so a linear-effect simulation would not exercise the failure mode
- The goal is a one-off descriptive analysis of an existing longitudinal dataset rather than method comparison against planted ground truth

## Inputs

- Public reference scRNA-seq dataset for sampling (e.g., public PBMC dataset with >60,000 sorted cells from eight immune cell types)
- Simulation design parameters: number of cells and genes sampled (e.g., 900 monocytes, 2000 genes), number of planted tDEGs (e.g., 300), time-effect levels (weak/medium/strong)
- Novel-cell scenario definition (which cell type emerges at which timepoints, e.g., B cells at t1/t2)
- Number of repetitions (e.g., 200) and evaluation threshold grid

## Outputs

- Simulated multi-timepoint count matrices (t0, t1, ...) with known cell labels and planted ground truth
- Per-method true/false discovery counts at each p-value threshold, averaged over repetitions
- TDR curves versus threshold comparing CASi, DESeq2, MAST, and Seurat Wilcoxon (e.g., tdr_curves.png)
- UMAP plots colored by cell-type-specific correlation and labels showing recovered novel-cell clusters (novel_cells_umap.png)

## How to apply

Sample cells and genes from a real public reference (the paper used 900 monocytes and 2000 genes from a public PBMC dataset of more than 60,000 sorted cells from eight immune cell types) to retain realistic count distributions, then inject perturbations in silico: either a novel cell type appearing only at later timepoints (e.g., B cells emerging at t1/t2 in scenario 3) or a fixed set of planted cell-type-specific tDEGs (e.g., 300) at graded time-effect levels (weak/medium/strong). Repeat the simulation many times (the paper used 200 repetitions) and run the candidate method plus baselines on identical simulated counts — for tDEGs, run CASi's FindTemporalDEGs with pct.cutoff=0.3 and p.cutoff=0.05 alongside DESeq2, MAST, and Seurat's Wilcoxon FindMarkers. Score each method against the known planted truth across a grid of p-value thresholds, average over repetitions, and plot true discovery rate (TDR) curves for all methods on one figure. Rationale: simulation is the only way to compute TDR, accuracy, and ARI against exact ground truth for cross-timepoint tasks, and varying effect strength reveals sensitivity trends (CASi's TDR rose with stronger time effects while competitors' fell).

## Related tools

- **CASi** (Package containing FindTemporalDEGs (GLM + iterative feature selection with pct.cutoff=0.3, p.cutoff=0.05) and the pbmc_demo / CASiList_tDEG_demo objects used to exercise the simulation benchmark) — github.com/yizhuo-wang/CASi
- **DESeq2** (Baseline differential-expression method run on the same simulated counts for TDR comparison)
- **MAST** (Baseline differential-expression method run on the same simulated counts for TDR comparison)
- **Seurat** (Provides Wilcoxon-rank-sum FindMarkers as a baseline and UMAP-based visualization of simulated/annotated populations)
- **keras** (Backend for CASi's CrossTimeAnnotation neural-network classifier evaluated in the novel-cell and annotation simulation scenarios)

## Examples

```
require(keras); require(CASi); FindTemporalDEGs(CASiList_tDEG_demo, pct.cutoff = 0.3, p.cutoff = 0.05)
```

## Evaluation signals

- Reproduced TDR curves must show CASi's TDR comparable to or above DESeq2/MAST/Seurat Wilcoxon at all thresholds, with CASi's TDR increasing (and competitors' decreasing) as the planted time effect strengthens from weak to medium to strong
- In the novel-cell scenario, CASi's prediction must closely match ground truth and recover most novel cells, while CHETAH recovers only a small portion and scPred/scmap recover none
- Per-method true-discovery and false-discovery counts must be computable at every repetition by comparing calls against the exact 300 planted tDEGs
- Averaged results over 200 repetitions should stabilize before plotting TDR versus threshold on the single comparison figure (tdr_curves.png)
- Install/execution sanity: devtools::install_github('yizhuo-wang/CASi') works and lazy-loaded demo objects pbmc_demo and CASiList_tDEG_demo load without error

## Limitations

- The GLM tDEG framework detects only linear time effects and time×condition interactions, so simulations graded on linear effect strength cannot probe nonlinear temporal changes
- Novel-cell detection may fail when the planted novel cell type is transcriptomically similar to known types — simulations should stratify similarity, but the paper notes this failure mode explicitly
- If the simulated t0 data differ dramatically from later timepoints, the neural-network classifier fails to borrow useful information and cannot be well trained
- ARI comparison results for the three simulation scenarios are only available in supplementary Figure 11, not the main text, limiting traceability of full benchmark numbers
- Version numbers of comparison tools (Seurat, BPSC, MAST, Monocle) are not stated, complicating exact reproduction

## Evidence

- [abstract] Simulation studies and real multi-timepoint single cell dataset applications showed robust and favorable performance versus existing methods with similar purposes: "we demonstrate the robust and favorable performance of the proposal versus existing methods serving similar purposes"
- [methods] PBMC reference dataset used as simulation source: "We obtain a publicly available dataset of peripheral blood mononuclear cells (PBMC)[1], containing more than 60,000 sorted cells from eight immune cell types."
- [methods] Demo objects shipped with the package support the tDEG simulation benchmark: "The demo pbmc data, "pbmc_demo", has been lazy-loaded with the package."
- [discussion] GLM scope limits what time effects can be planted and detected: "the current GLM framework can only detect linear effect, i.e., the linear change of gene expression by time or the interaction of time and condition"
- [discussion] Novel-cell detection failure mode relevant to simulation design: "the novel cell detection framework may not work well when the novel cell is transcriptomically similar to the known cell types"
- [readme] Setup requirement for running CASi-based simulation benchmarks: [evidence span withheld — anchor status: external_doc]
- [intro] Motivation for simulation: no existing tools for multi-timepoint scRNA-seq analysis: "At present, there are no available computational tools to comprehensively analyze multiple timepoints’ scRNA-seq data."
