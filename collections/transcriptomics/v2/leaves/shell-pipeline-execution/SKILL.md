---
name: shell-pipeline-execution
description: Use when a scRNA-seq target cell sub-cluster (e.g., AML1_cluster1, 100
  cells x 4K genes) must be mapped onto a reference cell atlas via the CCI repository's
  shell scripts, and a CCI score must be produced.
when_to_use_negative:
- Target cells have not been clustered yet — CCI projects cell clusters (not single
  cells) due to prevalent droplets in scRNA-seq, so single unclustered cells are the
  wrong input
- The task is de novo visualization or downstream plotting of CCI results — the README
  states users should visualize CCI results by their own
- No reference atlas in the data_ref/ format (expression matrix, gene list, cell types,
  identities) is available
edam_operation: http://edamontology.org/operation_2945
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_0622
tools:
- name: CCI (counterpart composite index)
  role: The composite-likelihood tool being executed; integrates expression level,
    co-expression and dimension-reduction embedding statistics to assign healthy counterparts
    to leukemia sub-clusters
  repo: https://github.com/pengfeeei/cci
- name: bash
  role: Shell driver used to compile the C++ code ('bash makefile.sh') and launch
    the run ('bash run_cci.sh AML1_cluster1')
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1038/s41421-020-00223-4
    title: Integrated decoding hematopoiesis and leukemogenesis using single-cell
      sequencing and its medical implication
  source_content_hash: sha256:1d2aad2f9f32453c0369877c98d8d0839fbc70803c3477ac1978fc6ba89d3df0
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
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.4444444444444444
n_claims: 9
---

# shell-pipeline-execution

## Summary

Run the CCI (counterpart composite index) computational pipeline as a bash-driven workflow — compiling the C++ code with 'bash makefile.sh' and executing 'bash run_cci.sh <sub-cluster>' to project leukemia cell sub-clusters onto a reference hematopoietic atlas and obtain CCI scores (BF probabilities).

## When to use

Use when a scRNA-seq target cell sub-cluster (e.g., AML1_cluster1, 100 cells x 4K genes) must be mapped onto a reference cell atlas via the CCI repository's shell scripts, and a CCI score must be produced.

## When NOT to use

- Target cells have not been clustered yet — CCI projects cell clusters (not single cells) due to prevalent droplets in scRNA-seq, so single unclustered cells are the wrong input
- The task is de novo visualization or downstream plotting of CCI results — the README states users should visualize CCI results by their own
- No reference atlas in the data_ref/ format (expression matrix, gene list, cell types, identities) is available

## Inputs

- Reference atlas files in data_ref/: expression matrix (cells as rows, genes as columns), gene list, cell types, and cell identity file (formatted as the shipped toy data)
- Target sub-cluster expression matrix file(s) in data_obj/ (e.g., AML1_cluster1; toy data: 100 cells x 4K genes)
- CCI repository source code requiring GNU c++ and R >3.4 with Seurat, ggplot2, RColorBrewer

## Outputs

- CCI score file in result/ expressed as a BF (Bayes factor) probability identifying the healthy counterpart assignment
- Example visualization plots as PDFs in result/

## How to apply

First compile the C++ sources of CCI by running 'bash makefile.sh', which should finish in seconds and requires GNU c++; ensure R >3.4 with the Seurat, ggplot2 and RColorBrewer packages installed. Prepare the inputs: reference cells in data_ref/ (expression matrix with cells as rows and genes as columns, gene list, cell types, and cell identity files formatted as the toy data) and one expression matrix file per target sub-cluster in data_obj/. Then execute 'bash run_cci.sh AML1_cluster1' (or the name of your sub-cluster file); for the toy data of 100 cells and 4K genes this finishes within several minutes on one normal CPU. Finally inspect the result/ folder for the CCI score expressed as a BF (Bayes factor) probability and the example PDF plots; visualization beyond the provided examples is left to the user.

## Related tools

- **CCI (counterpart composite index)** (The composite-likelihood tool being executed; integrates expression level, co-expression and dimension-reduction embedding statistics to assign healthy counterparts to leukemia sub-clusters) — https://github.com/pengfeeei/cci
- **bash** (Shell driver used to compile the C++ code ('bash makefile.sh') and launch the run ('bash run_cci.sh AML1_cluster1'))

## Examples

```
bash makefile.sh && bash run_cci.sh AML1_cluster1
```

## Evaluation signals

- 'bash makefile.sh' compiles without error in seconds
- 'bash run_cci.sh AML1_cluster1' executes end-to-end without error on a single normal CPU
- A CCI score (BF probability) file is present in the result/ folder upon completion
- Toy-scale run (100 cells x 4K genes) completes within several minutes, matching the README's stated runtime
- result/ contains the example PDF plots referenced in the README

## Limitations

- Identifying healthy counterparts based on a single signature is a stated unsolved challenge due to huge differences between leukemia and healthy cells; CCI mitigates this by integrating multiple statistics but the mapping remains challenging
- CCI projects clusters rather than single cells, so results depend on proper prior clustering of target cells
- Users must visualize CCI results themselves; only example PDF plots are provided

## Evidence

- [readme] bash makefile.sh: "Please compile cpp codes of CCI by GNU c++, to do this just run shells

   bash makefile.sh"
- [readme] run instructions: "To calculate CCI score, please simplely run 

   bash run_cci.sh AML1_cluster1 (toy data in data_obj/)"
- [readme] toy data runtime: "For the toy data which contains 100 cells and 4K genes, computating will be finished in several minutes by one normal CPU spread."
- [readme] result location: "CCI score (BF probability) could be found in folder result/."
- [readme] cluster-before-CCI: "CCI projects cell clusters to reference instead of single cell due to prevalent droplets in scRNA-seq. Please properly cluster your target cells before running CCI."
- [readme] dependencies: "CCI also need R >3.4 and several R packages available in your computer including Seurat, ggplot2 and RColorBrewer."
- [abstract] CCI purpose: "We developed counterpart composite index (CCI; available at GitHub: https://github.com/pengfeeei/cci) to search for the healthy counterpart of each leukemia cell subpopulation"
