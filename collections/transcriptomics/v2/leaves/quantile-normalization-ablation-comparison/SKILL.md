---
name: quantile-normalization-ablation-comparison
description: Use when building or validating a Ciclops cross-platform transfer model
  on transcriptomic datasets (e.g., RNA-seq train, microarray test) and you need to
  decide whether quantile normalization should be kept on or skipped via --no_quantile.
when_to_use_negative:
- Datasets that are already jointly normalized during preprocessing, where the protocol
  does not require quantile normalization
- Runs that return NaN evaluation metrics or AUPRC confidence intervals spanning [0,1];
  these must be flagged unreliable instead of compared
- Non-transcriptomic or single-platform tasks where no cross-platform transferring
  test exists to evaluate the normalization effect
edam_operation: http://edamontology.org/operation_2928
edam_topics:
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_0091
tools:
- name: Ciclops
  role: Pipeline run twice (default vs --no_quantile) to produce paired cross-validation
    and transferring-test metrics
  repo: https://github.com/GuanLab/ciclops.git
- name: Python
  role: CLI entry point for invoking ciclops and parsing ./performance/ metric tables
- name: pip
  role: Installs the ciclops package via 'pip install ciclops'
- name: LightGBM
  role: Default model type ('lgb') used so both ablation runs are comparable
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1016/j.xpro.2022.101583
    title: Protocol for using Ciclops to build models trained on cross-platform transcriptome
      data for clinical outcome prediction
  source_content_hash: sha256:f3509133a0f082a574da3d4842e18901442c1fd64c168b6ceebca4a784dc24c8
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/GuanLab/ciclops.git
  tools:
  - Ciclops
  - LightGBM
  - Python
  - pip
  edam_operation: http://edamontology.org/operation_2928
  edam_topics:
  - http://edamontology.org/topic_0203
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 12
---

# quantile-normalization-ablation-comparison

## Summary

An ablation protocol that compares Ciclops transfer prediction performance with quantile normalization enabled (default) versus disabled (--no_quantile) on the same cross-platform train/validation split (e.g., GSE151189 training, GSE59098 transferring test), to quantify the normalization's effect on transferring-test metrics.

## When to use

Use when building or validating a Ciclops cross-platform transfer model on transcriptomic datasets (e.g., RNA-seq train, microarray test) and you need to decide whether quantile normalization should be kept on or skipped via --no_quantile.

## When NOT to use

- Datasets that are already jointly normalized during preprocessing, where the protocol does not require quantile normalization
- Runs that return NaN evaluation metrics or AUPRC confidence intervals spanning [0,1]; these must be flagged unreliable instead of compared
- Non-transcriptomic or single-platform tasks where no cross-platform transferring test exists to evaluate the normalization effect

## Inputs

- Training dataset .csv (e.g., GSE151189) with sample names as first column and labels as last column
- Transfer validation dataset .csv (e.g., GSE59098) in the same format
- Ciclops pipeline with LightGBM default model and --no_quantile flag

## Outputs

- Metric tables under ./performance/ for 10-fold cross-validation and transferring test, one set per run (with and without quantile normalization)
- Paired comparison table quantile_normalization_comparison.csv containing matched metric rows and per-metric deltas

## How to apply

Format each GEO dataset as a .csv with sample names as the first column and labels as the last column, as required by Ciclops. Run the default pipeline ('python ciclops --train_path <train.csv> --valid_path <valid.csv>') with the default LightGBM model and default quantile normalization, collecting 10-fold cross-validation and transferring-test metric tables from ./performance/. Re-run the identical command with the --no_quantile flag so the same split is processed without quantile normalization. Parse both metric sets, compute per-metric deltas between the two runs, and write the paired metrics and deltas to a single comparison table (e.g., quantile_normalization_comparison.csv). The rationale comes from the original Ciclops study, where quantile normalization was key to a successful cross-platform prediction model, so the protocol recommends keeping it enabled unless ablation shows otherwise. Flag any run producing NaN evaluation metrics or confidence intervals spanning [0,1] for AUPRC as unreliable rather than treating its numbers as valid.

## Related tools

- **Ciclops** (Pipeline run twice (default vs --no_quantile) to produce paired cross-validation and transferring-test metrics) — https://github.com/GuanLab/ciclops.git
- **Python** (CLI entry point for invoking ciclops and parsing ./performance/ metric tables)
- **pip** (Installs the ciclops package via 'pip install ciclops')
- **LightGBM** (Default model type ('lgb') used so both ablation runs are comparable)

## Examples

```
python ciclops --train_path GSE151189.csv --valid_path GSE59098.csv --no_quantile
```

## Evaluation signals

- Both Ciclops runs (default and --no_quantile) complete and yield complete, non-NaN transferring-test metrics in ./performance/
- quantile_normalization_comparison.csv contains matched metric rows for the with-quantile and no-quantile runs of the same train/validation split
- 10-fold cross-validation metrics are generated from preprocessed training datasets and transferring-test metrics from the validation dataset
- Any NaN evaluation metrics or AUPRC confidence intervals spanning [0,1] are flagged as unreliable rather than reported as valid deltas

## Limitations

- Ciclops can fail silently when training and test sets share no common genes, producing csv splits with only sample names and labels and no gene expression columns
- Results can be unreliable without automatic safeguards, e.g., confidence intervals spanning [0, 1] for AUPRC or NaN values for evaluation metrics
- The original paper found quantile normalization was key to its cross-platform model; ablation deltas may be dataset- and condition-dependent due to data collection methods and environmental factors

## Evidence

- [methods] --no_quantile flag semantics: "--no_quantile         If specified, do not use quantile normalization."
- [abstract] Ciclops purpose: "freely available software that can build explainable models to deliver across cross-platform datasets for predicting clinical outcomes"
- [discussion] Unreliable metric failure mode: "confidence intervals spanning [0, 1] for AUPRC, or NaN values for any of the evaluation metrics"
- [readme] Performance folder contents: "preprocessed training datasets for model training and 10-fold cross validation"
