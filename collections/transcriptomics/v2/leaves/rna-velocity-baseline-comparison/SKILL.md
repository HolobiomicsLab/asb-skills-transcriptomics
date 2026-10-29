---
name: rna-velocity-baseline-comparison
description: Use when you need to benchmark an RNA-velocity- or manifold-based cell-cycle
  method (scVelo dynamical, Cyclum, Cyclone, Revelio, DeepCycle) on mESC or human
  fibroblast h5ad data and quantify agreement of inferred trajectories with annotated
  phases or DeepCycle's cell_cycle_theta.
when_to_use_negative:
- No cycling genes are detectable in the dataset (e.g. insufficient sequencing depth
  per cell) — the results will not be reliable per the README
- The input Anndata lacks spliced/unspliced layers or has not been preprocessed with
  scvelo.pp.moments, since all baselines operate on Mu/Ms moments
- The goal is phase assignment from gene-marker signatures only, which the paper states
  'lacks generalizability' across cellular models
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: scVelo
  role: Provides the pp.moments preprocessing (Mu/Ms layers) used by all baselines
    and serves as the extended dynamical-model baseline whose single-gene fits and
    latent time are evaluated
  repo: https://scvelo.readthedocs.io/
- name: DeepCycle
  role: Reference deep-learning method producing the continuous cell_cycle_theta trajectory
    used as the yardstick for baseline agreement
  repo: https://github.com/andreariba/DeepCycle
- name: velocyto
  role: Upstream preprocessing of the input Anndata alongside scVelo moments before
    running the baselines
- name: Cyclum
  role: Baseline tested for whether cells can be placed on a consistent circular 1D
    manifold
- name: Cyclone
  role: Baseline producing discrete phase assignments evaluated for consistency with
    annotated mESC phases
- name: Revelio
  role: Baseline whose trajectory agreement with DeepCycle's continuous cell_cycle_theta
    is quantified on both datasets
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1038/s41467-022-30545-8
    title: Cell cycle gene regulation dynamics revealed by RNA velocity and deep-learning
  source_content_hash: sha256:28e405d2c7ec2103f0b04802609881df410b1a7a44a17f3bb28a10d398fe4847
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - Cyclone
  - Cyclum
  - DeepCycle
  - Python
  - Revelio
  - numpy
  - scVelo
  - scipy
  - velocyto
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7
n_claims: 10
---

# rna-velocity-baseline-comparison

## Summary

A benchmarking procedure that reproduces the head-to-head comparison of DeepCycle against scVelo's extended dynamical model, Cyclum, Cyclone and Revelio on cell-cycle scRNA-seq datasets, testing whether each method fits single-gene unspliced-spliced dynamics and recovers the correct latent-time/cell-cycle trajectory.

## When to use

Use when you need to benchmark an RNA-velocity- or manifold-based cell-cycle method (scVelo dynamical, Cyclum, Cyclone, Revelio, DeepCycle) on mESC or human fibroblast h5ad data and quantify agreement of inferred trajectories with annotated phases or DeepCycle's cell_cycle_theta.

## When NOT to use

- No cycling genes are detectable in the dataset (e.g. insufficient sequencing depth per cell) — the results will not be reliable per the README
- The input Anndata lacks spliced/unspliced layers or has not been preprocessed with scvelo.pp.moments, since all baselines operate on Mu/Ms moments
- The goal is phase assignment from gene-marker signatures only, which the paper states 'lacks generalizability' across cellular models

## Inputs

- adata_mESC.h5ad and human fibroblast h5ad annotated with cell cycle phase (Zenodo 10.5281/zenodo.4719436)
- Anndata preprocessed with velocyto, scvelo.pp.moments (Mu/Ms layers)
- GOterm cell_cycle gene list for the organism (e.g. go_annotation/GO_cell_cycle_annotation_mouse.txt)
- DeepCycle output Anndata with obs['cell_cycle_theta'] as the reference trajectory for agreement metrics

## Outputs

- baseline_comparison_results.csv with per-method outputs (single-gene fits, phase assignments, manifold projections)
- Comparative panels reproducing Fig. 2 and Supplementary Figs. S3, S7, S9
- Per-method agreement statistics vs annotated cell cycle phases and DeepCycle cell_cycle_theta

## How to apply

