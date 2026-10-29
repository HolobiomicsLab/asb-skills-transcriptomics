---
name: cross-dataset-reproducibility-analysis
description: Use when you have scRNA-seq h5ad files annotated with adata.obs['cell_cycle_theta']
  from a DeepCycle run and need to verify cell cycle annotation or the RNA-doubling
  prior by quantifying UMI-vs-phase fold change across multiple cell systems.
when_to_use_negative:
- The anndata lacks obs['cell_cycle_theta'] or scVelo Ms/Mu layers, i.e. it was not
  preprocessed with velocyto/scvelo.pp.moments and run through DeepCycle — phase binning
  is undefined.
- The population contains no cycling genes/patterns; the README warns results 'will
  not be reliable' unless cycling patterns exist (may need deeper sequencing and more
  RNAs per cell).
- The question concerns gene-level cycling dynamics or phase-transition estimation
  (G1/S, S/G2, M/G1) rather than global RNA-content doubling.
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: DeepCycle
  role: Assigns the continuous circular transcriptional phase obs['cell_cycle_theta']
    that this skill bins cells by; run on h5ad preprocessed with scVelo moments (base
    gene Nusap1 for mESC, MELK for fibroblasts, expression_threshold 0.5).
  repo: https://github.com/andreariba/DeepCycle
- name: scVelo
  role: 'Preprocessing dependency: scvelo.pp.moments generates the imputed unspliced
    (Mu) and spliced (Ms) layers required in the input anndata.'
  repo: https://scvelo.readthedocs.io/
- name: numpy
  role: Circular binning of cell_cycle_theta and exponential trend fitting for the
    UMI-vs-phase profile.
- name: scipy
  role: Curve fitting of the exponential doubling segment preceding the mitotic drop
    in the binned UMI counts.
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1038/s41467-022-30545-8
    title: Cell cycle gene regulation dynamics revealed by RNA velocity and deep-learning
  source_content_hash: sha256:3d0324d26a1f4caad079691bf7c0645fd9be1305bacee0bdb401cce0a233883a
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/andreariba/DeepCycle
  tools:
  - DeepCycle
  - Python
  - numpy
  - scVelo
  - scipy
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7
n_claims: 10
---

# cross-dataset-reproducibility-analysis

## Summary

Quantitative cross-dataset validation that total RNA (UMI) content per cell approximately doubles across the transcriptional phase before mitosis, using DeepCycle phase-annotated scRNA-seq data from mESCs, ductal cell progenitors, and human fibroblasts (expected fold changes ~2.1, 2.0, 2.1).

## When to use

Use when you have scRNA-seq h5ad files annotated with adata.obs['cell_cycle_theta'] from a DeepCycle run and need to verify cell cycle annotation or the RNA-doubling prior by quantifying UMI-vs-phase fold change across multiple cell systems.

## When NOT to use

- The anndata lacks obs['cell_cycle_theta'] or scVelo Ms/Mu layers, i.e. it was not preprocessed with velocyto/scvelo.pp.moments and run through DeepCycle — phase binning is undefined.
- The population contains no cycling genes/patterns; the README warns results 'will not be reliable' unless cycling patterns exist (may need deeper sequencing and more RNAs per cell).
- The question concerns gene-level cycling dynamics or phase-transition estimation (G1/S, S/G2, M/G1) rather than global RNA-content doubling.

## Inputs

- DeepCycle-annotated h5ad files with adata.obs['cell_cycle_theta'] and scVelo 'Ms'/'Mu' layers (e.g. adata_mESC.h5ad and human fibroblast h5ad from Zenodo 10.5281/zenodo.4719436)
- Public ductal cell progenitor scRNA-seq dataset annotated with cell_cycle_theta
- Cell-cycle GO gene list per organism (e.g. GO_cell_cycle_annotation_mouse.txt) if phase annotation must be regenerated

## Outputs

- umi_fold_changes.csv with one fold change per dataset (expected ~2.1 mESCs, ~2.0 ductal, ~2.1 fibroblasts)
- Binned UMI-vs-phase profile (average total UMI counts per circular phase bin)

