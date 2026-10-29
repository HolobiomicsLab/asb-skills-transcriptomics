---
name: dsrna-region-feature-classification
description: Use when you have PacBio long-read RNA-seq (BAM) derived feature tables
  of candidate dsRNA regions and need to train/validate a classifier, or reproduce
  reported performance (89% 20-fold CV accuracy) and feature importances (Skip_ratio
  75.4%, Len_skip 15.9%).
when_to_use_negative:
- Input is already a whole-genome scan table (dsRID_whole.tsv) and you only want scoring
  — opt out of training and run extract_whole.py followed by model_predict.py directly
  instead of re-running dsRID.sh
- Data are short-read RNA-seq — dsRID is designed to leverage molecular traits of
  dsRNAs captured by long-read RNA sequencing
- You lack the paired positive/negative training tables or the packaged Pacbio_AD_data.tsv
  needed to fit and cross-validate the classifier
edam_operation: http://edamontology.org/operation_2990
edam_topics:
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_3474
tools:
- name: dsRID
  role: Provides the random-forest model, training script dsRID.sh, and model_predict.py
    for feature classification and importance estimation
  repo: https://github.com/gxiaolab/dsRID
- name: PacBio long-read RNA-seq
  role: Source data type from which dsRNA region features (e.g., Skip_ratio, Len_skip)
    are derived for training and prediction
- name: Python 3.x
  role: Runtime for dsRID scripts (model_predict.py, extract_whole.py) and pandas
    loading of feature tables
- name: pip
  role: Installs required packages from requirements.txt before running the classifier
provenance:
  source_task_ids:
  - task_002
  - task_005
  source_papers:
  - doi: 10.1093/bioinformatics/btad649
    title: 'dsRID: <i>in silico</i> identification of dsRNA regions using long-read
      RNA-seq data'
  source_content_hash: sha256:192d03abd85f94bcb633727fa955532270ab1e4913b7c8cfb17a60750e8c2183
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/gxiaolab/dsRID
  tools:
  - PacBio long-read RNA-seq
  - Python
  - Python 3.x
  - dsRID
  - pip
  edam_operation: http://edamontology.org/operation_2990
  edam_topics:
  - http://edamontology.org/topic_3308
  - http://edamontology.org/topic_3474
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.4166666666666667
n_claims: 12
---

# dsrna-region-feature-classification

## Summary

Train and evaluate a random-forest classifier (dsRID) that discriminates candidate dsRNA regions from null regions using molecular traits extracted from PacBio long-read RNA-seq data, quantifying both cross-validation accuracy and permutation-based feature contributions.

## When to use

Use when you have PacBio long-read RNA-seq (BAM) derived feature tables of candidate dsRNA regions and need to train/validate a classifier, or reproduce reported performance (89% 20-fold CV accuracy) and feature importances (Skip_ratio 75.4%, Len_skip 15.9%).

## When NOT to use

- Input is already a whole-genome scan table (dsRID_whole.tsv) and you only want scoring — opt out of training and run extract_whole.py followed by model_predict.py directly instead of re-running dsRID.sh
- Data are short-read RNA-seq — dsRID is designed to leverage molecular traits of dsRNAs captured by long-read RNA sequencing
- You lack the paired positive/negative training tables or the packaged Pacbio_AD_data.tsv needed to fit and cross-validate the classifier

## Inputs

- PacBio long-read RNA-seq data in BAM format
- Positive dsRNA region feature table dsRID_train.tsv
- Negative (null) dsRNA region feature table dsRID_null.tsv
- Packaged feature table /data/Pacbio_AD_data.tsv (AD brain PacBio data)
- Whole-genome scan predictions dsRID_whole.tsv from extract_whole.py

## Outputs

- dsRID_data.tsv (concatenated positive and negative training regions)
- feat_importance.tsv (permutation-based feature contribution scores)
- cv_scores.tsv (20-fold cross-validation accuracy results)
- dsRID_whole.tsv with predicted probability column from the random-forest model

## How to apply

Extract positive and negative dsRNA region training tables (dsRID_train.tsv and dsRID_null.tsv, concatenated into dsRID_data.tsv, or the packaged /data/Pacbio_AD_data.tsv from AD brain PacBio data) with 13,469 positive regions matched to an equal number of randomly selected null regions. Train the random-forest classifier using the TPOT-tuned hyperparameters shipped with dsRID, then run 20-fold cross-validation on the training set. Execute python src/model_predict.py to write feature importances to feat_importance.tsv and cross-validation accuracy to cv_scores.tsv. To score whole-genome scan candidates without retraining, run extract_whole.py first, then pass both /data/Pacbio_AD_data.tsv and dsRID_whole.tsv to model_predict.py. Validate by checking that mean 20-fold CV accuracy reproduces ~89% (Fig. 2A) and permutation contributions match Skip_ratio ~75.4% and Len_skip ~15.9% (Fig. 2E-F).

## Related tools

- **dsRID** (Provides the random-forest model, training script dsRID.sh, and model_predict.py for feature classification and importance estimation) — https://github.com/gxiaolab/dsRID
- **PacBio long-read RNA-seq** (Source data type from which dsRNA region features (e.g., Skip_ratio, Len_skip) are derived for training and prediction)
- **Python 3.x** (Runtime for dsRID scripts (model_predict.py, extract_whole.py) and pandas loading of feature tables)
- **pip** (Installs required packages from requirements.txt before running the classifier)

## Examples

```
python src/model_predict.py -i /data/Pacbio_AD_data.tsv -p <output_dir>/dsRID_whole.tsv -o <output_dir>
```

## Evaluation signals

- Mean 20-fold cross-validation accuracy recorded in cv_scores.tsv reproduces the reported ~89% accuracy on the PacBio AD data (Fig. 2A)
- Permutation feature contributions in feat_importance.tsv match Skip_ratio ~75.4% and Len_skip ~15.9% (Fig. 2E-F, Suppl. Fig. S4)
- Training set is balanced: 13,469 positive regions matched with an equal number of randomly selected negative regions
- dsRID_whole.tsv contains a column of predicted probability to be a candidate dsRNA region from the random-forest model
- All expected output files appear under output_dir: dsRID_train.tsv, dsRID_null.tsv, dsRID_whole.tsv, dsRID_data.tsv, feat_importance.tsv, cv_scores.tsv

## Limitations

- No version of dsRID or specific package versions are stated; required packages are only listed in requirements.txt
- Classification is trained on a single dataset (PacBio AD brain) with 13,469 positive regions; performance on other tissues/conditions depends on transferability of the pretrained model
- Skipping model training requires supplying the packaged /data/Pacbio_AD_data.tsv; otherwise the training process is mandatory via dsRID.sh

## Evidence

- [abstract] Using models trained with PacBio long-read RNA-seq data derived from Alzheimer's disease (AD) brain: "Using models trained with PacBio long-read RNA-seq data derived from Alzheimer's disease (AD) brain"
- [readme] concatenated positive/negative training data: "dsRID_data.tsv is a concatenated file of dsRID_train and dsRID_null each represents positive and negative dsRNA regions."
- [methods] outputs of model_predict.py: "After running model_predict.py, you should see feat_importance and cv_scores, each represents feature importance results and cross validation accuracy results."
- [readme] opt-out training path: "If you would like to opt out training process and directly proceed to prediction step, run"
