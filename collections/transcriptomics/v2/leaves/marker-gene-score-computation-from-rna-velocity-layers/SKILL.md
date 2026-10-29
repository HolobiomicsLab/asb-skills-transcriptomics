---
name: marker-gene-score-computation-from-rna-velocity-layers
description: Use when an AnnData annotated with DeepCycle's obs['cell_cycle_theta']
  needs marker-gene-derived phase-transition landmarks (Cyclin E peak, G2M-vs-S crossover)
  to assign cells to G1, S, or G2/M.
when_to_use_negative:
- When defining cell cycle phases from scratch without a continuous phase variable
  — the article warns that 'defining the cell cycle phases in RNA data based solely
  on gene markers lacks generalizability', so scores are only used to landmark the
  DeepCycle theta, not to replace it
- When the AnnData lacks scVelo Ms/Mu layers or obs['cell_cycle_theta'] (e.g. raw
  unpreprocessed count matrices not run through scvelo.pp.moments and DeepCycle)
- When no cycling genes were found in the dataset — the README states 'If you do not
  find any cycling genes the results will not be reliable'
edam_operation: http://edamontology.org/operation_0337
edam_topics:
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_3170
tools:
- name: scVelo
  role: Supplies the imputed spliced (Ms) and unspliced (Mu) layers from which the
    Cyclin E, S-phase, and G2/M marker scores are computed
  repo: https://scvelo.readthedocs.io/
- name: DeepCycle
  role: Provides the continuous circular transcriptional phase obs['cell_cycle_theta']
    that the marker scores landmark into phase transitions
  repo: https://github.com/andreariba/DeepCycle
- name: estimate_cell_cycle_transitions.py
  role: Repository script that consumes the marker scores and theta to estimate the
    G1/S, S/G2, and M/G1 transition thetas
  repo: https://github.com/andreariba/DeepCycle
- name: numpy
  role: Array computations for the per-cell marker scores and circular theta binning
- name: scipy
  role: Numerical routines supporting score computation and transition estimation
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1038/s41467-022-30545-8
    title: Cell cycle gene regulation dynamics revealed by RNA velocity and deep-learning
  source_content_hash: sha256:b839572b709e438e14422a1be0ce63353a2bf9a9bdd60e4b65e523690c98eded
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - DeepCycle
  - Python
  - estimate_cell_cycle_transitions.py
  - numpy
  - scVelo
  - scipy
  edam_operation: http://edamontology.org/operation_0337
  edam_topics:
  - http://edamontology.org/topic_0203
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5714285714285714
n_claims: 14
---

# marker-gene-score-computation-from-rna-velocity-layers

## Summary

Compute Cyclin E, S-phase, and G2/M marker gene scores per cell from the scVelo 'Ms' (spliced) and 'Mu' (unspliced) imputed layers of an AnnData object so that cell cycle phase transitions (G1/S, S/G2, M/G1) can be located along the DeepCycle-inferred circular transcriptional phase theta.

## When to use

Use when an AnnData annotated with DeepCycle's obs['cell_cycle_theta'] needs marker-gene-derived phase-transition landmarks (Cyclin E peak, G2M-vs-S crossover) to assign cells to G1, S, or G2/M.

## When NOT to use

- When defining cell cycle phases from scratch without a continuous phase variable — the article warns that 'defining the cell cycle phases in RNA data based solely on gene markers lacks generalizability', so scores are only used to landmark the DeepCycle theta, not to replace it
- When the AnnData lacks scVelo Ms/Mu layers or obs['cell_cycle_theta'] (e.g. raw unpreprocessed count matrices not run through scvelo.pp.moments and DeepCycle)
- When no cycling genes were found in the dataset — the README states 'If you do not find any cycling genes the results will not be reliable'

## Inputs

- AnnData (.h5ad) file annotated with obs['cell_cycle_theta'] and containing scVelo 'Ms' and 'Mu' layers (e.g. adata_mESC.h5ad from Zenodo 10.5281/zenodo.4719436)
- Cell-cycle marker gene sets: Cyclin E, S-phase, and G2/M marker genes for the organism of interest (e.g. GOterm cell_cycle gene lists)

## Outputs

- Per-cell Cyclin E, S-phase, and G2/M marker gene scores
- Estimated transition thetas for G1/S, S/G2, and M/G1 (from estimate_cell_cycle_transitions.py)
- Phase labels (G1, S, G2M) per cell after binning circular theta
- phase_fractions.csv with the fraction of cells per phase per dataset