## How to apply

Download the phase-annotated h5ad files (mESCs and human fibroblasts from Zenodo DOI 10.5281/zenodo.4719436; ductal cell progenitors from the public dataset), each carrying adata.obs['cell_cycle_theta']. Bin cells circularly by cell_cycle_theta into phase bins spanning the full 2*pi cycle, sum total UMI counts per cell, and average within each phase bin to get the UMI-vs-phase profile. Fit an exponential doubling trend with numpy/scipy curve fitting over the phase interval preceding the mitotic drop, then compute the fold change as the peak preceding mitosis relative to the preceding minimum. Repeat per dataset and write the three fold changes to umi_fold_changes.csv; the annotation itself assumes scVelo moments (Ms/Mu) were precomputed via scvelo.pp.moments before DeepCycle ran.

## Related tools

- **DeepCycle** (Assigns the continuous circular transcriptional phase obs['cell_cycle_theta'] that this skill bins cells by; run on h5ad preprocessed with scVelo moments (base gene Nusap1 for mESC, MELK for fibroblasts, expression_threshold 0.5).) — https://github.com/andreariba/DeepCycle
- **scVelo** (Preprocessing dependency: scvelo.pp.moments generates the imputed unspliced (Mu) and spliced (Ms) layers required in the input anndata.) — https://scvelo.readthedocs.io/
- **numpy** (Circular binning of cell_cycle_theta and exponential trend fitting for the UMI-vs-phase profile.)
- **scipy** (Curve fitting of the exponential doubling segment preceding the mitotic drop in the binned UMI counts.)

## Examples

```
python DeepCycle.py --input_adata adata_mESC.h5ad --gene_list go_annotation/GO_cell_cycle_annotation_mouse.txt --base_gene Nusap1 --expression_threshold 0.5 --gpu --hotelling --output_adata adata_mESC_DeepCycle.h5ad
```

## Evaluation signals

- Computed fold change per dataset matches the reported values within tolerance: 2.1 (mESCs), 2.0 (ductal), 2.1 (fibroblasts).
- The UMI-vs-phase profile shows a monotonic increase up to a peak preceding the mitotic drop across all three datasets.
- Circular binning covers the full 2*pi range of cell_cycle_theta with cells in each bin (no gap at the phase wraparound).
- umi_fold_changes.csv contains exactly one fold-change value per dataset and no NaNs in the fitted doubling segment.
- Input h5ad files contain obs['cell_cycle_theta'] and the Ms/Mu layers, confirming they came from a valid scVelo+DeepCycle run.

## Limitations

- If no cycling genes are found in the dataset, the DeepCycle phase annotation and hence the fold-change estimate are unreliable; deeper sequencing (more RNAs per cell) may be required.
- The expression threshold for gene inclusion must be tuned to sequencing depth (suggested around 1.0, 0.5 for the mESC rerun), affecting the quality of the theta annotation.
- Fold change depends on the binning resolution and on correctly identifying the doubling segment before the mitotic drop; a wrong phase-transitions estimate (G1/S, S/G2, M/G1) shifts the segment boundaries.

## Evidence

- [readme] The output Anndata contains a new observation variable adata.obs['cell_cycle_theta'] that represents the circular variable related to the cycling genes.: "The output Anndata contains a new observation variable adata.obs['cell_cycle_theta'] that represents the circular variable related to the cycling genes."
- [supplementary] h5ad files for mESCs and human fibroblasts already annotated with the cell cycle phase can be downloaded from Zenodo: "h5ad files for mESCs and human fibroblasts already annotated with the cell cycle phase can be downloaded from Zenodo"
- [readme] If you do not find any cycling genes the results will not be reliable.: "If you do not find any cycling genes the results will not be reliable. It is possible you have to sequence deeper and increase the number of RNAs per cell."
- [methods] In the current version a preprocessing step is required. The moments from scVelo have to be computed, i.e. you need to run pp.moments from scVelo: "In the current version a preprocessing step is required. The moments from scVelo have to be computed, i.e. you need to run pp.moments from scVelo"
