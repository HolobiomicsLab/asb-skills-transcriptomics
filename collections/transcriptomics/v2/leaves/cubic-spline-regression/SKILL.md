---
name: cubic-spline-regression
description: Use when you need an empirical cDNA fragment length distribution model
  from paired-end scRNA-seq fragment lengths (Forseti step 'fit_models'), or to test
  whether such a spline generalizes across species and tissue types.
when_to_use_negative:
- You need primer binding affinity rather than fragment lengths — that is the separate
  multi-layer perceptron (MLP) trained on oligo(dT) priming windows
- Your reads are single-end or otherwise lack paired-end fragment length measurements
  spanning the 1–1000 bp range
- You want the final splicing-status prediction of reads — the spline is only one
  component; scoring putative fragments against priming sites is a separate step
edam_operation: http://edamontology.org/operation_3659
edam_topics:
- http://edamontology.org/topic_3474
tools:
- name: Forseti
  role: Provides the spline-model fitting routine and the pre-built spline_model.pkl
    layout the fitted distribution must match
  repo: https://github.com/COMBINE-lab/forseti
- name: Python
  role: Implementation language for importing the forseti package and fitting/serializing/evaluating
    the cubic spline
  repo: https://github.com/COMBINE-lab/forseti
- name: forseti-experiments
  role: Nextflow pipeline (fit_models mode) containing the code that fits and evaluates
    the fragment length distribution
  repo: https://github.com/COMBINE-lab/forseti-experiments
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1093/bioinformatics/btae207
    title: '<tt>Forseti</tt> : a mechanistic and predictive model of the splicing
      status of scRNA-seq reads'
  source_content_hash: sha256:5ba8309172a0f595d17d9153c43152e06457d65dce8e014e54aed0655cc1af7b
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/COMBINE-lab/forseti
  tools:
  - Forseti
  - Python
  - forseti-experiments
  edam_operation: http://edamontology.org/operation_3659
  edam_topics:
  - http://edamontology.org/topic_3474
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.75
n_claims: 12
---

# cubic-spline-regression

## Summary

Fit a cubic spline to the empirical paired-end cDNA fragment length distribution (1–1000 bp) from scRNA-seq training datasets, and validate its generalization on held-out test datasets via RMSE and 5-fold cross-validation, as done in the Forseti fragment-length distribution model.

## When to use

Use when you need an empirical cDNA fragment length distribution model from paired-end scRNA-seq fragment lengths (Forseti step 'fit_models'), or to test whether such a spline generalizes across species and tissue types.

## When NOT to use

- You need primer binding affinity rather than fragment lengths — that is the separate multi-layer perceptron (MLP) trained on oligo(dT) priming windows
- Your reads are single-end or otherwise lack paired-end fragment length measurements spanning the 1–1000 bp range
- You want the final splicing-status prediction of reads — the spline is only one component; scoring putative fragments against priming sites is a separate step

## Inputs

- Empirical paired-end fragment length measurements spanning 1–1000 bp from eight training scRNA-seq datasets (Supplementary Table S1)
- Empirical fragment length distributions of two held-out test datasets
- Spline model layout/schema of the pre-built model in the forseti Python package

## Outputs

- spline_model.pkl — serialized cubic-spline fragment length distribution model matching the forseti package model layout
- fragment_length_rmse_metrics.json — per-dataset RMSE, mean RMSE, RMSE std, and 5-fold cross-validation variation

## How to apply

Load empirical paired-end fragment length measurements spanning 1–1000 bp from the eight training datasets listed in Supplementary Table S1, then fit a cubic spline to the empirical fragment length distribution using the Forseti spline-model fitting routine in Python. Serialize the fitted distribution as spline_model.pkl so its layout matches the pre-built model shipped with the forseti package. Evaluate the spline against the empirical fragment length distributions of the two held-out test datasets, computing per-dataset RMSE, mean RMSE, and RMSE standard deviation, and run 5-fold cross-validation on the training data to quantify fit variation. Accept the fit when mean RMSE across test datasets is < 3e-4, RMSE std is < 1e-4, and 5-fold cross-validation variation is minimal (Section 3.1, Fig. 2). This spline is one of Forseti's two trained components and is deliberately fit to generalize across datasets from different species and tissue types.

## Related tools

- **Forseti** (Provides the spline-model fitting routine and the pre-built spline_model.pkl layout the fitted distribution must match) — https://github.com/COMBINE-lab/forseti
- **Python** (Implementation language for importing the forseti package and fitting/serializing/evaluating the cubic spline) — https://github.com/COMBINE-lab/forseti
- **forseti-experiments** (Nextflow pipeline (fit_models mode) containing the code that fits and evaluates the fragment length distribution) — https://github.com/COMBINE-lab/forseti-experiments

## Examples

```
nextflow main.nf -resume --mode fit_models
```

## Evaluation signals

- Mean RMSE across the two held-out test datasets is < 3e-4
- RMSE standard deviation across test datasets is < 1e-4
- 5-fold cross-validation on the eight training datasets shows minimal variation of the fit
- Serialized spline_model.pkl matches the pre-built model layout of the forseti package
- Fitted distribution reproduces the reported results in Section 3.1 and Fig. 2 of the Forseti paper

## Limitations

- The spline is trained on ten selected paired-end scRNA-seq experiments (eight train / two test) and its 5-fold cross-validation variation must stay minimal; fits degrading on held-out species/tissue indicate overfitting
- Fitted fragment length support spans 1–1000 bp; fragment lengths outside this empirical range are not modeled
- As a mechanistic model, Forseti cannot distinguish fundamentally ambiguous reads (polyA-tail priming with read2 in the terminal exon vs. fragments arising from an exon) — the spline model does not resolve this ambiguity

## Evidence

- [intro] second, we fit a robust fragment length distribution model that generalizes well across datasets deriving from different species and tissue types.: "second, we fit a robust fragment length distribution model that generalizes well across datasets deriving from different species and tissue types."
- [readme] fit an emperical fragment length distribution using a cubic spline to represent the emperical distribution of the cDNA fragment length in scRNA-seq: [evidence span withheld — anchor status: external_doc]
- [results] The spline model achieves a low RMSE in predicting fragment length probability within the two test datasets: [evidence span withheld — anchor status: not_found]
