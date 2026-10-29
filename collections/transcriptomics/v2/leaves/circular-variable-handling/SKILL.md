---
name: circular-variable-handling
description: Use when cells carry a circular phase annotation (obs['cell_cycle_theta'])
  from DeepCycle and you must bin, average, or fit quantities (e.g. total UMI counts)
  across the full cell cycle without breaking the 0/2π wrap-around.
when_to_use_negative:
- The dataset has no spliced/unspliced layers and no obs['cell_cycle_theta'] — there
  is no circular variable to handle, and DeepCycle results are unreliable if no cycling
  genes are found
- You need phase labels defined purely by gene markers; the paper notes that defining
  cell cycle phases from gene markers alone lacks generalizability across cellular
  models
- The quantity being analyzed is genuinely linear (e.g. time since experiment start)
  — circular binning over 0–2π would be inappropriate
edam_operation: http://edamontology.org/operation_2945
edam_topics:
- http://edamontology.org/topic_0203
tools:
- name: DeepCycle
  role: Assigns the circular transcriptional phase obs['cell_cycle_theta'] via an
    autoencoder whose Circularize layer maps θ to (cos(θ), sin(θ)); base gene atan2
    gives the initial phase guess
  repo: https://github.com/andreariba/DeepCycle
- name: scVelo
  role: Produces the imputed 'Ms' and 'Mu' moments whose z-scored values feed the
    atan2 circular initialization of the phase
  repo: https://scvelo.readthedocs.io/
- name: numpy
  role: Circular binning of cell_cycle_theta over the full 2π range and trend fitting
    on binned UMI counts
- name: scipy
  role: Curve fitting of the exponential doubling trend to the binned UMI-vs-phase
    profile
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1038/s41467-022-30545-8
    title: Cell cycle gene regulation dynamics revealed by RNA velocity and deep-learning
  source_content_hash: sha256:7163f224988b748edb579b418e96a1874629a8074dc4287eee78a4d3f1022f4f
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
  edam_operation: http://edamontology.org/operation_2945
  edam_topics:
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 15
---

# circular-variable-handling

## Summary

Correctly treat DeepCycle's transcriptional phase (adata.obs['cell_cycle_theta']) as a circular variable on the 0–2π ring rather than a linear quantity, using atan2 initialization, cos/sin encoding, and circular binning across the full cell cycle.

## When to use

Use when cells carry a circular phase annotation (obs['cell_cycle_theta']) from DeepCycle and you must bin, average, or fit quantities (e.g. total UMI counts) across the full cell cycle without breaking the 0/2π wrap-around.

## When NOT to use

- The dataset has no spliced/unspliced layers and no obs['cell_cycle_theta'] — there is no circular variable to handle, and DeepCycle results are unreliable if no cycling genes are found
- You need phase labels defined purely by gene markers; the paper notes that defining cell cycle phases from gene markers alone lacks generalizability across cellular models
- The quantity being analyzed is genuinely linear (e.g. time since experiment start) — circular binning over 0–2π would be inappropriate

## Inputs

- h5ad anndata preprocessed with velocyto and scvelo.pp.moments (layers 'Ms' and 'Mu')
- anndata with obs['cell_cycle_theta'] from a DeepCycle run (e.g. adata_mESC.h5ad from Zenodo 10.5281/zenodo.4719436)
- transition thetas for G1/S, S/G2 and M/G1 from estimate_cell_cycle_transitions.py
- total UMI counts per cell (summed spliced counts)

## Outputs

- Circularly binned UMI-vs-phase profile spanning the full 0–2π cycle
- Exponential doubling fit over the phase segment preceding the mitotic drop
- umi_fold_changes.csv with fold changes per dataset (mESCs, ductal, fibroblasts)

## How to apply

