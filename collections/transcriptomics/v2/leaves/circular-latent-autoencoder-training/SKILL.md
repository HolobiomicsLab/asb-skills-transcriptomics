---
name: circular-latent-autoencoder-training
description: Use when an scRNA-seq dataset preprocessed with scvelo.pp.moments (Ms/Mu
  layers) should be given a continuous circular cell-cycle phase per cell, rather
  than a discrete marker-based G1/S/G2M label.
when_to_use_negative:
- The dataset has no spliced/unspliced layers and cannot be preprocessed with scvelo.pp.moments
  — DeepCycle requires the RNA velocity moments as input.
- Few or no genes pass the expression threshold and hotelling filter, indicating no
  detectable cycling genes — the README states the results will not be reliable and
  deeper sequencing may be needed.
- The goal is a discrete marker-based cell-cycle phase call rather than a continuous
  circular phase; the paper notes marker-only phase definitions lack generalizability
  across cellular models.
edam_topics:
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_3934
tools:
- name: DeepCycle
  role: Autoencoder with a single circular latent variable trained in two stages on
    unspliced-spliced reads of cell-cycle genes to output per-cell theta
  repo: https://github.com/andreariba/DeepCycle
- name: scVelo
  role: Provides the required pp.moments preprocessing that generates the imputed
    'Ms' and 'Mu' input layers
  repo: https://scvelo.readthedocs.io/
- name: velocyto
  role: Upstream spliced/unspliced annotation of the anndata used as DeepCycle input
- name: estimate_cell_cycle_transitions.py
  role: Downstream script in ./scripts/ estimating thetas of the G1/S, S/G2 and M/G1
    phase transitions from the fitted model output
  repo: https://github.com/andreariba/DeepCycle
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1038/s41467-022-30545-8
    title: Cell cycle gene regulation dynamics revealed by RNA velocity and deep-learning
  source_content_hash: sha256:9b2259d24ce6e6deae8e0f7acf54d91482d602b0209a1114ed5a94756a8dd93a
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/andreariba/DeepCycle
  tools:
  - DeepCycle
  - Python
  - estimate_cell_cycle_transitions.py
  - scVelo
  - velocyto
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3308
  - http://edamontology.org/topic_3934
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8571428571428571
n_claims: 14
---

# circular-latent-autoencoder-training

## Summary

Two-stage autoencoder training (DeepCycle) that maps imputed unspliced-spliced reads (scVelo Ms/Mu layers) of cell-cycle genes onto a single circular latent variable theta, transformed as (cos(theta), sin(theta)) in the decoder, to assign each single cell a continuous, high-resolution transcriptional cell-cycle phase.

## When to use

Use when an scRNA-seq dataset preprocessed with scvelo.pp.moments (Ms/Mu layers) should be given a continuous circular cell-cycle phase per cell, rather than a discrete marker-based G1/S/G2M label.

## When NOT to use

- The dataset has no spliced/unspliced layers and cannot be preprocessed with scvelo.pp.moments — DeepCycle requires the RNA velocity moments as input.
- Few or no genes pass the expression threshold and hotelling filter, indicating no detectable cycling genes — the README states the results will not be reliable and deeper sequencing may be needed.
- The goal is a discrete marker-based cell-cycle phase call rather than a continuous circular phase; the paper notes marker-only phase definitions lack generalizability across cellular models.

## Inputs

- Anndata file preprocessed with velocyto and scvelo (moments), containing 'Ms' and 'Mu' layers (e.g. adata_mESC.h5ad from Zenodo 10.5281/zenodo.4719436)
- List of candidate cycling genes for the organism of interest (e.g. GOterm cell_cycle annotation file such as go_annotation/GO_cell_cycle_annotation_mouse.txt)
- Base gene name used as initial condition for the transcriptional phase (e.g. Nusap1, MELK)
- Minimum average unspliced/spliced expression threshold (suggested ~1.0; 0.5 in the mESC example)
- Output anndata filename

## Outputs

- Output anndata with adata.obs['cell_cycle_theta'] holding the circular transcriptional phase per cell
- ./fits/ folder with unspliced-spliced pattern fits for visual quality control
- ./training/ plots of the MSE during training
- ./DeepCycle_model/ TensorFlow model
- input_data.npz with the training input data
- ./hotelling/ folder of genes showing more than one state of expression (only with --hotelling)

## How to apply

