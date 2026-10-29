---
name: single-cell-count-matrix-downsampling
description: Use when you need to test stability of a fitted single-cell dynamical
  model (e.g. scKINETICS velocities) against technical noise in transcript capture
  by perturbing raw counts at reduced sequencing depths and refitting.
when_to_use_negative:
- When the question is biological variation across samples or conditions rather than
  technical robustness of a single fit to sequencing depth.
- When you intend to vary the regulatory mask or model parameters along with the counts
  — this protocol requires those to stay fixed so only the count perturbation differs.
- When the input is not raw counts (e.g. already normalized or feature-selected matrices),
  since downsampling must operate on raw count totals to simulate capture-rate loss.
edam_topics:
- http://edamontology.org/topic_0203
tools:
- name: scKINETICS
  role: Dynamical model refit on each downsampled count matrix with the original regulatory
    mask and parameters to produce perturbed velocities
  repo: http://github.com/dpeerlab/scKINETICS
- name: Jupyter
  role: Python/Jupyter environment used to run the downsampling trials, compute cosine
    deviations, and export velocity_deviation.csv
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1093/bioinformatics/btad267
    title: 'scKINETICS: inference of regulatory velocity with single-cell transcriptomics
      data'
  source_content_hash: sha256:492e3338ef372bc323a00a2216e1f990f54136e147488e9e7fdcbb2de2ce44ec
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
yes_rate: 0.9
n_claims: 10
---

# single-cell-count-matrix-downsampling

## Summary

A robustness-testing skill that simulates reduced sequencing depth by randomly downsampling raw single-cell count matrices to several percentages of total counts (e.g. 15 trials per proportion) and refitting a fixed-configuration model — here scKINETICS velocity — to quantify how perturbed estimates deviate from the original fit.

## When to use

Use when you need to test stability of a fitted single-cell dynamical model (e.g. scKINETICS velocities) against technical noise in transcript capture by perturbing raw counts at reduced sequencing depths and refitting.

## When NOT to use

- When the question is biological variation across samples or conditions rather than technical robustness of a single fit to sequencing depth.
- When you intend to vary the regulatory mask or model parameters along with the counts — this protocol requires those to stay fixed so only the count perturbation differs.
- When the input is not raw counts (e.g. already normalized or feature-selected matrices), since downsampling must operate on raw count totals to simulate capture-rate loss.

## Inputs

- Original raw single-cell count matrix (e.g. GSE207943 scRNA-seq counts)
- Original scKINETICS velocity estimates from the baseline model fit
- Fixed regulatory mask used in the original scKINETICS fit
- Original scKINETICS parameter settings
- Downsampling proportions (percentages of total counts) to test

## Outputs

- Per-trial refit velocity estimates for each downsampled count matrix
- Cosine deviation values between perturbed and original velocities across cells
- velocity_deviation.csv containing trial-level and aggregated (per-proportion) deviations

## How to apply

Load the original raw count matrix (e.g. GSE207943) together with the original scKINETICS velocity estimates from the baseline fit. For each downsampling proportion, generate 15 independent random subsamples of the counts to the specified percentage of total counts, simulating reduced sequencing depth. Refit scKINETICS on each perturbed count matrix using the same fixed regulatory mask and parameter settings as the original fit, so only the count perturbation differs between runs. For each trial, compute the cosine deviation between the perturbed velocities and the original velocities across cells, then aggregate deviations across the 15 trials within each proportion to summarize sensitivity to depth reduction. Save trial-level and aggregated deviations to velocity_deviation.csv.

## Related tools

- **scKINETICS** (Dynamical model refit on each downsampled count matrix with the original regulatory mask and parameters to produce perturbed velocities) — http://github.com/dpeerlab/scKINETICS
- **Jupyter** (Python/Jupyter environment used to run the downsampling trials, compute cosine deviations, and export velocity_deviation.csv)

## Evaluation signals

- Refit scKINETICS on every perturbed matrix with the identical fixed regulatory mask and parameter settings as the original fit — any deviation from original velocities must be attributable to the count perturbation alone.
- Exactly 15 independent random subsamples per downsampling proportion are generated and evaluated.
- Trial-level cosine deviations, aggregated within each proportion, remain small relative to the expected magnitude of velocity change under perturbation (qualitative acceptance, since the source text states no numeric threshold).
- velocity_deviation.csv contains both trial-level and per-proportion aggregated deviation values with a row per trial and proportion.
- All comparisons are against the original velocity estimates computed across the same cells.

## Limitations

- The source text states no numeric acceptance threshold for cosine deviation, so validation is qualitative ('deviation remains small relative to the magnitude of velocity change expected under perturbation').
- Robustness is demonstrated only for transcript-capture downsampling perturbations; conclusions may not transfer to other technical noise sources.
- No changelog was found for the scKINETICS repository, so version drift between the original and refit runs must be checked manually.

## Evidence

- [other] workflow: cosine deviation protocol: [evidence span withheld — anchor status: not_found]
- [other] workflow: fixed configuration: [evidence span withheld — anchor status: not_found]
- [abstract] article: model definition: "a dynamical model of gene expression change which is fit with the simultaneous learning of per-cell transcriptional velocities and a governing gene regulatory network"
- [abstract] article: EM and priors: [evidence span withheld — anchor status: not_found]
- [readme] README: repo availability: "All python code and an accompanying Jupyter notebook with demonstrations are available"
