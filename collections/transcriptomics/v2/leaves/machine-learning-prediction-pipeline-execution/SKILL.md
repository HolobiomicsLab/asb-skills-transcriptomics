---
name: machine-learning-prediction-pipeline-execution
description: Use when a BAM file from PacBio long-read RNA-seq must be converted into
  predicted dsRNA regions with per-region probabilities, or when reproducing feature-importance
  (Skip_ratio 75.4%, Len_skip 15.9%) and cv_scores from the AD-trained model.
when_to_use_negative:
- You already have dsRID_whole.tsv or a dsRID feature table and only need model scoring
  — in that case run model_predict.py directly rather than re-running the dsRID.sh
  training process.
- Your input is short-read RNA-seq rather than PacBio long-read RNA-seq BAM data,
  since dsRID is designed to leverage long-read sequencing.
- You need calibrated per-base dsRNA annotations rather than region-level predicted
  probabilities from the random-forest model.
edam_operation: http://edamontology.org/operation_3936
edam_topics:
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_3512
tools:
- name: dsRID
  role: End-to-end pipeline (dsRID.sh, extract_whole.py, model_predict.py) that extracts
    dsRNA features from long-read alignments, trains a random forest, and scores candidate
    dsRNA regions
  repo: https://github.com/gxiaolab/dsRID
- name: Python 3.x
  role: Runtime for the dsRID scripts (extract_whole.py, model_predict.py) and package
    installation via pip/requirements.txt
- name: PacBio long-read RNA-seq
  role: Source data modality from which dsRNA features are extracted for model training
    and prediction
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1093/bioinformatics/btad649
    title: 'dsRID: <i>in silico</i> identification of dsRNA regions using long-read
      RNA-seq data'
  source_content_hash: sha256:52f77dc1c93aa867e3949ce533f0622e5548504eb2c3c213bada80a0a054cbc7
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/gxiaolab/dsRID
  tools:
  - PacBio long-read RNA-seq
  - Python
  - Python 3.x
  - dsRID
  edam_operation: http://edamontology.org/operation_3936
  edam_topics:
  - http://edamontology.org/topic_3308
  - http://edamontology.org/topic_3512
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.4
n_claims: 10
---

# machine-learning-prediction-pipeline-execution

## Summary

Executes the dsRID machine-learning prediction pipeline, which trains a random-forest model on PacBio long-read RNA-seq-derived dsRNA features and applies it to score candidate dsRNA regions genome-wide, yielding feature importance and cross-validation outputs.

## When to use

Use when a BAM file from PacBio long-read RNA-seq must be converted into predicted dsRNA regions with per-region probabilities, or when reproducing feature-importance (Skip_ratio 75.4%, Len_skip 15.9%) and cv_scores from the AD-trained model.

## When NOT to use

- You already have dsRID_whole.tsv or a dsRID feature table and only need model scoring — in that case run model_predict.py directly rather than re-running the dsRID.sh training process.
- Your input is short-read RNA-seq rather than PacBio long-read RNA-seq BAM data, since dsRID is designed to leverage long-read sequencing.
- You need calibrated per-base dsRNA annotations rather than region-level predicted probabilities from the random-forest model.

## Inputs

- PacBio long-read RNA-seq data in BAM format (input_file for dsRID.sh)
- Feature table /data/Pacbio_AD_data.tsv for the pretrained AD-trained model path
- Whole-genome scan predictions dsRID_whole.tsv produced by extract_whole.py

## Outputs

- dsRID_train.tsv and dsRID_null.tsv (positive and negative training dsRNA regions)
- dsRID_data.tsv (concatenated train + null feature table)
- dsRID_whole.tsv (whole-genome scan with predicted probability column per candidate dsRNA region)
- feat_importance.tsv (permutation-based feature contribution scores)
- cv_scores.tsv (cross-validation accuracy results)

## How to apply

