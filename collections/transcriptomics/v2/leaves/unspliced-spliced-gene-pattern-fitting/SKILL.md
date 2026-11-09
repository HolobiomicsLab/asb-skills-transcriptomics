---
name: unspliced-spliced-gene-pattern-fitting
description: Use when scRNA-seq data with spliced/unspliced layers (velocyto + scvelo.pp.moments)
  shows cycling patterns in unspliced-spliced space of cell-cycle genes and you need
  a high-resolution continuous cell cycle phase per cell.
when_to_use_negative:
- When no cycling genes are found after running DeepCycle — the results will not be
  reliable, likely because sequencing depth (RNAs per cell) is insufficient
- When the input Anndata lacks spliced/unspliced layers or has not been preprocessed
  with scvelo.pp.moments
- When the goal is cell-cycle phase labeling based purely on gene marker expression
  — the paper notes marker-based phase definitions lack generalizability, whereas
  this skill relies on dynamical unspliced-spliced features
edam_operation: http://edamontology.org/operation_3927
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_0602
tools:
- name: DeepCycle
  role: TensorFlow autoencoder that fits unspliced-spliced cycling patterns and assigns
    the continuous cell_cycle_theta phase to each cell
  repo: https://github.com/andreariba/DeepCycle
- name: scVelo
  role: Provides the pp.moments preprocessing step generating imputed Mu (unspliced)
    and Ms (spliced) layers required as input
  repo: https://scvelo.readthedocs.io/
- name: velocyto
  role: Produces the spliced/unspliced count layers in the input Anndata
- name: TensorFlow
  role: Backend for the DeepCycle autoencoder model, training with Adam on MSE with
    EarlyStopping and ReduceLROnPlateau callbacks
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1038/s41467-022-30545-8
    title: Cell cycle gene regulation dynamics revealed by RNA velocity and deep-learning
  source_content_hash: sha256:c38e705a319b523b1ba86619e1fd5ef6235d4663d64e5c26c88eacd1bb41da13
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
  edam_operation: http://edamontology.org/operation_3927
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_0602
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.75
n_claims: 8
---

# unspliced-spliced-gene-pattern-fitting

## Summary

Fits circular unspliced-spliced (Mu/Ms) expression patterns of cycling genes with DeepCycle, a TensorFlow autoencoder built on RNA velocity, to assign each cell a continuous cell-cycle angle (cell_cycle_theta) and build a high-resolution map of the cell cycle transcriptome.

## When to use

Use when scRNA-seq data with spliced/unspliced layers (velocyto + scvelo.pp.moments) shows cycling patterns in unspliced-spliced space of cell-cycle genes and you need a high-resolution continuous cell cycle phase per cell.

## When NOT to use

- When no cycling genes are found after running DeepCycle — the results will not be reliable, likely because sequencing depth (RNAs per cell) is insufficient
- When the input Anndata lacks spliced/unspliced layers or has not been preprocessed with scvelo.pp.moments
- When the goal is cell-cycle phase labeling based purely on gene marker expression — the paper notes marker-based phase definitions lack generalizability, whereas this skill relies on dynamical unspliced-spliced features

## Inputs

- Anndata (.h5ad) preprocessed with velocyto and scvelo.pp.moments (layers 'Mu' and 'Ms')
- List of candidate cycling genes for the organism of interest (e.g. GOterm cell_cycle annotation file)
- Base gene name used as initial condition for the transcriptional phase (e.g. MELK for human fibroblasts, Nusap1 for mESCs)
- Expression threshold value (suggested around 1.0, depending on sequencing depth per cell)

## Outputs

- Annotated Anndata with adata.obs['cell_cycle_theta'] (circular cell cycle phase variable)
- ./fits/ folder with unspliced-spliced pattern plots for checking fit quality
- ./training/ plots of the MSE during training
- ./DeepCycle_model/ TensorFlow model and input_data.npz
- ./hotelling/ folder listing genes with more than one state of expression (with --hotelling)

## How to apply

