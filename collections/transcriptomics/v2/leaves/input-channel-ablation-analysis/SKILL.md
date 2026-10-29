---
name: input-channel-ablation-analysis
description: Use when you need to test whether DeepCycle's accurate transcriptional
  phase inference depends on training with both the spliced (Ms) and unspliced (Mu)
  scVelo moment layers, by ablating one input channel and comparing the resulting
  cell_cycle_theta values between single-channel runs.
when_to_use_negative:
- You want the full published DeepCycle result on both channels — DeepCycle.py has
  no CLI flag for removing a channel, so this skill requires modifying the autoencoder
  input and is not a drop-in rerun of the README command
- The dataset contains no cycling genes (e.g. insufficient sequencing depth), since
  results will not be reliable regardless of channel configuration
- Your goal is phase-transition (G1/S, S/G2, M/G1) estimation via estimate_cell_cycle_transitions.py
  rather than ablation of the input channels
edam_topics:
- http://edamontology.org/topic_3474
- http://edamontology.org/topic_3512
- http://edamontology.org/topic_0203
tools:
- name: DeepCycle
  role: Autoencoder trained on scVelo moments; each ablation run trains it with one
    RNA channel removed from the autoencoder input
  repo: github.com/andreariba/DeepCycle
- name: scVelo
  role: Preprocessing via pp.moments to produce the imputed 'Ms' (spliced) and 'Mu'
    (unspliced) layers used as model input channels
  repo: https://scvelo.readthedocs.io/
- name: TensorFlow
  role: Backend for the two ablated DeepCycle training runs and their MSE training
    plots
- name: scipy
  role: Numerical routines for circular comparison of cell_cycle_theta between ablated
    runs
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1038/s41467-022-30545-8
    title: Cell cycle gene regulation dynamics revealed by RNA velocity and deep-learning
  source_content_hash: sha256:af8a1c51a5bcd5cacf59a99cb2e4b46d65a3d8f4027054f41799b96ae89d70f7
schema_version: 0.2.0
metadata:
  license_tier: open
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
  - http://edamontology.org/topic_3474
  - http://edamontology.org/topic_3512
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.75
n_claims: 12
---

# input-channel-ablation-analysis

## Summary

Reproduces DeepCycle's input-channel ablation: retrain the autoencoder on the mESC Anndata with either the spliced or the unspliced channel removed from the autoencoder input, then compare the inferred circular phases (adata.obs['cell_cycle_theta']) to show phase inconsistency, as reported in Supplementary Fig. S8.

## When to use

Use when you need to test whether DeepCycle's accurate transcriptional phase inference depends on training with both the spliced (Ms) and unspliced (Mu) scVelo moment layers, by ablating one input channel and comparing the resulting cell_cycle_theta values between single-channel runs.

## When NOT to use

- You want the full published DeepCycle result on both channels — DeepCycle.py has no CLI flag for removing a channel, so this skill requires modifying the autoencoder input and is not a drop-in rerun of the README command
- The dataset contains no cycling genes (e.g. insufficient sequencing depth), since results will not be reliable regardless of channel configuration
- Your goal is phase-transition (G1/S, S/G2, M/G1) estimation via estimate_cell_cycle_transitions.py rather than ablation of the input channels

## Inputs

- adata_mESC.h5ad Anndata preprocessed with scvelo.pp.moments (imputed 'Ms' and 'Mu' layers), from Zenodo doi:10.5281/zenodo.4719436
- GOterm cell_cycle gene list file (e.g. go_annotation/GO_cell_cycle_annotation_mouse.txt)
- Base cycling gene as initial phase condition (e.g. Nusap1 for mESC)
- Expression threshold (e.g. 1.0) for gene inclusion

## Outputs

- Two output Anndata files, each containing adata.obs['cell_cycle_theta'] from the spliced-ablated and unspliced-ablated runs
- MSE training plots in ./training/ and unspliced-spliced fits in ./fits/ for each ablated run
- Circular comparison (correlation/scatter) between the two ablated runs reproducing the phase inconsistency of Supplementary Fig. S8

## How to apply

Load adata_mESC.h5ad from Zenodo (10.5281/zenodo.4719436) preprocessed with scvelo.pp.moments so the imputed 'Ms' and 'Mu' layers are available. Train DeepCycle in Python/TensorFlow on the GOterm cell_cycle gene list with Nusap1 as the cycling base gene and expression_threshold 1.0, first with the spliced channel removed from the autoencoder input, then identically with the unspliced channel removed. Extract the circular variable adata.obs['cell_cycle_theta'] from both output Anndata files and compare the inferred transcriptional phases between the two ablated runs (e.g. circular correlation or scatter), expecting inconsistency as in Supplementary Fig. S8. The rationale is that DeepCycle is trained to reconstruct the unspliced-spliced reads, so removing one channel should degrade phase inference if both channels are required.

## Related tools

- **DeepCycle** (Autoencoder trained on scVelo moments; each ablation run trains it with one RNA channel removed from the autoencoder input) — github.com/andreariba/DeepCycle
- **scVelo** (Preprocessing via pp.moments to produce the imputed 'Ms' (spliced) and 'Mu' (unspliced) layers used as model input channels) — https://scvelo.readthedocs.io/
- **TensorFlow** (Backend for the two ablated DeepCycle training runs and their MSE training plots)
- **scipy** (Numerical routines for circular comparison of cell_cycle_theta between ablated runs)

## Examples

```
python DeepCycle.py --input_adata adata_mESC.h5ad --gene_list go_annotation/GO_cell_cycle_annotation_mouse.txt --base_gene Nusap1 --expression_threshold 1.0 --output_adata adata_mESC_DeepCycle_ablated.h5ad
```

## Evaluation signals

- Both ablated runs complete training, with MSE training plots generated in ./training/ and unspliced-spliced fits in ./fits/
- Each output Anndata contains the new observation variable adata.obs['cell_cycle_theta']
- The circular comparison of cell_cycle_theta between the spliced-ablated and unspliced-ablated runs shows phase inconsistency, matching Supplementary Fig. S8
- input_data.npz in each run directory confirms the ablated channel was actually removed from the training input
- Gene selection uses the same GOterm cell_cycle list and expression_threshold (around 1.0) as the full model so runs are comparable

## Limitations

- If no cycling genes are found (e.g. sequencing too shallow), the results are not reliable and the ablation comparison is meaningless
- The expression threshold (suggested around 1.0) must be adapted to sequencing depth per cell; a wrong threshold changes which genes enter both ablated runs
- Ablation is not exposed as a CLI option of DeepCycle.py, so the autoencoder input must be modified in code; the envisioned Variational Autoencoder extension is not implemented

## Evidence

- [methods] preprocessing with scvelo.pp.moments: "In the current version a preprocessing step is required. The moments from scVelo have to be computed, i.e. you need to run pp.moments from scVelo"
- [supplementary] two-step training reconstructing unspliced-spliced reads: "training encoder+decoder to reconstruct the unspliced-spliced reads"
- [readme] output circular variable: "The output Anndata contains a new observation variable adata.obs['cell_cycle_theta'] that represents the circular variable related to the cycling genes"
- [readme] expression threshold guidance: "a minimum value of average expression for a gene. Suggested value around 1.0, depending on your sequencing depth per cell."
