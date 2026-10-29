---
name: cell-cycle-phase-transition-estimation
description: Use when an h5ad annotated by DeepCycle (with obs['cell_cycle_theta']
  and scVelo Ms/Mu layers) needs cell-cycle phase labels and per-phase cell fractions,
  e.g. to validate inferred thetas against flow-cytometry proportions.
when_to_use_negative:
- The input AnnData has not been run through DeepCycle and lacks obs['cell_cycle_theta']
  (the script requires it) or lacks the scVelo Ms/Mu layers
- You only need a coarse binary cycling vs non-cycling classification rather than
  resolved G1/S and S/G2/M transitions
- No cycling genes were detected in the dataset — the README warns results will not
  be reliable in that case
edam_operation: http://edamontology.org/operation_0337
edam_topics:
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_3170
tools:
- name: DeepCycle
  role: Provides the inferred circular transcriptional phase (cell_cycle_theta) and
    repository scripts, including scripts/estimate_cell_cycle_transitions.py, used
    to detect the phase transitions
  repo: https://github.com/andreariba/DeepCycle
- name: scVelo
  role: Supplies the imputed spliced ('Ms') and unspliced ('Mu') layers from which
    Cyclin E, S-phase, and G2/M marker gene scores are computed per cell
  repo: https://scvelo.readthedocs.io/
- name: fcsparser
  role: Parses propidium iodide flow cytometry FCS files used as an external reference
    for the computed phase fractions
  repo: https://github.com/eyurtsev/fcsparser
- name: numpy
  role: Numerical handling of circular theta binning and score computations (tested
    with numpy 1.19.1)
- name: scipy
  role: Numerical routines supporting transition estimation on the circular theta
    variable (tested with scipy 1.5.2)
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1038/s41467-022-30545-8
    title: Cell cycle gene regulation dynamics revealed by RNA velocity and deep-learning
  source_content_hash: sha256:c111a56c065bf6c664261efb3cf2dc294aafbbb175ac39f395dbdd82238dd21a
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/andreariba/DeepCycle
  tools:
  - DeepCycle
  - Python
  - fcsparser
  - numpy
  - scVelo
  - scipy
  edam_operation: http://edamontology.org/operation_0337
  edam_topics:
  - http://edamontology.org/topic_0203
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8461538461538461
n_claims: 13
---

# cell-cycle-phase-transition-estimation

## Summary

Estimates the three theta transitions (G1/S, S/G2, M/G1) along DeepCycle's inferred circular transcriptional phase (adata.obs['cell_cycle_theta']) using Cyclin E marker scores, G2M-vs-S score crossovers, and UMI-depth drops, then assigns each cell to G1, S, or G2/M and computes phase fractions.

## When to use

Use when an h5ad annotated by DeepCycle (with obs['cell_cycle_theta'] and scVelo Ms/Mu layers) needs cell-cycle phase labels and per-phase cell fractions, e.g. to validate inferred thetas against flow-cytometry proportions.

## When NOT to use

- The input AnnData has not been run through DeepCycle and lacks obs['cell_cycle_theta'] (the script requires it) or lacks the scVelo Ms/Mu layers
- You only need a coarse binary cycling vs non-cycling classification rather than resolved G1/S and S/G2/M transitions
- No cycling genes were detected in the dataset — the README warns results will not be reliable in that case

## Inputs

- DeepCycle-annotated h5ad file (e.g. adata_mESC.h5ad from Zenodo doi:10.5281/zenodo.4719436) containing obs['cell_cycle_theta']
- AnnData layers 'Ms' and 'Mu' computed with scVelo pp.moments
- For human fibroblasts: the leiden cluster-0 subpopulation only

## Outputs

- Three estimated transition thetas (G1/S, S/G2, M/G1) on the circular cell_cycle_theta axis
- Per-cell phase assignment (G1, S, G2/M)
- phase_fractions.csv with the fraction of cells per phase per dataset

