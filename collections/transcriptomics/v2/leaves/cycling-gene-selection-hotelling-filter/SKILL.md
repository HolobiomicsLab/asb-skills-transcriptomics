---
name: cycling-gene-selection-hotelling-filter
description: Use when running DeepCycle on scRNA-seq data preprocessed with scvelo.pp.moments
  and you must pick a base gene (initial condition for cell_cycle_theta) from a GOterm
  cell_cycle gene list without prior knowledge of which genes cycle.
when_to_use_negative:
- No cycling genes can be found after the Hotelling screening — the results will not
  be reliable and deeper sequencing (more RNAs per cell) is needed instead of forcing
  the analysis
- The Anndata has not been preprocessed with scvelo.pp.moments to compute imputed
  unspliced (Mu) and spliced (Ms) layers, since DeepCycle requires them as input
- The goal is to define cell cycle phases in a cellular model based solely on gene
  markers, which the authors note lacks generalizability
edam_operation: http://edamontology.org/operation_2945
edam_topics:
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_3170
tools:
- name: DeepCycle
  role: Runs the Hotelling filter (--hotelling flag) over the candidate gene list
    to identify genes with more than one expression state, providing the base gene
    for phase initialization
  repo: https://github.com/andreariba/DeepCycle
- name: scVelo
  role: Computes the imputed spliced (Ms) and unspliced (Mu) moments that constitute
    the unspliced-spliced read space the filter operates on
  repo: https://scvelo.readthedocs.io/
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1038/s41467-022-30545-8
    title: Cell cycle gene regulation dynamics revealed by RNA velocity and deep-learning
  source_content_hash: sha256:e3cfba6fa79ee21f2d453ac5689332e4e7f966e030df4ca6aac8d3cd59af1c93
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/andreariba/DeepCycle
  tools:
  - DeepCycle
  - Python
  - TensorFlow
  - numpy
  - scVelo
  - scipy
  - velocyto
  edam_operation: http://edamontology.org/operation_2945
  edam_topics:
  - http://edamontology.org/topic_0203
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9230769230769231
n_claims: 13
---

# cycling-gene-selection-hotelling-filter

## Summary

Selects genuinely cycling genes from a cell-cycle GO-term candidate list using DeepCycle's Hotelling filter, which flags genes showing at least two distinct expression states in unspliced-spliced space so a reliable base gene can seed the transcriptional phase inference.

## When to use

Use when running DeepCycle on scRNA-seq data preprocessed with scvelo.pp.moments and you must pick a base gene (initial condition for cell_cycle_theta) from a GOterm cell_cycle gene list without prior knowledge of which genes cycle.

## When NOT to use

- No cycling genes can be found after the Hotelling screening — the results will not be reliable and deeper sequencing (more RNAs per cell) is needed instead of forcing the analysis
- The Anndata has not been preprocessed with scvelo.pp.moments to compute imputed unspliced (Mu) and spliced (Ms) layers, since DeepCycle requires them as input
- The goal is to define cell cycle phases in a cellular model based solely on gene markers, which the authors note lacks generalizability

## Inputs

- Anndata (.h5ad) preprocessed with velocyto and scvelo.pp.moments (Ms and Mu layers)
- Candidate cycling gene list for the organism of interest (e.g. GOterm cell_cycle annotation file such as go_annotation/GO_cell_cycle_annotation_mouse.txt or the human cell cycle GO annotation)

## Outputs

- ./hotelling/ subdirectory listing genes showing more than one state of expression
- Selected base gene to use as --base_gene in the final DeepCycle run
- Anndata output annotated with adata.obs['cell_cycle_theta'] once the full DeepCycle run completes

## How to apply

First run DeepCycle.py with a random gene from the gene_list file together with the --hotelling flag; DeepCycle writes the genes showing more than one state of expression into the ./hotelling/ subdirectory. Inspect these Hotelling-selected genes, verify they display multiple expression states, and choose one of them as the --base_gene for the definitive run (e.g. MELK for human fibroblasts, Nusap1 for mESCs). Concurrently apply an --expression_threshold around 1.0 (0.5 for mESCs, adjusted to sequencing depth per cell) to exclude weakly expressed genes from the inference. The rationale is that if the base gene is not truly cycling, the atan2 of its z-scored spliced and unspliced reads gives a meaningless initial phase and the whole theta inference collapses.

## Related tools

- **DeepCycle** (Runs the Hotelling filter (--hotelling flag) over the candidate gene list to identify genes with more than one expression state, providing the base gene for phase initialization) — https://github.com/andreariba/DeepCycle
- **scVelo** (Computes the imputed spliced (Ms) and unspliced (Mu) moments that constitute the unspliced-spliced read space the filter operates on) — https://scvelo.readthedocs.io/

## Examples

```
python DeepCycle.py --input_adata adata.h5ad --gene_list GO_cell_cycle_annotation_human.txt --base_gene RANDOM_GENE --expression_threshold 1.0 --hotelling --output_adata screen.h5ad
```

## Evaluation signals

- The ./hotelling/ directory is populated with genes flagged as showing at least two different expression states; an empty result indicates unreliable data or insufficient sequencing depth
- The chosen base gene appears among the Hotelling-selected cycling genes rather than being an arbitrary pick
- Expression threshold is tuned to sequencing depth (suggested ~1.0, e.g. 0.5 was used for the mESC dataset with Nusap1 as base gene)
- Subsequent ./fits/ unspliced-spliced pattern plots show proper cycling fits and ./training/ MSE plots show convergence, confirming the base gene initialization succeeded

## Limitations

- If you do not find any cycling genes the results will not be reliable; it is possible you have to sequence deeper and increase the number of RNAs per cell
- If cycling patterns are found but the fits are wrong, the DeepCycle author should be contacted — the method cannot self-correct a bad initialization
- A minimum average expression value is required for gene inclusion; the exact threshold depends on sequencing depth per cell

## Evidence

- [readme] A good strategy to detect a cycling gene is to first run DeepCycle with a random gene from the gene_list file together with the hotelling flag.: "A good strategy to detect a cycling gene is to first run DeepCycle with a random gene from the gene_list file together with the hotelling flag."
- [readme] ./hotelling/ [only with the option --hotelling]: the genes showing more than one state of expression.: "./hotelling/ [only with the option --hotelling]: the genes showing more than one state of expression."
- [readme] If you do not find any cycling genes the results will not be reliable. It is possible you have to sequence deeper and increase the number of RNAs per cell.: "If you do not find any cycling genes the results will not be reliable. It is possible you have to sequence deeper and increase the number of RNAs per cell."
- [methods] a minimum value of average expression for a gene. Suggested value around 1.0, depending on your sequencing depth per cell.: "a minimum value of average expression for a gene. Suggested value around 1.0, depending on your sequencing depth per cell."
