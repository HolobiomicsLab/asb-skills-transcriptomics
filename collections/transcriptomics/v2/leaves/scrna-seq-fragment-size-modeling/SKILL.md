---
name: scrna-seq-fragment-size-modeling
description: Use when paired-end scRNA-seq fragment lengths are available and you
  need a reusable empirical fragment length distribution (e.
when_to_use_negative:
- Data are single-end or otherwise lack paired-end alignments from which fragment
  lengths can be computed
- The goal is to model oligo(dT) primer binding affinity — that is the separate multi-layer
  perceptron (MLP) component of Forseti, not the fragment length distribution
- Fragment length distributions must already be available in a compatible form; re-fitting
  is not needed if the pre-built spline model in the forseti package suffices
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: Forseti
  role: Provides the spline-model fitting routine used to fit the cubic spline to
    the empirical fragment length distribution
  repo: https://github.com/COMBINE-lab/forseti
- name: Python
  role: Implementation language for the Forseti package and the spline fitting/serialization
    of spline_model.pkl
  repo: https://github.com/COMBINE-lab/forseti
- name: forseti-experiments
  role: Nextflow pipeline whose 'fit_models' step computes fragment lengths and fits
    the empirical fragment length distribution via cubic spline
  repo: https://github.com/COMBINE-lab/forseti-experiments
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1093/bioinformatics/btae207
    title: '<tt>Forseti</tt> : a mechanistic and predictive model of the splicing
      status of scRNA-seq reads'
  source_content_hash: sha256:f530124a69586fbf0ff45ead83d1fc8683b6f00dc830d2c1422f42da47f2be51
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/COMBINE-lab/forseti
  tools:
  - Forseti
  - Python
  - forseti-experiments
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7142857142857143
n_claims: 7
---

# scrna-seq-fragment-size-modeling

## Summary

Fit a cubic-spline model to the empirical paired-end cDNA fragment length distribution (spanning 1–1000 bp) from scRNA-seq training datasets so that fragment length probabilities can be predicted for held-out datasets and reused as a component of splicing-status prediction in Forseti.

## When to use

Use when paired-end scRNA-seq fragment lengths are available and you need a reusable empirical fragment length distribution (e.g. as the fragment-length component of a mechanistic splicing-status predictor), or to test how well such a model generalizes across datasets from different species and tissue types.

## When NOT to use

- Data are single-end or otherwise lack paired-end alignments from which fragment lengths can be computed
- The goal is to model oligo(dT) primer binding affinity — that is the separate multi-layer perceptron (MLP) component of Forseti, not the fragment length distribution
- Fragment length distributions must already be available in a compatible form; re-fitting is not needed if the pre-built spline model in the forseti package suffices

## Inputs

- Empirical paired-end fragment length measurements spanning 1–1000 bp from eight scRNA-seq training datasets (Supplementary Table S1)
- Empirical fragment length distributions from two held-out test datasets for evaluation

## Outputs

- spline_model.pkl — serialized cubic-spline fragment length distribution matching the forseti package model layout
- fragment_length_rmse_metrics.json — per-dataset RMSE, mean RMSE, RMSE std, and 5-fold cross-validation variation

## How to apply

Load empirical paired-end fragment length measurements spanning 1–1000 bp from the eight training datasets listed in Supplementary Table S1, then fit a cubic spline to the empirical fragment length distribution using the Forseti spline-model fitting routine in Python (part of the 'fit_models' step of the forseti-experiments Nextflow pipeline). Serialize the fitted model as spline_model.pkl so it matches the pre-built model layout of the forseti package. Evaluate the spline against the empirical fragment length distributions of the two held-out test datasets by computing per-dataset RMSE, and run 5-fold cross-validation on the training data to quantify fit variation. Accept the result only when mean RMSE across test datasets is < 3e-4, RMSE std is < 1e-4, and cross-validation variation is minimal, as reported in Section 3.1 and Fig. 2.

## Related tools

- **Forseti** (Provides the spline-model fitting routine used to fit the cubic spline to the empirical fragment length distribution) — https://github.com/COMBINE-lab/forseti
- **Python** (Implementation language for the Forseti package and the spline fitting/serialization of spline_model.pkl) — https://github.com/COMBINE-lab/forseti
- **forseti-experiments** (Nextflow pipeline whose 'fit_models' step computes fragment lengths and fits the empirical fragment length distribution via cubic spline) — https://github.com/COMBINE-lab/forseti-experiments

## Examples

```
nextflow main.nf -resume --mode fit_models
```

## Evaluation signals

- Mean RMSE across the two held-out test datasets is < 3e-4 when predicting fragment length probability
- RMSE standard deviation across test datasets is < 1e-4
- 5-fold cross-validation on the training data shows minimal variation of the fit (consistent with Section 3.1 and Fig. 2)
- spline_model.pkl loads in the forseti package and matches the pre-built model layout
- Fitted spline closely tracks the empirical distribution over the 1–1000 bp fragment length range

## Limitations

- The spline is fit to paired-end scRNA-seq fragment lengths in the 1–1000 bp range; behavior outside this range or on single-end protocols is unsupported
- Running the full forseti-experiments pipeline requires ~3 TB of disk space and a SLURM cluster (adaptable), and Nextflow cannot guarantee fetched FASTQ files are uncorrupted (github.com/nextflow-io/nextflow#2491), sometimes forcing partial re-runs by editing input_files/sample_url_sheet.csv
- The model is validated on a small number of held-out datasets (two test datasets), so generalization to new species/tissue combinations should be re-checked

## Evidence

- [intro] second, we fit a robust fragment length distribution model that generalizes well across datasets deriving from different species and tissue types.: "second, we fit a robust fragment length distribution model that generalizes well across datasets deriving from different species and tissue types."
- [readme] fit an emperical fragment length distribution using a cubic spline to represent the emperical distribution of the cDNA fragment length in scRNA-seq.: [evidence span withheld — anchor status: external_doc]
- [readme] we process the sequencing reads to compute the fragment length of the reads, and obtain the potentail priming windows of oligo(dT) primers indicated by the paired-end alignments.: [evidence span withheld — anchor status: external_doc]
- [readme] we fetch the sequencing reads from 10 selected paired-end scRNA-seq experiments and their corresponding reference sets: [evidence span withheld — anchor status: external_doc]
