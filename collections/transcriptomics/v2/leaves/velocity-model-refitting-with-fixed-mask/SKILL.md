---
name: velocity-model-refitting-with-fixed-mask
description: Use when you need to test whether inferred transcriptional velocities
  (e.g., from scKINETICS on raw UMI counts such as GSE207943) are stable under technical
  noise in gene expression, specifically perturbed transcript capture rates from downsampling.
when_to_use_negative:
- When testing biological (not technical) robustness — the perturbation here models
  transcript-capture noise via downsampling of raw counts, not biological variability
  between samples
- When the regulatory mask or model parameters should be re-optimized per fit — this
  skill requires a fixed mask and fixed parameters so only the count perturbation
  differs
- When velocity estimates do not yet exist for an unperturbed reference fit — the
  cosine deviation is always measured against the original fit
edam_topics:
- http://edamontology.org/topic_0203
tools:
- name: scKINETICS
  role: Dynamical model refit on each downsampled count matrix with the same fixed
    regulatory mask and parameter settings as the original fit; learns per-cell velocities
    via expectation-maximization
  repo: http://github.com/dpeerlab/scKINETICS
- name: Jupyter
  role: Notebook environment used to run the downsampling, refitting, and cosine-deviation
    aggregation workflow
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1093/bioinformatics/btad267
    title: 'scKINETICS: inference of regulatory velocity with single-cell transcriptomics
      data'
  source_content_hash: sha256:3236d8317b71b09ec16290b2c37334c071f15e01dd4b07c2b8a17123aaf1b71a
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
yes_rate: 0.8888888888888888
n_claims: 9
---

# velocity-model-refitting-with-fixed-mask

## Summary

A robustness-testing skill for dynamical velocity models: perturb raw single-cell counts (e.g., random downsampling to simulate reduced sequencing depth), refit scKINETICS with the same fixed regulatory mask and parameter settings, and quantify how far perturbed per-cell velocities deviate from the original fit via cosine deviation.

## When to use

Use when you need to test whether inferred transcriptional velocities (e.g., from scKINETICS on raw UMI counts such as GSE207943) are stable under technical noise in gene expression, specifically perturbed transcript capture rates from downsampling.

## When NOT to use

- When testing biological (not technical) robustness — the perturbation here models transcript-capture noise via downsampling of raw counts, not biological variability between samples
- When the regulatory mask or model parameters should be re-optimized per fit — this skill requires a fixed mask and fixed parameters so only the count perturbation differs
- When velocity estimates do not yet exist for an unperturbed reference fit — the cosine deviation is always measured against the original fit

## Inputs

- Raw single-cell count matrix (GSE207943)
- Original scKINETICS velocity estimates from the reference fit (task_002)
- Fixed regulatory mask used in the original scKINETICS fit
- Original scKINETICS parameter settings
- List of downsampling proportions (percentages of total counts)

## Outputs

- Trial-level cosine deviations between perturbed and original velocities (15 trials per proportion)
- Aggregated cosine deviations per downsampling proportion
- velocity_deviation.csv containing trial-level and aggregated deviations

## How to apply

Load the original raw count matrix (GSE207943) and the original scKINETICS velocity estimates from the reference fit. For each downsampling proportion, generate 15 independent random subsamples of the counts to the specified percentage of total counts, simulating reduced sequencing depth. Refit scKINETICS on each perturbed matrix using the identical fixed regulatory mask and parameter settings as the original expectation-maximization fit, so the only varying factor is the count perturbation — this isolates sensitivity of the learned per-cell velocities to capture-rate noise. For each trial, compute the cosine deviation between perturbed and original velocities across cells, then aggregate the 15 trials per proportion to summarize sensitivity. Write trial-level and aggregated deviations to velocity_deviation.csv. With no numeric threshold in the source, validate qualitatively: deviation should remain small relative to the magnitude of velocity change expected under perturbation.

## Related tools

- **scKINETICS** (Dynamical model refit on each downsampled count matrix with the same fixed regulatory mask and parameter settings as the original fit; learns per-cell velocities via expectation-maximization) — http://github.com/dpeerlab/scKINETICS
- **Jupyter** (Notebook environment used to run the downsampling, refitting, and cosine-deviation aggregation workflow)

## Examples

```
from scipy.spatial.distance import cosine; dev = 1 - cosine(v_original, v_downsampled_trial)  # per-trial cosine deviation, saved to velocity_deviation.csv
```

## Evaluation signals

- Perturbed-fit velocities show minimal cosine deviation from the original velocities under downsampling (qualitative, since no numeric threshold is stated in the source text)
- Each downsampling proportion has exactly 15 independent random subsample trials, isolating only the count perturbation (fixed mask, fixed parameters)
- velocity_deviation.csv contains both trial-level and aggregated deviations for every proportion tested
- Deviation remains small relative to the magnitude of velocity change expected under perturbation
- The same regulatory mask and parameter settings used in the original fit are confirmed to be reused across all refits

## Limitations

- No numeric threshold for acceptable cosine deviation is stated in the source; robustness is accepted only qualitatively
- Robustness conclusions are specific to transcript-capture perturbation via random downsampling of raw counts; other noise modes (e.g., dropout patterns, batch effects) are not tested
- Results depend on the specific dataset (GSE207943) and on keeping the regulatory mask and parameters fixed; refitting with a re-learned mask would confound the perturbation effect

## Evidence

- [abstract] dynamical model refit rationale: "a dynamical model of gene expression change which is fit with the simultaneous learning of per-cell transcriptional velocities and a governing gene regulatory network"
- [readme] EM estimation step: "This is accomplished through an expectation-maximization approach derived to learn the impact of each regulator on its target genes, leveraging biologically-motivated priors"
- [readme] demo scope: [evidence span withheld — anchor status: external_doc]
