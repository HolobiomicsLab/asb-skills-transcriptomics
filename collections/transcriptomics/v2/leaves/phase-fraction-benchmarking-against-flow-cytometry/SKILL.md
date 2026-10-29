---
name: phase-fraction-benchmarking-against-flow-cytometry
description: Use when a scRNA-seq cell cycle phase assignment (e.g. from DeepCycle
  cell_cycle_theta) needs independent validation against flow cytometry phase fractions,
  or when reproducing the paper's mESC (~19%/40%/41%) and fibroblast (~76%/15%/9%)
  phase proportion benchmarks.
when_to_use_negative:
- The input h5ad lacks obs['cell_cycle_theta'] or scVelo Ms/Mu layers (DeepCycle has
  not been run); transition estimation will not work
- No cycling genes were detected by DeepCycle — the README warns results will not
  be reliable, so phase fractions should not be benchmarked
- The biological system has no matching flow cytometry reference data and only marker-gene
  phases are available, which the discussion notes lacks generalizability across cellular
  models
edam_operation: http://edamontology.org/operation_0337
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_2229
tools:
- name: DeepCycle
  role: Infers the circular cell_cycle_theta per cell whose binned phases are benchmarked
    against flow cytometry
  repo: https://github.com/andreariba/DeepCycle
- name: scVelo
  role: Provides the Ms/Mu moment layers used to compute the Cyclin E, S-phase, and
    G2/M marker scores for transition detection
  repo: https://scvelo.readthedocs.io/
- name: fcsparser
  role: Parses propidium iodide flow cytometry FCS files to obtain reference DNA-content
    phase fractions
  repo: https://github.com/eyurtsev/fcsparser
- name: Python 3.7.9 / numpy 1.19.1 / scipy 1.5.2
  role: Execution environment for binning theta, computing fractions, and comparing
    to references
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1038/s41467-022-30545-8
    title: Cell cycle gene regulation dynamics revealed by RNA velocity and deep-learning
  source_content_hash: sha256:3fe81b1c823a47775509073f7d10023697862f4a015cc0c521c5710807d5a2df
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/andreariba/DeepCycle
  tools:
  - DeepCycle
  - Python
  - Python 3.7.9 / numpy 1.19.1 / scipy 1.5.2
  - fcsparser
  - numpy
  - scVelo
  - scipy
  edam_operation: http://edamontology.org/operation_0337
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_2229
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 11
---

# phase-fraction-benchmarking-against-flow-cytometry

## Summary

Validates cell cycle phase fractions inferred from DeepCycle's transcriptional theta (G1, S, G2/M) by comparing them to propidium iodide flow cytometry reference proportions in mESCs and human fibroblasts, demonstrating that RNA-velocity-based phase assignment reproduces DNA-content-based measurements.

## When to use

Use when a scRNA-seq cell cycle phase assignment (e.g. from DeepCycle cell_cycle_theta) needs independent validation against flow cytometry phase fractions, or when reproducing the paper's mESC (~19%/40%/41%) and fibroblast (~76%/15%/9%) phase proportion benchmarks.

## When NOT to use

- The input h5ad lacks obs['cell_cycle_theta'] or scVelo Ms/Mu layers (DeepCycle has not been run); transition estimation will not work
- No cycling genes were detected by DeepCycle — the README warns results will not be reliable, so phase fractions should not be benchmarked
- The biological system has no matching flow cytometry reference data and only marker-gene phases are available, which the discussion notes lacks generalizability across cellular models

## Inputs

- Zenodo-annotated h5ad files (adata_mESC.h5ad, human fibroblast dataset) with obs['cell_cycle_theta'] and scVelo 'Ms'/'Mu' layers
- Gene marker lists for Cyclin E, S-phase, and G2/M signatures
- Propidium iodide flow cytometry FCS files (readable with fcsparser, FCS 2.0/3.0/3.1)
- Repository script scripts/estimate_cell_cycle_transitions.py

## Outputs

- Per-cell phase labels (G1, S, G2/M) derived from binned cell_cycle_theta
- phase_fractions.csv with fraction of cells per phase for each dataset
- Comparison table of scRNA-seq-derived vs flow-cytometry phase fractions

## How to apply

