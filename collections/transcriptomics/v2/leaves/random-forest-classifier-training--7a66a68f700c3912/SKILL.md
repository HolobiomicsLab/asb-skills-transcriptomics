---
name: random-forest-classifier-training
description: Use when you have PacBio long-read RNA-seq BAM data processed by dsRID
  into positive (dsRID_train.tsv) and negative (dsRID_null.tsv) dsRNA region tables
  and need to train and validate a classifier that discriminates candidate dsRNA regions.
when_to_use_negative:
- Input is already a scored dsRID_whole.tsv and the goal is only prediction — run
  extract_whole.py/model_predict.py with the pre-trained PacBio AD model instead of
  retraining
- Data are short-read RNA-seq or non-BAM format — dsRID's training pipeline is built
  around PacBio long-read RNA-seq BAM inputs
- You need the global dsRNA expression profile of the ENCODE AD cohort rather than
  model training — use the pre-packaged dsRID prediction outputs (dsRID_novel_dsRNA.bed,
  Encode_novel_dsRNA_source.tsv) instead
edam_operation: http://edamontology.org/operation_2990
edam_topics:
- http://edamontology.org/topic_3474
- http://edamontology.org/topic_3308
tools:
- name: dsRID
  role: Pipeline that extracts training regions from BAM and trains the random-forest
    classifier (dsRID.sh) with TPOT-tuned hyperparameters
  repo: https://github.com/gxiaolab/dsRID
- name: model_predict.py
  role: Trains/evaluates the random forest on dsRID_data.tsv and writes feat_importance.tsv
    and cv_scores.tsv; with -i /data/Pacbio_AD_data.tsv it transfers the PacBio AD-trained
    model
  repo: https://github.com/gxiaolab/dsRID
- name: extract_whole.py
  role: Extracts the whole-genome candidate dsRNA region feature table (dsRID_whole.tsv)
    from a BAM when the training step is skipped
  repo: https://github.com/gxiaolab/dsRID
- name: Python
  role: Runtime environment (Python 3.x or higher) for dsRID scripts and model training
- name: pip
  role: Installs required packages from requirements.txt before training
provenance:
  source_task_ids:
  - task_002
  - task_003
  - task_004
  source_papers:
  - doi: 10.1093/bioinformatics/btad649
    title: 'dsRID: <i>in silico</i> identification of dsRNA regions using long-read
      RNA-seq data'
  source_content_hash: sha256:decc84aee0d5417d10911533db12ca634e276546329a37042c74464cd8e16c73
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/gxiaolab/dsRID
  tools:
  - Python
  - dsRID
  - extract_whole.py
  - model_predict.py
  - pip
  edam_operation: http://edamontology.org/operation_2990
  edam_topics:
  - http://edamontology.org/topic_3474
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 9
---

# random-forest-classifier-training

## Summary

Train the dsRID random-forest classifier on positive and negative dsRNA region feature tables extracted from PacBio long-read RNA-seq data (e.g., the 13,469 positive regions from AD brain), then estimate performance with 20-fold cross-validation, producing feature importances and cross-validation scores.

## When to use

Use when you have PacBio long-read RNA-seq BAM data processed by dsRID into positive (dsRID_train.tsv) and negative (dsRID_null.tsv) dsRNA region tables and need to train and validate a classifier that discriminates candidate dsRNA regions.

## When NOT to use

- Input is already a scored dsRID_whole.tsv and the goal is only prediction — run extract_whole.py/model_predict.py with the pre-trained PacBio AD model instead of retraining
- Data are short-read RNA-seq or non-BAM format — dsRID's training pipeline is built around PacBio long-read RNA-seq BAM inputs
- You need the global dsRNA expression profile of the ENCODE AD cohort rather than model training — use the pre-packaged dsRID prediction outputs (dsRID_novel_dsRNA.bed, Encode_novel_dsRNA_source.tsv) instead

## Inputs

- PacBio long-read RNA-seq BAM file
- dsRID_train.tsv (positive dsRNA regions)
- dsRID_null.tsv (negative dsRNA regions)
- dsRID_data.tsv (concatenated positive/negative training table; packaged reference: /data/Pacbio_AD_data.tsv)
- dsRID_whole.tsv (whole-genome candidate dsRNA region feature table from extract_whole.py)

## Outputs

- feat_importance.tsv (random-forest feature importances)
- cv_scores.tsv (per-fold cross-validation accuracy, plus AUC and AUPRC)
- dsRID_whole.tsv (whole-genome predictions with predicted probability of being a candidate dsRNA region)

## How to apply

