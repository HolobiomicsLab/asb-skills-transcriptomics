---
name: scrna-ann-data-expression-aggregation
description: Use when a Scanpy/AnnData scRNA-seq object already carries gene age class
  annotations (an orthomap matched via oggmap) and the goal is relative expression
  per age class aggregated by cell type or sampling timepoint.
when_to_use_negative:
- The AnnData object has not yet been annotated with gene age classes via an oggmap
  orthomap — gene ages must be matched first
- The goal is per-cell TEI or partial TEI profiles rather than per-group relative
  expression per age class (use oggmap's TEI/partial TEI functionality instead)
- The data are bulk RNA-seq or not stored as a Scanpy/AnnData scRNA-seq object
edam_operation: http://edamontology.org/operation_3800
edam_topics:
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_3170
tools:
- name: oggmap
  role: Provides get_rematrix, which extracts mean/relative expression per gene age
    class grouped by annotated observations, starting from raw counts or implemented
    expression transformation options
  repo: https://github.com/kullrich/oggmap
- name: Scanpy
  role: Toolkit used to load and handle the scRNA sequencing objects (AnnData) that
    get_rematrix operates on
- name: Python
  role: Runtime environment (Python 3.10 for oggmap v0.0.2) in which the aggregation
    workflow is executed
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1093/bioinformatics/btad657
    title: 'oggmap: a Python package to extract gene ages per orthogroup and link
      them with single-cell RNA data'
  source_content_hash: sha256:356a0f4f7ac2e10fd41abc88f63febed484f56fad0dbcf739c5df0a192a40a31
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/kullrich/oggmap
  tools:
  - Python
  - Scanpy
  - oggmap
  edam_operation: http://edamontology.org/operation_3800
  edam_topics:
  - http://edamontology.org/topic_0203
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6
n_claims: 5
---

# scrna-ann-data-expression-aggregation

## Summary

Compute mean and relative expression per gene age class from scRNA-seq AnnData objects using oggmap's get_rematrix, grouping by any annotated observation (e.g. cell type or sampling timepoint) so that age-class expression shares within each group sum to 1.

## When to use

Use when a Scanpy/AnnData scRNA-seq object already carries gene age class annotations (an orthomap matched via oggmap) and the goal is relative expression per age class aggregated by cell type or sampling timepoint.

## When NOT to use

- The AnnData object has not yet been annotated with gene age classes via an oggmap orthomap — gene ages must be matched first
- The goal is per-cell TEI or partial TEI profiles rather than per-group relative expression per age class (use oggmap's TEI/partial TEI functionality instead)
- The data are bulk RNA-seq or not stored as a Scanpy/AnnData scRNA-seq object

## Inputs

- TEI-annotated AnnData (h5ad) scRNA-seq object with gene age class annotations matched via oggmap
- Name of an annotated observation column in adata.obs (e.g. cell type or sampling timepoint) used as grouping variable
- Choice of expression matrix source: raw counts or an implemented transformation layer (e.g. normalized/log-transformed)

## Outputs

- relative_expression_per_age_class.csv with rows = observation groups and columns = gene age classes, relative shares summing to 1 per group
- Intermediate mean expression per gene age class table from get_rematrix

## How to apply

Load the TEI-annotated AnnData object (e.g. the zebrafish object with gene age classes matched via oggmap) with Scanpy. Choose the expression source: raw counts by default, or one of get_rematrix's implemented expression transformation options such as a normalized/log-transformed layer, keeping the choice consistent with the input options of the function. Run oggmap's get_rematrix, passing the annotated observation column (e.g. cell type or sampling timepoint) as the grouping variable so that mean expression is aggregated per gene age class. Then normalize each group's age-class values to relative shares so that within each observation group the values sum to 1. Write the resulting table (rows = observation groups, columns = gene age classes) to relative_expression_per_age_class.csv.

## Related tools

- **oggmap** (Provides get_rematrix, which extracts mean/relative expression per gene age class grouped by annotated observations, starting from raw counts or implemented expression transformation options) — https://github.com/kullrich/oggmap
- **Scanpy** (Toolkit used to load and handle the scRNA sequencing objects (AnnData) that get_rematrix operates on)
- **Python** (Runtime environment (Python 3.10 for oggmap v0.0.2) in which the aggregation workflow is executed)

## Examples

```
from oggmap import get_rematrix; relmat = get_rematrix(adata, gene_id_col='gene_id', obs_class='celltype'); relmat.to_csv('relative_expression_per_age_class.csv')
```

## Evaluation signals

- Output CSV relative_expression_per_age_class.csv exists and is non-empty
- The table contains one column per gene age class present in the orthomap annotations
- Relative expression shares within each observation group sum to 1
- Every relative share value falls within (0, 1)
- Rows of the table correspond to levels of the chosen annotated observation column (e.g. cell types or sampling timepoints)

## Limitations

- oggmap v0.0.2 was developed with Python 3.10 and does not support Python 2.7x or Python <=3.10
- The aggregation depends on prior orthomap gene-age assignment; gene/transcript IDs must match those in the scRNA data set
- Pre-annotated orthomaps are available for a fixed set of species (more than thousand eukaryotic species); other species require running OrthoFinder/eggNOG/PLAZA extraction first

## Evidence

- [other] oggmap's get_rematrix function extracts the relative expression per gene age class, grouped by any annotated observation such as cell-type or sampling timepoint, starting either from raw counts or using the implemented expression transformation options.: [evidence span withheld — anchor status: not_found]
- [methods] calculate mean/relative expression per gene age class.: "- calculate mean/relative expression per gene age class."
- [abstract] can be further used to calculate gene age-weighted expression data from scRNA sequencing objects using the Python Scanpy toolkit: "can be further used to calculate gene age-weighted expression data from scRNA sequencing objects using the Python Scanpy toolkit"
- [methods] oggmap v0.0.2 was developed using Python 3.10. We do not support Python 2.7x or Python <=3.10.: "oggmap v0.0.2 was developed using Python `3.10`. We do not support Python `2.7x` or Python `<=3.10`."
- [methods] match gene and transcript IDs and gene age class with scRNA data sets: "match gene and transcript IDs and their corresponding gene age class with scRNA data sets."
