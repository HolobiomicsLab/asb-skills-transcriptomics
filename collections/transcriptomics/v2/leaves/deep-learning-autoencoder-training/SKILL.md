---
name: deep-learning-autoencoder-training
description: Use when a scvelo.pp.moments-preprocessed Anndata (Ms/Mu layers) must
  be assigned a high-resolution circular cell-cycle phase, or when testing whether
  DeepCycle's phase inference depends on training with both spliced and unspliced
  input channels.
when_to_use_negative:
- The input Anndata lacks scVelo moments ('Ms'/'Mu' layers) — DeepCycle requires the
  scvelo.pp.moments preprocessing step before training
- No cycling genes can be found in the dataset at the chosen expression threshold
  — the README states the results will not be reliable and deeper sequencing may be
  needed
- The goal is a discrete marker-gene-based phase assignment rather than DeepCycle's
  continuous circular transcriptional phase from RNA velocity dynamics
edam_topics:
- http://edamontology.org/topic_0203
tools:
- name: DeepCycle
  role: Provides DeepCycle.py, the autoencoder (encoder trained on atan2 of z-scored
    spliced/unspliced reads, then encoder+decoder reconstructing unspliced-spliced
    reads) that outputs cell_cycle_theta; supports the ablation by removing an input
    channel
  repo: https://github.com/andreariba/DeepCycle
- name: scVelo
  role: scvelo.pp.moments generates the imputed 'Mu' and 'Ms' layers that serve as
    the autoencoder input
  repo: https://scvelo.readthedocs.io/
- name: velocyto
  role: Upstream preprocessor of the Anndata input file (spliced/unspliced counts)
    before scvelo moments
- name: TensorFlow
  role: Backend that trains the encoder/decoder with EarlyStopping and ReduceLROnPlateau
    callbacks; the model is saved to ./DeepCycle_model/
- name: scipy
  role: Supports circular comparison (e.g. circular correlation) of cell_cycle_theta
    between ablated runs
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1038/s41467-022-30545-8
    title: Cell cycle gene regulation dynamics revealed by RNA velocity and deep-learning
  source_content_hash: sha256:0efa33fd06faeb0dee0e59b8a9c4c5bbecbde6cfda3ec869e2036c472623dab6
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
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6
n_claims: 10
---

# deep-learning-autoencoder-training

## Summary

Train the DeepCycle TensorFlow autoencoder on imputed spliced (Ms) and unspliced (Mu) RNA layers to assign each cell a continuous circular transcriptional phase (adata.obs['cell_cycle_theta']), including single-channel ablation runs that test whether phase inference requires both inputs.

## When to use

Use when a scvelo.pp.moments-preprocessed Anndata (Ms/Mu layers) must be assigned a high-resolution circular cell-cycle phase, or when testing whether DeepCycle's phase inference depends on training with both spliced and unspliced input channels.

## When NOT to use

- The input Anndata lacks scVelo moments ('Ms'/'Mu' layers) — DeepCycle requires the scvelo.pp.moments preprocessing step before training
- No cycling genes can be found in the dataset at the chosen expression threshold — the README states the results will not be reliable and deeper sequencing may be needed
- The goal is a discrete marker-gene-based phase assignment rather than DeepCycle's continuous circular transcriptional phase from RNA velocity dynamics

## Inputs

- Anndata preprocessed with velocyto and scvelo.pp.moments (e.g. adata_mESC.h5ad from Zenodo doi 10.5281/zenodo.4719436) with 'Ms' and 'Mu' layers
- Gene list of candidate cycling genes (e.g. go_annotation/GO_cell_cycle_annotation_mouse.txt; human cell cycle GO annotation for fibroblasts)
- Base gene name used as initial condition for the transcriptional phase (e.g. Nusap1 for mESC, MELK for human fibroblast leiden cluster 0)
- Expression threshold for gene inclusion (suggested around 1.0)
- CLI options: --gpu, --hotelling, --output_adata

## Outputs

- Output Anndata with adata.obs['cell_cycle_theta'] (circular transcriptional phase per cell)
- ./fits/: unspliced-spliced cycling pattern fits
- ./training/: plots of the MSE during training
- ./DeepCycle_model/: TensorFlow model
- input_data.npz: the input data used to train the model
- ./hotelling/: genes showing more than one state of expression (with --hotelling)

## How to apply