Preprocess the Anndata with scvelo.pp.moments to obtain imputed Mu and Ms layers, then run DeepCycle.py with an organism-specific cycling-gene list (e.g. GOterm cell_cycle), a base gene as initial condition for the transcriptional phase (MELK for human fibroblasts, Nusap1 for mESCs), an --expression_threshold around 1.0 (adjusted to sequencing depth), and optionally --gpu and --hotelling to keep genes showing at least two expression states. The autoencoder trains in two steps — first the encoder on phases estimated from the input gene (atan2 of z-scored spliced and unspliced reads), then encoder+decoder to reconstruct unspliced-spliced reads — with Adam on MSE, early stopping (patience 20), ReduceLROnPlateau, 17% validation cells, and batches of 5 cells. The output Anndata gains obs['cell_cycle_theta'], the circular variable of the cycling genes. For human fibroblasts, first subset to the leiden cluster 0 subpopulation before running. Optionally run estimate_cell_cycle_transitions.py to detect the G1/S, S/G2 and M/G1 transition thetas.

## Related tools

- **DeepCycle** (TensorFlow autoencoder that fits unspliced-spliced cycling patterns and assigns the continuous cell_cycle_theta phase to each cell) — https://github.com/andreariba/DeepCycle
- **scVelo** (Provides the pp.moments preprocessing step generating imputed Mu (unspliced) and Ms (spliced) layers required as input) — https://scvelo.readthedocs.io/
- **velocyto** (Produces the spliced/unspliced count layers in the input Anndata)
- **TensorFlow** (Backend for the DeepCycle autoencoder model, training with Adam on MSE with EarlyStopping and ReduceLROnPlateau callbacks)

## Examples

```
python DeepCycle.py --input_adata adata_mESC.h5ad --gene_list go_annotation/GO_cell_cycle_annotation_mouse.txt --base_gene Nusap1 --expression_threshold 0.5 --gpu --hotelling --output_adata adata_mESC_DeepCycle.h5ad
```

## Evaluation signals

- Inspect ./fits/ unspliced-spliced pattern plots to confirm the fitting procedure worked properly
- Inspect ./training/ MSE plots to confirm training converged (early stopping on val_loss plateau with patience 20)
- Presence of cycling genes: if no cycling genes are found the results will not be reliable
- Output Anndata contains a well-defined obs['cell_cycle_theta'] circular variable; estimate_cell_cycle_transitions.py requires it plus Ms/Mu layers to detect G1/S, S/G2 and M/G1 thetas
- Downstream self-consistency: G0/G1 transition markers behave as expected (FOXM1 downregulated; MXD4/CDC42EP5/CLIP3/MIR22HG upregulated; ESCO2/MCM10/MYBL2/NUF2 downregulated in fibroblasts)

## Limitations

- If you do not find any cycling genes the results will not be reliable; it may be necessary to sequence deeper and increase the number of RNAs per cell
- The method is untested outside the reported stack (Python 3.7.9, scipy 1.5.2, numpy 1.19.1); preprocessing with scvelo.pp.moments is mandatory in the current version
- Defining cell cycle phases based solely on gene markers lacks generalizability, so phase labels derived from theta require the dynamical fitting rather than marker-only assignment; a VAE extension to model distributions of unspliced-spliced RNAs is envisioned but not implemented
- If cycling patterns are found but the fits are wrong, manual inspection of ./fits/ is needed (the author requests being contacted)

## Evidence

- [methods] In the current version a preprocessing step is required. The moments from scVelo have to be computed, i.e. you need to run pp.moments from scVelo: "In the current version a preprocessing step is required. The moments from scVelo have to be computed, i.e. you need to run pp.moments from scVelo"
- [methods] a minimum value of average expression for a gene. Suggested value around 1.0, depending on your sequencing depth per cell.: "a minimum value of average expression for a gene. Suggested value around 1.0, depending on your sequencing depth per cell."
- [readme] The output Anndata contains a new observation variable adata.obs['cell_cycle_theta'] that represents the circular variable related to the cycling genes.: "The output Anndata contains a new observation variable adata.obs['cell_cycle_theta'] that represents the circular variable related to the cycling genes."
- [supplementary] training the encoder on the phases estimated from the input gene (atan2 of z-scored spliced and unspliced reads): "training the encoder on the phases estimated from the input gene (atan2 of z-scored spliced and unspliced reads)"
- [readme] If you do not find any cycling genes the results will not be reliable. It is possible you have to sequence deeper and increase the number of RNAs per cell.: "If you do not find any cycling genes the results will not be reliable. It is possible you have to sequence deeper and increase the number of RNAs per cell."