## How to apply

Download the Zenodo-annotated h5ad files (doi:10.5281/zenodo.4719436) for mESCs and human fibroblasts — for fibroblasts restrict to the leiden cluster-0 subpopulation — and load them with scanpy/AnnData. Compute Cyclin E, S-phase, and G2/M marker gene scores from the scVelo Ms/Mu layers of each cell. Run scripts/estimate_cell_cycle_transitions.py on adata.obs['cell_cycle_theta'] to estimate the three transition thetas: the Cyclin E score peak marks G1/S, the G2M-vs-S score crossover marks S/G2, and the UMI-depth drop marks M/G1. Bin the circular theta values by the estimated transitions to assign each cell to G1, S, or G2M, then compute and save per-phase fractions (e.g. phase_fractions.csv). The rationale is that gene-marker-based phase definitions alone lack generalizability, so transitions are anchored on dynamical features (score peaks/crossovers and sequencing-depth drops) along the continuous theta.

## Related tools

- **DeepCycle** (Provides the inferred circular transcriptional phase (cell_cycle_theta) and repository scripts, including scripts/estimate_cell_cycle_transitions.py, used to detect the phase transitions) — https://github.com/andreariba/DeepCycle
- **scVelo** (Supplies the imputed spliced ('Ms') and unspliced ('Mu') layers from which Cyclin E, S-phase, and G2/M marker gene scores are computed per cell) — https://scvelo.readthedocs.io/
- **fcsparser** (Parses propidium iodide flow cytometry FCS files used as an external reference for the computed phase fractions) — https://github.com/eyurtsev/fcsparser
- **numpy** (Numerical handling of circular theta binning and score computations (tested with numpy 1.19.1))
- **scipy** (Numerical routines supporting transition estimation on the circular theta variable (tested with scipy 1.5.2))

## Examples

```
python scripts/estimate_cell_cycle_transitions.py --input_adata adata_mESC_DeepCycle.h5ad
```

## Evaluation signals

- mESC phase fractions should approximate ~19% G1, 40% S, 41% G2/M; fibroblast (leiden cluster 0) fractions ~76% G1, 15% S, 9% G2/M
- Fractions should agree with propidium iodide flow cytometry references: fibroblasts ~79%/13%/8% and mESCs 22–26% G1, 42–51% S, 27–32% G2/M
- The three estimated thetas must be ordered consistently around the circular theta axis (G1/S before S/G2 before M/G1) and bin all cells exactly once
- Output CSV (phase_fractions.csv) must contain one fraction per phase per dataset with fractions summing to ~1

## Limitations

- Defining cell cycle phases from RNA data based solely on gene markers lacks generalizability across cellular models, given variability in cell cycle signatures
- If no cycling genes are found, DeepCycle results (and hence transition estimates) are unreliable; deeper sequencing with more RNAs per cell may be needed
- Future methods should replace gene markers with approaches relying on dynamical features of gene expression

## Evidence

- [supplementary] estimate_cell_cycle_transitions script: "The script estimate_cell_cycle_transitions.py in ./scripts/ runs the estimation of the thetas associated to the different phase transitions (G1/S, S/G2 and M/G1)."
- [readme] script input requirements: "The input anndata has to contains the obs['cell_cycle_theta'] and the layers 'Ms' and 'Mu' from scVelo."
- [discussion] marker limitation: "Given the variability in the cell cycle signatures among cellular models, defining the cell cycle phases in RNA data based solely on gene markers lacks generalizability."
- [supplementary] fibroblast subpopulation: "To reproduce the results for the human fibroblast, you have to run the DeepCycle on the subpopulation identified by the leiden clustering as 0"
- [supplementary] annotated h5ad data availability: "h5ad files for mESCs and human fibroblasts already annotated with the cell cycle phase can be downloaded from Zenodo"
- [readme] no cycling genes warning: "If you do not find any cycling genes the results will not be reliable."