## How to apply

Start from the Zenodo-annotated h5ad files (adata_mESC.h5ad for mESCs; for fibroblasts restrict to the leiden cluster-0 subpopulation) that already contain scVelo Ms/Mu layers and obs['cell_cycle_theta']. Compute three per-cell scores from these imputed layers: a Cyclin E gene score whose peak marks the G1/S transition, an S-phase marker score, and a G2/M marker score whose crossover with the S-phase score marks the S/G2 transition. These scores are then fed, together with the circular theta, to the repository script scripts/estimate_cell_cycle_transitions.py, which estimates the three transition thetas (the UMI-depth drop supplies the M/G1 transition). The rationale is that gene-marker-based landmark signals in the velocity-imputed expression data anchor the abstract theta to biologically interpretable phase boundaries. Success is judged by binning theta at the estimated transitions and obtaining phase fractions close to the reported values (mESC ~19% G1 / 40% S / 41% G2/M; fibroblasts ~76% G1 / 15% S / 9% G2/M), which are validated against propidium iodide flow cytometry.

## Related tools

- **scVelo** (Supplies the imputed spliced (Ms) and unspliced (Mu) layers from which the Cyclin E, S-phase, and G2/M marker scores are computed) — https://scvelo.readthedocs.io/
- **DeepCycle** (Provides the continuous circular transcriptional phase obs['cell_cycle_theta'] that the marker scores landmark into phase transitions) — https://github.com/andreariba/DeepCycle
- **estimate_cell_cycle_transitions.py** (Repository script that consumes the marker scores and theta to estimate the G1/S, S/G2, and M/G1 transition thetas) — https://github.com/andreariba/DeepCycle
- **numpy** (Array computations for the per-cell marker scores and circular theta binning)
- **scipy** (Numerical routines supporting score computation and transition estimation)

## Examples

```
python scripts/estimate_cell_cycle_transitions.py --adata adata_mESC_DeepCycle.h5ad --output phase_fractions.csv
```

## Evaluation signals

- Binned phase fractions for mESCs should match the DeepCycle-reported values (~19% G1, 40% S, 41% G2/M) and flow cytometry references (22-26% G1, 42-51% S, 27-32% G2/M)
- Fibroblast phase fractions (leiden cluster 0) should be ~76% G1, 15% S, 9% G2/M, matching propidium iodide flow cytometry (79% / 13% / 8%)
- Input AnnData must contain obs['cell_cycle_theta'] and both 'Ms' and 'Mu' layers before running the transition estimation script
- The Cyclin E score peak should align with a theta that partitions G1 from S; the G2M-vs-S score crossover should partition S from G2/M; the UMI-depth drop should mark M/G1
- Output phase_fractions.csv should contain three fractions per dataset that sum to ~1

## Limitations

- Gene-marker-based phase definition is model-dependent: 'the variability in the cell cycle signatures among cellular models, defining the cell cycle phases in RNA data based solely on gene markers lacks generalizability'
- For fibroblasts the scores and transitions must be computed only on the leiden cluster-0 subpopulation, not the whole dataset
- If sequencing depth is too low and no cycling genes are detected, downstream marker-score landmarking is unreliable

## Evidence

- [supplementary] the layers 'Ms' and 'Mu' from scVelo: "the layers 'Ms' and 'Mu' from scVelo"
- [supplementary] The script estimate_cell_cycle_transitions.py in ./scripts/ runs the estimation of the thetas: "The script estimate_cell_cycle_transitions.py in ./scripts/ runs the estimation of the thetas associated to the different phase transitions (G1/S, S/G2 and M/G1)."
- [discussion] Given the variability in the cell cycle signatures among cellular models: "Given the variability in the cell cycle signatures among cellular models, defining the cell cycle phases in RNA data based solely on gene markers lacks generalizability."
- [readme] The moments from scVelo have to be computed, i.e. you need to run pp.moments from scVelo: "The moments from scVelo have to be computed, i.e. you need to run pp.moments from scVelo (https://scvelo.readthedocs.io/) to generate imputed unspliced (Mu) and spliced (Ms) expressions."
- [supplementary] h5ad files for mESCs and human fibroblasts already annotated with the cell cycle phase: "h5ad files for mESCs and human fibroblasts already annotated with the cell cycle phase can be downloaded from Zenodo"