Preprocess the input anndata with scvelo.pp.moments to generate imputed unspliced (Mu) and spliced (Ms) expression layers, then load the organism's cycling-gene list (e.g. GOterm cell_cycle GO:0007049) and pick a base gene to seed the initial phase (Nusap1 for mESCs, MELK for human fibroblasts; for fibroblasts restrict to the leiden cluster 0 subpopulation). Filter genes by a minimum average unspliced/spliced expression around 1.0 (0.5 in the mESC command example), adjusted to sequencing depth per cell, and optionally enable the --hotelling filter that selects genes with more than one expression state. Train in two stages with Adam on the MSE: first the encoder on per-cell phases given by atan2 of z-scored spliced and unspliced reads of the base gene, then encoder+decoder reconstructing the unspliced-spliced reads (early stopping with patience 20, ReduceLROnPlateau factor 0.8 with min_lr 0.00001, 17% validation cells, batch size 5). The encoder concatenates the atan2 of the base gene with dense-layer features (layers sized 4x the number of genes, leaky ReLU; input/output layers 2x) and outputs a real theta that the decoder's Circularize layer converts to (cos(theta), sin(theta)). Save the anndata with the new observation variable adata.obs['cell_cycle_theta'] and inspect ./fits/, ./training/ MSE plots and ./DeepCycle_model/ to confirm the fit; if no cycling genes are found or fits look wrong, rerun with a different base gene or deeper data.

## Related tools

- **DeepCycle** (Autoencoder with a single circular latent variable trained in two stages on unspliced-spliced reads of cell-cycle genes to output per-cell theta) — https://github.com/andreariba/DeepCycle
- **scVelo** (Provides the required pp.moments preprocessing that generates the imputed 'Ms' and 'Mu' input layers) — https://scvelo.readthedocs.io/
- **velocyto** (Upstream spliced/unspliced annotation of the anndata used as DeepCycle input)
- **estimate_cell_cycle_transitions.py** (Downstream script in ./scripts/ estimating thetas of the G1/S, S/G2 and M/G1 phase transitions from the fitted model output) — https://github.com/andreariba/DeepCycle

## Examples

```
python DeepCycle.py --input_adata adata_mESC.h5ad --gene_list go_annotation/GO_cell_cycle_annotation_mouse.txt --base_gene Nusap1 --expression_threshold 0.5 --gpu --hotelling --output_adata adata_mESC_DeepCycle.h5ad
```

## Evaluation signals

- Output anndata contains a finite, circularly distributed adata.obs['cell_cycle_theta'] for cells of the analysed (sub)population.
- ./fits/ unspliced-spliced patterns visually follow the fitted cycling curves for the selected genes.
- ./training/ MSE plots show plateaued loss with EarlyStopping(patieance=20, restore_best_weights=True) and ReduceLROnPlateau(factor=0.8, min_lr=0.00001) converging rather than diverging.
- estimate_cell_cycle_transitions.py yields consistent theta values for the G1/S, S/G2 and M/G1 transitions from anndata containing obs['cell_cycle_theta'] plus Ms/Mu layers.
- Recomputed phases reproduce the annotated phases in the Zenodo mESC/human-fibroblast h5ad files (10.5281/zenodo.4719436).

## Limitations

- If no cycling genes are found the results are unreliable; deeper sequencing (more RNAs per cell) or a different base gene may be required.
- A mandatory preprocessing step with scvelo.pp.moments is required; the method cannot run directly on raw counts.
- The human fibroblast result requires running on the leiden cluster 0 subpopulation and switching to the human GO cell-cycle gene list and MELK as base gene.
- Constant-rate RNA velocity assumptions are relaxed here, and a Variational Autoencoder extension for modelling distributions over unspliced-spliced inputs is envisioned but not implemented.
- theta is a circular variable that must be further mapped to cell-cycle stages (e.g. via estimate_cell_cycle_transitions.py); the angle alone is not a stage label.

## Evidence

- [readme] DeepCycle is a method that assign an angle (transcritional phase) to each cell in your dataset, that can be associated to the cell cycle stage with further analysis.: "DeepCycle is a method that assign an angle (transcritional phase) to each cell in your dataset, that can be associated to the cell cycle stage with further analysis."
- [methods] a minimum value of average expression for a gene. Suggested value around 1.0, depending on your sequencing depth per cell.: "a minimum value of average expression for a gene. Suggested value around 1.0, depending on your sequencing depth per cell."
- [supplementary] training the encoder on the phases estimated from the input gene (atan2 of z-scored spliced and unspliced reads): "training the encoder on the phases estimated from the input gene (atan2 of z-scored spliced and unspliced reads)"
- [readme] The output Anndata contains a new observation variable adata.obs['cell_cycle_theta'] that represents the circular variable related to the cycling genes.: "The output Anndata contains a new observation variable adata.obs['cell_cycle_theta'] that represents the circular variable related to the cycling genes."
- [readme] If you do not find any cycling genes the results will not be reliable. It is possible you have to sequence deeper and increase the number of RNAs per cell.: "If you do not find any cycling genes the results will not be reliable. It is possible you have to sequence deeper and increase the number of RNAs per cell."
