---
name: circular-trajectory-evaluation
description: Use when you have inferred a circular latent time (e.g. adata.obs['cell_cycle_theta'])
  from cycling genes in scRNA-seq data and need to verify the fits, phase-transition
  angles, and agreement with discrete/continuous baseline methods.
when_to_use_negative:
- No cycling genes can be detected in the dataset — the README states results will
  not be reliable and deeper sequencing (more RNAs per cell) may be needed
- The input Anndata lacks Mu/Ms moments from scvelo.pp.moments or unspliced/spliced
  layers, since the circular inference and its evaluation are based on unspliced-spliced
  reads
- The cell-cycle signature is being defined from gene markers alone rather than dynamical
  features — the article notes marker-based phase definition lacks generalizability
  across cellular models
edam_topics:
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_3170
tools:
- name: DeepCycle
  role: Produces the circular cell_cycle_theta trajectory whose fits and transition
    angles are evaluated
  repo: https://github.com/andreariba/DeepCycle
- name: scVelo
  role: Computes Mu/Ms moments (pp.moments) required as input; its extended dynamical
    model is the negative baseline shown to fail single-gene fits and latent-time
    capture
  repo: https://scvelo.readthedocs.io/
- name: estimate_cell_cycle_transitions.py
  role: Estimates the thetas associated with G1/S, S/G2 and M/G1 phase transitions
    to validate angular ordering
  repo: https://github.com/andreariba/DeepCycle
- name: Cyclum
  role: Baseline tested for whether cells can be placed on a consistent circular 1D
    manifold
- name: Cyclone
  role: Baseline producing discrete phase assignments compared against annotated phases
- name: Revelio
  role: Baseline whose trajectory agreement with DeepCycle cell_cycle_theta is quantified
- name: velocyto
  role: Preprocessing of the Anndata together with scvelo moments before running DeepCycle
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1038/s41467-022-30545-8
    title: Cell cycle gene regulation dynamics revealed by RNA velocity and deep-learning
  source_content_hash: sha256:b9fc23b9cae503ea013865102ce1b41d346b82fd3b38e05e7f54ae29f4f13884
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/andreariba/DeepCycle
  tools:
  - Cyclone
  - Cyclum
  - DeepCycle
  - Python
  - Revelio
  - estimate_cell_cycle_transitions.py
  - numpy
  - scVelo
  - scipy
  - velocyto
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0203
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8333333333333334
n_claims: 18
---

# circular-trajectory-evaluation

## Summary

A skill for validating circular (1D angular) cell-cycle latent trajectories inferred from scRNA-seq: checking that single-gene unspliced-spliced fits are plausible, that the inferred theta recovers known phase transitions (G1/S, S/G2, M/G1), and comparing against baselines such as scVelo's dynamical model, Cyclum, Cyclone and Revelio.

## When to use

Use when you have inferred a circular latent time (e.g. adata.obs['cell_cycle_theta']) from cycling genes in scRNA-seq data and need to verify the fits, phase-transition angles, and agreement with discrete/continuous baseline methods.

## When NOT to use

- No cycling genes can be detected in the dataset — the README states results will not be reliable and deeper sequencing (more RNAs per cell) may be needed
- The input Anndata lacks Mu/Ms moments from scvelo.pp.moments or unspliced/spliced layers, since the circular inference and its evaluation are based on unspliced-spliced reads
- The cell-cycle signature is being defined from gene markers alone rather than dynamical features — the article notes marker-based phase definition lacks generalizability across cellular models

## Inputs

- Anndata .h5ad file preprocessed with scvelo.pp.moments (Mu/Ms layers), e.g. adata_mESC.h5ad or the human fibroblast h5ad from Zenodo 10.5281/zenodo.4719436
- Organism-specific cycling gene list (e.g. GOterm cell_cycle annotation file such as GO_cell_cycle_annotation_mouse.txt)
- Annotated cell cycle phase labels for cells (mESC/human fibroblast phase annotations)
- Baseline method outputs: scVelo dynamical-model fits/latent time, Cyclum manifold projections, Cyclone phase assignments, Revelio trajectories

## Outputs

- Output Anndata with adata.obs['cell_cycle_theta'] circular trajectory
- Unspliced-spliced fit plots in ./fits/ used to judge fit quality
- Estimated transition angles (G1/S, S/G2, M/G1) from estimate_cell_cycle_transitions.py
- baseline_comparison_results.csv with per-method fits, phase assignments, and manifold projections
- Comparative panels reproducing Fig. 2 and Supplementary Figs. S3, S7, S9

