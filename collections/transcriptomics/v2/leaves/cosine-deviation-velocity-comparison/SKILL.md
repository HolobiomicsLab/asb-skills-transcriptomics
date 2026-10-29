---
name: cosine-deviation-velocity-comparison
description: Use when assessing whether velocity estimates from scKINETICS (or a comparable
  EM-fitted velocity model) remain stable under technical noise, such as reduced transcript-capture
  depth simulated by random downsampling of raw counts.
when_to_use_negative:
- Comparing velocities fitted under different regulatory masks or model parameters,
  where differences do not isolate the effect of count perturbation
- Evaluating robustness of the learned gene regulatory network itself rather than
  the per-cell velocity vectors
edam_operation: http://edamontology.org/operation_2945
edam_topics:
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_3308
tools:
- name: scKINETICS
  role: Refit the dynamical velocity model on each perturbed count matrix using a
    fixed regulatory mask and parameters
  repo: http://github.com/dpeerlab/scKINETICS
- name: Jupyter
  role: Environment for running the downsampling, refitting, and cosine-deviation
    aggregation analysis
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1093/bioinformatics/btad267
    title: 'scKINETICS: inference of regulatory velocity with single-cell transcriptomics
      data'
  source_content_hash: sha256:10486cfb17a22b6d99323f311128a161c880da5c6c5d216483fdbd41e64eb3d2
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - Jupyter
  - scKINETICS
  edam_operation: http://edamontology.org/operation_2945
  edam_topics:
  - http://edamontology.org/topic_0203
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8888888888888888
n_claims: 9
---

# cosine-deviation-velocity-comparison

## Summary

Quantifies robustness of single-cell velocity estimates by refitting scKINETICS on downsampled raw counts (e.g. 15 random trials per capture proportion) and measuring the cosine deviation between perturbed-fit velocities and the original velocities across cells.

## When to use

Use when assessing whether velocity estimates from scKINETICS (or a comparable EM-fitted velocity model) remain stable under technical noise, such as reduced transcript-capture depth simulated by random downsampling of raw counts.

## When NOT to use

- Comparing velocities fitted under different regulatory masks or model parameters, where differences do not isolate the effect of count perturbation
- Evaluating robustness of the learned gene regulatory network itself rather than the per-cell velocity vectors

## Inputs

- Raw single-cell count matrix (e.g. GSE207943)
- Original scKINETICS velocity estimates from the unperturbed fit
- Fixed regulatory mask used in the original fit
- Fixed scKINETICS parameter settings from the original fit
- Downsampling proportions (percentages of total counts)

## Outputs

- Trial-level cosine deviations between perturbed and original velocities
- Aggregated cosine deviations per downsampling proportion (e.g. velocity_deviation.csv)

## How to apply

Load the original raw count matrix (e.g. GSE207943) together with the original scKINETICS velocity estimates from a prior fit. For each downsampling proportion, generate 15 independent random subsamples of counts to a specified percentage of total counts to simulate reduced sequencing depth, then refit scKINETICS on each perturbed matrix using the same fixed regulatory mask and parameter settings so only the count perturbation differs. For each trial, compute the cosine deviation between perturbed velocities and original velocities across all cells; aggregate deviations across the 15 trials within each proportion to summarize sensitivity to depth reduction. Save trial-level and aggregated values (e.g. to velocity_deviation.csv); since no numeric threshold is stated, accept qualitatively that deviation stays small relative to the expected magnitude of velocity change under perturbation.

## Related tools

- **scKINETICS** (Refit the dynamical velocity model on each perturbed count matrix using a fixed regulatory mask and parameters) — http://github.com/dpeerlab/scKINETICS
- **Jupyter** (Environment for running the downsampling, refitting, and cosine-deviation aggregation analysis)

## Examples

```
cos_dev = 1 - np.dot(v_orig, v_pert) / (np.linalg.norm(v_orig) * np.linalg.norm(v_pert))
```

## Evaluation signals

- Perturbed-fit velocities show minimal cosine deviation from original velocities, consistent with the reported downsampling robustness finding
- Deviations are aggregated across 15 independent random subsamples per downsampling proportion
- Only the count perturbation differs from the original fit — regulatory mask and parameters are identical
- Trial-level and aggregated deviations are both recorded (e.g. velocity_deviation.csv)

## Limitations

- No numeric threshold is stated for 'minimal' deviation, so the robustness judgment is qualitative relative to expected velocity change magnitude
- Downsampling raw counts simulates only reduced transcript-capture depth; other technical noise modes are not covered

## Evidence

- [abstract] a dynamical model of gene expression change which is fit with the simultaneous learning of per-cell transcriptional velocities and a governing gene regulatory network: "a dynamical model of gene expression change which is fit with the simultaneous learning of per-cell transcriptional velocities and a governing gene regulatory network"
- [readme] The demo shows the main functionalities of the package including input file generation, GRN construction, EM estimation, visualizations, and the TF ablation experiment.: [evidence span withheld — anchor status: external_doc]
