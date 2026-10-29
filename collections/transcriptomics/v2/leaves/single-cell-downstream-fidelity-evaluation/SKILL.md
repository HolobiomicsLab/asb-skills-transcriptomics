---
name: single-cell-downstream-fidelity-evaluation
description: Use when a computationally inferred or semi-profiled single-cell cohort
  must be validated against a real-profiled cohort by comparing downstream outputs
  such as marker genes, GO enrichment, CellChat interaction matrices, and Monocle3
  pseudotime.
when_to_use_negative:
- The cohort being evaluated is real-profiled data with no AI-generated or semi-profiled
  counterpart, so there is no fidelity comparison to make
- Only bulk-level deconvolution accuracy (cell-type proportions) is needed — this
  skill evaluates true single-cell-resolution downstream analyses, not cell-type-fraction
  correlation
- The inferred cohort lacks a matched real-profiled ground truth on shared cells,
  making RRHO, CellChat, and pseudotime correlations undefined
edam_operation: http://edamontology.org/operation_0282
edam_topics:
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_0091
tools:
- name: scSemiProfiler
  role: Generates the semi-profiled cohort whose downstream fidelity is being evaluated
    against the real-profiled cohort
  repo: https://github.com/mcgilldinglab/scSemiProfiler
- name: SCANPY
  role: Identifies top 50 positive/negative markers and top 100 cell-type signature
    genes and supplies UMAP coordinates for both cohorts
- name: GSEApy
  role: Runs GO/Reactome enrichment on the top-100 signature gene sets to compare
    enriched terms between cohorts
- name: CellChat
  role: R package for inferring cell-cell interactions per disease-severity/tissue/condition
    group on both datasets
- name: Monocle3
  role: Performs pseudotime analysis on shared UMAP coordinates with roots placed
    at identical UMAP positions
- name: SciPy
  role: Computes Pearson correlations between interaction matrices and 20-by-20 pseudotime
    grids
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1038/s41467-024-50150-1
    title: 'scSemiProfiler: Advancing large-scale single-cell studies through semi-profiling
      with deep generative models and active learning'
  source_content_hash: sha256:1ccaf31a76874d2c0ee0ae42f26df759c6f7ad75a97094547fcd5a8a24d75498
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
  - scSemiProfiler
  edam_operation: http://edamontology.org/operation_0282
  edam_topics:
  - http://edamontology.org/topic_3308
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9375
n_claims: 16
---

# single-cell-downstream-fidelity-evaluation

## Summary

A benchmarking workflow for verifying that semi-profiled (AI-generated) single-cell cohorts reproduce downstream single-cell analyses — marker/RRHO overlap, GO/Reactome enrichment, cell-cell interactions, and pseudotime — at the fidelity levels achieved by scSemiProfiler on the COVID-19 cohort.

## When to use

Use when a computationally inferred or semi-profiled single-cell cohort must be validated against a real-profiled cohort by comparing downstream outputs such as marker genes, GO enrichment, CellChat interaction matrices, and Monocle3 pseudotime.

## When NOT to use

- The cohort being evaluated is real-profiled data with no AI-generated or semi-profiled counterpart, so there is no fidelity comparison to make
- Only bulk-level deconvolution accuracy (cell-type proportions) is needed — this skill evaluates true single-cell-resolution downstream analyses, not cell-type-fraction correlation
- The inferred cohort lacks a matched real-profiled ground truth on shared cells, making RRHO, CellChat, and pseudotime correlations undefined

## Inputs

- Semi-profiled single-cell cohort (e.g., produced by scSemiProfiler from bulk data plus representative single-cell references)
- Real-profiled single-cell cohort serving as ground truth
- Cell type annotations for both cohorts

## Outputs

- RRHO overlap summaries of per-cell-type top-50 positive/negative marker lists
- GO/Reactome enrichment overlap statistics (top-100 signature genes, expected 95/100 overlap, p=9.10e-196)
- Cell-cell interaction strength matrices per disease-severity/tissue/condition group with Pearson correlation (expected 0.996)
- Pseudotime grids (20-by-20 UMAP-space averages) with Pearson correlation (expected 0.809)
- downstream_fidelity.csv aggregating all agreement statistics

