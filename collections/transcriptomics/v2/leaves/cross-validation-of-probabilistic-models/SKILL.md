---
name: cross-validation-of-probabilistic-models
description: Use when a probabilistic distribution model (e.g. the cubic-spline fragment
  length model in Forseti) must be trained on empirical measurements and its accuracy/variation
  on held-out data must be quantified with RMSE and k-fold cross-validation.
when_to_use_negative:
- The goal is predicting splicing status of scRNA-seq reads (that is the downstream
  forseti_predictor step, which consumes the fitted model, not builds it)
- The target model is the oligo(dT) primer binding affinity MLP trained on priming
  windows, not the fragment length distribution
- Input reads are single-end or lack empirical paired-end fragment lengths spanning
  the 1–1000 bp range needed to fit the spline
edam_topics:
- http://edamontology.org/topic_3474
tools:
- name: Forseti
  role: Provides the spline-model fitting routine and the pre-built spline_model.pkl
    layout into which the fitted fragment length distribution is serialized
  repo: https://github.com/COMBINE-lab/forseti
- name: Python
  role: Language in which Forseti is implemented and in which the spline fitting,
    RMSE evaluation, and 5-fold cross-validation are executed
  repo: https://github.com/COMBINE-lab/forseti
- name: Nextflow (forseti-experiments)
  role: Pipeline wrapper whose 'fit_models' step trains the binding affinity MLP and
    fits the empirical fragment length distribution via cubic spline
  repo: https://github.com/COMBINE-lab/forseti-experiments
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1093/bioinformatics/btae207
    title: '<tt>Forseti</tt> : a mechanistic and predictive model of the splicing
      status of scRNA-seq reads'
  source_content_hash: sha256:3dacd8e35e4fd2d23c9fc2d00e160615944ea5f2c328c7fef8ead67dd854a1f6
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/COMBINE-lab/forseti
  tools:
  - Forseti
  - Nextflow (forseti-experiments)
  - Python
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3474
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 9
---

# cross-validation-of-probabilistic-models

## Summary

Fit and validate the Forseti empirical cDNA fragment length distribution using a cubic spline over paired-end fragment lengths (1–1000 bp), then quantify generalizability via held-out test datasets and 5-fold cross-validation with RMSE metrics.

## When to use

Use when a probabilistic distribution model (e.g. the cubic-spline fragment length model in Forseti) must be trained on empirical measurements and its accuracy/variation on held-out data must be quantified with RMSE and k-fold cross-validation.

## When NOT to use

- The goal is predicting splicing status of scRNA-seq reads (that is the downstream forseti_predictor step, which consumes the fitted model, not builds it)
- The target model is the oligo(dT) primer binding affinity MLP trained on priming windows, not the fragment length distribution
- Input reads are single-end or lack empirical paired-end fragment lengths spanning the 1–1000 bp range needed to fit the spline

## Inputs

- Empirical paired-end fragment length measurements spanning 1–1000 bp from the eight training scRNA-seq datasets (Supplementary Table S1)
- Empirical fragment length distributions of the two held-out test datasets
- Forseti spline-model fitting routine (Python) and forseti pre-built model layout

## Outputs

- spline_model.pkl — serialized cubic-spline fragment length distribution matching the forseti package model layout
- fragment_length_rmse_metrics.json — per-dataset RMSE, mean RMSE, RMSE std, and 5-fold cross-validation variation

## How to apply

Load empirical paired-end fragment length measurements spanning 1–1000 bp from the eight training datasets (Supplementary Table S1) and fit a cubic spline to the empirical fragment length distribution using the Forseti spline-model fitting routine in Python. Serialize the fitted distribution as spline_model.pkl, matching the pre-built model layout of the forseti package so it can be consumed directly by the predictor. Evaluate the fitted spline against the empirical fragment length distributions of the two held-out test datasets, computing per-dataset RMSE, then run 5-fold cross-validation on the training data to quantify fit variation. Write per-dataset RMSE, mean RMSE, RMSE std, and cross-validation variation to fragment_length_rmse_metrics.json. Accept the result only when mean RMSE across test datasets is < 3e-4, RMSE std is < 1e-4, and the 5-fold cross-validation variation is minimal, as reported in Section 3.1 and Fig. 2; the forseti-experiments pipeline implements this inside its 'fit_models' Nextflow step alongside MLP training of the binding affinity model.

## Related tools

- **Forseti** (Provides the spline-model fitting routine and the pre-built spline_model.pkl layout into which the fitted fragment length distribution is serialized) — https://github.com/COMBINE-lab/forseti
- **Python** (Language in which Forseti is implemented and in which the spline fitting, RMSE evaluation, and 5-fold cross-validation are executed) — https://github.com/COMBINE-lab/forseti
- **Nextflow (forseti-experiments)** (Pipeline wrapper whose 'fit_models' step trains the binding affinity MLP and fits the empirical fragment length distribution via cubic spline) — https://github.com/COMBINE-lab/forseti-experiments

## Examples

```
nextflow main.nf -resume --mode fit_models
```

## Evaluation signals

- Mean RMSE across the two test datasets is below 3e-4 (reported as lower than 3 × e−4 in Section 3.1 and Fig. 2)
- RMSE standard deviation across test datasets is below 1e-4
- 5-fold cross-validation variation on the eight training datasets is minimal
- spline_model.pkl loads into the forseti package with the same layout as its pre-built model
- fragment_length_rmse_metrics.json contains per-dataset RMSE, mean RMSE, RMSE std, and cross-validation variation keys

## Limitations

- The RMSE thresholds (mean < 3e-4, std < 1e-4) were validated only on the two specific held-out test datasets; new species/tissue datasets should re-run cross-validation before reuse
- As a mechanistic model, Forseti cannot distinguish fundamentally ambiguous reads (polyA-tail priming with read2 inside the terminal exon vs. fragments entirely within an exon), so the fragment length model alone cannot resolve such cases
- The full forseti-experiments pipeline requires ~3 TB of disk space, and Nextflow may fetch corrupted FASTQ files at the prepare_data step, requiring manual removal of successfully processed SRR lines from input_files/sample_url_sheet.csv before re-running

## Evidence

- [results] The spline model achieves a low RMSE in predicting fragment length probability within the two test datasets (mean RMSE lower than 3 × e−4 and RMSE standard deviation lower than 1 × e−4): [evidence span withheld — anchor status: not_found]
- [intro] second, we fit a robust fragment length distribution model that generalizes well across datasets deriving from different species and tissue types.: "second, we fit a robust fragment length distribution model that generalizes well across datasets deriving from different species and tissue types."
- [readme] fit an emperical fragment length distribution using a cubic spline to represent the emperical distribution of the cDNA fragment length in scRNA-seq.: [evidence span withheld — anchor status: external_doc]
- [readme] we have included the trained binding affinity model, and the output of the evaluation scripts in the workflow_output directory.: [evidence span withheld — anchor status: external_doc]
