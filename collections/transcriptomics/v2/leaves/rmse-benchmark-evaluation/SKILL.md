---
name: rmse-benchmark-evaluation
description: Use when you need to validate that a fitted fragment length distribution
  model generalizes across scRNA-seq datasets, i.e., when fitting Forseti's spline
  model and quantifying held-out prediction accuracy via RMSE.
when_to_use_negative:
- You only have a single scRNA-seq dataset with no held-out test data, so cross-dataset
  generalization RMSE cannot be computed
- You are evaluating Forseti's MLP binding affinity component rather than the fragment
  length distribution (that component is assessed via AUC, not fragment-length RMSE)
- Input fragment length data does not come from paired-end scRNA-seq libraries (e.g.,
  bulk RNA-seq or short-read single-end data), since the spline models the scRNA-seq
  cDNA fragment length distribution
edam_operation: http://edamontology.org/operation_2428
edam_topics:
- http://edamontology.org/topic_0203
tools:
- name: Forseti
  role: Provides the spline-model fitting routine for the fragment length distribution
    and the pre-built model layout that spline_model.pkl must match
  repo: https://github.com/COMBINE-lab/forseti
- name: Python
  role: Language in which the spline model is fit, serialized, and evaluated (import
    forseti package; fitting is done via the forseti-experiments fit_models step)
  repo: https://github.com/COMBINE-lab/forseti
- name: forseti-experiments
  role: Nextflow pipeline whose fit_models step trains the MLP binding affinity model
    and fits the cubic-spline fragment length distribution; evaluation scripts output
    the RMSE metrics
  repo: https://github.com/COMBINE-lab/forseti-experiments
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1093/bioinformatics/btae207
    title: '<tt>Forseti</tt> : a mechanistic and predictive model of the splicing
      status of scRNA-seq reads'
  source_content_hash: sha256:2475d9d2a8b4d6705d090d6897dbd93ac534c0a6d42c4ff997736266dacee3d4
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/COMBINE-lab/forseti
  tools:
  - Forseti
  - Python
  - forseti-experiments
  edam_operation: http://edamontology.org/operation_2428
  edam_topics:
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5555555555555556
n_claims: 18
---

# rmse-benchmark-evaluation

## Summary

Reproduce and benchmark Forseti's cubic-spline cDNA fragment length distribution model by fitting it to paired-end fragment lengths from the eight training datasets, then computing held-out RMSE and 5-fold cross-validation variation to verify cross-dataset generalizability.

## When to use

Use when you need to validate that a fitted fragment length distribution model generalizes across scRNA-seq datasets, i.e., when fitting Forseti's spline model and quantifying held-out prediction accuracy via RMSE.

## When NOT to use

- You only have a single scRNA-seq dataset with no held-out test data, so cross-dataset generalization RMSE cannot be computed
- You are evaluating Forseti's MLP binding affinity component rather than the fragment length distribution (that component is assessed via AUC, not fragment-length RMSE)
- Input fragment length data does not come from paired-end scRNA-seq libraries (e.g., bulk RNA-seq or short-read single-end data), since the spline models the scRNA-seq cDNA fragment length distribution

## Inputs

- Empirical paired-end fragment length distributions (1–1000 bp) from the eight training scRNA-seq datasets listed in Supplementary Table S1
- Empirical fragment length distributions from the two held-out test datasets
- forseti package pre-built spline model layout (for matching serialized model structure)

## Outputs

- spline_model.pkl — serialized cubic-spline fragment length distribution model compatible with the forseti package
- fragment_length_rmse_metrics.json — per-dataset RMSE, mean RMSE, RMSE std, and 5-fold cross-validation variation

## How to apply

Load empirical paired-end fragment length measurements spanning 1–1000 bp from the eight training datasets listed in Supplementary Table S1, and fit a cubic spline to the empirical fragment length distribution using the Forseti spline-model fitting routine in Python. Serialize the fitted distribution as spline_model.pkl, matching the layout of the pre-built model shipped with the forseti package so downstream prediction can load it unchanged. Evaluate the fitted spline against the empirical fragment length distributions of the two held-out test datasets by computing per-dataset RMSE, then run 5-fold cross-validation on the training data to quantify fit variation. Accept the reproduction when mean RMSE across the two test datasets is below 3e-4, RMSE standard deviation is below 1e-4, and 5-fold CV variation is minimal (as reported in Section 3.1 and Fig. 2). The rationale is that a robust spline model must generalize across datasets from different species and tissue types, so benchmarking on held-out datasets rather than the training fit is the correctness criterion.

## Related tools

- **Forseti** (Provides the spline-model fitting routine for the fragment length distribution and the pre-built model layout that spline_model.pkl must match) — https://github.com/COMBINE-lab/forseti
- **Python** (Language in which the spline model is fit, serialized, and evaluated (import forseti package; fitting is done via the forseti-experiments fit_models step)) — https://github.com/COMBINE-lab/forseti
- **forseti-experiments** (Nextflow pipeline whose fit_models step trains the MLP binding affinity model and fits the cubic-spline fragment length distribution; evaluation scripts output the RMSE metrics) — https://github.com/COMBINE-lab/forseti-experiments

## Examples

```
nextflow main.nf -resume --mode fit_models
```

## Evaluation signals

- Mean RMSE across the two held-out test datasets is < 3e-4 (matches Section 3.1 / Fig. 2 report)
- RMSE standard deviation across test datasets is < 1e-4
- 5-fold cross-validation on training data shows minimal variation of the fit
- Serialized spline_model.pkl loads into the forseti package and matches the pre-built model layout
- Fitted distribution covers fragment lengths spanning 1–1000 bp as in the empirical measurements

## Limitations

- The spline is an empirical model of cDNA fragment length in scRNA-seq; it is not a mechanistic model and cannot extrapolate to library-preparation protocols unlike those in the ten training/test experiments
- Fundamentally ambiguous reads (polyA-tail priming with read2 inside the terminal exon, or fragments arising from an exon) cannot be distinguished by Forseti, so held-out evaluation focuses on the distribution model's accuracy rather than full prediction accuracy
- Nextflow pipeline results depend on the datasets sheet (sample_url_sheet.csv); corrupted FASTQ fetches during prepare_data can fail the fit step and require removing processed SRR lines and re-running

## Evidence

- [intro] second, we fit a robust fragment length distribution model that generalizes well across datasets deriving from different species and tissue types.: "second, we fit a robust fragment length distribution model that generalizes well across datasets deriving from different species and tissue types."
- [readme] (2) fit an emperical fragment length distribution using a cubic spline to represent the emperical distribution of the cDNA fragment length in scRNA-seq.: [evidence span withheld — anchor status: external_doc]
- [full_text] The spline model achieves a low RMSE in predicting fragment length probability within the two test datasets (mean RMSE lower than 3 × e−4 and RMSE standard deviation lower than 1 × e−4), indicating high accuracy.: [evidence span withheld — anchor status: not_found]
- [full_text] 5. Run 5-fold cross-validation on the training data to quantify the variation of the fit.: [evidence span withheld — anchor status: not_found]