Prepare the PacBio BAM and run `bash dsRID.sh <input_file> <output_dir>` so dsRID generates the positive region table (dsRID_train.tsv), matched negative regions (dsRID_null.tsv), their concatenation (dsRID_data.tsv), and the whole-genome candidate table (dsRID_whole.tsv). Train the random forest on dsRID_data.tsv using the TPOT-tuned hyperparameters shipped with dsRID (13,469 positive and an equal number of randomly selected negative regions in the PacBio AD reference set). Run 20-fold cross-validation on the training set and, for subsampling analyses, evaluate self-trained models on a 5% held-out test split repeated 20 times. Execute `python src/model_predict.py -i <training_table> -p <dsRID_whole.tsv> -o <output_dir>` to write the random-forest feature importances to feat_importance.tsv and the per-fold cross-validation accuracy, AUC, and AUPRC to cv_scores.tsv. To skip training, reuse the packaged PacBio AD-trained model via `-i /data/Pacbio_AD_data.tsv` for transfer prediction.

## Related tools

- **dsRID** (Pipeline that extracts training regions from BAM and trains the random-forest classifier (dsRID.sh) with TPOT-tuned hyperparameters) — https://github.com/gxiaolab/dsRID
- **model_predict.py** (Trains/evaluates the random forest on dsRID_data.tsv and writes feat_importance.tsv and cv_scores.tsv; with -i /data/Pacbio_AD_data.tsv it transfers the PacBio AD-trained model) — https://github.com/gxiaolab/dsRID
- **extract_whole.py** (Extracts the whole-genome candidate dsRNA region feature table (dsRID_whole.tsv) from a BAM when the training step is skipped) — https://github.com/gxiaolab/dsRID
- **Python** (Runtime environment (Python 3.x or higher) for dsRID scripts and model training)
- **pip** (Installs required packages from requirements.txt before training)

## Examples

```
bash dsRID.sh <input_file> <output_dir>; python src/model_predict.py -i dsRID_data.tsv -p <output_dir>/dsRID_whole.tsv -o <output_dir>
```

## Evaluation signals

- cv_scores.tsv exists and its mean 20-fold cross-validation accuracy reproduces ~89% for the PacBio AD reference model (Fig. 2A)
- Across the 10 ENCODE datasets, average cross-validation accuracy is 84.1% self-trained and 88.3% for the PacBio AD transfer condition, with average AUC ~0.95 and AUPRC ~0.94
- Subsampled PacBio AD models show self-trained accuracy above 80% even at 0.43M reads and transfer accuracy consistently above 88%, plateauing near 1.5–2M reads (Supplementary Fig. S2)
- All six expected outputs (dsRID_train.tsv, dsRID_null.tsv, dsRID_whole.tsv, dsRID_data.tsv, feat_importance.tsv, cv_scores.tsv) are present under the output directory
- Training table contains 13,469 positive dsRNA regions matched by an equal number of randomly selected negative regions

## Limitations

- No specific dsRID version or pinned package versions are stated; only 'Python version 3.x or higher' and the requirements.txt file are specified
- Accuracy somewhat plateaus at around 1.5–2 million sequencing reads, so much shallower data may degrade self-trained model performance
- Transfer prediction depends on the packaged /data/Pacbio_AD_data.tsv model; behavior on tissues/conditions outside AD brain long-read data is not characterized here

## Evidence

- [readme] Run the dsRID script to include the training process with the following command: bash dsRID.sh <input_file> <output_dir>: "Run the dsRID script to include the training process with the following command: bash dsRID.sh <input_file> <output_dir>"
- [readme] After running model_predict.py, you should see feat_importance and cv_scores, each represents feature importance results and cross validation accuracy results.: "After running model_predict.py, you should see feat_importance and cv_scores, each represents feature importance results and cross validation accuracy results."
- [results] We evaluated the performance of dsRID using PacBio long-read RNA-seq data derived from Alzheimer's disease (AD) brain: "We evaluated the performance of dsRID using PacBio long-read RNA-seq data derived from Alzheimer's disease (AD) brain"
- [intro] a machine-learning-based method designed to predict dsRNA regions in silico by leveraging the power of long-read RNA sequencing (RNA-seq) and molecular traits of dsRNAs: "a machine-learning-based method designed to predict dsRNA regions in silico by leveraging the power of long-read RNA sequencing (RNA-seq) and molecular traits of dsRNAs"
- [results] In 20-fold cross-validation across the ENCODE datasets, the average accuracy was 84.1% when trained on each respective dataset, and was higher at 88.3% using the PacBio AD-trained model.: [evidence span withheld — anchor status: not_found]
