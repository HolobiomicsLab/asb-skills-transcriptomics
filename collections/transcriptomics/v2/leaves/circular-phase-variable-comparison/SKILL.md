---
name: circular-phase-variable-comparison
description: Use when comparing circular cell-cycle phase assignments from two DeepCycle
  runs (e.g. ablated inputs or replicate fits) to quantify phase consistency or reproduce
  the Supplementary Fig. S8 inconsistency.
when_to_use_negative:
- The quantities to compare are linear (e.g. expression levels) rather than angular
  — a circular embedding or circular statistic is required only for phase angles
- Only a single DeepCycle run exists and there is no ablated or replicate run to compare
  against
- No cycling genes were found in the dataset — per the README, results will not be
  reliable and deeper sequencing is needed rather than a phase comparison
tools:
- name: DeepCycle
  role: Trained twice with one input channel (spliced or unspliced) removed to produce
    ablated cell_cycle_theta outputs for comparison
  repo: https://github.com/andreariba/DeepCycle
- name: scVelo
  role: Provides the pp.moments imputed 'Ms' and 'Mu' layers used as the autoencoder
    input channels being ablated
  repo: https://scvelo.readthedocs.io/
- name: TensorFlow
  role: Backend in which the DeepCycle autoencoder (encoder+decoder, Circularize layer)
    is trained per ablated configuration
- name: scipy
  role: Used for the circular correlation / scatter comparison of cell_cycle_theta
    between runs
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1038/s41467-022-30545-8
    title: Cell cycle gene regulation dynamics revealed by RNA velocity and deep-learning
  source_content_hash: sha256:a16b26663ac6e281c15b9871b4c0455a94144a00aec592d93e13813f3a972d60
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
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 12
---

# circular-phase-variable-comparison

## Summary

Compares the circular transcriptional phase variable (adata.obs['cell_cycle_theta']) inferred by DeepCycle across two or more runs — e.g. spliced-only vs unspliced-only input ablations — to test whether phase inference depends on both RNA velocity channels.

## When to use

Use when comparing circular cell-cycle phase assignments from two DeepCycle runs (e.g. ablated inputs or replicate fits) to quantify phase consistency or reproduce the Supplementary Fig. S8 inconsistency.

## When NOT to use

- The quantities to compare are linear (e.g. expression levels) rather than angular — a circular embedding or circular statistic is required only for phase angles
- Only a single DeepCycle run exists and there is no ablated or replicate run to compare against
- No cycling genes were found in the dataset — per the README, results will not be reliable and deeper sequencing is needed rather than a phase comparison

## Inputs

- adata_mESC.h5ad (Zenodo doi:10.5281/zenodo.4719436) preprocessed with velocyto and scvelo.pp.moments, with 'Ms' and 'Mu' layers
- GO cell cycle annotation gene list (e.g. go_annotation/GO_cell_cycle_annotation_mouse.txt)
- Base gene for initial phase guess (Nusap1 for mESC; MELK for human fibroblasts)
- expression_threshold (ablation runs: 1.0; README example: 0.5)
- Two output Anndata files from ablated DeepCycle runs, each containing obs['cell_cycle_theta']

## Outputs

- Comparison of inferred transcriptional phases between ablated runs (circular correlation / scatter) reproducing Supplementary Fig. S8
- Per-run artifacts: adata.obs['cell_cycle_theta'], ./fits/ unspliced-spliced fits, ./training/ MSE plots, ./DeepCycle_model/ TensorFlow model, input_data.npz

## How to apply

Load the mESC Anndata (adata_mESC.h5ad from Zenodo) preprocessed with scvelo.pp.moments, using the imputed 'Ms' and 'Mu' layers as the model inputs. Run two DeepCycle trainings on the GOterm cell_cycle gene list with base gene Nusap1 and an expression_threshold of 1.0, removing the spliced channel from the autoencoder input in one run and the unspliced channel in the other (this requires modifying the model input, since DeepCycle.py exposes no ablation flag). Extract the circular variable adata.obs['cell_cycle_theta'] from both output Anndata files. Because theta is an angle, compare runs with circular statistics — e.g. Pearson correlation of cos(2*theta)/sin(2*theta) embeddings or a circular scatter — never a raw Pearson on theta values. Accept the reproduction when both ablated runs complete training (MSE plots in ./training/, unspliced-spliced fits in ./fits/) and the comparison shows the phase inconsistency reported in Supplementary Fig. S8.

## Related tools

- **DeepCycle** (Trained twice with one input channel (spliced or unspliced) removed to produce ablated cell_cycle_theta outputs for comparison) — https://github.com/andreariba/DeepCycle
- **scVelo** (Provides the pp.moments imputed 'Ms' and 'Mu' layers used as the autoencoder input channels being ablated) — https://scvelo.readthedocs.io/
- **TensorFlow** (Backend in which the DeepCycle autoencoder (encoder+decoder, Circularize layer) is trained per ablated configuration)
- **scipy** (Used for the circular correlation / scatter comparison of cell_cycle_theta between runs)

## Examples

```
from scipy.stats import pearsonr; import numpy as np; r = pearsonr(np.cos(2*adata1.obs['cell_cycle_theta']), np.cos(2*adata2.obs['cell_cycle_theta']))[0]
```

## Evaluation signals

- Both ablated runs complete training with early stopping, generating MSE plots in ./training/ and unspliced-spliced fits in ./fits/
- Each output Anndata contains adata.obs['cell_cycle_theta']
- input_data.npz for each run confirms only the intended channel (spliced or unspliced) was removed from the input
- The circular correlation/scatter comparison shows phase inconsistency between the single-channel runs, as in Supplementary Fig. S8
- Phase comparisons are computed on sin/cos (or 2*theta) embeddings of the angle, not raw theta values

## Limitations

- Removing an input channel is not a supported CLI option of DeepCycle.py (flags cover only --gpu, --hotelling, --expression_threshold, etc.), so the ablation requires code modification
- If no cycling genes are found the results will not be reliable; deeper sequencing may be required
- The phase variable is circular, so naive linear correlation on raw theta values is misleading; for human fibroblasts the run must be restricted to the leiden cluster 0 subpopulation with base gene MELK

## Evidence

- [readme] The output Anndata contains a new observation variable adata.obs['cell_cycle_theta'] that represents the circular variable related to the cycling genes.: "The output Anndata contains a new observation variable adata.obs['cell_cycle_theta'] that represents the circular variable related to the cycling genes."
- [supplementary] training the encoder on the phases estimated from the input gene (atan2 of z-scored spliced and unspliced reads): "training the encoder on the phases estimated from the input gene (atan2 of z-scored spliced and unspliced reads)"
- [supplementary] Anndata input file preprocessed with velocyto, scvelo (moments) and DeepCycle.: "Anndata input file preprocessed with velocyto, scvelo (moments) and DeepCycle."
- [readme] the folder contains unspliced-spliced patterns to check if the fitting procedure worked properly: "the folder contains unspliced-spliced patterns to check if the fitting procedure worked properly"
- [intro] a deep learning method to ascribe a continuous high-resolution cell cycle trajectory to single cells based on RNA velocity: "a deep learning method to ascribe a continuous high-resolution cell cycle trajectory to single cells based on RNA velocity"
