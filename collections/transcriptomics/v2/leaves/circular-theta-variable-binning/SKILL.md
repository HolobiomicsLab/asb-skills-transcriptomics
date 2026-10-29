---
name: circular-theta-variable-binning
description: Use when an AnnData object already carries a continuous circular cell
  cycle phase (obs['cell_cycle_theta']) and discrete phase assignments plus per-phase
  fractions are needed for validation against flow cytometry.
when_to_use_negative:
- The input AnnData lacks obs['cell_cycle_theta'] and the scVelo Ms/Mu layers, i.e.
  DeepCycle has not been run — rerun DeepCycle first with a gene list and base gene
  (Nusap1 for mESC, MELK for fibroblasts).
- No cycling genes are detected in the dataset; the README warns the results will
  not be reliable and deeper sequencing is likely needed.
- Phase assignment should rely on gene markers alone as a generalizable method across
  cellular models — the article notes marker-based definitions lack generalizability
  across cellular models.
tools:
- name: DeepCycle
  role: Produces the circular obs['cell_cycle_theta'] that is binned, and provides
    scripts/estimate_cell_cycle_transitions.py for the transition thetas
  repo: https://github.com/andreariba/DeepCycle
- name: scVelo
  role: Supplies the imputed Ms and Mu layers used to compute Cyclin E, S-phase, and
    G2/M marker gene scores
  repo: https://scvelo.readthedocs.io/
- name: scanpy/AnnData
  role: Loads and manipulates the .h5ad datasets, including subsetting fibroblasts
    to the leiden cluster-0 population
- name: fcsparser
  role: Parses propidium iodide flow cytometry FCS files used to validate the computed
    phase fractions
  repo: https://github.com/eyurtsev/fcsparser
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1038/s41467-022-30545-8
    title: Cell cycle gene regulation dynamics revealed by RNA velocity and deep-learning
  source_content_hash: sha256:aab7ba525d54ad297f28a36d92109dd58611fa40558692c2758d08783ff297ff
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
  - scanpy/AnnData
  - scipy
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7142857142857143
n_claims: 14
---

# circular-theta-variable-binning

## Summary

Converts DeepCycle's circular transcriptional phase (adata.obs['cell_cycle_theta']) into discrete G1, S, and G2/M cell cycle phase labels by estimating three transition thetas (G1/S, S/G2, M/G1) from marker scores and UMI depth, then binning the circular variable to compute per-phase cell fractions.

## When to use

Use when an AnnData object already carries a continuous circular cell cycle phase (obs['cell_cycle_theta']) and discrete phase assignments plus per-phase fractions are needed for validation against flow cytometry.

## When NOT to use

- The input AnnData lacks obs['cell_cycle_theta'] and the scVelo Ms/Mu layers, i.e. DeepCycle has not been run — rerun DeepCycle first with a gene list and base gene (Nusap1 for mESC, MELK for fibroblasts).
- No cycling genes are detected in the dataset; the README warns the results will not be reliable and deeper sequencing is likely needed.
- Phase assignment should rely on gene markers alone as a generalizable method across cellular models — the article notes marker-based definitions lack generalizability across cellular models.

## Inputs

- AnnData .h5ad file annotated with obs['cell_cycle_theta'] and scVelo 'Ms'/'Mu' layers (e.g. adata_mESC.h5ad from Zenodo 10.5281/zenodo.4719436)
- Leiden cluster-0 subpopulation selection for the human fibroblast dataset
- Marker gene score inputs (Cyclin E, S-phase, G2/M) computed from Ms/Mu layers; per-cell UMI depth for the M/G1 transition

## Outputs

- Per-cell discrete phase labels (G1, S, G2M) derived from binned circular theta
- phase_fractions.csv with the fraction of cells per phase for each dataset
- Estimated transition thetas for G1/S, S/G2, and M/G1

## How to apply

Load the Zenodo-annotated h5ad files (adata_mESC.h5ad; for human fibroblasts restrict to the leiden cluster-0 subpopulation) with scanpy/AnnData, which must contain obs['cell_cycle_theta'] and the scVelo 'Ms' and 'Mu' layers. Compute Cyclin E, S-phase, and G2/M marker gene scores from the Ms/Mu layers per cell. Run the repository script scripts/estimate_cell_cycle_transitions.py to estimate the three transition thetas: the Cyclin E score peak defines G1/S, the G2M-vs-S score crossover defines S/G2, and the UMI-depth drop defines M/G1. Bin the circular theta values by these transitions to assign each cell to G1, S, or G2M, then compute the fraction of cells per phase. Validate against DeepCycle-reported fractions (mESC ~19%/40%/41%; fibroblasts ~76%/15%/9%) and propidium iodide flow cytometry references (fibroblasts 79%/13%/8%; mESC 22-26%/42-51%/27-32%).

## Related tools

- **DeepCycle** (Produces the circular obs['cell_cycle_theta'] that is binned, and provides scripts/estimate_cell_cycle_transitions.py for the transition thetas) — https://github.com/andreariba/DeepCycle
- **scVelo** (Supplies the imputed Ms and Mu layers used to compute Cyclin E, S-phase, and G2/M marker gene scores) — https://scvelo.readthedocs.io/
- **scanpy/AnnData** (Loads and manipulates the .h5ad datasets, including subsetting fibroblasts to the leiden cluster-0 population)
- **fcsparser** (Parses propidium iodide flow cytometry FCS files used to validate the computed phase fractions) — https://github.com/eyurtsev/fcsparser

## Examples

```
python scripts/estimate_cell_cycle_transitions.py
```

## Evaluation signals

- mESC phase fractions should approach the reported ~19% G1, 40% S, 41% G2/M.
- Fibroblast (leiden cluster-0) fractions should approach the reported ~76% G1, 15% S, 9% G2M.
- Fractions should be consistent with propidium iodide flow cytometry: fibroblasts 79%/13%/8%, mESC 22-26% G1, 42-51% S, 27-32% G2/M.
- Estimated transition thetas should follow the expected anchors: Cyclin E score peak at G1/S, G2M-vs-S score crossover at S/G2, UMI-depth drop at M/G1.
- phase_fractions.csv should be produced and every cell assigned to exactly one of the three phases.

## Limitations

- Defining cell cycle phases based solely on gene markers lacks generalizability across cellular models due to variability in cell cycle signatures.
- If no cycling genes are found (e.g. low RNAs per cell), the DeepCycle theta and downstream phase binning will not be reliable; deeper sequencing may be required.
- The human fibroblast result is only valid when DeepCycle/binning is applied to the leiden cluster-0 subpopulation with the human GO cell cycle gene list and MELK as base gene.

## Evidence

- [readme] transition theta estimation script: "The script estimate_cell_cycle_transitions.py in ./scripts/ runs the estimation of the thetas associated to the different phase transitions (G1/S, S/G2 and M/G1)."
- [readme] theta observation variable: "The output Anndata contains a new observation variable adata.obs['cell_cycle_theta'] that represents the circular variable related to the cycling genes."
- [supplementary] input layer requirements: "The input anndata has to contains the obs['cell_cycle_theta'] and the layers 'Ms' and 'Mu' from scVelo."
- [results] mESC phase fractions and flow cytometry validation: [evidence span withheld — anchor status: not_found]
- [supplementary] fibroblast subpopulation restriction: "To reproduce the results for the human fibroblast, you have to run the DeepCycle on the subpopulation identified by the leiden clustering as 0"