Load the annotated mESC and human fibroblast h5ad files from the Zenodo deposit (10.5281/zenodo.4719436) and preprocess with scvelo.pp.moments to obtain the Mu and Ms layers. Run scVelo's extended dynamical model on the GOterm cell_cycle gene set (expression threshold around 1.0) and inspect single-gene unspliced-spliced fits — the paper reports scVelo's fits fail and its inferred latent time misses the correct cell-level dynamics (Supplementary Fig. S3), because constant-rate assumptions cannot capture cycling transcription patterns. Run Cyclum (circular 1D manifold), Cyclone (discrete phase assignment on mESC, scored against annotated phases) and Revelio (agreement with DeepCycle's continuous cell_cycle_theta) on the same preprocessed data. Collect per-method fits, phase assignments and manifold projections into baseline_comparison_results.csv and reproduce the comparative panels of Fig. 2 and Supplementary Figs. S3, S7, S9. For DeepCycle itself, the README command uses base gene Nusap1 (mESC) or MELK (human fibroblast leiden-0 subpopulation) with --expression_threshold 0.5 and --hotelling.

## Related tools

- **scVelo** (Provides the pp.moments preprocessing (Mu/Ms layers) used by all baselines and serves as the extended dynamical-model baseline whose single-gene fits and latent time are evaluated) — https://scvelo.readthedocs.io/
- **DeepCycle** (Reference deep-learning method producing the continuous cell_cycle_theta trajectory used as the yardstick for baseline agreement) — https://github.com/andreariba/DeepCycle
- **velocyto** (Upstream preprocessing of the input Anndata alongside scVelo moments before running the baselines)
- **Cyclum** (Baseline tested for whether cells can be placed on a consistent circular 1D manifold)
- **Cyclone** (Baseline producing discrete phase assignments evaluated for consistency with annotated mESC phases)
- **Revelio** (Baseline whose trajectory agreement with DeepCycle's continuous cell_cycle_theta is quantified on both datasets)

## Examples

```
python DeepCycle.py --input_adata adata_mESC.h5ad --gene_list go_annotation/GO_cell_cycle_annotation_mouse.txt --base_gene Nusap1 --expression_threshold 0.5 --gpu --hotelling --output_adata adata_mESC_DeepCycle.h5ad
```

## Evaluation signals

- scVelo's single-gene unspliced-spliced fits on cell-cycle genes visibly fail, and its inferred latent time does not track annotated cell-level phase order (reproduces Supplementary Fig. S3)
- Cyclum cells fall on a consistent circular 1D manifold comparable to DeepCycle's cell_cycle_theta
- Cyclone's discrete phase assignments are consistent with the annotated cell cycle phases in adata_mESC.h5ad
- Revelio trajectory shows quantifiable agreement with DeepCycle's cell_cycle_theta on both mESC and fibroblast datasets
- All methods consumed the identical preprocessed Anndata (same Mu/Ms layers, same GOterm cell_cycle gene set, expression threshold ~1.0), so comparisons are apples-to-apples

## Limitations

- scVelo's extended model assumes constant rates, which the paper notes should be relaxed to fit the transcriptional model to cycling-gene data
- If no cycling genes are found, results are unreliable and deeper sequencing (more RNAs per cell) is needed
- Baseline agreement is dataset- and organism-specific: gene lists and base genes differ between mouse (Nusap1) and human fibroblast (MELK, leiden cluster 0)

## Evidence

- [readme] scVelo preprocessing of h5ad inputs used by all baselines: "In the current version a preprocessing step is required. The moments from scVelo have to be computed, i.e. you need to run pp.moments from scVelo"
- [supplementary] Reported failure of scVelo's dynamical model on cell-cycle data (Supplementary Fig. S3): [evidence span withheld — anchor status: not_found]
- [discussion] Rationale for why existing velocity models fail to fit cycling genes: "Likely the assumptions in the previous model (constant rates) should be relaxed to fit the transcriptional model to the data."
- [readme] Baseline Anndata inputs available for benchmarking: "h5ad files for mESCs and human fibroblasts already annotated with the cell cycle phase can be downloaded from Zenodo"
- [readme] Reliability caveat for shallow-sequenced datasets: "If you do not find any cycling genes the results will not be reliable. It is possible you have to sequence deeper and increase the number of RNAs per cell."
