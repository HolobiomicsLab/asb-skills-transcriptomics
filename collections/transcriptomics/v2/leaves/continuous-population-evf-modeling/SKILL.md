---
name: continuous-population-evf-modeling
description: Use when you need simulated scRNA-seq data with a known continuous trajectory
  (true pseudotime from the root and branch labels) to evaluate trajectory inference
  methods under varied sigma (within-population variability) and alpha (capture efficiency).
when_to_use_negative:
- When the task is clustering or differential expression benchmarking on discrete
  populations — use tree-based discrete simulations with getDEgenes instead of evf_type='continuous'
- When input data are already real observed scRNA-seq with no simulation needed (SymSim's
  BestMatchParams/parameter-fitting workflow, not this trajectory benchmark)
- When the target is experimental design for rare-population detection (cell-number
  requirement analysis), which is a separate SymSim use case
edam_operation: http://edamontology.org/operation_0284
edam_topics:
- http://edamontology.org/topic_0102
tools:
- name: SymSim
  role: Generates true transcript counts with continuous EVFs along the Phyla5() tree
    and converts them to observed non-UMI counts (SimulateTrueCounts, True2ObservedCounts,
    getTrajectoryGenes)
  repo: https://github.com/YosefLab/SymSim
- name: Monocle
  role: Trajectory inference method scored in the benchmark (version 2.6.4) on each
    observed-count dataset
- name: Slingshot
  role: Trajectory inference method scored in the benchmark (version 0.99.12); clearly
    outperformed Monocle and the MST method
- name: dynmethods
  role: Provides the wrapped minimum-spanning-tree trajectory inference implementation
    (dynwrap 0.1.0) used as the third competing method
  repo: https://github.com/dynverse/dynmethods
- name: dynwrap
  role: Wraps trajectory inference inputs/outputs into a common trajectory model so
    predictions can be compared against the SymSim gold standard
  repo: https://github.com/dynverse/dynwrap
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1038/s41467-019-10500-w
    title: Simulating multiple faceted variability in single cell RNA sequencing
  source_content_hash: sha256:4f3d4af4df3e8eac47156136627f1e1632923a0520a7dc1450a625f9cfc117b0
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/YosefLab/SymSim
  tools:
  - Monocle
  - Slingshot
  - SymSim
  - dynmethods
  - dynwrap
  edam_operation: http://edamontology.org/operation_0284
  edam_topics:
  - http://edamontology.org/topic_0102
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6428571428571429
n_claims: 14
---

# continuous-population-evf-modeling

## Summary

Generates gold-standard continuous single-cell populations with SymSim's extrinsic-variation-factor (EVF) model along a simulated tree (evf_type='continuous', Phyla5()), and uses the true pseudotime and branch labels to benchmark trajectory inference methods such as Monocle, Slingshot, and the dynverse MST.

## When to use

Use when you need simulated scRNA-seq data with a known continuous trajectory (true pseudotime from the root and branch labels) to evaluate trajectory inference methods under varied sigma (within-population variability) and alpha (capture efficiency).

## When NOT to use

