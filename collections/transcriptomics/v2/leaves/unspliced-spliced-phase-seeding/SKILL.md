---
name: unspliced-spliced-phase-seeding
description: Use when scRNA-seq data preprocessed with scvelo.pp.moments (Ms/Mu layers)
  should be assigned a continuous high-resolution cell-cycle phase per cell, and a
  cycling base gene (e.g. Nusap1 or MELK) is available for phase initialization.
when_to_use_negative:
- When no cycling genes can be found (e.g. shallow sequencing with too few RNAs per
  cell) — the results will not be reliable
- When the dataset lacks unspliced/spliced read layers or has not been preprocessed
  with scvelo.pp.moments
- When the goal is discrete cell-cycle phase labeling from marker gene scores rather
  than a continuous circular transcriptional phase
edam_operation: http://edamontology.org/operation_3891
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_3308
tools:
- name: DeepCycle
  role: Autoencoder with a single circular latent variable theta; trains on atan2-seeded
    phases from the base gene then reconstructs unspliced-spliced reads to assign
    cell_cycle_theta
  repo: https://github.com/andreariba/DeepCycle
- name: scVelo
  role: Provides the imputed 'Ms' and 'Mu' moments layers used as DeepCycle's input
    via scvelo.pp.moments
  repo: https://scvelo.readthedocs.io/
- name: velocyto
  role: Preprocessing of the anndata input file with spliced/unspliced reads prior
    to scvelo moments
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1038/s41467-022-30545-8
    title: Cell cycle gene regulation dynamics revealed by RNA velocity and deep-learning
  source_content_hash: sha256:6b80f62d1da583b665f91a26482bd529f75ced747c44635450b8af3f9d410cbf
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/andreariba/DeepCycle
  tools:
  - DeepCycle
  - Python
  - scVelo
  - velocyto
  edam_operation: http://edamontology.org/operation_3891
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9230769230769231
n_claims: 13
---

# unspliced-spliced-phase-seeding

## Summary

Seeds and infers a continuous circular transcriptional phase (theta) for each single cell by using the atan2 of z-scored spliced and unspliced reads of a cycling base gene as the initial condition for DeepCycle's autoencoder, then training the encoder and decoder to reconstruct the unspliced-spliced patterns of cell-cycle genes.

## When to use

Use when scRNA-seq data preprocessed with scvelo.pp.moments (Ms/Mu layers) should be assigned a continuous high-resolution cell-cycle phase per cell, and a cycling base gene (e.g. Nusap1 or MELK) is available for phase initialization.

## When NOT to use

- When no cycling genes can be found (e.g. shallow sequencing with too few RNAs per cell) — the results will not be reliable
- When the dataset lacks unspliced/spliced read layers or has not been preprocessed with scvelo.pp.moments
- When the goal is discrete cell-cycle phase labeling from marker gene scores rather than a continuous circular transcriptional phase

## Inputs

- anndata (.h5ad) preprocessed with velocyto and scvelo.pp.moments, containing 'Ms' and 'Mu' layers
- gene list of candidate cycling genes (e.g. GOterm cell_cycle GO:0007049 annotation file for the organism)
- base gene name used for the initial phase guess (e.g. Nusap1, MELK)
- average unspliced/spliced expression threshold (suggested ~1.0, 0.5 for mESC run)

## Outputs

- output anndata (.h5ad) with new observation variable adata.obs['cell_cycle_theta']
- ./fits/ folder with unspliced-spliced fitting patterns
- ./DeepCycle_model/ TensorFlow model
- ./training/ MSE plots during training
- input_data.npz of model training inputs
- ./hotelling/ genes showing more than one state of expression (with --hotelling)

## How to apply

Preprocess the anndata with scvelo.pp.moments to obtain imputed spliced (Ms) and unspliced (Mu) layers, then select a cycling base gene (Nusap1 for mESCs, MELK for human fibroblasts; a random gene from the GO cell_cycle gene list plus the --hotelling flag helps detect one). Run DeepCycle in two training stages: first train the encoder on phases estimated by atan2 of z-scored spliced and unspliced reads of the base gene, then train encoder+decoder to reconstruct the unspliced-spliced reads of the candidate cycling genes, filtering genes below an average expression threshold (~0.5-1.0 depending on sequencing depth) and optionally applying the Hotelling filter. The decoder transforms the latent theta into (cos(theta), sin(theta)) via a Circularize layer, enforcing the circular structure. Training uses Adam on MSE with EarlyStopping (patience 20) and ReduceLROnPlateau, 17% validation cells, and batches of 5 cells; save the output anndata containing adata.obs['cell_cycle_theta'].

## Related tools

- **DeepCycle** (Autoencoder with a single circular latent variable theta; trains on atan2-seeded phases from the base gene then reconstructs unspliced-spliced reads to assign cell_cycle_theta) — https://github.com/andreariba/DeepCycle
- **scVelo** (Provides the imputed 'Ms' and 'Mu' moments layers used as DeepCycle's input via scvelo.pp.moments) — https://scvelo.readthedocs.io/
- **velocyto** (Preprocessing of the anndata input file with spliced/unspliced reads prior to scvelo moments)

## Examples

```
python DeepCycle.py --input_adata adata_mESC.h5ad --gene_list go_annotation/GO_cell_cycle_annotation_mouse.txt --base_gene Nusap1 --expression_threshold 0.5 --gpu --hotelling --output_adata adata_mESC_DeepCycle.h5ad
```

## Evaluation signals

- adata.obs['cell_cycle_theta'] is present in the output anndata and behaves as a circular (0-2pi) variable
- Unspliced-spliced patterns in ./fits/ show correct circular fitting; wrong fits indicate failure
- MSE plots in ./training/ decrease and plateau via EarlyStopping, indicating converged training in both stages
- At least some cycling genes are found (with --hotelling, ./hotelling/ shows genes with more than one expression state); finding none means results are unreliable
- For human fibroblasts, verification that DeepCycle was run only on the leiden cluster 0 subpopulation

## Limitations

- If no cycling genes are found the results will not be reliable; deeper sequencing (more RNAs per cell) may be required
- Expression threshold must be adjusted to sequencing depth per cell (suggested around 1.0; the mESC run used 0.5)
- Defining cell cycle phases based solely on gene markers lacks generalizability across cellular models; future methods should rely on dynamical features
- Extending DeepCycle as a Variational Autoencoder to model distributions over inputs is envisioned but not yet implemented

## Evidence

- [supplementary] training the encoder on the phases estimated from the input gene (atan2 of z-scored spliced and unspliced reads): "training the encoder on the phases estimated from the input gene (atan2 of z-scored spliced and unspliced reads)"
- [methods] The moments from scVelo have to be computed: "The moments from scVelo have to be computed, i.e. you need to run pp.moments from scVelo"
- [readme] a gene to use as initial condition for the transcriptional phase: "a gene to use as initial condition for the transcriptional phase. A good strategy to detect a cycling gene is to first run DeepCycle with a random gene from the gene_list file together with the"
- [readme] If you do not find any cycling genes the results will not be reliable: "If you do not find any cycling genes the results will not be reliable. It is possible you have to sequence deeper and increase the number of RNAs per cell."
- [readme] The real number is the input of the decoder that transforms it in (cos(theta), sin(theta)): "The real number is the input of the decoder that transforms it in (cos(𝜽), sin(𝜽)) with the layer Circularize"