Install dsRID by cloning https://github.com/gxiaolab/dsRID and running `pip install -r requirements.txt` under Python 3.x. For the full pipeline, run `bash dsRID.sh <input_file> <output_dir>` with the input PacBio BAM; the output folder contains dsRID_train.tsv, dsRID_null.tsv, dsRID_whole.tsv, dsRID_data.tsv, feat_importance.tsv, and cv_scores.tsv, where dsRID_data.tsv concatenates positive (dsRID_train) and negative (dsRID_null) dsRNA regions for random-forest training. To skip training and only predict, first run extract_whole.py to produce the whole-genome scan table dsRID_whole.tsv, then run model_predict.py with `-i /data/Pacbio_AD_data.tsv` (the pretrained AD-trained model's feature table) and `-p <output_dir>/dsRID_whole.tsv`; dsRID_whole.tsv carries a column of predicted probability to be a candidate dsRNA region. After model_predict.py, check feat_importance.tsv (feature contribution scores; skipping ratio Skip_ratio ~75.4% and length of skipping Len_skip ~15.9% in the PacBio AD-trained model, Fig. 2E–F and Suppl. Fig. S4) and cv_scores.tsv (cross-validation accuracy).

## Related tools

- **dsRID** (End-to-end pipeline (dsRID.sh, extract_whole.py, model_predict.py) that extracts dsRNA features from long-read alignments, trains a random forest, and scores candidate dsRNA regions) — https://github.com/gxiaolab/dsRID
- **Python 3.x** (Runtime for the dsRID scripts (extract_whole.py, model_predict.py) and package installation via pip/requirements.txt)
- **PacBio long-read RNA-seq** (Source data modality from which dsRNA features are extracted for model training and prediction)

## Examples

```
python src/model_predict.py -i /data/Pacbio_AD_data.tsv -p <output_dir>/dsRID_whole.tsv -o <output_dir>
```

## Evaluation signals

- feat_importance.tsv reproduces the reported permutation contributions for the PacBio AD-trained model: Skip_ratio ~75.4% and Len_skip ~15.9% (Fig. 2E–F, Suppl. Fig. S4).
- cv_scores.tsv contains cross-validation accuracy results for the random-forest model and is non-empty after model_predict.py runs.
- dsRID_whole.tsv includes a column of predicted probability to be a candidate dsRNA region for every scanned region.
- All six expected output files (dsRID_train.tsv, dsRID_null.tsv, dsRID_whole.tsv, dsRID_data.tsv, feat_importance.tsv, cv_scores.tsv) are present under <output_dir>/ after dsRID.sh.
- R2 evaluated on a held-out validation split of the AD data is consistent with the reported model performance before trusting permutation contributions.

## Limitations

- No specific version of dsRID or pinned package versions are stated; only that required packages are listed in requirements.txt.
- Training requires PacBio long-read RNA-seq BAM input; the README does not describe adaptation to other sequencing platforms.
- The trained model is a random forest specific to the PacBio AD dataset (e.g., /data/Pacbio_AD_data.tsv); porting to other tissues or conditions is not validated in the provided context.

## Evidence

- [abstract] dsRID is a machine learning-based method to predict dsRNA regions in silico using long-read RNA-seq data: "we report dsRID (double-stranded RNA identifier), a machine-learning-based method to predict dsRNA regions in silico, leveraging the power of long-read RNA-sequencing"
- [readme] Pipeline invocation and outputs: "Run the dsRID script to include the training process with the following command: bash dsRID.sh <input_file> <output_dir>"
- [readme] Probability column in dsRID_whole: "dsRID_whole will contain column of predicted probability to be a candidate dsRNA region from the random forest model trained on dsRID_data.tsv"
- [methods] Feature importance and cv scores outputs: "After running model_predict.py, you should see feat_importance and cv_scores, each represents feature importance results and cross validation accuracy results."
- [methods] Opt-out prediction path: "If you would like to opt out training process and directly proceed to prediction step, run"