Load the Zenodo-annotated h5ad files (adata_mESC.h5ad; for fibroblasts restrict to the leiden cluster-0 subpopulation) with scanpy/annData. Compute Cyclin E, S-phase, and G2/M marker gene scores from the scVelo Ms/Mu layers, then run scripts/estimate_cell_cycle_transitions.py on adata.obs['cell_cycle_theta'] to find the G1/S (Cyclin E score peak), S/G2 (G2M-vs-S score crossover), and M/G1 (UMI-depth drop) transition thetas. Bin the circular theta values by these transitions to assign each cell to G1, S, or G2M and compute per-phase fractions. Validate the fractions against propidium iodide flow cytometry references (mESC 22-26% G1, 42-51% S, 27-32% G2/M; fibroblasts 79%/13%/8%) using fcsparser to read FCS files if needed; the rationale is that gene-marker-only phase definitions lack generalizability, so an orthogonal DNA-content measurement provides an independent check of DeepCycle's continuous phase assignment.

## Related tools

- **DeepCycle** (Infers the circular cell_cycle_theta per cell whose binned phases are benchmarked against flow cytometry) — https://github.com/andreariba/DeepCycle
- **scVelo** (Provides the Ms/Mu moment layers used to compute the Cyclin E, S-phase, and G2/M marker scores for transition detection) — https://scvelo.readthedocs.io/
- **fcsparser** (Parses propidium iodide flow cytometry FCS files to obtain reference DNA-content phase fractions) — https://github.com/eyurtsev/fcsparser
- **Python 3.7.9 / numpy 1.19.1 / scipy 1.5.2** (Execution environment for binning theta, computing fractions, and comparing to references)

## Examples

```
python scripts/estimate_cell_cycle_transitions.py --input_adata adata_mESC_DeepCycle.h5ad; # then bin cell_cycle_theta and save fractions to phase_fractions.csv
```

## Evaluation signals

- mESC phase fractions should approximate 19% G1 / 40% S / 41% G2/M from DeepCycle theta binning
- mESC fractions should fall within the propidium iodide flow cytometry ranges 22-26% G1, 42-51% S, 27-32% G2/M
- Human fibroblast (leiden cluster 0) fractions should approximate 76% G1 / 15% S / 9% G2/M, close to the flow reference 79%/13%/8%
- The three estimated transition thetas should correspond to biologically sensible signals: Cyclin E score peak (G1/S), G2M-vs-S score crossover (S/G2), and UMI-depth drop (M/G1)
- Output phase_fractions.csv exists with three phase fractions per dataset summing to 1

## Limitations

- Defining cell cycle phases based solely on gene markers lacks generalizability among cellular models, motivating the RNA-velocity-based approach and the flow cytometry cross-check
- If no cycling genes are found, DeepCycle results are unreliable — sequencing depth (RNAs per cell) may need to increase
- For fibroblasts, the annotation only reproduces when DeepCycle is restricted to the leiden cluster-0 subpopulation with the human GO cell cycle gene list and MELK as base gene

## Evidence

- [results] Assigns mESCs 19% to G1, 40% to S, and 41% to G2/M, validated by propidium iodide flow cytometry: [evidence span withheld — anchor status: not_found]
- [supplementary] The script estimate_cell_cycle_transitions.py runs the estimation of the thetas: "The script estimate_cell_cycle_transitions.py in ./scripts/ runs the estimation of the thetas associated to the different phase transitions (G1/S, S/G2 and M/G1)."
- [readme] The input anndata has to contains the obs['cell_cycle_theta'] and the layers 'Ms' and 'Mu' from scVelo.: "The input anndata has to contains the obs['cell_cycle_theta'] and the layers 'Ms' and 'Mu' from scVelo."
- [supplementary] h5ad files for mESCs and human fibroblasts already annotated with the cell cycle phase can be downloaded from Zenodo: "h5ad files for mESCs and human fibroblasts already annotated with the cell cycle phase can be downloaded from Zenodo"
- [discussion] defining the cell cycle phases in RNA data based solely on gene markers lacks generalizability: "Given the variability in the cell cycle signatures among cellular models, defining the cell cycle phases in RNA data based solely on gene markers lacks generalizability."
- [readme] If you do not find any cycling genes the results will not be reliable.: "If you do not find any cycling genes the results will not be reliable. It is possible you have to sequence deeper and increase the number of RNAs per cell."
