---
name: single-cell-expression-matrix-processing
description: Use when scRNA-seq target cells have been clustered into sub-clusters
  and each must be written as an expression matrix file (e.g. data_obj/AML1_cluster1,
  100 cells x 4K genes) for downstream counterpart mapping.
when_to_use_negative:
- You need counterpart assignment for individual single cells — CCI explicitly projects
  cell clusters, not single cells, because of prevalent droplets in scRNA-seq data
- Your target cells have not been clustered yet; the README requires proper clustering
  into sub-clusters before running CCI
- The analysis is single-signature-based counterpart identification, which the paper
  states is unreliable due to huge differences between leukemia and healthy cells
  — CCI was developed to integrate multiple statistics instead
edam_operation: http://edamontology.org/operation_2935
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_2229
tools:
- name: CCI (counterpart composite index)
  role: Consumes the per-sub-cluster expression matrices and formatted reference atlas
    to compute BF-probability scores identifying each leukemia subpopulation's healthy
    counterpart
  repo: https://github.com/pengfeeei/cci
- name: bash
  role: Drives compilation (makefile.sh) and execution of the CCI pipeline via run_cci.sh
    on a named sub-cluster input
provenance:
  source_task_ids:
  - task_002
  - task_001
  source_papers:
  - doi: 10.1038/s41421-020-00223-4
    title: Integrated decoding hematopoiesis and leukemogenesis using single-cell
      sequencing and its medical implication
  source_content_hash: sha256:32697c9ab53d9ab622d4865dd309564894f75a7c6426f9f671de2fe46c27ff69
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/pengfeeei/cci
  tools:
  - CCI (counterpart composite index)
  - bash
  edam_operation: http://edamontology.org/operation_2935
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_2229
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.42857142857142855
n_claims: 7
---

# single-cell-expression-matrix-processing

## Summary

Prepare per-sub-cluster single-cell expression matrices and a formatted reference cell atlas so the CCI (counterpart composite index) tool can project leukemia cell subpopulations onto healthy hematopoietic reference cells.

## When to use

Use when scRNA-seq target cells have been clustered into sub-clusters and each must be written as an expression matrix file (e.g. data_obj/AML1_cluster1, 100 cells x 4K genes) for downstream counterpart mapping.

## When NOT to use

- You need counterpart assignment for individual single cells — CCI explicitly projects cell clusters, not single cells, because of prevalent droplets in scRNA-seq data
- Your target cells have not been clustered yet; the README requires proper clustering into sub-clusters before running CCI
- The analysis is single-signature-based counterpart identification, which the paper states is unreliable due to huge differences between leukemia and healthy cells — CCI was developed to integrate multiple statistics instead

## Inputs

- Expression matrix file for each target sub-cluster (cells x genes), placed in data_obj/ (e.g. AML1_cluster1: 100 cells x 4K genes)
- Reference atlas in data_ref/: expression matrix (cells as rows, genes as columns), gene list, cell type annotations, and per-cell identity labels matching the expression matrix rows

## Outputs

- CCI score per reference cell type as a BF (Bayes factor) probability, written to the result/ folder
- Example visualization PDFs in the result/ folder (users are expected to generate their own plots)

## How to apply

First cluster the target cells properly, because CCI projects cell clusters (not single cells) to the reference due to prevalent droplets in scRNA-seq. Export one expression matrix file per sub-cluster into data_obj/ (toy example: AML1_cluster1 with 100 cells and 4K genes). Construct the reference atlas in data_ref/ with four files matching the toy data format: expression matrix (cells as rows, genes as columns), gene list, cell types, and cell identity per row of the expression matrix. Compile the C++ code with 'bash makefile.sh' (needs GNU c++; R >3.4 with Seurat, ggplot2, RColorBrewer), then run 'bash run_cci.sh AML1_cluster1', which finishes in several minutes on one normal CPU for the toy data. The CCI score, expressed as a BF (Bayes factor) probability, is written to the result/ folder.

## Related tools

- **CCI (counterpart composite index)** (Consumes the per-sub-cluster expression matrices and formatted reference atlas to compute BF-probability scores identifying each leukemia subpopulation's healthy counterpart) — https://github.com/pengfeeei/cci
- **bash** (Drives compilation (makefile.sh) and execution of the CCI pipeline via run_cci.sh on a named sub-cluster input)

## Examples

```
bash run_cci.sh AML1_cluster1
```

## Evaluation signals

- 'bash run_cci.sh AML1_cluster1' executes without error and a CCI BF-probability score file appears in the result/ folder
- For the toy data (100 cells, 4K genes), computation completes within several minutes on a single normal CPU
- Reimplemented CCI BF-probability scores show high rank agreement and near-zero maximum deviation versus the released tool's scores across cell/sub-cluster pairs
- Reference files in data_ref/ conform to the four-file toy-data format: expression matrix (cells as rows, genes as columns), gene list, cell types, and cell identities

## Limitations

- CCI requires target cells to be pre-clustered; it assigns counterparts at the sub-cluster level, not per single cell
- Visualization is left to the user — only example PDFs are provided in result/
- Counterpart identification based on a single signature is unreliable because of huge differences between leukemia and healthy cells; the multi-statistic integration is what gives CCI its power

## Evidence

- [readme] CCI projects cell clusters to reference instead of single cell due to prevalent droplets in scRNA-seq. Please properly cluster your target cells before running CCI.: "CCI projects cell clusters to reference instead of single cell due to prevalent droplets in scRNA-seq. Please properly cluster your target cells before running CCI."
- [readme] Reference cells were stored in folder data_ref/ including 4 necessary files as 1)expression matrix (cell as rows and gene as columns), 2)gene list, 3)cell types and 4)identity of each cell: "Reference cells were stored in folder data_ref/ including 4 necessary files as 1)expression matrix (cell as rows and gene as columns), 2)gene list, 3)cell types and 4)identity of each cell"
- [methods] To calculate CCI score, please simplely run bash run_cci.sh AML1_cluster1 (toy data in data_obj/): "To calculate CCI score, please simplely run bash run_cci.sh AML1_cluster1 (toy data in data_obj/)"
- [full_text] CCI combines multiple statistics — expression level, co-expression, and dimension-reduction embedding space — within a composite likelihood statistical framework: [evidence span withheld — anchor status: not_found]
