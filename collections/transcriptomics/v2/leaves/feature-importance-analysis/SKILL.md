---
name: feature-importance-analysis
description: Use when a trained dsRID random-forest model on PacBio AD long-read RNA-seq
  regions (dsRID_data.tsv or /data/Pacbio_AD_data.tsv) needs feature importance ranking
  and cross-validation accuracy reported.
when_to_use_negative:
- Input is a raw PacBio BAM file rather than an extracted region feature table — run
  extract_whole.py / dsRID.sh first to generate dsRID_train.tsv, dsRID_null.tsv and
  dsRID_whole.tsv.
- You only need genome-wide dsRNA candidate predictions (dsRID_whole.tsv probabilities)
  without model interpretability — skip model_predict.py's training/evaluation path.
- The model was trained on data other than PacBio long-read RNA-seq dsRNA regions,
  since the TPOT-tuned hyperparameters and 89% accuracy benchmark are specific to
  that setting.
edam_topics:
- http://edamontology.org/topic_3474
tools:
- name: dsRID
  role: Provides model_predict.py, the random-forest model, TPOT-tuned hyperparameters,
    and the packaged Pacbio_AD_data.tsv training table used to compute feature importance
    and cross-validation scores
  repo: https://github.com/gxiaolab/dsRID
- name: Python 3.x
  role: Runtime for model_predict.py; pandas loads the positive and negative dsRNA
    region training tables
- name: PacBio
  role: Source of the long-read RNA-seq data from Alzheimer's disease brain used to
    train the model and derive the feature table
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1093/bioinformatics/btad649
    title: 'dsRID: <i>in silico</i> identification of dsRNA regions using long-read
      RNA-seq data'
  source_content_hash: sha256:b9a2d558e85e65ae21bb78195ee4ad95e7faa653be0d6affa031b640eb570c58
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/gxiaolab/dsRID
  tools:
  - PacBio
  - Python
  - Python 3.x
  - dsRID
  - pip
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3474
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.4
n_claims: 10
---

# feature-importance-analysis

## Summary

Extract and report feature importance from the dsRID random-forest classifier alongside cross-validation accuracy, using the model_predict.py step that writes feat_importance.tsv and cv_scores.tsv. This quantifies which molecular traits of dsRNA regions drive discrimination of true dsRNA regions from null regions in PacBio long-read RNA-seq data.

## When to use

Use when a trained dsRID random-forest model on PacBio AD long-read RNA-seq regions (dsRID_data.tsv or /data/Pacbio_AD_data.tsv) needs feature importance ranking and cross-validation accuracy reported.

## When NOT to use

- Input is a raw PacBio BAM file rather than an extracted region feature table — run extract_whole.py / dsRID.sh first to generate dsRID_train.tsv, dsRID_null.tsv and dsRID_whole.tsv.
- You only need genome-wide dsRNA candidate predictions (dsRID_whole.tsv probabilities) without model interpretability — skip model_predict.py's training/evaluation path.
- The model was trained on data other than PacBio long-read RNA-seq dsRNA regions, since the TPOT-tuned hyperparameters and 89% accuracy benchmark are specific to that setting.

## Inputs

- dsRID_data.tsv — concatenated positive (dsRID_train.tsv) and negative (dsRID_null.tsv) dsRNA region feature tables
- /data/Pacbio_AD_data.tsv — packaged feature table from PacBio AD brain long-read RNA-seq data (13,469 positive regions plus matched negatives)
- dsRID_whole.tsv — whole-genome scan candidates with predicted probability column (as the -p prediction input)

## Outputs

- feat_importance.tsv — feature importance results from the trained random-forest model
- cv_scores.tsv — 20-fold cross-validation accuracy results (mean should approximate 89%, Fig. 2A)

## How to apply

Load the concatenated positive/negative training table (dsRID_train.tsv + dsRID_null.tsv = dsRID_data.tsv, or the packaged /data/Pacbio_AD_data.tsv from AD brain PacBio long-read RNA-seq data) into pandas. Train the random-forest classifier on 13,469 positive dsRNA regions and an equal number of matched null regions using the TPOT-tuned hyperparameters shipped with dsRID. Run python src/model_predict.py, which performs 20-fold cross-validation and writes two files to the output directory: feat_importance.tsv (feature importance results) and cv_scores.tsv (cross-validation accuracy results). The rationale is that feature importance reveals which dsRNA molecular traits the classifier relies on, while cross-validation scores confirm the model discriminates candidate dsRNA regions from null regions at the reported level.

## Related tools

- **dsRID** (Provides model_predict.py, the random-forest model, TPOT-tuned hyperparameters, and the packaged Pacbio_AD_data.tsv training table used to compute feature importance and cross-validation scores) — https://github.com/gxiaolab/dsRID
- **Python 3.x** (Runtime for model_predict.py; pandas loads the positive and negative dsRNA region training tables)
- **PacBio** (Source of the long-read RNA-seq data from Alzheimer's disease brain used to train the model and derive the feature table)

## Examples

```
python src/model_predict.py -i /data/Pacbio_AD_data.tsv -p <output_dir>/dsRID_whole.tsv -o <output_dir>
```

## Evaluation signals

- Mean cross-validation accuracy recorded in cv_scores.tsv reproduces the reported dsRID accuracy of approximately 89% (Fig. 2A).
- feat_importance.tsv exists in the output directory and contains a ranked feature importance table.
- cv_scores.tsv exists and reflects 20-fold cross-validation over the training set.
- The model was trained on balanced classes: 13,469 positive regions and an equal number of randomly selected negative regions from the PacBio AD data.
- The prediction input dsRID_whole.tsv carries a column of predicted probability to be a candidate dsRNA region from the random forest trained on dsRID_data.tsv.

## Limitations

- No versions of dsRID or its Python packages are stated; only that required packages are listed in requirements.txt, so exact environment reproducibility is uncertain.
- The 89% accuracy benchmark applies to the PacBio AD training data; performance on other datasets must be re-validated.
- Feature importances come from a single random-forest model configuration (TPOT-tuned hyperparameters); no alternative model comparison or statistical significance testing of importances is described.

## Evidence

- [methods] model_predict.py writes feature importance and CV accuracy: "After running model_predict.py, you should see feat_importance and cv_scores, each represents feature importance results and cross validation accuracy results."
- [methods] model_predict.py CLI invocation: "python src/model_predict.py -i /data/Pacbio_AD_data.tsv -p <output_dir>/dsRID_whole.tsv -o <output_dir>"
- [readme] random-forest probability column in dsRID_whole: "dsRID_whole will contain column of predicted probability to be a candidate dsRNA region from the random forest model trained on dsRID_data.tsv."
- [results] 89% accuracy from 20-fold cross-validation on PacBio AD data: [evidence span withheld — anchor status: not_found]
- [abstract] training data source is PacBio AD brain long-read RNA-seq: "Using models trained with PacBio long-read RNA-seq data derived from Alzheimer's disease (AD) brain"
