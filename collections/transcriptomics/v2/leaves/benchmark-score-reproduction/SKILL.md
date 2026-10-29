---
name: benchmark-score-reproduction
description: Use when a published cell-cell communication score (e.g., ICELLNET score
  1527 for CM3->CT3b vs 1123 for CM3->CT0a) must be exactly reproduced from raw expression
  data to validate a pipeline or benchmark a method.
when_to_use_negative:
- Input is bulk RNA-seq or microarray data from sorted populations without single-cell
  cluster annotations and the goal is de novo discovery rather than benchmark reproduction
  of a published scRNA-seq score
- The target interaction database must include non-human interactions or downstream
  signaling effectors, which ICELLNET explicitly does not integrate at this stage
- Only approximate relative rankings are needed and exact numeric reproduction of
  published scores is not required
edam_operation: http://edamontology.org/operation_0282
edam_topics:
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_3170
tools:
- name: ICELLNET
  role: Computes transcriptome-based communication scores using a manually curated
    ligand-receptor database with multi-subunit complex rules
  repo: https://github.com/soumelis-lab/ICELLNET
- name: devtools
  role: Installs the ICELLNET package from GitHub (install_github with ref='master',
    subdir='icellnet')
  repo: https://github.com/soumelis-lab/ICELLNET
- name: data.table
  role: R dependency used for fast loading and manipulation of the expression matrix
    and score tables
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1038/s41467-021-21244-x
    title: Dissection of intercellular communication using the transcriptome-based
      framework ICELLNET
  source_content_hash: sha256:55347a2696217544f466d7ae8f427d72351b186bdced81e8a43c75dc974a68e2
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/soumelis-lab/ICELLNET
  tools:
  - ICELLNET
  - R
  - data.table
  - devtools
  edam_operation: http://edamontology.org/operation_0282
  edam_topics:
  - http://edamontology.org/topic_0203
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.46153846153846156
n_claims: 13
---

# benchmark-score-reproduction

## Summary

Reproduces published intercellular communication scores (e.g., ICELLNET global and per-interaction scores between dendritic cell cluster CM3 and T-cell clusters CT0a/CT3b in lupus nephritis scRNA-seq) to numerically validate an analysis pipeline against benchmark values.

## When to use

Use when a published cell-cell communication score (e.g., ICELLNET score 1527 for CM3->CT3b vs 1123 for CM3->CT0a) must be exactly reproduced from raw expression data to validate a pipeline or benchmark a method.

## When NOT to use

- Input is bulk RNA-seq or microarray data from sorted populations without single-cell cluster annotations and the goal is de novo discovery rather than benchmark reproduction of a published scRNA-seq score
- The target interaction database must include non-human interactions or downstream signaling effectors, which ICELLNET explicitly does not integrate at this stage
- Only approximate relative rankings are needed and exact numeric reproduction of published scores is not required

## Inputs

- scRNA-seq expression matrix (normalized counts) for the lupus nephritis dataset
- cell-to-cluster annotation table mapping cells to clusters (e.g., CM3, CT0a, CT3b)
- ICELLNET curated human ligand-receptor interactions database (>1600 interactions)

## Outputs

- icellnet_scores.csv with global communication scores per cell pair (CM3->CT3b, CM3->CT0a)
- per-interaction scores from the score matrix (e.g., CD86/CD28 92 vs 40, CCL22/CCR4 100 vs 39)
- gene x cluster average-expression profile matrix

## How to apply

Load the target scRNA-seq expression matrix and cell-to-cluster annotations in R, then subset cells for the specified clusters (e.g., CM3 cDC, CT0a effector memory CD4+ T cells, CT3b TFH-like cells) and compute per-cluster mean normalized expression to build a gene x cluster average-expression profile matrix in the cell-by-gene input format required by the ICELLNET scoring function. Install ICELLNET with devtools::install_github('soumelis-lab/ICELLNET', ref='master', subdir='icellnet') so the manually curated human ligand-receptor database (>1600 interactions, extended to 1669 in v2) with multi-subunit ligand/receptor complex rules is used, matching the original publication. Run the communication-score quantification for the source->target pairs, then extract individual interaction scores (e.g., CD86/CD28, PDCD1LG2/PDCD1, CCL22/CCR4, VEGFA/FLT1) from the score matrix. Write global and per-interaction scores to a results file (e.g., icellnet_scores.csv) and compare against published values as the acceptance criterion. Match the R version to the ICELLNET version (v2.0.0 requires R < 4.1; use v2.2.x for R >= 4.1).

## Related tools

- **ICELLNET** (Computes transcriptome-based communication scores using a manually curated ligand-receptor database with multi-subunit complex rules) — https://github.com/soumelis-lab/ICELLNET
- **devtools** (Installs the ICELLNET package from GitHub (install_github with ref='master', subdir='icellnet')) — https://github.com/soumelis-lab/ICELLNET
- **data.table** (R dependency used for fast loading and manipulation of the expression matrix and score tables)

## Examples

```
library(devtools); install_github("soumelis-lab/ICELLNET",ref="master",subdir="icellnet"); library(icellnet); cc <- ICELLI(ss=scores, db=LRdb, source="CM3", target="CT3b")
```

## Evaluation signals

- Global ICELLNET score for CM3->CT3b equals the published value 1527 and CM3->CT0a equals 1123
- Individual interaction scores match: CD86/CD28 92 vs 40, CD86/CTLA4 92 vs 14, PDCD1LG2/PDCD1 72 vs 19, CCL22/CCR4 100 vs 39, VEGFA/FLT1 21 vs 0
- The gene x cluster profile matrix covers all genes present in the ICELLNET ligand-receptor database for the subsetted clusters
- ICELLNET version and R version pairing is consistent (v2.0.0 with R < 4.1; v2.2.x with R >= 4.1)
- Output CSV contains both global and per-interaction scores for every queried pair

## Limitations

- ICELLNET v2.0.0 requires R version < 4.1; for latest R versions (> 4.1.x), use ICELLNET v2.2.x
- ICELLNET includes a relatively small number of interactions compared to other databases and will in time be extended
- Downstream effectors are not integrated in ICELLNET at this stage, and approaches integrating downstream signaling pathways can lead to unpredictable false positives and false negatives

## Evidence

- [methods] Global score validation: [evidence span withheld — anchor status: not_found]
- [readme] Database curation rationale: "curated exclusively from **human** studies, ICELLNET database takes into account the **multiple** subunits of ligand and receptor complexes (more than 1600 interactions)"
- [readme] Version constraint: "ICELLNET v2.0.0 requires R version < 4.1. For latest R versions (> 4.1.x), please use ICELLNET v2.2.x."
- [readme] Install command: "install_github("soumelis-lab/ICELLNET",ref="master", subdir="icellnet")"
- [discussion] Scope limitation: "Due to these limitations, we have decided not to integrate downstream effectors in ICELLNET, at this stage."
