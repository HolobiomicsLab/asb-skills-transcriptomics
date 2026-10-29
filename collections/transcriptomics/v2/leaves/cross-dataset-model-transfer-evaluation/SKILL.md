---
name: cross-dataset-model-transfer-evaluation
description: Use when you have candidate-feature tables (e.g. dsRID_whole.tsv) from
  multiple ENCODE long-read RNA-seq datasets and need to decide whether a self-trained
  random forest or the PacBio AD-trained transferred model performs better under 20-fold
  cross-validation.
when_to_use_negative:
- Input is already a scored dsRID_whole.tsv and you only want predictions, not a train-vs-transfer
  benchmark — run prediction only (extract_whole.py + model_predict.py) instead.
- Data is short-read RNA-seq or lacks the PacBio long-read RNA-seq BAM needed to build
  dsRID_train.tsv/dsRID_null.tsv, so the self-trained condition cannot be constructed.
edam_operation: http://edamontology.org/operation_2939
edam_topics:
- http://edamontology.org/topic_3474
tools:
- name: dsRID
  role: Pipeline whose dsRID.sh, extract_whole.py, and model_predict.py scripts generate
    the training data, feature tables, and 20-fold cross-validation scores compared
    here
  repo: https://github.com/gxiaolab/dsRID
- name: Python
  role: Runtime for extract_whole.py and model_predict.py (random forest training,
    cross-validation scoring)
- name: PacBio long-read RNA-seq
  role: Data modality supplying the BAM inputs and the AD-brain training data used
    for transfer
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1093/bioinformatics/btad649
    title: 'dsRID: <i>in silico</i> identification of dsRNA regions using long-read
      RNA-seq data'
  source_content_hash: sha256:ae9fdc4f1c5ccb514069c6d0e1289c13edb7adb968483857c31d7fe83f3a829f
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/gxiaolab/dsRID
  tools:
  - PacBio long-read RNA-seq
  - Python
  - dsRID
  edam_operation: http://edamontology.org/operation_2939
  edam_topics:
  - http://edamontology.org/topic_3474
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 12
---

# cross-dataset-model-transfer-evaluation

## Summary

Compares a random forest dsRNA-prediction model trained on the target dataset itself against a model transferred from PacBio AD brain long-read RNA-seq data, using 20-fold cross-validation across the 10 ENCODE datasets with accuracy, AUC, and AUPRC as metrics.

## When to use

Use when you have candidate-feature tables (e.g. dsRID_whole.tsv) from multiple ENCODE long-read RNA-seq datasets and need to decide whether a self-trained random forest or the PacBio AD-trained transferred model performs better under 20-fold cross-validation.

## When NOT to use

- Input is already a scored dsRID_whole.tsv and you only want predictions, not a train-vs-transfer benchmark — run prediction only (extract_whole.py + model_predict.py) instead.
- Data is short-read RNA-seq or lacks the PacBio long-read RNA-seq BAM needed to build dsRID_train.tsv/dsRID_null.tsv, so the self-trained condition cannot be constructed.

## Inputs

- PacBio long-read RNA-seq BAM file per ENCODE dataset
- dsRID_data.tsv (concatenated dsRID_train.tsv positives and dsRID_null.tsv negatives) per dataset
- dsRID_whole.tsv (whole-genome candidate dsRNA region feature table)
- Pre-trained PacBio AD model data file (/data/Pacbio_AD_data.tsv) for the transfer condition

## Outputs

- cv_scores.tsv with per-fold cross-validation accuracy, AUC, and AUPRC for all 10 datasets under both self-trained and transfer conditions
- feat_importance.tsv with random-forest feature importance

## How to apply

For each ENCODE dataset, first extract the whole-genome candidate dsRNA region feature table with extract_whole.py from the PacBio long-read RNA-seq BAM file, and run bash dsRID.sh to generate the training positives (dsRID_train.tsv), negatives (dsRID_null.tsv), and their concatenation dsRID_data.tsv. For the self-trained condition, run model_predict.py with -i dsRID_data.tsv and -p dsRID_whole.tsv so the random forest is trained on that dataset's own labels and scored under 20-fold cross-validation. For the transfer condition, run model_predict.py with -i /data/Pacbio_AD_data.tsv so the model trained on the PacBio AD brain data is applied to each dataset's dsRID_whole.tsv under the same 20-fold scheme. Aggregate per-fold results into cv_scores.tsv (accuracy, AUC, AUPRC for both conditions across all 10 datasets) and random-forest feature importance into feat_importance.tsv, then compare the average accuracies to judge which condition generalizes better.

## Related tools

- **dsRID** (Pipeline whose dsRID.sh, extract_whole.py, and model_predict.py scripts generate the training data, feature tables, and 20-fold cross-validation scores compared here) — https://github.com/gxiaolab/dsRID
- **Python** (Runtime for extract_whole.py and model_predict.py (random forest training, cross-validation scoring))
- **PacBio long-read RNA-seq** (Data modality supplying the BAM inputs and the AD-brain training data used for transfer)

## Examples

```
python src/model_predict.py -i /data/Pacbio_AD_data.tsv -p <output_dir>/dsRID_whole.tsv -o <output_dir>
```

## Evaluation signals

- cv_scores.tsv exists with per-fold records for all 10 ENCODE datasets and both conditions (self-trained and transfer).
- Average cross-validation accuracy reproduces the reported values: 84.1% self-trained vs 88.3% PacBio AD transfer.
- Average AUC ≈ 0.95 and average AUPRC ≈ 0.94 across the 20-fold cross-validation.
- feat_importance.tsv is produced alongside cv_scores.tsv, as model_predict.py outputs both feature importance and cross-validation accuracy results.
- Both dsRID_train.tsv (positives) and dsRID_null.tsv (negatives) were generated and concatenated into dsRID_data.tsv before training.

## Limitations

- No version of dsRID or specific package versions is stated; required packages are only listed in requirements.txt, so exact reproducibility of cross-validation scores may depend on environment.
- Transfer performance relies on models trained with PacBio long-read RNA-seq data derived from Alzheimer's disease (AD) brain; datasets far outside this tissue/protocol may behave differently.
- Validation in the source card is exact-value based (84.1% / 88.3%, AUC 0.95, AUPRC 0.94), which may not hold exactly for re-run data.

## Evidence

- [results] self-trained vs transfer accuracies: [evidence span withheld — anchor status: not_found]
- [methods] extract_whole.py invocation: "python src/extract_whole.py -b <input_bam_file> -o <output_dir>/dsRID_whole.tsv"
- [methods] transfer-condition invocation: "python src/model_predict.py -i /data/Pacbio_AD_data.tsv -p <output_dir>/dsRID_whole.tsv -o <output_dir>"
- [readme] model_predict.py outputs: "After running model_predict.py, you should see feat_importance and cv_scores, each represents feature importance results and cross validation accuracy results."
- [readme] training data composition: "dsRID_data.tsv is a concatenated file of dsRID_train and dsRID_null each represents positive and negative dsRNA regions."
