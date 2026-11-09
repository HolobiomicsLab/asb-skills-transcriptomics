---
name: leukemia-counterpart-mapping-onto-hematopoietic-reference
description: Use when scRNA-seq data from a leukemia (or carcinoma) cell subpopulation
  must be projected onto a healthy reference atlas to identify its healthy counterpart,
  subtype, and lineage/developmental stage.
when_to_use_negative:
- Goal is to analyze relationships only among leukemia subpopulations without a healthy
  reference — the paper notes such studies 'lost the whole picture of leukemogenesis'
- 'Target cells are unclustered single cells: CCI projects cell clusters to the reference
  rather than single cells, so cells must be properly clustered first'
- No healthy reference atlas is available or the reference files cannot be formatted
  as the required four-file toy-data format (expression matrix, gene list, cell types,
  cell identity)
edam_operation: http://edamontology.org/operation_3801
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_2640
tools:
- name: CCI (counterpart composite index)
  role: Core tool that integrates multiple statistics (expression level, co-expression,
    dimension-reduction embedding) in a composite likelihood framework to map leukemia
    sub-clusters onto reference hematopoietic cells and output BF-probability counterpart
    scores
  repo: https://github.com/pengfeeei/cci
- name: bash
  role: Shell entry point for compiling (bash makefile.sh) and running the pipeline
    (bash run_cci.sh <sub-cluster>)
  repo: https://github.com/pengfeeei/cci
- name: Seurat
  role: R package dependency (with ggplot2 and RColorBrewer) required for CCI's R
    environment, e.g. clustering and handling scRNA-seq objects
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1038/s41421-020-00223-4
    title: Integrated decoding hematopoiesis and leukemogenesis using single-cell
      sequencing and its medical implication
  source_content_hash: sha256:0954d268bc4c8bc9964f11fb06a773574c1b61930907d54fa68ce8b080433a39
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/pengfeeei/cci
  tools:
  - CCI (counterpart composite index)
  - Seurat
  - bash
  edam_operation: http://edamontology.org/operation_3801
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_2640
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 10
---

# leukemia-counterpart-mapping-onto-hematopoietic-reference

## Summary

Applies the CCI (counterpart composite index) tool, which integrates multiple statistics (expression level, co-expression, and dimension-reduction embedding space) in a composite likelihood/Bayes-factor framework, to map a leukemia cell subpopulation onto a healthy hematopoietic reference atlas and assign it a healthy counterpart.

## When to use

Use when scRNA-seq data from a leukemia (or carcinoma) cell subpopulation must be projected onto a healthy reference atlas to identify its healthy counterpart, subtype, and lineage/developmental stage.

## When NOT to use

- Goal is to analyze relationships only among leukemia subpopulations without a healthy reference — the paper notes such studies 'lost the whole picture of leukemogenesis'
- Target cells are unclustered single cells: CCI projects cell clusters to the reference rather than single cells, so cells must be properly clustered first
- No healthy reference atlas is available or the reference files cannot be formatted as the required four-file toy-data format (expression matrix, gene list, cell types, cell identity)

## Inputs

- Reference atlas in data_ref/: expression matrix (cells as rows, genes as columns), gene list, cell types, and cell identity files (toy format provided; a normal BMMC atlas is available)
- Target sub-cluster expression matrix file per cluster in data_obj/ (e.g. toy AML1_cluster1: 100 cells x 4K genes)
- Sub-cluster assignments obtained by clustering the target cells beforehand

## Outputs

- CCI score expressed as a BF (Bayes factor) probability per reference cell type, written to result/, identifying the healthy counterpart assignment
- Example visualization plots as PDFs in result/ (custom visualization by the user)

## How to apply

First construct a normal-cell reference atlas in data_ref/ with four files: expression matrix (cells as rows, genes as columns), gene list, cell types, and per-cell identity, formatted like the toy data. Second, cluster the target cells before running CCI (CCI projects clusters rather than single cells due to prevalent droplets), and store one expression matrix file per sub-cluster in data_obj/. Third, run CCI via the provided shell script (e.g. 'bash run_cci.sh AML1_cluster1'); the C++ code should first be compiled with 'bash makefile.sh' using GNU c++, with R >3.4 plus Seurat, ggplot2 and RColorBrewer available. Finally, retrieve the CCI score (a BF, Bayes factor, probability) identifying the healthy counterpart from the result/ folder; visualization is left to the user, with example PDF plots provided in result/. The composite index integrates multiple measurements because identifying counterparts from a single signature is unreliable given the huge differences between leukemia and healthy cells.

## Related tools

- **CCI (counterpart composite index)** (Core tool that integrates multiple statistics (expression level, co-expression, dimension-reduction embedding) in a composite likelihood framework to map leukemia sub-clusters onto reference hematopoietic cells and output BF-probability counterpart scores) — https://github.com/pengfeeei/cci
- **bash** (Shell entry point for compiling (bash makefile.sh) and running the pipeline (bash run_cci.sh <sub-cluster>)) — https://github.com/pengfeeei/cci
- **Seurat** (R package dependency (with ggplot2 and RColorBrewer) required for CCI's R environment, e.g. clustering and handling scRNA-seq objects)

## Examples

```
bash makefile.sh; bash run_cci.sh AML1_cluster1
```

## Evaluation signals

- 'bash run_cci.sh AML1_cluster1' executes without error end-to-end (compilation via bash makefile.sh completed in seconds; runtime is several minutes on one normal CPU for the 100-cell x 4K-gene toy data)
- A CCI score file expressed as BF probability is present in the result/ folder upon completion
- The assigned healthy counterpart places the subpopulation at a plausible position on the hematopoietic tree (e.g. relapsed cells map closer to the root of the hematopoietic tree)
- Input files match the required reference format: expression matrix with cells as rows and genes as columns, plus gene list, cell types, and per-cell identity files
- Assignment is stable at cluster level (CCI scores clusters, not single cells), consistent with the sub-cluster structure in data_obj/

## Limitations

- Identifying healthy counterparts based on a single signature is a stated unsolved challenge due to huge differences between leukemia and healthy cells; CCI mitigates but does not eliminate this by integrating multiple statistics
- CCI projects cell clusters rather than single cells because of prevalent droplets in scRNA-seq, so results depend on the quality of prior clustering
- The mutant-stem-cell model explaining why subpopulations have different counterparts is a hypothesis, not a validated result; multi-time-point clinical analysis was demonstrated in only a single patient (ALL3)

## Evidence

- [abstract] CCI assigns healthy counterparts by integrating multiple statistics: "We developed counterpart composite index (CCI; available at GitHub: https://github.com/pengfeeei/cci) to search for the healthy counterpart of each leukemia cell subpopulation"
- [methods] Run command and toy scale: "bash run_cci.sh AML1_cluster1 (toy data in data_obj/)"
- [readme] Runtime and output location: "For the toy data which contains 100 cells and 4K genes, computating will be finished in several minutes by one normal CPU spread. CCI score (BF probability) could be found in folder result/."
- [readme] Cluster-level projection rationale: "CCI projects cell clusters to reference instead of single cell due to prevalent droplets in scRNA-seq. Please properly cluster your target cells before running CCI."
- [readme] Reference atlas file requirements: "Reference cells were stored in folder data_ref/ including 4 necessary files as 1)expression matrix (cell as rows and gene as columns), 2)gene list, 3)cell types and 4)identity of each cell"
- [discussion] Single-signature challenge motivating CCI: "it is a big challenge to identify the healthy counterparts of each leukemia cell subpopulation based on single signature due to the huge differences"
