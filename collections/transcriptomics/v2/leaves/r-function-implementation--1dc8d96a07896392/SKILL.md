---
name: r-function-implementation
description: Use when a task requires re-implementing an R package's internal statistic
  as a standalone function, and correctness can be verified by correlating the function's
  outputs with the package's outputs on the same inputs.
when_to_use_negative:
- The goal is biological interpretation of pathway perturbations rather than validating/reproducing
  the statistical method — call compare_pathways directly instead.
- Input is not single-cell expression data with condition metadata (SCPA requires
  per-cell matrices from two or more conditions).
edam_operation: http://edamontology.org/operation_3791
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_0203
tools:
- name: SCPA
  role: Reference package whose compare_pathways output provides ground-truth Qvals
    for validating the standalone implementation; also supplies seurat_extract for
    matrix extraction
  repo: https://github.com/jackbibby1/SCPA
- name: Seurat
  role: Holds the naive CD4+ T cell object that seurat_extract subsets by metadata
    into condition-specific expression matrices
- name: tidyverse
  role: Used for data loading, joining package and standalone result tables by Pathway,
    and writing qval_comparison.csv
- name: dplyr
  role: Data manipulation when joining and comparing per-pathway Qval results
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1016/j.celrep.2022.111697
    title: Systematic single-cell pathway analysis to characterize early T cell activation
  source_content_hash: sha256:63d6bc4398c6877ae372804903b54d3859f84e5068e595edcfea6c7c5fc51567
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/jackbibby1/SCPA
  tools:
  - R
  - SCPA
  - Seurat
  - dplyr
  - tidyverse
  edam_operation: http://edamontology.org/operation_3791
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.4666666666666667
n_claims: 15
---

# r-function-implementation

## Summary

Implement a standalone R function that reproduces a package's core statistical method — here, SCPA's graph-based nonparametric multisample distribution test — and validate it by comparing its outputs (Qvals) against the original package on identical inputs.

## When to use

Use when a task requires re-implementing an R package's internal statistic as a standalone function, and correctness can be verified by correlating the function's outputs with the package's outputs on the same inputs.

## When NOT to use

- The goal is biological interpretation of pathway perturbations rather than validating/reproducing the statistical method — call compare_pathways directly instead.
- Input is not single-cell expression data with condition metadata (SCPA requires per-cell matrices from two or more conditions).

## Inputs

- Seurat object of naive CD4+ T cells (naive_cd4.rds) with metadata columns 'Cell_Type' and 'Hour'
- Pathway gene set file (combined_metabolic_pathways.csv, Hallmark/KEGG/Reactome) loadable via get_paths
- Two normalized total count expression matrices from seurat_extract (Hour 0 resting and Hour 24 activated cells)

## Outputs

- Standalone R function computing pval, adjusted pval, qval, and two-sample fold-change enrichment per pathway
- compare_pathways result table for the same samples and pathways
- qval_comparison.csv joining package and standalone Qvals per Pathway

## How to apply

Load the naive CD4+ T cell Seurat object (naive_cd4.rds) and curated pathway gene sets (e.g. combined_metabolic_pathways.csv from Hallmark/KEGG/Reactome) with SCPA, Seurat, and tidyverse. Extract condition-specific expression matrices with seurat_extract (meta1 = 'Cell_Type', meta2 = 'Hour': resting Hour 0 vs activated Hour 24). Write a standalone function that builds pathway-specific expression matrices, downsamples to the package default of 500 cells per population, and computes a statistic whose pval/adjusted pval/qval reflect divergence in the pathway's multivariate distribution, adding a fold-change enrichment score only for the two-sample case. Run both the standalone function and compare_pathways(samples = list(resting, activated), pathways = pathways) on the same inputs, join results by Pathway, and write per-pathway package-vs-standalone Qvals to qval_comparison.csv. The rationale is that SCPA defines pathway activity as a change in the multivariate distribution of a pathway's genes, so the reimplementation must mirror that distributional (not enrichment-based) framing, with qval as the primary statistic.

## Related tools

- **SCPA** (Reference package whose compare_pathways output provides ground-truth Qvals for validating the standalone implementation; also supplies seurat_extract for matrix extraction) — https://github.com/jackbibby1/SCPA
- **Seurat** (Holds the naive CD4+ T cell object that seurat_extract subsets by metadata into condition-specific expression matrices)
- **tidyverse** (Used for data loading, joining package and standalone result tables by Pathway, and writing qval_comparison.csv)
- **dplyr** (Data manipulation when joining and comparing per-pathway Qval results)

## Examples

```
resting <- seurat_extract(seu, meta1 = 'Cell_Type', value_meta1 = 'CD4 T', meta2 = 'Hour', value_meta2 = '0'); activated <- seurat_extract(seu, meta1 = 'Cell_Type', value_meta1 = 'CD4 T', meta2 = 'Hour', value_meta2 = '24'); scpa_out <- compare_pathways(samples = list(resting, activated), pathways = pathways)
```

## Evaluation signals

- The standalone function runs without error on the two-sample resting (Hour 0) vs activated (Hour 24) input.
- Qvals from the standalone function show strong agreement with compare_pathways output (high rank/linear correlation across tested pathways).
- Per-pathway package-vs-standalone Qvals are successfully joined by Pathway and written to qval_comparison.csv.
- Downsampling matches the package default of 500 cells per population.
- Fold-change enrichment is computed only in the two-sample case, mirroring compare_pathways behavior.

## Limitations

- SCPA accepts total normalized count matrices from two or more conditions; single-condition input is outside its design.
- qval should be the primary statistic; pathways can show large qvals without fold change/enrichment (e.g. arachidonic acid metabolism), so fold change alone is misleading.
- Installation requires pinned dependency versions (crossmatch 1.3.1, multicross 2.1.0); installation errors usually mean missing dependencies must be installed manually.

## Evidence

- [readme] Installation requires pinned dependency versions.: [evidence span withheld — anchor status: external_doc]
