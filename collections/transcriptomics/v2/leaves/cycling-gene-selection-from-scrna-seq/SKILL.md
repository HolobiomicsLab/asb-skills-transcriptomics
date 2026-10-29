---
name: cycling-gene-selection-from-scrna-seq
description: Use when preprocessing an AnnData (velocyto + scVelo pp.moments, Ms/Mu
  layers) for DeepCycle and you must select reliable cycling genes from a candidate
  list (e.g. GOterm cell_cycle) before inferring transcriptional phases.
when_to_use_negative:
- If you do not find any cycling genes after filtering, the DeepCycle results will
  not be reliable — the underlying data may need deeper sequencing (more RNAs per
  cell) rather than re-tuning the filter.
- When your input AnnData lacks velocyto/scVelo preprocessing (no imputed Ms and Mu
  layers), since the mixture fitting operates on imputed unspliced-spliced moments.
- For datasets without a candidate cycling gene list for the organism of interest
  (e.g. no GOterm cell_cycle annotation), since the filter only evaluates genes from
  that predefined subset.
edam_operation: http://edamontology.org/operation_3659
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_0203
tools:
- name: DeepCycle
  role: Main pipeline whose --hotelling flag runs the Hotelling filter on the gene
    list; selected genes are initial conditions for transcriptional phase estimation
  repo: https://github.com/andreariba/DeepCycle
- name: scVelo
  role: Provides the pp.moments preprocessing that generates imputed unspliced (Mu)
    and spliced (Ms) expression layers used by the filter
  repo: https://scvelo.readthedocs.io/
- name: velocyto
  role: Preprocessing of the input AnnData with spliced/unspliced read counts prior
    to moment imputation
- name: scipy
  role: Statistical library (tested on 1.5.2) used for the Hotelling T2 test and Gaussian
    mixture fitting
- name: numpy
  role: Numerical library (tested on 1.19.1) supporting the bivariate Gaussian mixture
    fitting
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1038/s41467-022-30545-8
    title: Cell cycle gene regulation dynamics revealed by RNA velocity and deep-learning
  source_content_hash: sha256:31348b3a3955cd7d83d39ea57b9d34877c2da6f4ef16e975ac708e9564420cbc
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/andreariba/DeepCycle
  tools:
  - DeepCycle
  - numpy
  - scVelo
  - scipy
  - velocyto
  edam_operation: http://edamontology.org/operation_3659
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7
n_claims: 10
---

# cycling-gene-selection-from-scRNA-seq

## Summary

Identifies reliably cycling genes from scRNA-seq data by fitting a two-component bivariate Gaussian mixture to each gene's (spliced, unspliced) read distribution and applying a Hotelling T2 test, retaining genes that show more than one expression state. These selected genes serve as reliable inputs for DeepCycle so that the inferred transcriptional phase (cell_cycle_theta) is robust.

## When to use

Use when preprocessing an AnnData (velocyto + scVelo pp.moments, Ms/Mu layers) for DeepCycle and you must select reliable cycling genes from a candidate list (e.g. GOterm cell_cycle) before inferring transcriptional phases.

## When NOT to use

- If you do not find any cycling genes after filtering, the DeepCycle results will not be reliable — the underlying data may need deeper sequencing (more RNAs per cell) rather than re-tuning the filter.
- When your input AnnData lacks velocyto/scVelo preprocessing (no imputed Ms and Mu layers), since the mixture fitting operates on imputed unspliced-spliced moments.
- For datasets without a candidate cycling gene list for the organism of interest (e.g. no GOterm cell_cycle annotation), since the filter only evaluates genes from that predefined subset.

## Inputs

- AnnData (.h5ad) preprocessed with velocyto and scVelo pp.moments, containing imputed 'Ms' and 'Mu' layers
- Organism-specific candidate cycling gene list (e.g. GOterm cell_cycle annotation file, such as go_annotation/GO_cell_cycle_annotation_mouse.txt)
- Base gene used as initial condition for the transcriptional phase (e.g. Nusap1 for mESC, MELK for human fibroblasts)
- Average expression threshold value (suggested around 1.0, depending on sequencing depth per cell)

## Outputs

- ./hotelling/ directory containing the genes showing more than one state of expression (selected cycling genes)
- Filtered gene subset suitable as reliable DeepCycle inputs; for the mESC dataset this corresponds to n = 158 cycling genes reported in Supplementary Data 1

