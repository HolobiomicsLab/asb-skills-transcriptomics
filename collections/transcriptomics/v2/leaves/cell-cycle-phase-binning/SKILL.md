---
name: cell-cycle-phase-binning
description: Use when h5ad data carry a continuous DeepCycle phase annotation (adata.obs['cell_cycle_theta'])
  with Ms/Mu layers, and you need to quantify how per-cell RNA content (total UMIs)
  changes around the cell cycle or locate phase transitions.
when_to_use_negative:
- Input anndata lacks a continuous phase annotation (adata.obs['cell_cycle_theta'])
  — run DeepCycle first (scvelo.pp.moments preprocessing, base gene Nusap1/MELK, expression
  threshold ~0.5–1.0) before binning.
- The population is not asynchronously cycling (e.g. fibroblasts that have exited
  the cycle into G0) — the UMI-vs-phase doubling profile is only meaningful for cells
  traversing G1/S/G2/M.
- DeepCycle found no cycling genes (shallow sequencing, few RNAs per cell) — the README
  states the results will not be reliable in this case, so downstream phase binning
  is unfounded.
edam_operation: http://edamontology.org/operation_3656
edam_topics:
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_0102
tools:
- name: DeepCycle
  role: Produces the continuous circular transcriptional phase (adata.obs['cell_cycle_theta'])
    that serves as the binning coordinate
  repo: https://github.com/andreariba/DeepCycle
- name: scVelo
  role: Preprocessing (pp.moments) generating the imputed 'Ms' and 'Mu' layers required
    in the input anndata
  repo: https://github.com/andreariba/DeepCycle
- name: numpy
  role: Circular binning of cell_cycle_theta into phase bins and averaging UMI counts
    per bin
- name: scipy
  role: Curve fitting of the exponential doubling trend to the binned UMI counts preceding
    the mitotic drop
provenance:
  source_task_ids:
  - task_003
  - task_004
  - task_005
  - task_001
  source_papers:
  - doi: 10.1038/s41467-022-30545-8
    title: Cell cycle gene regulation dynamics revealed by RNA velocity and deep-learning
  source_content_hash: sha256:41d59c1f3242899672dd90d25fcac6f1bafa4e941ecbfc8c7d3e052200225565
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
  edam_operation: http://edamontology.org/operation_3656
  edam_topics:
  - http://edamontology.org/topic_0203
  - http://edamontology.org/topic_0102
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 10
---

# cell-cycle-phase-binning

## Summary

Bins cells circularly by their DeepCycle transcriptional phase (adata.obs['cell_cycle_theta']) across the full 0–2π cycle and summarizes total RNA (UMI) content per phase bin, enabling quantitative verification that RNA counts per cell approximately double (fold change ~2.0–2.1) before mitosis.

## When to use

Use when h5ad data carry a continuous DeepCycle phase annotation (adata.obs['cell_cycle_theta']) with Ms/Mu layers, and you need to quantify how per-cell RNA content (total UMIs) changes around the cell cycle or locate phase transitions.

## When NOT to use

- Input anndata lacks a continuous phase annotation (adata.obs['cell_cycle_theta']) — run DeepCycle first (scvelo.pp.moments preprocessing, base gene Nusap1/MELK, expression threshold ~0.5–1.0) before binning.
- The population is not asynchronously cycling (e.g. fibroblasts that have exited the cycle into G0) — the UMI-vs-phase doubling profile is only meaningful for cells traversing G1/S/G2/M.
- DeepCycle found no cycling genes (shallow sequencing, few RNAs per cell) — the README states the results will not be reliable in this case, so downstream phase binning is unfounded.

## Inputs

- h5ad anndata files annotated with adata.obs['cell_cycle_theta'] from a DeepCycle run (e.g. adata_mESC.h5ad, human fibroblast subpopulation leiden cluster 0)
- Layers 'Ms' and 'Mu' from scvelo.pp.moments present in the anndata
- Total UMI counts per cell (sum over the count matrix)
- Theta values of cell cycle phase transitions (G1/S, S/G2, M/G1) from estimate_cell_cycle_transitions.py

## Outputs

