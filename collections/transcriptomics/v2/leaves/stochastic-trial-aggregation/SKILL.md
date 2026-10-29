---
name: stochastic-trial-aggregation
description: Use when assessing whether model outputs (e.g., scKINETICS per-cell transcriptional
  velocities) are stable under technical noise, by repeating a stochastic perturbation
  such as downsampling raw counts across multiple independent trials per proportion
  and aggregating deviation statistics.
when_to_use_negative:
- The input is already normalized or processed expression data — downsampling to simulate
  reduced sequencing depth requires raw integer counts
- The perturbation is intended to change the model itself (mask or parameters) rather
  than isolate the effect of the count perturbation
- The goal is to test biological, not technical, reproducibility — this skill only
  simulates technical transcript-capture noise via random count subsampling
edam_topics:
- http://edamontology.org/topic_0203
tools:
- name: scKINETICS
  role: Model refit on each downsampled count matrix with the fixed regulatory mask
    and original parameter settings, producing perturbed velocities for comparison
  repo: http://github.com/dpeerlab/scKINETICS
- name: Jupyter
  role: Notebook environment in which the downsampling trials, scKINETICS refits,
    and cosine-deviation aggregation are executed
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1093/bioinformatics/btad267
    title: 'scKINETICS: inference of regulatory velocity with single-cell transcriptomics
      data'
  source_content_hash: sha256:d17f764537732a6b78effaf86ab179f1e66b3475c7888507f7ed3ecb82ab457e
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - Jupyter
  - scKINETICS
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 6
---

# stochastic-trial-aggregation

## Summary

A robustness-testing pattern for model-based single-cell analyses: repeatedly perturb raw inputs (e.g., random downsampling of raw counts), refit the identical model (scKINETICS with fixed regulatory mask and parameters) on each trial, and aggregate a per-cell deviation metric (cosine deviation of velocities) across trials to quantify sensitivity to the perturbation.

## When to use

Use when assessing whether model outputs (e.g., scKINETICS per-cell transcriptional velocities) are stable under technical noise, by repeating a stochastic perturbation such as downsampling raw counts across multiple independent trials per proportion and aggregating deviation statistics.

## When NOT to use

- The input is already normalized or processed expression data — downsampling to simulate reduced sequencing depth requires raw integer counts
- The perturbation is intended to change the model itself (mask or parameters) rather than isolate the effect of the count perturbation
- The goal is to test biological, not technical, reproducibility — this skill only simulates technical transcript-capture noise via random count subsampling

## Inputs

- Raw integer count matrix (e.g., GSE207943 scRNA-seq raw counts) — downsampling requires raw counts, not normalized data
- Original scKINETICS per-cell velocity estimates from the reference fit (from task_002)
- Fixed regulatory mask used in the original scKINETICS fit
- Original scKINETICS parameter settings (held fixed across all trials)

## Outputs

- velocity_deviation.csv containing trial-level cosine deviations and per-proportion aggregated deviations across the 15 trials

## How to apply

Load the original raw count matrix (GSE207943 in the source study) and the original scKINETICS velocity estimates from the reference fit. For each downsampling proportion, generate 15 independent random subsamples of the counts to the specified percentage of total counts, simulating reduced sequencing depth. Refit scKINETICS on each perturbed count matrix using the same fixed regulatory mask and parameter settings as the original fit, so that only the count perturbation differs between trials. For each trial, compute the cosine deviation between perturbed and original velocities across cells, then aggregate the 15 trial-level deviations within each proportion to summarize sensitivity to depth reduction. Save trial-level and aggregated deviations to velocity_deviation.csv and judge success qualitatively: perturbed-fit velocities should show minimal cosine deviation from the originals, since no numeric threshold is stated in the source.

## Related tools

- **scKINETICS** (Model refit on each downsampled count matrix with the fixed regulatory mask and original parameter settings, producing perturbed velocities for comparison) — http://github.com/dpeerlab/scKINETICS
- **Jupyter** (Notebook environment in which the downsampling trials, scKINETICS refits, and cosine-deviation aggregation are executed)

## Evaluation signals

- Each downsampling proportion has exactly 15 independent random subsample trials
- Every trial refit uses the same regulatory mask and parameter settings as the original scKINETICS fit — only the counts differ
- Per-trial cosine deviation between perturbed and original velocities is small relative to expected velocity-change magnitude under perturbation (qualitative check; no numeric threshold in source)
- velocity_deviation.csv contains both trial-level deviations and per-proportion aggregations
- Deviation trend across proportions is inspectable (higher downsampling proportions removed should not silently produce larger-than-expected drift without comment)

## Limitations

- No numeric deviation threshold is stated in the source text, so validation is qualitative — accept that deviation remains small relative to expected velocity-change magnitude
- Random subsampling makes trial results stochastic; 15 trials per proportion are used to aggregate away sampling variability, but single-trial results should not be over-interpreted
- The test isolates only transcript-capture (depth) noise; other noise sources (dropout structure, batch effects) are not simulated by uniform downsampling

## Evidence

- [abstract] simultaneous learning of per-cell transcriptional velocities and a governing gene regulatory network: "a dynamical model of gene expression change which is fit with the simultaneous learning of per-cell transcriptional velocities and a governing gene regulatory network"
- [readme] demo covers input generation, GRN construction, EM estimation: [evidence span withheld — anchor status: external_doc]
- [intro] EM learns regulator-target impacts (the fit refit in each trial): "This is accomplished through an expectation-maximization approach derived to learn the impact of each regulator on its target genes"
