---
name: composite-likelihood-scoring
description: Use when projecting a clustered target scRNA-seq subpopulation (e.g.,
  a leukemia sub-cluster) onto a reference cell atlas to identify its closest healthy
  counterpart, especially where single-signature similarity is unreliable due to large
  leukemia-vs-healthy differences.
when_to_use_negative:
- Target cells are unclustered single cells rather than sub-clusters — CCI projects
  cell clusters to reference instead of single cells due to prevalent droplets in
  scRNA-seq
- No reference normal-cell atlas is available or the target is not carcinoma/leukemia
  cells being projected onto healthy reference cells
- The goal is analyzing relationships only among leukemia subpopulations without mapping
  onto a healthy hematopoietic reference
edam_operation: http://edamontology.org/operation_2945
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_2640
tools:
- name: CCI (counterpart composite index)
  role: 'The core tool being reimplemented: integrates multiple statistics in a composite-likelihood/Bayes-factor
    framework to score healthy counterparts of leukemia subpopulations'
  repo: https://github.com/pengfeeei/cci
- name: bash
  role: 'Used to run the pipeline entry points: ''bash makefile.sh'' for C++ compilation
    and ''bash run_cci.sh AML1_cluster1'' to compute CCI scores'
  repo: https://github.com/pengfeeei/cci
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1038/s41421-020-00223-4
    title: Integrated decoding hematopoiesis and leukemogenesis using single-cell
      sequencing and its medical implication
  source_content_hash: sha256:7081bfe3bd58e24a35ff09cdef94ebd6cb2646afdbe0518144526c48cc88a5cc
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/pengfeeei/cci
  tools:
  - CCI (counterpart composite index)
  - bash
  edam_operation: http://edamontology.org/operation_2945
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_2640
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7857142857142857
n_claims: 14
---

# composite-likelihood-scoring

## Summary

Implements CCI (Counterpart Composite Index), a composite-likelihood framework that combines multiple similarity statistics — expression-level distance, Spearman rank correlation, PCA-space distance, and gene-set-enrichment AUC differences — into a Bayes-factor probability that maps each leukemia cell subpopulation to its healthy counterpart among reference hematopoietic cells.

## When to use

Use when projecting a clustered target scRNA-seq subpopulation (e.g., a leukemia sub-cluster) onto a reference cell atlas to identify its closest healthy counterpart, especially where single-signature similarity is unreliable due to large leukemia-vs-healthy differences.

## When NOT to use

- Target cells are unclustered single cells rather than sub-clusters — CCI projects cell clusters to reference instead of single cells due to prevalent droplets in scRNA-seq
- No reference normal-cell atlas is available or the target is not carcinoma/leukemia cells being projected onto healthy reference cells
- The goal is analyzing relationships only among leukemia subpopulations without mapping onto a healthy hematopoietic reference

## Inputs

- Reference atlas files in data_ref/: expression matrix (cells as rows, genes as columns), gene list, cell-type labels, and per-cell identity matching the matrix rows
- Target sub-cluster expression matrix file(s) in data_obj/ (e.g., AML1_cluster1 toy data: 100 cells, 4000 genes)
- Gene sets for the compared sub-clusters (used for the enrichment-AUC component)

## Outputs

- CCI BF-probability scores per target-cell/reference-sub-cluster pair, written to the result/ folder
- Example visualization plots as PDFs in result/

## How to apply

Construct a reference atlas in data_ref/ with four files (expression matrix with cells as rows and genes as columns, gene list, cell types, and per-cell identity), then pre-cluster the target cells because CCI projects cell clusters, not single cells, due to prevalent droplets in scRNA-seq. Place each sub-cluster's expression matrix in data_obj/ and run the CCI pipeline, which computes four component statistics for leukemia-vs-reference cell pairs: Euclidean distance of expression profiles, Spearman rank correlation of expression, Euclidean distance in the space of the top 30 principal components, and the AUC difference of gene-set enrichment scores for the compared sub-clusters. The four statistics are combined into a composite likelihood expressed as a Bayes factor (BF) probability, mirroring the formula in the released cci package source. Compile the C++ code with 'bash makefile.sh' (requires GNU c++, R >3.4, Seurat, ggplot2, RColorBrewer) and run per sub-cluster; for the toy data (100 cells, 4000 genes) computation takes a few minutes on one normal CPU. Correctness is judged by comparing reimplemented BF probabilities against the reference scores in result/ (e.g., rank correlation and near-zero max absolute deviation across cell/sub-cluster pairs).