DeepCycle outputs θ as a real number that the decoder transforms into (cos(θ), sin(θ)) via a Circularize layer, so all downstream statistics must respect the circular structure: bin cells circularly by cell_cycle_theta into phase bins spanning the full 0–2π cycle rather than sorting θ linearly. The initial phase guess comes from the atan2 of z-scored spliced and unspliced reads of a base gene (e.g. Nusap1 for mESCs, MELK for fibroblasts), computed on the 'Ms' and 'Mu' layers produced by scvelo.pp.moments. To anchor θ to biological phases, run the estimate_cell_cycle_transitions.py script, which estimates the thetas of the G1/S, S/G2 and M/G1 transitions. For quantitative summaries such as RNA content doubling, sum total UMI counts per cell, average within each circular phase bin, and fit the exponential trend over the phase interval preceding the mitotic drop (peak relative to preceding minimum). Treat values near θ≈0/2π as adjacent, not as extremes of a linear axis, otherwise profiles split at the mitotic boundary will be distorted.

## Related tools

- **DeepCycle** (Assigns the circular transcriptional phase obs['cell_cycle_theta'] via an autoencoder whose Circularize layer maps θ to (cos(θ), sin(θ)); base gene atan2 gives the initial phase guess) — https://github.com/andreariba/DeepCycle
- **scVelo** (Produces the imputed 'Ms' and 'Mu' moments whose z-scored values feed the atan2 circular initialization of the phase) — https://scvelo.readthedocs.io/
- **numpy** (Circular binning of cell_cycle_theta over the full 2π range and trend fitting on binned UMI counts)
- **scipy** (Curve fitting of the exponential doubling trend to the binned UMI-vs-phase profile)

## Examples

```
import numpy as np; from scipy.stats import binned_statistic; bins = np.linspace(0, 2*np.pi, 25); binned_means, _, _ = binned_statistic(adata.obs['cell_cycle_theta'], adata.obs['n_counts'], statistic='mean', bins=bins)
```

## Evaluation signals

- All phase statistics must be computed on bins covering the complete 0–2π cycle; a UMI-vs-phase profile must show a single continuous rise-and-drop with no artificial split at θ≈0/2π
- The decoder side of the model must satisfy the (cos(θ), sin(θ)) unit-circle constraint, so |(cos θ, sin θ)| ≈ 1 for every cell
- The computed UMI fold change across the doubling segment should reproduce ~2.1 (mESCs), ~2.0 (ductal), ~2.1 (fibroblasts) within a small tolerance
- The three transition thetas (G1/S, S/G2, M/G1) estimated by estimate_cell_cycle_transitions.py must fall in the expected order around the cycle
- The base gene used for atan2 initialization must itself show cycling patterns (verified via the Hotelling filter output); if no cycling genes are found the results are unreliable

## Limitations

- If no cycling genes are detected, DeepCycle results (and hence θ) are not reliable; deeper sequencing or more RNAs per cell may be needed
- Defining cell cycle phases in RNA data based solely on gene markers lacks generalizability among cellular models
- For human fibroblasts, DeepCycle must be run on the leiden cluster 0 subpopulation only; running on the full dataset breaks the circular phase assignment

## Evidence

- [readme] The output Anndata contains a new observation variable adata.obs['cell_cycle_theta'] that represents the circular variable related to the cycling genes.: "The output Anndata contains a new observation variable adata.obs['cell_cycle_theta'] that represents the circular variable related to the cycling genes."
- [readme] The real number is the input of the decoder that transforms it in (cos(𝜽), sin(𝜽)) with the layer Circularize.: "The real number is the input of the decoder that transforms it in (cos(𝜽), sin(𝜽)) with the layer Circularize."
- [supplementary] training the encoder on the phases estimated from the input gene (atan2 of z-scored spliced and unspliced reads): "training the encoder on the phases estimated from the input gene (atan2 of z-scored spliced and unspliced reads)"
- [supplementary] The script estimate_cell_cycle_transitions.py in ./scripts/ runs the estimation of the thetas associated to the different phase transitions (G1/S, S/G2 and M/G1).: "The script estimate_cell_cycle_transitions.py in ./scripts/ runs the estimation of the thetas associated to the different phase transitions (G1/S, S/G2 and M/G1)."
