---
name: cell-cell-interaction-network-comparison
description: 'Use when validating AI-inferred single-cell data against a real-profiled
  ground-truth cohort: you have matched (semi-profiled vs real-profiled) single-cell
  datasets with shared cell-type annotations and need a quantitative fidelity metric
  for cell-cell interaction inference.'
when_to_use_negative:
- The datasets lack a real-profiled counterpart or shared cell-type annotations, since
  no paired network comparison is possible
- The goal is single-cell inference itself rather than fidelity evaluation — use the
  scSemiProfiler semi-profiling or single-cell level deconvolution pipelines instead
- The input is bulk RNA-seq without a single-cell reference; CellChat requires single-cell-resolution
  data
tools:
- name: CellChat
  role: R package used to infer cell-cell interaction/communication strengths per
    disease-severity/tissue/condition group in both the semi-profiled and real-profiled
    cohorts
- name: SciPy
  role: Computes the Pearson correlation between the inferred interaction-strength
    matrices of the two cohorts
- name: scSemiProfiler
  role: Generates the semi-profiled single-cell cohort whose interaction networks
    are compared against the real-profiled ground truth
  repo: https://github.com/mcgilldinglab/scSemiProfiler
- name: UMAP
  role: Provides the shared embedding context in which cell types, the basis of interaction
    inference, are compared between cohorts
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1038/s41467-024-50150-1
    title: 'scSemiProfiler: Advancing large-scale single-cell studies through semi-profiling
      with deep generative models and active learning'
  source_content_hash: sha256:eac3a877236576f799e6e292447f5b958776bb9ab3036f1a0bd62ffa430673a1
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/mcgilldinglab/scSemiProfiler
  tools:
  - CellChat
  - GSEApy
  - Monocle3
  - SCANPY
  - SciPy
  - UMAP
  - scSemiProfiler
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 11
---

# cell-cell-interaction-network-comparison

## Summary

Compares inferred cell-cell ligand-receptor interaction networks between a semi-profiled (scSemiProfiler-generated) single-cell cohort and its real-profiled counterpart by running CellChat per disease-severity/tissue/condition group on both datasets and computing the Pearson correlation between interaction-strength matrices.

## When to use

Use when validating AI-inferred single-cell data against a real-profiled ground-truth cohort: you have matched (semi-profiled vs real-profiled) single-cell datasets with shared cell-type annotations and need a quantitative fidelity metric for cell-cell interaction inference.

## When NOT to use

- The datasets lack a real-profiled counterpart or shared cell-type annotations, since no paired network comparison is possible
- The goal is single-cell inference itself rather than fidelity evaluation — use the scSemiProfiler semi-profiling or single-cell level deconvolution pipelines instead
- The input is bulk RNA-seq without a single-cell reference; CellChat requires single-cell-resolution data

## Inputs

- Semi-profiled single-cell cohort produced by the scSemiProfiler pipeline (task_001 output)
- Real-profiled (ground-truth) single-cell cohort with matched samples
- Shared cell-type annotations and disease-severity/tissue/condition group labels

## Outputs

- Per-group CellChat cell-cell interaction strength matrices for both cohorts
- Pearson correlation statistic between semi-profiled and real-profiled interaction matrices (target: 0.996)
- Row in downstream_fidelity.csv recording the cell-cell interaction correlation

## How to apply

Run the R package CellChat independently on the semi-profiled and real-profiled COVID-19 cohort datasets, stratifying the analysis per disease-severity/tissue/condition group so that interactions are inferred within comparable biological contexts. For each group, infer communication probabilities between cell-type pairs and extract the interaction-strength matrix. Compute the Pearson correlation (SciPy) between the corresponding interaction-strength matrices from the two cohorts; the article's benchmark is a correlation of 0.996. Assemble the correlation statistic into downstream_fidelity.csv alongside the RRHO, GO enrichment overlap, and pseudotime results, and validate it against the reported Fig. 3 agreement value within tolerance.

## Related tools

- **CellChat** (R package used to infer cell-cell interaction/communication strengths per disease-severity/tissue/condition group in both the semi-profiled and real-profiled cohorts)
- **SciPy** (Computes the Pearson correlation between the inferred interaction-strength matrices of the two cohorts)
- **scSemiProfiler** (Generates the semi-profiled single-cell cohort whose interaction networks are compared against the real-profiled ground truth) — https://github.com/mcgilldinglab/scSemiProfiler
- **UMAP** (Provides the shared embedding context in which cell types, the basis of interaction inference, are compared between cohorts)

## Examples

```
library(CellChat); cc <- createCellChat(object = seurat_obj, group.by = "celltype"); cc <- computeCommunProb(cc)
```

## Evaluation signals

- Pearson correlation between semi-profiled and real-profiled cell-cell interaction matrices is within tolerance of the reported 0.996
- Interaction matrices are computed over the same cell-type pairs and group stratifications (disease severity, tissue, condition) in both cohorts
- The correlation statistic appears as a row in downstream_fidelity.csv alongside RRHO, GO enrichment overlap (95/100, p=9.10e-196), and pseudotime correlation (0.809) that also match Fig. 3 values
- Semi-profiled cohort shows near-identical cell type proportions across disease conditions to the real-profiled ground truth, a prerequisite for comparable networks

## Limitations

- The comparison requires a real-profiled ground-truth cohort; the article notes scSemiProfiler's model requires a single-cell reference ('provided there is a single-cell reference')
- The 0.996 agreement was established on the COVID-19 cohort (124 samples) and other heterogeneous datasets; fidelity on new tissue types or modalities (e.g. proteomics) is unvalidated
- Interaction inference inherits errors from the semi-profiled inference; the article's fidelity evidence is aggregate, not per-interaction

## Evidence

- [intro] True single-cell resolution allows for in-depth analyses essential for understanding cellular heterogeneity, such as UMAP27, pathway activation pattern analysis, biomarker discovery: "True single-cell resolution allows for in-depth analyses essential for understanding cellular heterogeneity, such as UMAP27, pathway activation pattern analysis, biomarker discovery"
- [full_text] Run cell-cell interaction analysis on both datasets with the R package CellChat (per disease-severity/tissue/condition group): [evidence span withheld — anchor status: not_found]
- [readme] semi-profiling, which uses deep generative learning and active learning to generate a single-cell cohort with 1/10 to 1/3 sequencing cost: "semi-profiling, which uses deep generative learning and active learning to generate a single-cell cohort with 1/10 to 1/3 sequencing cost"
- [discussion] The striking resemblance between semi-profiled and real-profiled data in facets like UMAP visualization and cell type proportions ascertains the robustness of scSemiProfiler.: "The striking resemblance between semi-profiled and real-profiled data in facets like UMAP visualization and cell type proportions ascertains the robustness of scSemiProfiler."
