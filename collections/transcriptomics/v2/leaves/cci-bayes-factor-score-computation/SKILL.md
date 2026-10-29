---
name: cci-bayes-factor-score-computation
description: Use when a clustered scRNA-seq target population (e.g. AML1_cluster1)
  must be assigned a healthy counterpart from a reference atlas via a composite-likelihood/Bayes-factor
  score.
when_to_use_negative:
- Your input is single cells without prior clustering — CCI projects cell clusters
  rather than individual cells due to prevalent droplets in scRNA-seq
- You need counterpart assignment from a single gene signature — the article states
  this is a big challenge due to huge differences between leukemia and healthy cells,
  which is why the multi-statistic composite index is required
- You lack a formatted reference atlas in data_ref/ (expression matrix, gene list,
  cell types, identities) matching the toy-data format
edam_operation: http://edamontology.org/operation_3801
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_2640
tools:
- name: CCI (counterpart composite index)
  role: Core tool that integrates multiple statistics (expression level, co-expression,
    dimension-reduction embedding space) in a composite likelihood framework to compute
    the BF-probability counterpart assignment
  repo: https://github.com/pengfeeei/cci
- name: bash
  role: Shell driver used to compile (bash makefile.sh) and launch the score computation
    (bash run_cci.sh AML1_cluster1)
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1038/s41421-020-00223-4
    title: Integrated decoding hematopoiesis and leukemogenesis using single-cell
      sequencing and its medical implication
  source_content_hash: sha256:14269b09ed48181d9e4d379c82308b7b4ae0245c8fc67661060195a5d76d49bd
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/pengfeeei/cci
  tools:
  - CCI (counterpart composite index)
  - bash
  edam_operation: http://edamontology.org/operation_3801
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_2640
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.42857142857142855
n_claims: 7
---

# cci-bayes-factor-score-computation

## Summary

Computes the CCI (counterpart composite index) score, expressed as a Bayes factor (BF) probability, that maps a leukemia/carcinoma cell sub-cluster onto a healthy reference cell atlas to identify its healthy counterpart by integrating expression, co-expression and embedding-space statistics.

## When to use

Use when a clustered scRNA-seq target population (e.g. AML1_cluster1) must be assigned a healthy counterpart from a reference atlas via a composite-likelihood/Bayes-factor score.

## When NOT to use

- Your input is single cells without prior clustering — CCI projects cell clusters rather than individual cells due to prevalent droplets in scRNA-seq
- You need counterpart assignment from a single gene signature — the article states this is a big challenge due to huge differences between leukemia and healthy cells, which is why the multi-statistic composite index is required
- You lack a formatted reference atlas in data_ref/ (expression matrix, gene list, cell types, identities) matching the toy-data format

## Inputs

- Reference atlas in data_ref/: expression matrix (cells as rows, genes as columns), gene list, cell type annotations, and cell identity file (e.g. comprehensive normal BMMC atlas)
- Target sub-cluster expression matrix file in data_obj/ (one file per cluster, e.g. AML1_cluster1; toy data: 100 cells x 4K genes)
- Pre-computed cluster labels for the target cells (CCI projects clusters, not single cells)

## Outputs

- CCI score as BF (Bayes factor) probability identifying the healthy counterpart assignment, written to result/
- Example PDF visualization plots in result/ (user is expected to generate their own visualizations)

## How to apply

First compile the C++ code with `bash makefile.sh` (requires GNU c++, R >3.4 with Seurat, ggplot2, RColorBrewer) and place a reference atlas (expression matrix, gene list, cell types, cell identities) in data_ref/, formatted like the toy data. Cluster the target cells beforehand because CCI projects cell clusters — not single cells — due to prevalent droplets in scRNA-seq, storing one expression matrix per sub-cluster in data_obj/. Then run `bash run_cci.sh AML1_cluster1`; for the toy dataset of 100 cells x 4K genes the computation finishes in several minutes on one normal CPU. Retrieve the CCI score (BF probability) and example PDF plots from the result/ folder; CCI integrates multiple statistics (expression level, co-expression, dimension-reduction embedding space) in a composite likelihood framework, so the BF probability — not a single signature — identifies the counterpart.

## Related tools

- **CCI (counterpart composite index)** (Core tool that integrates multiple statistics (expression level, co-expression, dimension-reduction embedding space) in a composite likelihood framework to compute the BF-probability counterpart assignment) — https://github.com/pengfeeei/cci
- **bash** (Shell driver used to compile (bash makefile.sh) and launch the score computation (bash run_cci.sh AML1_cluster1))

## Examples

```
bash run_cci.sh AML1_cluster1
```

## Evaluation signals

- `bash run_cci.sh AML1_cluster1` executes without error end-to-end after `bash makefile.sh` compilation
- A CCI score (BF probability) file is present in the result/ folder upon completion
- Example PDF plots are produced in result/ alongside the score
- Runtime scales as documented: toy data (100 cells x 4K genes) completes within several minutes on a single normal CPU
- The BF probability output names a specific healthy hematopoietic counterpart for the sub-cluster, consistent with mapping onto the reference hematopoietic tree

## Limitations

- Identifying healthy counterparts based on a single signature is a stated unsolved challenge due to huge differences between leukemia and healthy cells; CCI's composite multi-statistic framework is required, but individual statistics alone are insufficient
- Users must visualize CCI results on their own; the repository only provides example PDF plots
- Target cells must be properly clustered beforehand; CCI projects clusters, not single cells
- The proposed model that mutant stem cells partially maintain their original developmental trajectory is presented as a hypothesis, so counterpart assignments should be interpreted as mappings rather than validated lineage relationships

## Evidence

- [methods] To calculate CCI score, please simplely run: "To calculate CCI score, please simplely run"
- [methods] For the toy data which contains 100 cells and 4K genes: "For the toy data which contains 100 cells and 4K genes"
- [readme] CCI projects cell clusters to reference instead of single cell due to prevalent droplets in scRNA-seq.: "CCI projects cell clusters to reference instead of single cell due to prevalent droplets in scRNA-seq."
- [readme] CCI score (BF probability) could be found in folder result/.: "CCI score (BF probability) could be found in folder result/."
- [discussion] We developed CCI that integrates multiple measurements for searching the healthy counterpart of each leukemia cell subpopulation in reference hematopoietic cells.: "We developed CCI that integrates multiple measurements for searching the healthy counterpart of each leukemia cell subpopulation in reference hematopoietic cells."