## How to apply

Start from an Anndata preprocessed with scvelo.pp.moments so the Mu/Ms layers exist, and run the circular method (DeepCycle) on the GOterm cell_cycle gene list with an expression threshold around 0.5–1.0 and the Hotelling filter. Inspect the ./fits/ directory for single-gene unspliced-spliced patterns to confirm the fitting procedure worked — scVelo's extended dynamical model is the known negative baseline, failing both single-gene fits and latent-time ordering (Supplementary Fig. S3), because its constant-rate assumptions should be relaxed. Run the baselines on the same preprocessed data: Cyclum (test whether cells lie on a consistent circular 1D manifold), Cyclone (discrete phase assignments, check consistency with annotated phases), and Revelio (quantify agreement with the DeepCycle cell_cycle_theta). Finally, run estimate_cell_cycle_transitions.py to estimate the thetas of the G1/S, S/G2 and M/G1 transitions and check that the angular ordering matches the annotated phases. Collect per-method outputs (fits, phase assignments, manifold projections) into a baseline_comparison_results.csv and reproduce the comparative panels (Fig. 2, Supplementary Figs. S3, S7, S9).

## Related tools

- **DeepCycle** (Produces the circular cell_cycle_theta trajectory whose fits and transition angles are evaluated) — https://github.com/andreariba/DeepCycle
- **scVelo** (Computes Mu/Ms moments (pp.moments) required as input; its extended dynamical model is the negative baseline shown to fail single-gene fits and latent-time capture) — https://scvelo.readthedocs.io/
- **estimate_cell_cycle_transitions.py** (Estimates the thetas associated with G1/S, S/G2 and M/G1 phase transitions to validate angular ordering) — https://github.com/andreariba/DeepCycle
- **Cyclum** (Baseline tested for whether cells can be placed on a consistent circular 1D manifold)
- **Cyclone** (Baseline producing discrete phase assignments compared against annotated phases)
- **Revelio** (Baseline whose trajectory agreement with DeepCycle cell_cycle_theta is quantified)
- **velocyto** (Preprocessing of the Anndata together with scvelo moments before running DeepCycle)

## Examples

```
python DeepCycle.py --input_adata adata_mESC.h5ad --gene_list go_annotation/GO_cell_cycle_annotation_mouse.txt --base_gene Nusap1 --expression_threshold 0.5 --gpu --hotelling --output_adata adata_mESC_DeepCycle.h5ad
```

## Evaluation signals

- ./fits/ plots show sensible unspliced-spliced patterns per cycling gene (scVelo's dynamical model fails this check on the same data)
- The inferred cell_cycle_theta orders annotated phases consistently, and estimated transition thetas for G1/S, S/G2, M/G1 match phase boundaries
- Revelio trajectory shows quantitative agreement with the DeepCycle continuous cell_cycle_theta on both mESC and human fibroblast datasets
- Cyclone discrete phase assignments are consistent with the annotated cell cycle phases; Cyclum places cells on a consistent circular 1D manifold
- At least some cycling genes are found (results are unreliable if none are detected), and training MSE curves in ./training/ plateau via the EarlyStopping/ReduceLROnPlateau callbacks

## Limitations

- If no cycling genes are found, results are unreliable; deeper sequencing to increase RNAs per cell may be required
- scVelo's extended RNA velocity model could not fit actual single-gene dynamics or capture correct latent-time dynamics in the deep-sequenced cell cycle datasets
- Defining cell cycle phases based solely on gene markers lacks generalizability across cellular models; constant-rate assumptions of previous RNA velocity models should be relaxed
- For the human fibroblast data, DeepCycle must be run only on the leiden cluster 0 subpopulation, otherwise the trajectory evaluation is not comparable to the published results

## Evidence

- [readme] readme: "The output Anndata contains a new observation variable adata.obs['cell_cycle_theta'] that represents the circular variable related to the cycling genes."
- [supplementary] supplementary: "The script estimate_cell_cycle_transitions.py in ./scripts/ runs the estimation of the thetas associated to the different phase transitions (G1/S, S/G2 and M/G1)."
- [discussion] discussion: "Likely the assumptions in the previous model (constant rates) should be relaxed to fit the transcriptional model to the data."
