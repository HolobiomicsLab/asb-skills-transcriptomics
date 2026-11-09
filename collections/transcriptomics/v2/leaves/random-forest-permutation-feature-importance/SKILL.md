---
name: random-forest-permutation-feature-importance
description: Use when a random-forest model (e.g., dsRID trained on Pacbio_AD_data.tsv)
  needs per-feature contribution scores rather than generic impurity-based importance,
  to explain which molecular traits drive dsRNA region prediction.
when_to_use_negative:
- When the goal is to train the model itself rather than explain an already-trained
  model — use the full dsRID.sh training pipeline instead.
- When the input is raw PacBio BAM reads rather than an extracted feature table —
  run extract_whole.py first.
- When a model class other than random forest is used, since the dsRID permutation
  workflow is built around its random-forest predictions.
edam_operation: http://edamontology.org/operation_3658
edam_topics:
- http://edamontology.org/topic_3474
- http://edamontology.org/topic_0080
tools:
- name: dsRID
  role: Provides model_predict.py, which performs the permutation-based feature importance
    analysis and writes feat_importance.tsv and cv_scores.tsv
  repo: https://github.com/gxiaolab/dsRID
- name: Python 3.x
  role: Runtime environment for executing model_predict.py and the permutation analysis
    scripts
- name: PacBio long-read RNA-seq
  role: Source of the AD brain training data on which the random-forest model being
    explained was trained
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1093/bioinformatics/btad649
    title: 'dsRID: <i>in silico</i> identification of dsRNA regions using long-read
      RNA-seq data'
  source_content_hash: sha256:6f7614fa8184dcc43d83b65f44db546d4692f0ac28b1cd8a9576a34b1650b959
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/gxiaolab/dsRID
  tools:
  - PacBio long-read RNA-seq
  - Python
  - Python 3.x
  - dsRID
  edam_operation: http://edamontology.org/operation_3658
  edam_topics:
  - http://edamontology.org/topic_3474
  - http://edamontology.org/topic_0080
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.4
n_claims: 10
---

# random-forest-permutation-feature-importance

## Summary

Quantify feature contributions of a trained random-forest model (e.g., the dsRID model trained on PacBio AD long-read RNA-seq data) by permuting each feature individually and measuring the impact on predictions, as implemented in dsRID's model_predict.py.

## When to use

Use when a random-forest model (e.g., dsRID trained on Pacbio_AD_data.tsv) needs per-feature contribution scores rather than generic impurity-based importance, to explain which molecular traits drive dsRNA region prediction.

## When NOT to use

- When the goal is to train the model itself rather than explain an already-trained model — use the full dsRID.sh training pipeline instead.
- When the input is raw PacBio BAM reads rather than an extracted feature table — run extract_whole.py first.
- When a model class other than random forest is used, since the dsRID permutation workflow is built around its random-forest predictions.

## Inputs

- /data/Pacbio_AD_data.tsv (PacBio AD feature table used for model input/validation)
- <output_dir>/dsRID_whole.tsv (whole-genome scan candidate dsRNA regions from extract_whole.py)
- Pretrained dsRID random-forest model

## Outputs

- feat_importance.tsv (permutation-based feature contribution scores)
- cv_scores.tsv (cross-validation / R2 validation metrics)

## How to apply

Load the PacBio AD feature table /data/Pacbio_AD_data.tsv and the whole-genome scan predictions dsRID_whole.tsv produced by extract_whole.py. Run model_predict.py with the pretrained random-forest model (-i /data/Pacbio_AD_data.tsv -p <output_dir>/dsRID_whole.tsv -o <output_dir>) to score candidate dsRNA regions, evaluate on a held-out validation split with R2, then permute each feature vector individually to quantify its contribution to predictions. Write permutation scores to feat_importance.tsv and validation metrics to cv_scores.tsv. Sanity-check that the dominant contributions match the published pattern: skipping ratio (Skip_ratio) ~75.4% and length of skipping (Len_skip) ~15.9%, as shown in Fig. 2E–F and Suppl. Fig. S4.

## Related tools

- **dsRID** (Provides model_predict.py, which performs the permutation-based feature importance analysis and writes feat_importance.tsv and cv_scores.tsv) — https://github.com/gxiaolab/dsRID
- **Python 3.x** (Runtime environment for executing model_predict.py and the permutation analysis scripts)
- **PacBio long-read RNA-seq** (Source of the AD brain training data on which the random-forest model being explained was trained)

## Examples

```
python src/model_predict.py -i /data/Pacbio_AD_data.tsv -p <output_dir>/dsRID_whole.tsv -o <output_dir>
```

## Evaluation signals

- feat_importance.tsv shows Skip_ratio as the top contributor at ~75.4%.
- feat_importance.tsv shows Len_skip as the second contributor at ~15.9%.
- cv_scores.tsv contains validation R2 metrics on the held-out split of the PacBio AD data.
- Both feat_importance.tsv and cv_scores.tsv appear under <output_dir> after model_predict.py completes.
- Reproduced contributions match Fig. 2E–F and Suppl. Fig. S4 of the paper.

## Limitations

- No version of dsRID or specific package versions are stated; only that required packages are listed in requirements.txt.
- Contribution values are dataset-specific: the 75.4%/15.9% split reflects the PacBio AD-trained model and may not transfer to other models or datasets.
- dsRID_whole.tsv probabilities are predicted only for candidate dsRNA regions from the random-forest model trained on dsRID_data.tsv.

## Evidence

- [methods] After running model_predict.py, you should see feat_importance and cv_scores, each represents feature importance results and cross validation accuracy results.: "After running model_predict.py, you should see feat_importance and cv_scores, each represents feature importance results and cross validation accuracy results."
- [readme] python src/model_predict.py -i /data/Pacbio_AD_data.tsv -p <output_dir>/dsRID_whole.tsv -o <output_dir>: "python src/model_predict.py -i /data/Pacbio_AD_data.tsv -p <output_dir>/dsRID_whole.tsv -o <output_dir>"
- [readme] dsRID_whole will contain column of predicted probability to be a candidate dsRNA region from the random forest model trained on dsRID_data.tsv.: "dsRID_whole will contain column of predicted probability to be a candidate dsRNA region from the random forest model trained on dsRID_data.tsv."
- [results] We evaluated the performance of dsRID using PacBio long-read RNA-seq data derived from Alzheimer's disease (AD) brain: "We evaluated the performance of dsRID using PacBio long-read RNA-seq data derived from Alzheimer's disease (AD) brain"
