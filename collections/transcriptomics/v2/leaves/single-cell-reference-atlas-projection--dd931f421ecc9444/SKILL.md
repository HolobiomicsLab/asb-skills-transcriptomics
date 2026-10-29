---
name: single-cell-reference-atlas-projection
description: Use when single-cell RNA-seq clusters (e.g., a leukemia sub-cluster such
  as AML1_cluster1) must be assigned a healthy counterpart cell type from a reference
  atlas to infer lineage stage, leukemia subtype, or stemness.
when_to_use_negative:
- Input cells are not clustered into subpopulations — CCI projects cell clusters to
  reference instead of single cells due to prevalent droplets in scRNA-seq
- The analysis only needs relationships among leukemia subpopulations themselves;
  the article notes such studies 'lost the whole picture of leukemogenesis' — reference-atlas
  projection is the approach for that whole picture
- Attempting counterpart assignment from a single signature alone is a stated unsolved
  challenge ('a big challenge to identify the healthy counterparts of each leukemia
  cell subpopulation based on single signature due to the huge differences') — use
  the multi-statistic CCI framework instead
edam_topics:
- http://edamontology.org/topic_3170
tools:
- name: CCI (counterpart composite index)
  role: 'Core algorithm: integrates multiple statistics (expression level, co-expression,
    dimension-reduction embedding) in a composite likelihood framework to map leukemia
    sub-clusters onto the reference hematopoietic lineage and output BF-probability
    counterpart scores'
  repo: https://github.com/pengfeeei/cci
- name: bash
  role: 'Shell entry point to compile and launch CCI: `bash makefile.sh` to build
    C++ sources, then `bash run_cci.sh AML1_cluster1` to compute the CCI score for
    a sub-cluster'
  repo: https://github.com/pengfeeei/cci
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1038/s41421-020-00223-4
    title: Integrated decoding hematopoiesis and leukemogenesis using single-cell
      sequencing and its medical implication
  source_content_hash: sha256:6bba1d8a8ec0a7225776ff8a19bb131366e13822fbed2b75f906d373f27fc114
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/pengfeeei/cci
  tools:
  - CCI (counterpart composite index)
  - bash
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7
n_claims: 10
---

# single-cell-reference-atlas-projection

## Summary

Project a leukemia/carcinoma cell subpopulation onto a reference healthy cell atlas using CCI (counterpart composite index), which integrates multiple statistics — expression level, co-expression, and dimension-reduction embedding space — within a composite likelihood (Bayes factor) framework to assign each subpopulation a healthy counterpart.

## When to use

Use when single-cell RNA-seq clusters (e.g., a leukemia sub-cluster such as AML1_cluster1) must be assigned a healthy counterpart cell type from a reference atlas to infer lineage stage, leukemia subtype, or stemness.

## When NOT to use

- Input cells are not clustered into subpopulations — CCI projects cell clusters to reference instead of single cells due to prevalent droplets in scRNA-seq
- The analysis only needs relationships among leukemia subpopulations themselves; the article notes such studies 'lost the whole picture of leukemogenesis' — reference-atlas projection is the approach for that whole picture
- Attempting counterpart assignment from a single signature alone is a stated unsolved challenge ('a big challenge to identify the healthy counterparts of each leukemia cell subpopulation based on single signature due to the huge differences') — use the multi-statistic CCI framework instead

## Inputs

- Reference atlas in data_ref/: expression matrix (cells as rows, genes as columns), gene list, cell types, and per-cell identity of each cell, formatted like the toy data (a comprehensive normal BMMC atlas is provided)
- Target sub-cluster expression matrix file(s) in data_obj/, one per cluster (e.g., AML1_cluster1, 100 cells x 4K genes in the toy data)
- Pre-clustered target cells (CCI requires clusters, not single cells)

## Outputs

- CCI score file in result/ expressing the healthy counterpart assignment as a BF (Bayes factor) probability
- Example plots as PDFs in result/ (visualization of CCI results is left to the user)

## How to apply