## Related tools

- **CCI (counterpart composite index)** (The core tool being reimplemented: integrates multiple statistics in a composite-likelihood/Bayes-factor framework to score healthy counterparts of leukemia subpopulations) — https://github.com/pengfeeei/cci
- **bash** (Used to run the pipeline entry points: 'bash makefile.sh' for C++ compilation and 'bash run_cci.sh AML1_cluster1' to compute CCI scores) — https://github.com/pengfeeei/cci

## Examples

```
bash makefile.sh && bash run_cci.sh AML1_cluster1
```

## Evaluation signals

- Reimplemented CCI BF-probability scores match the released tool's scores in result/ with high rank agreement (e.g., Spearman rank correlation of BF probabilities across cell/sub-cluster pairs)
- Maximum absolute deviation of BF probabilities between reimplementation and reference scores is near zero within tight numerical tolerance
- Pipeline completes end-to-end on the toy data (100 cells, 4000 genes) within a few minutes on a single normal CPU
- Component statistics (expression Euclidean distance, Spearman correlation, top-30-PC Euclidean distance, enrichment AUC difference) are each computed and combined in the same order/formula as the cci package source
- Output BF probabilities land in the result/ folder in the expected schema (probability scores per reference sub-cluster)

## Limitations

- Identification of healthy counterparts based on a single signature is a stated unsolved challenge due to huge differences between leukemia and healthy cells — hence multiple statistics must be combined, and single-signature variants will underperform
- Target cells must be pre-clustered and formatted exactly like the toy data in data_ref/ and data_obj/; CCI projects clusters, not individual cells
- Visualization is not provided by the tool — users must visualize CCI results themselves, though example plots are given as PDFs in result/
- Requires a C++ compiler and specific R environment (R >3.4 with Seurat, ggplot2, RColorBrewer); CCI v1.0 was designed for projecting carcinoma cells, so scope beyond that setting is untested

## Evidence

- [abstract] We developed counterpart composite index (CCI; available at GitHub: https://github.com/pengfeeei/cci) to search for the healthy counterpart of each leukemia cell subpopulation: "We developed counterpart composite index (CCI; available at GitHub: https://github.com/pengfeeei/cci) to search for the healthy counterpart of each leukemia cell subpopulation"
- [readme] CCI score (BF probability) could be found in folder result/.: "CCI score (BF probability) could be found in folder result/."
- [readme] CCI projects cell clusters to reference instead of single cell due to prevalent droplets in scRNA-seq.: "CCI projects cell clusters to reference instead of single cell due to prevalent droplets in scRNA-seq."
- [readme] Reference cells were stored in folder data_ref/ including 4 necessary files as 1)expression matrix (cell as rows and gene as columns), 2)gene list, 3)cell types and 4)identity of each cell as in expression matrix.: "Reference cells were stored in folder data_ref/ including 4 necessary files as 1)expression matrix (cell as rows and gene as columns), 2)gene list, 3)cell types and 4)identity of each cell as in"
- [readme] For the toy data which contains 100 cells and 4K genes, computating will be finished in several minutes by one normal CPU spread.: "For the toy data which contains 100 cells and 4K genes, computating will be finished in several minutes by one normal CPU spread."
- [discussion] it is a big challenge to identify the healthy counterparts of each leukemia cell subpopulation based on single signature due to the huge differences: "it is a big challenge to identify the healthy counterparts of each leukemia cell subpopulation based on single signature due to the huge differences"