## How to apply

Load an AnnData preprocessed with velocyto and scVelo pp.moments so imputed Ms and Mu layers exist. Restrict the analysis to candidate cycling genes from the organism-specific gene list (e.g. GOterm cell_cycle, such as go_annotation/GO_cell_cycle_annotation_mouse.txt for mESCs) that pass the average-expression threshold (--expression_threshold; suggested around 1.0, though the mESC rerun example uses 0.5, depending on sequencing depth per cell). For each gene, fit a two-component bivariate Gaussian mixture to the (spliced, unspliced) read distribution with scipy/numpy and apply the Hotelling T2 significance test to assess whether the two fitted components are statistically distinguishable. Retain genes whose fitted mixture exhibits at least two distinct maxima, i.e. more than one expression state; the resulting gene list is written to the ./hotelling/ output directory. This is enabled with the --hotelling flag, and the selected genes (plus a base gene such as Nusap1 for mESC or MELK for human fibroblasts) provide the initial condition for DeepCycle's transcriptional phase estimation. The rationale is that cycling genes present multiple maxima in the unspliced-spliced space, so genes with a single expression state are unreliable inputs for phase inference.

## Related tools

- **DeepCycle** (Main pipeline whose --hotelling flag runs the Hotelling filter on the gene list; selected genes are initial conditions for transcriptional phase estimation) — https://github.com/andreariba/DeepCycle
- **scVelo** (Provides the pp.moments preprocessing that generates imputed unspliced (Mu) and spliced (Ms) expression layers used by the filter) — https://scvelo.readthedocs.io/
- **velocyto** (Preprocessing of the input AnnData with spliced/unspliced read counts prior to moment imputation)
- **scipy** (Statistical library (tested on 1.5.2) used for the Hotelling T2 test and Gaussian mixture fitting)
- **numpy** (Numerical library (tested on 1.19.1) supporting the bivariate Gaussian mixture fitting)

## Examples

```
python DeepCycle.py --input_adata adata_mESC.h5ad --gene_list go_annotation/GO_cell_cycle_annotation_mouse.txt --base_gene Nusap1 --expression_threshold 0.5 --gpu --hotelling --output_adata adata_mESC_DeepCycle.h5ad
```

## Evaluation signals

- For the mESC dataset, the reconstructed gene set should agree in membership and count with the n = 158 cycling genes reported in Supplementary Data 1 of the paper.
- The ./hotelling/ output directory must be created (only when --hotelling is set) and must list genes showing more than one state of expression.
- Each retained gene's fitted two-component Gaussian mixture must exhibit at least two distinct maxima in the (spliced, unspliced) space, i.e. the Hotelling T2 test must distinguish the two components.
- All selected genes must pass the average-expression threshold (--expression_threshold, suggested ~1.0) and belong to the organism-specific gene list (e.g. GOterm cell_cycle).
- The output Anndata must contain adata.obs['cell_cycle_theta'] if downstream DeepCycle ran successfully on the filtered gene set; absence of any cycling genes signals unreliable results.

## Limitations

- If no cycling genes are found the DeepCycle results will not be reliable; it may be necessary to sequence deeper and increase the number of RNAs per cell.
- The choice of expression threshold depends on sequencing depth per cell (suggested ~1.0, but 0.5 is used in the mESC example command), so the cutoff must be adapted per dataset.
- If the cycling patterns are found but the fits are wrong, the fitting procedure needs checking (the ./fits/ folder contains unspliced-spliced patterns for this purpose).
- The gene-selection procedure can surface cycling genes not yet included in the GO term:cell_cycle annotation, meaning the candidate list alone is incomplete.

## Evidence

- [readme] Hotelling filter rationale and output location: "The subdirectory DeepCycle/hotelling will show genes selected to show at least two different states of expression."
- [readme] Preprocessing requirement: "The moments from scVelo have to be computed, i.e. you need to run pp.moments from scVelo"
- [readme] Expression threshold guidance: "a minimum value of average expression for a gene. Suggested value around 1.0, depending on your sequencing depth per cell."
- [results] Filter finding and count: [evidence span withheld — anchor status: not_found]
- [readme] Reliability caveat: "If you do not find any cycling genes the results will not be reliable. It is possible you have to sequence deeper and increase the number of RNAs per cell."