Preprocess the Anndata (e.g. adata_mESC.h5ad from Zenodo) with scvelo.pp.moments so imputed 'Mu' and 'Ms' layers exist, then run DeepCycle.py with a GOterm cell_cycle gene list, a cycling base gene as initial phase condition (Nusap1 for mESC, MELK for fibroblast leiden cluster 0), and an expression_threshold around 1.0 (the README mESC command uses 0.5), optionally with --gpu and --hotelling. Training proceeds in two steps: first the encoder is trained on the phase estimated from the input gene (atan2 of z-scored spliced and unspliced reads), then encoder+decoder are trained to reconstruct the unspliced-spliced reads, using Adam on the MSE with EarlyStopping (patience 20, restore_best_weights), ReduceLROnPlateau (factor 0.8, min_lr 0.00001), 17% validation cells and batches of 5. For the ablation, rerun the identical training with the spliced channel removed from the autoencoder input, then with the unspliced channel removed, and extract adata.obs['cell_cycle_theta'] from both outputs. Correct application is judged by completed MSE training plots in ./training/, plausible unspliced-spliced cycling fits in ./fits/, and the two ablated runs showing mutually inconsistent phases (circular correlation/scatter), reproducing Supplementary Fig. S8.

## Related tools

- **DeepCycle** (Provides DeepCycle.py, the autoencoder (encoder trained on atan2 of z-scored spliced/unspliced reads, then encoder+decoder reconstructing unspliced-spliced reads) that outputs cell_cycle_theta; supports the ablation by removing an input channel) — https://github.com/andreariba/DeepCycle
- **scVelo** (scvelo.pp.moments generates the imputed 'Mu' and 'Ms' layers that serve as the autoencoder input) — https://scvelo.readthedocs.io/
- **velocyto** (Upstream preprocessor of the Anndata input file (spliced/unspliced counts) before scvelo moments)
- **TensorFlow** (Backend that trains the encoder/decoder with EarlyStopping and ReduceLROnPlateau callbacks; the model is saved to ./DeepCycle_model/)
- **scipy** (Supports circular comparison (e.g. circular correlation) of cell_cycle_theta between ablated runs)

## Examples

```
python DeepCycle.py --input_adata adata_mESC.h5ad --gene_list go_annotation/GO_cell_cycle_annotation_mouse.txt --base_gene Nusap1 --expression_threshold 0.5 --gpu --hotelling --output_adata adata_mESC_DeepCycle.h5ad
```

## Evaluation signals

- Both training steps terminate via EarlyStopping(monitor='val_loss', patience=20, restore_best_weights=True) and MSE training plots are generated in ./training/
- Unspliced-spliced fit plots in ./fits/ show the expected cycling patterns (wrong fits indicate a failed run)
- Output Anndata contains the circular variable adata.obs['cell_cycle_theta'] for every cell
- In the ablation, both single-channel runs complete, and the extracted thetas from the spliced-removed and unspliced-removed runs are mutually inconsistent (circular correlation/scatter), reproducing Supplementary Fig. S8
- Training used 17% of cells as validation set, batches of 5 cells, and Adam on the MSE, matching the documented setup

## Limitations

- If no cycling genes are found the results will not be reliable; sequencing deeper and increasing RNAs per cell may be necessary
- In the current version a preprocessing step (scvelo.pp.moments) is mandatory before training
- The envisioned extension of DeepCycle as a Variational Autoencoder (VAE) to model distributions of unspliced-spliced RNAs is not yet implemented
- Removing either the spliced or the unspliced channel from the input makes the inferred transcriptional phase inconsistent, so single-channel runs are not individually trustworthy

## Evidence

- [supplementary] training the encoder on the phases estimated from the input gene (atan2 of z-scored spliced and unspliced reads): "training the encoder on the phases estimated from the input gene (atan2 of z-scored spliced and unspliced reads)"
- [readme] The optimization has been performed with Adam on the Mean Squared Error (MSE) between the input and the output.: "The optimization has been performed with Adam on the Mean Squared Error (MSE) between the input and the output."
- [methods] a minimum value of average expression for a gene. Suggested value around 1.0: "a minimum value of average expression for a gene. Suggested value around 1.0, depending on your sequencing depth per cell."
- [readme] adata.obs['cell_cycle_theta'] that represents the circular variable related to the cycling genes: "adata.obs['cell_cycle_theta'] that represents the circular variable related to the cycling genes"
- [supplementary] the layers 'Ms' and 'Mu' from scVelo: "the layers 'Ms' and 'Mu' from scVelo"
- [readme] If you do not find any cycling genes the results will not be reliable.: "If you do not find any cycling genes the results will not be reliable."
