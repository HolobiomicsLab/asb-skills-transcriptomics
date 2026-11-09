---
name: mse-training-convergence-monitoring
description: Use when training the DeepCycle autoencoder on scVelo moments (Mu/Ms)
  of cycling genes and you must confirm the loss plateaued rather than diverged before
  trusting the inferred cell_cycle_theta phases.
when_to_use_negative:
- When the input dataset lacks cycling genes (DeepCycle reports results will not be
  reliable); convergence curves alone cannot rescue this — deeper sequencing is needed
- When you need gene-level fit diagnostics rather than training convergence — inspect
  ./fits/ unspliced-spliced patterns instead
edam_topics:
- http://edamontology.org/topic_3474
tools:
- name: DeepCycle
  role: Autoencoder trained in two stages on unspliced-spliced reads; writes MSE training
    plots and TensorFlow model used for convergence monitoring
  repo: https://github.com/andreariba/DeepCycle
- name: TensorFlow / tf.keras
  role: Provides the Adam MSE optimization, EarlyStopping (patience 20, restore_best_weights)
    and ReduceLROnPlateau (factor 0.8, patience 5, min_lr 0.00001) callbacks whose
    behavior the monitoring validates
- name: scVelo
  role: Generates the imputed Mu/Ms layers that constitute the autoencoder's reconstruction
    target whose MSE is monitored
  repo: https://scvelo.readthedocs.io/
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1038/s41467-022-30545-8
    title: Cell cycle gene regulation dynamics revealed by RNA velocity and deep-learning
  source_content_hash: sha256:84e53ad5675c6983f3c9ccb30d520d4c6a9724046e7aa232a263de4c18e7d77b
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/andreariba/DeepCycle
  tools:
  - DeepCycle
  - Python
  - TensorFlow / tf.keras
  - scVelo
  - velocyto
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3474
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9230769230769231
n_claims: 13
---

# mse-training-convergence-monitoring

## Summary

Verifies that the DeepCycle autoencoder training converged by inspecting the Mean Squared Error (MSE) training/validation plots written to ./training/, alongside the early-stopping and learning-rate-reduction callbacks used during the two-stage (encoder, then encoder+decoder) training on unspliced-spliced reads.

## When to use

Use when training the DeepCycle autoencoder on scVelo moments (Mu/Ms) of cycling genes and you must confirm the loss plateaued rather than diverged before trusting the inferred cell_cycle_theta phases.

## When NOT to use

- When the input dataset lacks cycling genes (DeepCycle reports results will not be reliable); convergence curves alone cannot rescue this — deeper sequencing is needed
- When you need gene-level fit diagnostics rather than training convergence — inspect ./fits/ unspliced-spliced patterns instead

## Inputs

- Trained DeepCycle model run on an anndata preprocessed with scvelo.pp.moments (Ms/Mu layers)
- ./training/ directory with plots of the MSE during training
- ./fits/ directory with unspliced-spliced pattern fits
- input_data.npz (input data used to train the model)
- ./DeepCycle_model/ TensorFlow model directory

## Outputs

- Assessment of training convergence from MSE training/validation curves
- Decision to accept the output anndata's adata.obs['cell_cycle_theta'] or rerun with a different base gene or deeper data

## How to apply

After running DeepCycle, open the MSE plots written to ./training/ and check that the training and validation losses plateau with a small gap, indicating the autoencoder fit the unspliced-spliced reads without overfitting. DeepCycle trains in two steps — first the encoder on atan2 phases of the base gene's z-scored spliced/unspliced reads, then encoder+decoder reconstruction — and each step uses tf.keras EarlyStopping(monitor='val_loss', patience=20, restore_best_weights=True) plus ReduceLROnPlateau(factor=0.8, patience=5, min_lr=0.00001), so a plateau is expected. Training uses Adam optimized on the MSE between input and output, with 17% of cells held out as a validation set and batches of 5 cells, so judge convergence by the val_loss trend relative to training loss. If ./fits/ shows unspliced-spliced patterns that fit poorly or no cycling genes were found, the results are unreliable and should be rerun with a different base gene or deeper sequencing, regardless of how the MSE curves look.

## Related tools

- **DeepCycle** (Autoencoder trained in two stages on unspliced-spliced reads; writes MSE training plots and TensorFlow model used for convergence monitoring) — https://github.com/andreariba/DeepCycle
- **TensorFlow / tf.keras** (Provides the Adam MSE optimization, EarlyStopping (patience 20, restore_best_weights) and ReduceLROnPlateau (factor 0.8, patience 5, min_lr 0.00001) callbacks whose behavior the monitoring validates)
- **scVelo** (Generates the imputed Mu/Ms layers that constitute the autoencoder's reconstruction target whose MSE is monitored) — https://scvelo.readthedocs.io/

## Examples

```
python DeepCycle.py --input_adata adata_mESC.h5ad --gene_list go_annotation/GO_cell_cycle_annotation_mouse.txt --base_gene Nusap1 --expression_threshold 0.5 --gpu --hotelling --output_adata adata_mESC_DeepCycle.h5ad
```

## Evaluation signals

- Both training steps stopped on a val_loss plateau via tf.keras.callbacks.EarlyStopping(monitor='val_loss', patience=20, restore_best_weights=True), visible as flattened MSE curves in ./training/
- Validation MSE tracks training MSE closely (17% of cells are the validation set), indicating no overfitting despite GaussianNoise layers
- MSE curves in ./training/ correspond to a model saved in ./DeepCycle_model/ and fits in ./fits/ that visually match the observed unspliced-spliced cycling patterns
- Output anndata contains adata.obs['cell_cycle_theta'] only when the reconstruction (minimized MSE) succeeded; diverging loss or no cycling genes signals unreliable phases

## Limitations

- If no cycling genes are found, results are unreliable regardless of the MSE convergence; deeper sequencing (more RNAs per cell) may be required
- Early stopping uses val_loss plateau with patience=20 and min_delta=0.0, so a very slow loss improvement can still be cut off or a slow drift accepted
- Convergence of the MSE does not guarantee biologically correct phases — the base gene initial guess and Hotelling filtering affect the fit

## Evidence

- [readme] ./training/: plots of the MSE during the training: "./training/: plots of the MSE during the training"
- [readme] Both training steps have an early stop when they reach a plateau: "Both training steps have an early stop when they reach a plateau"
- [readme] The optimization has been performed with Adam on the Mean Squared Error (MSE) between the input and the output.: "The optimization has been performed with Adam on the Mean Squared Error (MSE) between the input and the output."
- [readme] 17% of the input cells are used as validation set and the training is performed in batches of 5 cells.: "17% of the input cells are used as validation set and the training is performed in batches of 5 cells."
- [readme] If you do not find any cycling genes the results will not be reliable.: "If you do not find any cycling genes the results will not be reliable."
