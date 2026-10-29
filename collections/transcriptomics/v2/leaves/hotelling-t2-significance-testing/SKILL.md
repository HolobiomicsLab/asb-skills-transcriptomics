---
name: hotelling-t2-significance-testing
description: Use when preprocessing AnnData (Ms/Mu layers from scvelo.pp.moments)
  before running DeepCycle and you need to select a trustworthy base gene or restrict
  inference to genes showing at least two distinct expression states in unspliced-spliced
  space.
when_to_use_negative:
- If no cycling genes are found after filtering, DeepCycle results will not be reliable
  — likely the data need deeper sequencing and more RNAs per cell rather than further
  filtering
- If you only need a cell-cycle phase annotation and already have a validated base
  gene, the optional Hotelling filter is not required for the DeepCycle run
- When the organism of interest has no curated GOterm cell_cycle gene list to restrict
  candidates, the filter has no defined candidate set to operate on
edam_operation: http://edamontology.org/operation_2239
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: DeepCycle
  role: Provides the --hotelling flag and ./hotelling/ output directory that implement
    the Gaussian-mixture/Hotelling gene filter over candidate cycling genes
  repo: https://github.com/andreariba/DeepCycle
- name: scVelo
  role: Produces the imputed 'Ms' and 'Mu' moments layers that supply the (spliced,
    unspliced) read distributions fed to the filter
- name: scipy
  role: Statistical library (tested on 1.5.2) used for the Hotelling T2 significance
    test and Gaussian-mixture fitting
- name: numpy
  role: Numerical backend (tested on 1.19.1) for fitting the bivariate Gaussian mixtures
    to spliced/unspliced reads
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1038/s41467-022-30545-8
    title: Cell cycle gene regulation dynamics revealed by RNA velocity and deep-learning
  source_content_hash: sha256:9fe9350de3dae3e82b6aff68e66abdd7d809322105aa286731d10845946f24b9
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/andreariba/DeepCycle
  tools:
  - DeepCycle
  - numpy
  - scVelo
  - scipy
  edam_operation: http://edamontology.org/operation_2239
  edam_topics:
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5555555555555556
n_claims: 9
---

# hotelling-t2-significance-testing

## Summary

A gene-filtering skill for DeepCycle that identifies reliably cycling genes by fitting a two-component bivariate Gaussian mixture to each gene's (spliced, unspliced) reads and retaining genes whose fitted mixture shows more than one expression state, validated with the Hotelling filter on n=158 cycling genes.

## When to use

Use when preprocessing AnnData (Ms/Mu layers from scvelo.pp.moments) before running DeepCycle and you need to select a trustworthy base gene or restrict inference to genes showing at least two distinct expression states in unspliced-spliced space.

## When NOT to use

- If no cycling genes are found after filtering, DeepCycle results will not be reliable — likely the data need deeper sequencing and more RNAs per cell rather than further filtering
- If you only need a cell-cycle phase annotation and already have a validated base gene, the optional Hotelling filter is not required for the DeepCycle run
- When the organism of interest has no curated GOterm cell_cycle gene list to restrict candidates, the filter has no defined candidate set to operate on

## Inputs

- AnnData (.h5ad) preprocessed with velocyto and scvelo.pp.moments, containing 'Ms' and 'Mu' layers
- Organism-specific candidate cycling gene list (e.g. go_annotation/GO_cell_cycle_annotation_mouse.txt from GOterm cell_cycle)
- Average-expression threshold for gene inclusion (unspliced/spliced, suggested ~1.0; 0.5 used in the mESC run)

## Outputs

- ./hotelling/ directory listing genes showing more than one state of expression (at least two distinct maxima in unspliced-spliced space)
- A base_gene candidate suitable as initial condition for DeepCycle's transcriptional phase

## How to apply

Start from an AnnData preprocessed with velocyto and scvelo.pp.moments (imputed Ms and Mu layers), restricted to candidate cycling genes from the organism-specific GOterm cell_cycle list passing the average-expression threshold (~1.0, or 0.5 in the mESC example command, depending on sequencing depth). Run DeepCycle with the --hotelling flag, which fits a two-component bivariate Gaussian mixture to each candidate gene's (spliced, unspliced) read distribution (implemented with scipy/numpy) and applies the Hotelling T2 significance test to decide whether the two fitted Gaussian components are statistically distinguishable. Genes whose mixture exhibits at least two distinct maxima — i.e. more than one expression state — are written to the ./hotelling/ output subdirectory. A good strategy is to first run DeepCycle with a random gene from the gene_list file together with the hotelling flag to detect a cycling gene to use as the base_gene initial condition for the transcriptional phase (e.g. Nusap1 for mESC, MELK for human fibroblast leiden cluster 0). Validation compares the reconstructed gene list against the n=158 cycling genes in Supplementary Data 1 of the paper, which includes cycling genes not yet annotated in GOterm cell_cycle.

## Related tools

- **DeepCycle** (Provides the --hotelling flag and ./hotelling/ output directory that implement the Gaussian-mixture/Hotelling gene filter over candidate cycling genes) — https://github.com/andreariba/DeepCycle
- **scVelo** (Produces the imputed 'Ms' and 'Mu' moments layers that supply the (spliced, unspliced) read distributions fed to the filter)
- **scipy** (Statistical library (tested on 1.5.2) used for the Hotelling T2 significance test and Gaussian-mixture fitting)
- **numpy** (Numerical backend (tested on 1.19.1) for fitting the bivariate Gaussian mixtures to spliced/unspliced reads)

## Examples

```
python DeepCycle.py --input_adata adata_mESC.h5ad --gene_list go_annotation/GO_cell_cycle_annotation_mouse.txt --base_gene Nusap1 --expression_threshold 0.5 --gpu --hotelling --output_adata adata_mESC_DeepCycle.h5ad
```

## Evaluation signals

- The ./hotelling/ directory is populated with genes exhibiting more than one state of expression; an empty or missing directory means the filter ran without selections
- The selected gene list agrees in membership and count with the n=158 cycling genes reported in Supplementary Data 1 of the paper
- Retained genes include cycling genes not already present in the GOterm cell_cycle annotation, indicating the filter recovers novel cycling genes
- The chosen base gene (e.g. Nusap1 for mESC, MELK for human fibroblasts) yields coherent cycling patterns and correct unspliced-spliced fits in ./fits/
- Every retained gene passed the average-expression threshold (suggested ~1.0; 0.5 in the mESC command) before mixture fitting

## Limitations

- If you do not find any cycling genes the results will not be reliable — sequencing depth may need to be increased to capture more RNAs per cell
- The expression threshold is depth-dependent; a fixed cutoff can mis-select genes across datasets with different sequencing depths
- The filter is optional; without a validated base gene and cycling-gene set, DeepCycle phase inference may be unreliable

## Evidence

- [methods] Hotelling filter as optional gene-selection step: "decide to run it or not with the hotelling filter on the list of genes."
- [methods] Hotelling flag semantics: "--hotelling [HOTELLING] Use Hotelling filter."
- [readme] README output description: "./hotelling/ [only with the option --hotelling]: the genes showing more than one state of expression."
- [readme] README strategy for picking a base gene: "A good strategy to detect a cycling gene is to first run DeepCycle with a random gene from the gene_list file together with the hotelling flag."
- [readme] Expression threshold guidance: "a minimum value of average expression for a gene. Suggested value around 1.0, depending on your sequencing depth per cell."