- When the task is clustering or differential expression benchmarking on discrete populations — use tree-based discrete simulations with getDEgenes instead of evf_type='continuous'
- When input data are already real observed scRNA-seq with no simulation needed (SymSim's BestMatchParams/parameter-fitting workflow, not this trajectory benchmark)
- When the target is experimental design for rare-population detection (cell-number requirement analysis), which is a separate SymSim use case

## Inputs

- Five-leaf reference tree from SymSim Phyla5()
- Simulation grid over within-population variability (Sigma) and capture efficiency (alpha_mean)
- Human transcript-length pool (gene_len_pool) for assigning gene lengths
- Observed non-UMI count matrices (500 cells x 500 genes) from True2ObservedCounts

## Outputs

- True transcript count matrices with continuous EVFs per cell
- Gold-standard cell pseudotime (distance from root) and branch labels via getTrajectoryGenes
- Per-method inferred pseudotime and trajectory models from Monocle, Slingshot, and the dynverse MST
- cor_pseudotime (Spearman correlation) and cal_KNNP (KNN purity) scores per (Sigma, alpha) setting

## How to apply

Build the five-leaf reference tree with SymSim's Phyla5(), then call SimulateTrueCounts with evf_type='continuous', ncells_total=500, ngenes=500, nevf=20, n_de_evf=12, vary='s', and a fixed randseed for reproducibility. Convert true counts to non-UMI observed counts with True2ObservedCounts, setting alpha_mean to the capture-efficiency grid value and sampling gene lengths from the human transcript-length pool gene_len_pool; vary Sigma (intrinsic noise) and alpha to sweep the simulation grid. Extract gold-standard pseudotime (distance from root) and branch labels from the simulated metadata with getTrajectoryGenes. Run Monocle 2.6.4, Slingshot 0.99.12, and the dynmethods (dynwrap 0.1.0) minimum-spanning-tree implementation on each observed-count matrix, then score each with cor_pseudotime (Spearman correlation between inferred and true pseudotime) and cal_KNNP (k-nearest-neighbor purity). Rationale: SymSim explicitly models intrinsic transcriptional noise, extrinsic cell-state variation, and technical variation, so performance trends in sigma/alpha have a mechanistic interpretation.

## Related tools

- **SymSim** (Generates true transcript counts with continuous EVFs along the Phyla5() tree and converts them to observed non-UMI counts (SimulateTrueCounts, True2ObservedCounts, getTrajectoryGenes)) — https://github.com/YosefLab/SymSim
- **Monocle** (Trajectory inference method scored in the benchmark (version 2.6.4) on each observed-count dataset)
- **Slingshot** (Trajectory inference method scored in the benchmark (version 0.99.12); clearly outperformed Monocle and the MST method)
- **dynmethods** (Provides the wrapped minimum-spanning-tree trajectory inference implementation (dynwrap 0.1.0) used as the third competing method) — https://github.com/dynverse/dynmethods
- **dynwrap** (Wraps trajectory inference inputs/outputs into a common trajectory model so predictions can be compared against the SymSim gold standard) — https://github.com/dynverse/dynwrap

## Examples

```
phyla <- Phyla5(); true_counts_res <- SimulateTrueCounts(ncells_total=500, ngenes=500, nevf=20, n_de_evf=12, vary='s', evf_type='continuous', phyla=phyla, randseed=42)
```

## Evaluation signals

- Validation run accepted when Spearman correlation (cor_pseudotime) and KNN purity (cal_KNNP) across the Sigma/alpha grid reproduce the trends and magnitudes of Fig. 7a-b
- cor_pseudotime and cal_KNNP must decrease as Sigma increases (at alpha=0.1) and increase with alpha (at sigma=0.6) for all three methods
- Slingshot should clearly outperform Monocle and the dynverse MST on both metrics
- Fixed randseed in SimulateTrueCounts yields bit-identical gold-standard pseudotime and branch labels across reruns
- Each dataset is 500 cells x 500 genes with 20 EVFs and 12 DE EVFs, as specified in the simulation call

## Limitations

- The simulation pipeline omits several steps, including reverse transcription and library cleaning up
- As scVI is only applicable to large datasets, some downstream processing choices depend on protocol (UMI vs non-UMI)
- Performance trends depend on the specific Phyla5() tree topology and the 500-cell/500-gene setting; extrapolation to other topologies or depths is not established by this workflow

## Evidence

- [methods] SimulateTrueCounts and True2ObservedCounts as the two major dataset-generation functions: "**SimulateTrueCounts( )** and **True2ObservedCounts( )** are the two major functions to generate datasets."
- [methods] getTrajectoryGenes returns the simulated trajectory gold standard: "Function **getTrajectoryGenes( )** returns the simulated trajectory information of cells."
- [discussion] dynverse packages used for trajectory inference comparison: "We use the R packages dynwrap (https://github.com/dynverse/dynwrap, version 0.1.0)"
- [readme] SymSim as an R package for single cell RNA-Seq simulation: "SymSim (Synthetic model of multiple variability factors for Simulation) is an R package for simulation of single cell RNA-Seq data."
- [methods] Gene lengths sampled from human transcript lengths: "Each genes needs to be assigned a gene length. We sample lengths from human transcript lengths."