## How to apply

Load the semi-profiled cohort and the real-profiled ground-truth cohort of single-cell data. With SCANPY, derive each cell type's top 50 positive and top 50 negative markers in both datasets and build RRHO plots from hypergeometric-test p-values between the ranked lists; separately identify the top 100 cell-type signature genes per dataset and run GO/Reactome enrichment with GSEApy, recording term overlap (target: 95/100 overlap with hypergeometric p = 9.10e-196). Run CellChat in R on both cohorts grouped by disease severity/tissue/condition and compute the Pearson correlation of inferred interaction strengths (target: 0.996). For pseudotime, run Monocle3 on shared UMAP coordinates with roots at identical UMAP positions, average pseudotime over a 20-by-20 grid of UMAP space, and compute the Pearson correlation between grids (target: 0.809). Assemble all statistics into downstream_fidelity.csv and check each against the reported Fig. 3 agreement values.

## Related tools

- **scSemiProfiler** (Generates the semi-profiled cohort whose downstream fidelity is being evaluated against the real-profiled cohort) — https://github.com/mcgilldinglab/scSemiProfiler
- **SCANPY** (Identifies top 50 positive/negative markers and top 100 cell-type signature genes and supplies UMAP coordinates for both cohorts)
- **GSEApy** (Runs GO/Reactome enrichment on the top-100 signature gene sets to compare enriched terms between cohorts)
- **CellChat** (R package for inferring cell-cell interactions per disease-severity/tissue/condition group on both datasets)
- **Monocle3** (Performs pseudotime analysis on shared UMAP coordinates with roots placed at identical UMAP positions)
- **SciPy** (Computes Pearson correlations between interaction matrices and 20-by-20 pseudotime grids)

## Examples

```
import scanpy as sc; sc.tl.rank_genes_groups(adata, groupby='celltype'); Rscript -e "library(CellChat); chat <- createCellChat(obj=seurat_obj); chat <- computeCommunProb(chat)"
```

## Evaluation signals

- Enrichment overlap of top-100 CD4 signature genes reaches 95/100 between semi-profiled and real-profiled cohorts with hypergeometric p = 9.10e-196, and the enrichment output contains the reported GO terms
- Cell-cell interaction matrix Pearson correlation between cohorts equals 0.996 (within tolerance)
- Pseudotime grid Pearson correlation (20-by-20 UMAP-space averages, identical roots) equals 0.809 (within tolerance)
- RRHO plots of top-50 positive/negative markers per cell type show consistent rank-rank overlap between the two cohorts
- downstream_fidelity.csv is produced and every statistic falls within tolerance of the reported Fig. 3 agreement values

## Limitations

- Validated agreement values (95/100, 0.996, 0.809) are specific to the COVID-19 cohort as reported in Fig. 3; other datasets may yield different fidelity levels
- Cell-cell interaction and pseudotime components require R packages (CellChat, Monocle3), so the workflow spans both Python (SCANPY, GSEApy) and R environments
- The article does not state software versions or dataset accession IDs for the evaluated cohorts, complicating exact reproduction

## Evidence

- [methods] Fig. 3 agreement values: [evidence span withheld — anchor status: not_found]
- [methods] pseudotime method: [evidence span withheld — anchor status: not_found]
- [results] quantitative agreement: [evidence span withheld — anchor status: not_found]
- [intro] downstream goal: "True single-cell resolution allows for in-depth analyses essential for understanding cellular heterogeneity, such as UMAP27, pathway activation pattern analysis, biomarker discovery"
- [discussion] cohort resemblance: "The striking resemblance between semi-profiled and real-profiled data in facets like UMAP visualization and cell type proportions ascertains the robustness of scSemiProfiler."
- [readme] README fidelity claim: [evidence span withheld — anchor status: external_doc]
