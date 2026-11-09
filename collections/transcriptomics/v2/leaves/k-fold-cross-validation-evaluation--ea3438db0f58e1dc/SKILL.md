---
name: k-fold-cross-validation-evaluation
description: Use when a dsRID random-forest model must be evaluated on candidate dsRNA
  region feature tables (dsRID_data.tsv / dsRID_whole.tsv) and you need per-fold accuracy,
  AUC, and AUPRC rather than a single train/test split.
when_to_use_negative:
- The goal is prediction only (no performance estimate) — in that case opt out of
  training and run extract_whole.py followed by model_predict.py in transfer mode
  directly.
- There is no labeled training table (dsRID_data.tsv with positive and negative dsRNA
  regions), so there is nothing to partition into folds.
- Input RNA-seq is not PacBio long-read data; dsRID's feature tables and models are
  built for PacBio BAM inputs.
edam_operation: http://edamontology.org/operation_0303
edam_topics:
- http://edamontology.org/topic_3474
- http://edamontology.org/topic_3308
tools:
- name: dsRID
  role: Provides model_predict.py and the dsRID.sh pipeline that generate training
    tables, run the 20-fold cross-validated random forest, and write cv_scores.tsv
    and feat_importance.tsv.
  repo: https://github.com/gxiaolab/dsRID
- name: Python 3.x
  role: Runtime for model_predict.py and extract_whole.py, with packages installed
    from requirements.txt via pip.
  repo: https://github.com/gxiaolab/dsRID
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1093/bioinformatics/btad649
    title: 'dsRID: <i>in silico</i> identification of dsRNA regions using long-read
      RNA-seq data'
  source_content_hash: sha256:dc74354c89bb4e5636a11ca05035efd59e40bc2076182159446d641a4c79c4f1
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/gxiaolab/dsRID
  tools:
  - Python
  - Python 3.x
  - dsRID
  edam_operation: http://edamontology.org/operation_0303
  edam_topics:
  - http://edamontology.org/topic_3474
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7
n_claims: 10
---

# k-fold Cross-Validation Evaluation

## Summary

Runs k-fold cross-validation of the dsRID random-forest model on PacBio long-read RNA-seq-derived dsRNA feature tables to obtain per-fold accuracy, AUC, and AUPRC, and compares self-trained versus PacBio AD-transferred models.

## When to use

Use when a dsRID random-forest model must be evaluated on candidate dsRNA region feature tables (dsRID_data.tsv / dsRID_whole.tsv) and you need per-fold accuracy, AUC, and AUPRC rather than a single train/test split.

## When NOT to use

- The goal is prediction only (no performance estimate) — in that case opt out of training and run extract_whole.py followed by model_predict.py in transfer mode directly.
- There is no labeled training table (dsRID_data.tsv with positive and negative dsRNA regions), so there is nothing to partition into folds.
- Input RNA-seq is not PacBio long-read data; dsRID's feature tables and models are built for PacBio BAM inputs.

## Inputs

- dsRID_data.tsv — concatenated positive (dsRID_train.tsv) and negative (dsRID_null.tsv) dsRNA region feature tables
- dsRID_whole.tsv — whole-genome candidate dsRNA region feature table with features extracted from the PacBio BAM
- /data/Pacbio_AD_data.tsv — pre-built training table from PacBio Alzheimer's disease brain data (transfer condition)
- PacBio long-read RNA-seq BAM file (input to extract_whole.py / dsRID.sh)

## Outputs

- cv_scores.tsv — per-fold cross-validation accuracy, AUC, and AUPRC for each dataset and condition
- feat_importance.tsv — random-forest feature importance results

## How to apply

Run `python src/model_predict.py` on the concatenated positive/negative training table: for the self-trained condition pass `-i dsRID_data.tsv` (produced by `bash dsRID.sh <input_bam> <output_dir>` as dsRID_train.tsv + dsRID_null.tsv); for the transfer condition pass `-i /data/Pacbio_AD_data.tsv` so the random forest trained on PacBio AD brain data is applied to each ENCODE dataset's `dsRID_whole.tsv`. In both cases pass `-p dsRID_whole.tsv` (candidate dsRNA regions extracted by `extract_whole.py -b <input_bam> -o <output_dir>/dsRID_whole.tsv`) and collect the 20-fold cross-validation results. dsRID uses 20 folds, and each fold's accuracy (plus AUC and AUPRC) is written to cv_scores.tsv while the random-forest feature importances go to feat_importance.tsv. Aggregate per-fold results across conditions to judge transferability: the article reports mean CV accuracy of 84.1% self-trained versus 88.3% with the PacBio AD-transferred model, with average AUC 0.95 and AUPRC 0.94.

## Related tools

- **dsRID** (Provides model_predict.py and the dsRID.sh pipeline that generate training tables, run the 20-fold cross-validated random forest, and write cv_scores.tsv and feat_importance.tsv.) — https://github.com/gxiaolab/dsRID
- **Python 3.x** (Runtime for model_predict.py and extract_whole.py, with packages installed from requirements.txt via pip.) — https://github.com/gxiaolab/dsRID

## Examples

```
python src/model_predict.py -i dsRID_data.tsv -p dsRID_whole.tsv -o <output_dir>
```

## Evaluation signals

- cv_scores.tsv exists and contains per-fold records for all 10 ENCODE datasets under both the self-trained and PacBio-transfer conditions.
- Mean cross-validation accuracy reproduces the reported values: ~84.1% self-trained and ~88.3% with the PacBio AD-transferred model.
- Average cross-validation AUC is ~0.95 and average AUPRC ~0.94 across datasets.
- feat_importance.tsv is produced alongside cv_scores.tsv, confirming the random-forest training inside each fold completed.

## Limitations

- No version of dsRID or specific package versions is stated; dependency reproducibility relies on the unversioned requirements.txt.
- Cross-validation performance depends on transferability of the PacBio AD brain-trained model; performance on non-brain or non-ENCODE datasets is not established by these results.
- The README documents cv_scores as 'cross validation accuracy results' only; AUC and AUPRC reporting comes from the article's evaluation protocol.

## Evidence

- [methods] model_predict.py produces per-fold CV scores and feature importance: "After running model_predict.py, you should see feat_importance and cv_scores, each represents feature importance results and cross validation accuracy results."
- [methods] Transfer-mode invocation applying the PacBio AD-trained model to whole-genome candidate features: "python src/model_predict.py -i /data/Pacbio_AD_data.tsv -p <output_dir>/dsRID_whole.tsv -o <output_dir>"
- [readme] dsRID.sh generates the training tables needed for cross-validation: "Run the dsRID script to include the training process with the following command: bash dsRID.sh <input_file> <output_dir>"
- [results] dsRID performance was validated on PacBio AD brain long-read RNA-seq data: "We evaluated the performance of dsRID using PacBio long-read RNA-seq data derived from Alzheimer's disease (AD) brain"
- [readme] dsRID output files include the cross-validation scores file: "Output will be the list of following files under /output_dir/ dsRID_train.tsv dsRID_null.tsv dsRID_whole.tsv dsRID_data.tsv feat_importance.tsv cv_scores.tsv"