- UMI-vs-phase profile: mean total UMI counts per circular phase bin spanning 0–2π
- Fitted exponential doubling trend over the segment preceding the mitotic drop
- Per-dataset UMI fold change (peak preceding mitosis / preceding minimum), e.g. written to umi_fold_changes.csv

## How to apply

Start from the phase-annotated h5ad files (mESC and human fibroblast from Zenodo 10.5281/zenodo.4719436, or ductal progenitors) that already contain adata.obs['cell_cycle_theta'] from a DeepCycle run. Sum total UMI counts per cell, then bin cells circularly by cell_cycle_theta into bins spanning the full cycle and average UMI content within each bin to obtain the UMI-vs-phase profile. Use the theta values of the phase transitions estimated by scripts/estimate_cell_cycle_transitions.py (G1/S, S/G2, M/G1) to delimit the segment preceding the mitotic drop in UMI content. Fit an exponential doubling trend to this segment with numpy/scipy (e.g. scipy.optimize curve fitting) and compute the fold change of the peak preceding mitosis relative to the preceding minimum. Report the fold change per dataset — expected ~2.1 for mESCs, ~2.0 for ductal cells, ~2.1 for fibroblasts, consistent with a mother cell roughly doubling in volume — e.g. to umi_fold_changes.csv.

## Related tools

- **DeepCycle** (Produces the continuous circular transcriptional phase (adata.obs['cell_cycle_theta']) that serves as the binning coordinate) — https://github.com/andreariba/DeepCycle
- **scVelo** (Preprocessing (pp.moments) generating the imputed 'Ms' and 'Mu' layers required in the input anndata) — https://github.com/andreariba/DeepCycle
- **numpy** (Circular binning of cell_cycle_theta into phase bins and averaging UMI counts per bin)
- **scipy** (Curve fitting of the exponential doubling trend to the binned UMI counts preceding the mitotic drop)

## Examples

```
import numpy as np; bin_id = np.digitize(adata.obs['cell_cycle_theta'], np.linspace(0, 2*np.pi, 49))
```

## Evaluation signals

- Computed UMI fold change across the doubling segment is ~2.1 (mESCs), ~2.0 (ductal), ~2.1 (fibroblasts) within a small tolerance of the reported values.
- The UMI-vs-phase profile spans the full 0–2π circular range with no empty bins and shows the characteristic drop at mitosis (M/G1).
- The exponential fit to the binned UMI counts over the pre-mitotic segment converges and the peak precedes the mitotic drop in phase.
- umi_fold_changes.csv contains one fold-change row per dataset matching the expected values, confirming the doubling of RNA content before mitosis.
- The binning is self-consistent with the transition thetas: the doubling segment falls between the preceding minimum and the G1/S/S/G2/M transition thetas from estimate_cell_cycle_transitions.py.

## Limitations

- If no cycling genes are found (insufficient sequencing depth, few RNAs per cell), the DeepCycle phases — and hence any binned UMI profile — are unreliable.
- The gene set driving cell_cycle_theta depends on the expression threshold (~1.0 suggested, ~0.5 used for mESC in the README) and sequencing depth per cell, so the phase axis is depth- and threshold-dependent.
- Fold-change estimates assume the DeepCycle circular latent variable correctly orders cells through G1→S→G2→M; baseline comparisons showed alternative methods (scVelo, Cyclum, Cyclone, Revelio) fail to capture these latent dynamics.

## Evidence

- [readme] The output Anndata contains a new observation variable adata.obs['cell_cycle_theta'] that represents the circular variable related to the cycling genes.: "The output Anndata contains a new observation variable adata.obs['cell_cycle_theta'] that represents the circular variable related to the cycling genes."
- [readme] h5ad files already annotated with cell cycle phase downloadable from Zenodo: "h5ad files for mESCs and human fibroblasts already annotated with the cell cycle phase can be downloaded from Zenodo"
- [supplementary] estimate_cell_cycle_transitions.py estimates thetas of phase transitions: "The script estimate_cell_cycle_transitions.py in ./scripts/ runs the estimation of the thetas associated to the different phase transitions (G1/S, S/G2 and M/G1)."
- [abstract] Major transcriptional waves during G1 and stage characterization: "Characterizing the cell cycle in embryonic and somatic cells, we identify major waves of transcription during the G1 phase and systematically study the stages of the cell cycle"