First construct or supply a reference atlas (e.g., the comprehensive normal BMMC atlas in data_ref/) containing four files: an expression matrix (cells as rows, genes as columns), a gene list, cell types, and per-cell identity, formatted like the shipped toy data. Cluster the target cells before running CCI, since CCI projects cell clusters rather than single cells due to prevalent droplets in scRNA-seq data; each sub-cluster's expression matrix goes into data_obj/. Then run the CCI calculation (e.g., `bash run_cci.sh AML1_cluster1`); the tool integrates expression level, co-expression, and dimension-reduction embedding statistics in a composite likelihood framework. For the shipped toy dataset (100 cells x 4K genes) the run finishes in several minutes on one normal CPU. Retrieve the CCI score — expressed as a BF (Bayes factor) probability — from the result/ folder; a higher BF probability identifies the healthy counterpart assignment. Judging correctness: the run should complete without error and a CCI (BF probability) result file must appear in result/; downstream, healthy counterpart calls should be biologically consistent (e.g., relapsed leukemia cells' counterparts closer to the root of the hematopoietic tree).

## Related tools

- **CCI (counterpart composite index)** (Core algorithm: integrates multiple statistics (expression level, co-expression, dimension-reduction embedding) in a composite likelihood framework to map leukemia sub-clusters onto the reference hematopoietic lineage and output BF-probability counterpart scores) — https://github.com/pengfeeei/cci
- **bash** (Shell entry point to compile and launch CCI: `bash makefile.sh` to build C++ sources, then `bash run_cci.sh AML1_cluster1` to compute the CCI score for a sub-cluster) — https://github.com/pengfeeei/cci

## Examples

```
bash makefile.sh; bash run_cci.sh AML1_cluster1
```

## Evaluation signals

- `bash makefile.sh` compiles the C++ code in seconds and `bash run_cci.sh AML1_cluster1` executes without error
- A CCI score (BF probability) file is present in the result/ folder after the run completes
- Toy-scale run (100 cells x 4K genes) finishes within several minutes on a single normal CPU
- The reference atlas in data_ref/ contains all four required files (expression matrix, gene list, cell types, cell identities) formatted like the toy data
- Assigned counterparts are biologically consistent with expectations, e.g., relapsed leukemia cells map to counterparts closer to the root of the hematopoietic tree

## Limitations

- CCI projects cell clusters rather than single cells, so targets must be properly pre-clustered; incorrect clustering propagates into counterpart assignments
- Visualization is not automated — users must visualize CCI results themselves; only example plots as PDFs are provided in result/
- Healthy counterpart identification based on a single signature is unreliable due to huge differences between leukemia and healthy cells, which is why CCI integrates multiple statistics; the multi-time-point clinical analysis was limited to a single patient (ALL3)

## Evidence

- [abstract] CCI assigns healthy counterparts by integrating multiple statistics: "We developed counterpart composite index (CCI; available at GitHub: https://github.com/pengfeeei/cci) to search for the healthy counterpart of each leukemia cell subpopulation"
- [discussion] it is a big challenge to identify the healthy counterparts of each leukemia cell subpopulation based on single signature: "it is a big challenge to identify the healthy counterparts of each leukemia cell subpopulation based on single signature due to the huge differences"
- [readme] CCI projects cell clusters to reference instead of single cell due to prevalent droplets in scRNA-seq: "CCI projects cell clusters to reference instead of single cell due to prevalent droplets in scRNA-seq. Please properly cluster your target cells before running CCI."
- [readme] Reference cells were stored in folder data_ref/ including 4 necessary files: "Reference cells were stored in folder data_ref/ including 4 necessary files as 1)expression matrix (cell as rows and gene as columns), 2)gene list, 3)cell types and 4)identity of each cell"
- [readme] For the toy data which contains 100 cells and 4K genes, computating will be finished in several minutes by one normal CPU: "For the toy data which contains 100 cells and 4K genes, computating will be finished in several minutes by one normal CPU spread."
- [readme] CCI score (BF probability) could be found in folder result/: "CCI score (BF probability) could be found in folder result/."
