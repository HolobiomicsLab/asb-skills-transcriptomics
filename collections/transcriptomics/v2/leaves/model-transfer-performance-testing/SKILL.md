---
name: model-transfer-performance-testing
description: Use when you must determine the minimum sequencing depth at which a trained
  model retains acceptable accuracy, by training on descending random subsamples and
  comparing self-trained vs full-data-transfer accuracy curves.
when_to_use_negative:
- Input RNA-seq data is short-read rather than long-read; dsRID was developed and
  evaluated on PacBio long-read RNA-seq data
- You only need to run prediction at one fixed depth with an existing model (skip
  retraining-per-depth and use extract_whole.py + model_predict.py directly)
- Input is already a feature table or accuracy summary — this skill requires re-running
  the training pipeline on each depth subsample
edam_topics:
- http://edamontology.org/topic_3474
- http://edamontology.org/topic_3308
tools:
- name: dsRID
  role: Training pipeline (bash dsRID.sh) run on each depth subsample to generate
    dsRID_train.tsv, dsRID_null.tsv, dsRID_data.tsv, and cv_scores.tsv; extract_whole.py
    and model_predict.py score whole-genome regions with the full-data-trained model
    for transfer accuracy
  repo: https://github.com/gxiaolab/dsRID
- name: PacBio long-read RNA-seq
  role: Source dataset (Alzheimer's disease brain BAM) that is subsampled at descending
    read depths to build the accuracy-vs-depth curves
- name: Python
  role: Runtime for the dsRID scripts (extract_whole.py, model_predict.py) and the
    random forest training/evaluation loop with 20 repeated splits
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1093/bioinformatics/btad649
    title: 'dsRID: <i>in silico</i> identification of dsRNA regions using long-read
      RNA-seq data'
  source_content_hash: sha256:c924a7890b39ae5c7bd631e8f2d97daa0fc97d854b102f0125b49c6a617662b2
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/gxiaolab/dsRID
  tools:
  - PacBio long-read RNA-seq
  - Python
  - dsRID
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3474
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6
n_claims: 10
---

# model-transfer-performance-testing

## Summary

Tests how a machine learning model's accuracy holds up as sequencing depth decreases by training models on randomly subsampled data (self-trained accuracy) and also scoring the same subsamples with a model trained on the full dataset (transfer accuracy), to identify the depth at which performance plateaus.

## When to use

Use when you must determine the minimum sequencing depth at which a trained model retains acceptable accuracy, by training on descending random subsamples and comparing self-trained vs full-data-transfer accuracy curves.

## When NOT to use

- Input RNA-seq data is short-read rather than long-read; dsRID was developed and evaluated on PacBio long-read RNA-seq data
- You only need to run prediction at one fixed depth with an existing model (skip retraining-per-depth and use extract_whole.py + model_predict.py directly)
- Input is already a feature table or accuracy summary — this skill requires re-running the training pipeline on each depth subsample

## Inputs

- PacBio long-read RNA-seq data in BAM format (full dataset, e.g., 4.27M reads)
- Randomly subsampled BAM files at descending depths (down to ~0.43M reads)
- dsRID_data.tsv (concatenated dsRID_train.tsv positives and dsRID_null.tsv negatives) per subsample
- Full-data-trained model / feature table (e.g., /data/Pacbio_AD_data.tsv) for transfer prediction

## Outputs

- Cross-validation accuracy per subsample depth (cv_scores.tsv), from 20 repeated 5% held-out test splits
- Transfer-prediction probabilities for whole-genome regions scored by the full-data-trained model
- Accuracy-vs-read-depth curves (self-trained vs transfer) comparable to Supplementary Fig. S2
- Estimated depth plateau (≈1.5–2 million reads) as a sequencing-depth recommendation

## How to apply

Subsample the full PacBio AD long-read RNA-seq BAM dataset (4.27M reads) at multiple descending depths (down to ~0.43M reads), and run the dsRID training pipeline (bash dsRID.sh) on each subsample to generate positive regions (dsRID_train.tsv), negative regions (dsRID_null.tsv), and the concatenated training table (dsRID_data.tsv) used to fit the random forest model. Evaluate each self-trained subsample model on a 5% held-out test split, repeating the train/test split 20 times, and record cross-validation accuracy. For transfer testing, run extract_whole.py on each subsample and score the extracted whole-genome regions with the full-data-trained model via model_predict.py (-i /data/Pacbio_AD_data.tsv) to obtain candidate dsRNA prediction probabilities. Compile self-trained and transfer accuracies against read depth (e.g., cv_scores.tsv) and compare the resulting curves against the paper's Supplementary Fig. S2 trends: accuracy rises with depth and plateaus at roughly 1.5–2 million reads, which is the recommended minimum depth.

## Related tools

- **dsRID** (Training pipeline (bash dsRID.sh) run on each depth subsample to generate dsRID_train.tsv, dsRID_null.tsv, dsRID_data.tsv, and cv_scores.tsv; extract_whole.py and model_predict.py score whole-genome regions with the full-data-trained model for transfer accuracy) — https://github.com/gxiaolab/dsRID
- **PacBio long-read RNA-seq** (Source dataset (Alzheimer's disease brain BAM) that is subsampled at descending read depths to build the accuracy-vs-depth curves)
- **Python** (Runtime for the dsRID scripts (extract_whole.py, model_predict.py) and the random forest training/evaluation loop with 20 repeated splits)

## Examples

```
bash dsRID.sh <input_file> <output_dir>
```

## Evaluation signals

- Self-trained accuracy at the smallest subsample (~0.43M reads) is above 80%
- Transfer accuracy (full-data-trained model scoring subsample-derived regions) is consistently above 88% across depths
- Accuracy-vs-depth curves show a plateau at roughly 1.5–2 million reads, matching Supplementary Fig. S2 trends
- cv_scores.tsv is produced for each subsample with 20 repeated train/test splits recorded
- Each subsample run produces the expected dsRID.sh outputs: dsRID_train.tsv, dsRID_null.tsv, dsRID_whole.tsv, dsRID_data.tsv, feat_importance.tsv, cv_scores.tsv

## Limitations

- Plateau estimate (~1.5–2M reads) is specific to the 4.27M-read PacBio AD dataset; other datasets may plateau at different depths
- The dsRID README does not state tool or package versions (requirements are only given via requirements.txt), so depth-sweep results may vary across versions
- Accuracy improvement with depth is 'somewhat' plateauing per the paper — modest gains beyond 2M reads may still occur

## Evidence

- [readme] Prepare the PacBio long-read RNA-seq data in the BAM format.: "Prepare the PacBio long-read RNA-seq data in the BAM format."
- [methods] Run the dsRID script to include the training process with the following command: bash dsRID.sh <input_file> <output_dir>: "Run the dsRID script to include the training process with the following command: bash dsRID.sh <input_file> <output_dir>"
- [methods] After running model_predict.py, you should see feat_importance and cv_scores, each represents feature importance results and cross validation accuracy results.: "After running model_predict.py, you should see feat_importance and cv_scores, each represents feature importance results and cross validation accuracy results."
- [methods] python src/model_predict.py -i /data/Pacbio_AD_data.tsv -p <output_dir>/dsRID_whole.tsv -o <output_dir>: "python src/model_predict.py -i /data/Pacbio_AD_data.tsv -p <output_dir>/dsRID_whole.tsv -o <output_dir>"
- [readme] dsRID_whole will contain column of predicted probability to be a candidate dsRNA region from the random forest model trained on dsRID_data.tsv.: "dsRID_whole will contain column of predicted probability to be a candidate dsRNA region from the random forest model trained on dsRID_data.tsv."
- [results] We evaluated the performance of dsRID using PacBio long-read RNA-seq data derived from Alzheimer's disease (AD) brain: "We evaluated the performance of dsRID using PacBio long-read RNA-seq data derived from Alzheimer's disease (AD) brain"
