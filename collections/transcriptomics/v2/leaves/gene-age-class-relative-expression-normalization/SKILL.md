---
name: gene-age-class-relative-expression-normalization
description: Use when a TEI-annotated AnnData (h5ad) with gene age class annotations
  is available and you need a table of relative expression shares per gene age class
  per observation group (cell type, timepoint).
when_to_use_negative:
- The AnnData object lacks gene age class annotations (no orthomap matched to genes
  yet) — first run the orthomap extraction/matching steps.
- You need gene age-weighted summary indices such as TEI or partial TEI per cell rather
  than per-group relative expression per age class.
- The goal is extracting the orthomap itself (of2orthomap, eggNOG, PLAZA) rather than
  computing expression per age class.
edam_operation: http://edamontology.org/operation_2945
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_3308
tools:
- name: oggmap
  role: Provides the get_rematrix function that extracts relative expression per gene
    age class grouped by any annotated observation, from raw counts or implemented
    transformation options
  repo: https://github.com/kullrich/oggmap
- name: Scanpy
  role: Loads the TEI-annotated AnnData scRNA object whose expression matrix and obs
    annotations feed get_rematrix
- name: Python
  role: Runtime environment for oggmap (developed and supported with Python 3.10)
    and for the normalization/CSV export logic
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1093/bioinformatics/btad657
    title: 'oggmap: a Python package to extract gene ages per orthogroup and link
      them with single-cell RNA data'
  source_content_hash: sha256:cc7b4491c624a846af89fa3e7f68ad2d4169bc47d007e027269f87b540387aa3
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/kullrich/oggmap
  tools:
  - Python
  - Scanpy
  - oggmap
  edam_operation: http://edamontology.org/operation_2945
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 9
---

# gene-age-class-relative-expression-normalization

## Summary

Computes relative expression per gene age class in scRNA data using oggmap's get_rematrix, grouping mean expression by an annotated observation (e.g. cell type or sampling timepoint) and normalizing each group's age-class values to shares summing to 1.

## When to use

Use when a TEI-annotated AnnData (h5ad) with gene age class annotations is available and you need a table of relative expression shares per gene age class per observation group (cell type, timepoint).

## When NOT to use

- The AnnData object lacks gene age class annotations (no orthomap matched to genes yet) — first run the orthomap extraction/matching steps.
- You need gene age-weighted summary indices such as TEI or partial TEI per cell rather than per-group relative expression per age class.
- The goal is extracting the orthomap itself (of2orthomap, eggNOG, PLAZA) rather than computing expression per age class.

## Inputs

- TEI-annotated zebrafish AnnData (Scanpy .h5ad) object containing gene age class annotations matched via oggmap
- Expression matrix layer: raw counts (default) or a normalized/log-transformed layer
- Annotated observation column in adata.obs (e.g. cell type or sampling timepoint) used as grouping variable

## Outputs

- relative_expression_per_age_class.csv: rows = observation groups, columns = gene age classes, values = relative shares in (0,1) summing to 1 per group

## How to apply

Load the TEI-annotated AnnData object (produced by the earlier gene-to-orthomap matching step) with Scanpy/AnnData. Select the expression matrix source: raw counts by default, or one of get_rematrix's implemented transformation options (e.g. a normalized or log-transformed layer). Run oggmap's get_rematrix, passing the annotated observation column (e.g. cell type or sampling timepoint) as the grouping so that mean expression is aggregated per gene age class. Finally normalize each observation group's age-class expression values to relative shares so that values within each group sum to 1, and write the resulting table to relative_expression_per_age_class.csv. This implements the article's stated capability to 'calculate mean/relative expression per gene age class'.

## Related tools

- **oggmap** (Provides the get_rematrix function that extracts relative expression per gene age class grouped by any annotated observation, from raw counts or implemented transformation options) — https://github.com/kullrich/oggmap
- **Scanpy** (Loads the TEI-annotated AnnData scRNA object whose expression matrix and obs annotations feed get_rematrix)
- **Python** (Runtime environment for oggmap (developed and supported with Python 3.10) and for the normalization/CSV export logic)

## Examples

```
import scanpy as sc; from oggmap.datasets import datasets; adata = sc.read_h5ad('zebrafish_tei_annotated.h5ad'); rematrix = datasets.get_rematrix(adata=adata, obs_type='cell_type'); rematrix.to_csv('relative_expression_per_age_class.csv')
```

## Evaluation signals

- relative_expression_per_age_class.csv exists and contains one column per gene age class
- Within each observation group (row), the relative expression shares sum to 1 (floating-point tolerance)
- Every relative share value lies strictly in (0,1)
- Rows correspond exactly to the levels of the chosen annotated observation column (e.g. cell types or sampling timepoints) in adata.obs
- Mean expression per age class recomputed independently from the same layer matches get_rematrix output before normalization

## Limitations

- Requires gene and transcript IDs to already be matched with their gene age class via oggmap (orthomap/TEI annotation step must precede this skill).
- oggmap v0.0.2 was developed with Python 3.10; Python 2.7x and <=3.10 (as stated) are not supported, so older environments may fail.
- Choice of expression matrix source (raw counts vs. transformation option) affects the computed means and must be consistent with the downstream interpretation.

## Evidence

- [methods] get_rematrix function extracts the relative expression per gene age class, grouped by any annotated observation such as cell-type or sampling timepoint: [evidence span withheld — anchor status: not_found]
- [methods] calculate mean/relative expression per gene age class: "- calculate mean/relative expression per gene age class."
- [full_text] start from raw counts or use the implemented expression transformation options: "starting either from raw counts or using the implemented expression transformation options"
- [methods] match gene and transcript IDs and their corresponding gene age class with scRNA data sets: "match gene and transcript IDs and their corresponding gene age class with scRNA data sets."
