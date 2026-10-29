---
name: hematopoietic-progenitor-cell-type-ranking
description: Use when per-cell pluripotency/stemness scores must be compared across
  hematopoietic progenitor types (HSC, MPP, GMP, etc.) to check whether a method orders
  cells according to the classical HSC-first hierarchy.
when_to_use_negative:
- The goal is trajectory inference from scratch rather than quantifying pluripotency
  per cell type — ORIGINS scores are intended to identify stem/progenitor cells as
  trajectory origins, not to build the trajectory itself.
- Expression data are not a normalized genes-by-cells matrix; ORIGINS expects a matrix
  or data frame with genes in rows and cells in columns, and PyOrigins normalizes
  by edge count and per-cell mean expression before comparing PPINs or samples.
- The biological process of interest is not cell differentiation — the differentiation
  PPIN in ORIGINS is specific; use ORIGINS2, which provides PPINs for 18 biological
  processes and a build_ppin function for custom gene lists.
edam_operation: http://edamontology.org/operation_0337
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_0602
tools:
- name: ORIGINS
  role: R package used to compute per-cell activity of the differentiation-associated
    PPIN from the expression matrix via activity(expression_matrix, differentiation_edges)
  repo: https://github.com/danielasenraoka/ORIGINS
- name: LandSCENT
  role: Benchmark pluripotency scoring method computed on the same expression matrix
    so its per-cell-type ranking is directly comparable to ORIGINS
- name: CytoTRACE
  role: Benchmark pluripotency scoring method computed on the same expression matrix
    for cross-method ranking comparison
- name: remotes
  role: R package used to install ORIGINS from GitHub before running the activity
    computation
  repo: https://github.com/danielasenraoka/ORIGINS
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1016/j.mex.2022.101778
    title: 'ORIGINS: A protein network-based approach to quantify cell pluripotency
      from scRNA-seq data'
  source_content_hash: sha256:311ad2976ae3a7fed9ea3186c3717c56fab9b66591590f941bcf31e4387ac9d9
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/danielasenraoka/ORIGINS
  tools:
  - CytoTRACE
  - LandSCENT
  - ORIGINS
  - R
  - remotes
  edam_operation: http://edamontology.org/operation_0337
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_0602
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.75
n_claims: 8
---

# hematopoietic-progenitor-cell-type-ranking

## Summary

Benchmark pluripotency scoring methods (ORIGINS, LandSCENT, CytoTRACE) on the hematopoietic bone marrow scRNA-seq dataset GSE117498 by aggregating per-cell scores over progenitor cell types (HSC, MPP, GMP, etc.) and ranking cell types by average potency to test recovery of the classical hematopoietic hierarchy.

## When to use

Use when per-cell pluripotency/stemness scores must be compared across hematopoietic progenitor types (HSC, MPP, GMP, etc.) to check whether a method orders cells according to the classical HSC-first hierarchy.

## When NOT to use

- The goal is trajectory inference from scratch rather than quantifying pluripotency per cell type — ORIGINS scores are intended to identify stem/progenitor cells as trajectory origins, not to build the trajectory itself.
- Expression data are not a normalized genes-by-cells matrix; ORIGINS expects a matrix or data frame with genes in rows and cells in columns, and PyOrigins normalizes by edge count and per-cell mean expression before comparing PPINs or samples.
- The biological process of interest is not cell differentiation — the differentiation PPIN in ORIGINS is specific; use ORIGINS2, which provides PPINs for 18 biological processes and a build_ppin function for custom gene lists.

## Inputs

- scRNA-seq count matrix for hematopoietic bone marrow cells (GSE117498), genes in rows, cells in columns
- Cell-type annotations per cell (HSC, MPP, GMP, etc.) for aggregating scores by progenitor type
- differentiation_edges data frame (differentiation PPIN shipped inside the ORIGINS package)

## Outputs

- Per-cell potency/differentiation-activity scores from ORIGINS, LandSCENT, and CytoTRACE on the same expression matrix
- Score table and per-cell-type ranking verdict written to hematopoietic_stemness_scores.csv

## How to apply

Download the hematopoietic bone marrow scRNA-seq count matrix (GSE117498) from GEO and format it as a genes-by-cells matrix. Compute ORIGINS differentiation activity per cell with activity(expression_matrix, differentiation_edges), where the differentiation PPIN edges ship inside the ORIGINS package. Compute LandSCENT and CytoTRACE potency scores on the same expression matrix so all methods are directly comparable. Aggregate the scores by annotated progenitor cell type (HSC, MPP, GMP, etc.), compute the average potency per type, and rank cell types under each method. The agreement criterion from the source task is that GMP must show the highest average potency under all methods, and no method's ordering should follow the classical HSC-first hierarchy; write the score table and ranking verdict to hematopoietic_stemness_scores.csv.

## Related tools

- **ORIGINS** (R package used to compute per-cell activity of the differentiation-associated PPIN from the expression matrix via activity(expression_matrix, differentiation_edges)) — https://github.com/danielasenraoka/ORIGINS
- **LandSCENT** (Benchmark pluripotency scoring method computed on the same expression matrix so its per-cell-type ranking is directly comparable to ORIGINS)
- **CytoTRACE** (Benchmark pluripotency scoring method computed on the same expression matrix for cross-method ranking comparison)
- **remotes** (R package used to install ORIGINS from GitHub before running the activity computation) — https://github.com/danielasenraoka/ORIGINS

## Examples

```
remotes::install_github("danielasenraoka/ORIGINS"); library(ORIGINS); diff_activity <- activity(expression_matrix, differentiation_edges)
```

## Evaluation signals

- ORIGINS, LandSCENT, and CytoTRACE are computed on the identical expression matrix so per-cell-type rankings are directly comparable.
- Average potency is aggregated per annotated progenitor type (HSC, MPP, GMP, etc.) rather than left as raw per-cell scores.
- Ranking verdict check: all three methods agree on which cell type has the highest average potency (in the source task, GMP, not HSC).
- The score table and ranking verdict are written to hematopoetic_stemness_scores.csv (expected output file hematopoietic_stemness_scores.csv).
- The activity() call uses the differentiation_edges data frame loaded from within the ORIGINS package, not user-supplied edges.

## Limitations

- None of the benchmarked methods adequately ordered cells according to pluripotency under the classical hematopoietic model: all methods agreed GMP had the highest average potency rather than HSCs.
- ORIGINS is a differentiation PPIN-based score intended to identify stem and progenitor cells as trajectory-inference origins; it does not guarantee recovery of canonical developmental hierarchies.
- The complete example workflow in the article (count matrix to trajectory inference) is illustrated using the breast dataset, not the hematopoietic one.

## Evidence

- [abstract] ORIGINS quantifies pluripotency from scRNA-seq data using a differentiation-associated PPI network and the expression matrix to compute a differentiation activity score per cell: "This approach uses the protein-protein interaction (PPI) network associated with the differentiation process as a scaffold and the gene expression matrix to calculate a score that we call"
- [methods] Activity computation call from the ORIGINS methods/README.: "diff_activity <- activity(expression_matrix, differentiation_edges)"
- [readme] Expression matrix orientation and differentiation_edges provenance from the README.: "expression_matrix is a matrix or data frame, where each row is a gene and each column is a cell. differentiation_edges is a data frame containing the edges of the Protein-protein"
- [readme] ORIGINS purpose as stated in the README.: "This package was originally created to identify stem and progenitor cells to select those cells as an origin when performing trajectory inference."
- [abstract] Benchmarked methods and datasets from the abstract.: [evidence span withheld — anchor status: not_found]
