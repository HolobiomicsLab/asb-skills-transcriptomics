---
name: sequencing-depth-perturbation-analysis
description: Use when a single-cell velocity model's outputs must be shown stable
  under technical noise, specifically when perturbing raw count matrices by downsampling
  to reduced sequencing-depth proportions.
when_to_use_negative:
- Input is not raw counts (e.g., normalized or dimensionality-reduced data), since
  downsampling to a percentage of total counts requires raw integer UMI counts
- The scKINETICS mask or parameters would be re-optimized per trial, which confounds
  the depth perturbation with model refitting and invalidates the comparison
- The goal is biological, not technical, perturbation analysis — downsampling only
  simulates reduced sequencing depth, not real biological variation
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: scKINETICS
  role: Velocity inference model refit on each downsampled count matrix with the same
    fixed regulatory mask and parameters as the original fit
  repo: github.com/dpeerlab/scKINETICS
- name: Jupyter
  role: Notebook environment used to run downsampling trials, cosine deviation computation,
    and aggregation into velocity_deviation.csv
  repo: github.com/dpeerlab/scKINETICS
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1093/bioinformatics/btad267
    title: 'scKINETICS: inference of regulatory velocity with single-cell transcriptomics
      data'
  source_content_hash: sha256:e7932b52073d5af4ffb3dd92e047e1e0f32f22a2a35b27edeb7b3eb6e0cd4165
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - Jupyter
  - scKINETICS
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7777777777777778
n_claims: 9
---

# sequencing-depth-perturbation-analysis

## Summary

A robustness test for scKINETICS velocity estimates that randomly downsamples raw single-cell counts to simulate reduced transcript capture depth, refits the model, and quantifies cosine deviation of perturbed velocities from the original fit.

## When to use

Use when a single-cell velocity model's outputs must be shown stable under technical noise, specifically when perturbing raw count matrices by downsampling to reduced sequencing-depth proportions.

## When NOT to use

- Input is not raw counts (e.g., normalized or dimensionality-reduced data), since downsampling to a percentage of total counts requires raw integer UMI counts
- The scKINETICS mask or parameters would be re-optimized per trial, which confounds the depth perturbation with model refitting and invalidates the comparison
- The goal is biological, not technical, perturbation analysis — downsampling only simulates reduced sequencing depth, not real biological variation

## Inputs

- Original raw single-cell count matrix (e.g., GSE207943)
- Original scKINETICS velocity estimates from a prior unperturbed fit
- Fixed regulatory mask and scKINETICS parameter settings from the original fit
- List of downsampling proportions to test

## Outputs

- Trial-level cosine deviations between perturbed and original velocities (15 trials per proportion)
- Aggregated per-proportion deviation summaries
- velocity_deviation.csv containing trial-level and aggregated deviations

## How to apply

Load the original raw count matrix (e.g., GSE207943) together with the original scKINETICS velocity estimates so the perturbed and unperturbed runs share everything but the counts. For each downsampling proportion, generate 15 independent random subsamples of counts to the specified percentage of total counts. Refit scKINETICS on each perturbed matrix keeping the regulatory mask and parameter settings fixed, so only the count perturbation varies. For each trial, compute the cosine deviation between perturbed and original velocities across cells, then aggregate across the 15 trials within each proportion to summarize sensitivity to depth reduction. Since no numeric acceptance threshold is given, judge success qualitatively: perturbed velocities should deviate minimally, consistent with the paper's robustness finding.

## Related tools

- **scKINETICS** (Velocity inference model refit on each downsampled count matrix with the same fixed regulatory mask and parameters as the original fit) — github.com/dpeerlab/scKINETICS
- **Jupyter** (Notebook environment used to run downsampling trials, cosine deviation computation, and aggregation into velocity_deviation.csv) — github.com/dpeerlab/scKINETICS

## Examples

```
from scipy.spatial.distance import cosine; devs = [cosine(v_refit[c], v_orig[c]) for c in cells]  # per trial, then aggregate 15 trials per proportion into velocity_deviation.csv
```

## Evaluation signals

- Refitted velocities on downsampled counts show minimal cosine deviation from the original velocities across cells (qualitative, since no numeric threshold is stated in the source)
- Deviation is summarized consistently across 15 independent random trials per downsampling proportion
- Regulatory mask and parameter settings are verified identical between original and refit runs, so only the count perturbation differs
- velocity_deviation.csv contains both trial-level and aggregated deviation records for every tested proportion

## Limitations

- No numeric threshold is stated in the source for what counts as acceptable deviation; acceptance is qualitative relative to expected velocity change under perturbation
- Robustness is only demonstrated for transcript-capture/depth perturbations; other noise modalities are not covered
- Results are tied to the specific dataset (GSE207943) and fixed mask/parameters; different masks or hyperparameters may respond differently to downsampling

## Evidence

- [abstract] dynamical model fit with simultaneous velocity and GRN learning: "a dynamical model of gene expression change which is fit with the simultaneous learning of per-cell transcriptional velocities and a governing gene regulatory network"
- [readme] code availability in Jupyter notebook: "All python code and an accompanying Jupyter notebook with demonstrations are available"
- [abstract] benchmarking improvement over existing velocity approaches: "scKINETICS successfully extends and improves existing velocity approaches to generate interpretable, mechanistic models of gene regulatory dynamics"
