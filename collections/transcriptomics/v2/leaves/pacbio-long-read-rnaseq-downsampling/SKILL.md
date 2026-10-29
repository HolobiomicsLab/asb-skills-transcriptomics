---
name: pacbio-long-read-rnaseq-downsampling
description: 'Use when deciding the minimum PacBio long-read RNA-seq depth needed
  for dsRID training/prediction: you have a full-depth BAM (e.g. 4.27M reads) and
  must justify how many reads (1.5–2M plateau) suffice.'
when_to_use_negative:
- Input is short-read RNA-seq data — dsRID and this depth analysis are designed for
  PacBio long-read RNA-seq
- Input is already a feature table such as dsRID_data.tsv or dsRID_whole.tsv (BAM
  input is required for the training/extraction steps)
- Goal is simply to predict dsRNA regions at full sequencing depth without assessing
  depth sufficiency — use the standard dsRID workflow instead
edam_operation: http://edamontology.org/operation_2945
edam_topics:
- http://edamontology.org/topic_3168
tools:
- name: dsRID
  role: machine-learning pipeline (dsRID.sh, random forest) re-run at each subsampled
    depth to generate dsRID_train.tsv, dsRID_null.tsv, dsRID_data.tsv and cv_scores.tsv
  repo: https://github.com/gxiaolab/dsRID
- name: extract_whole.py
  role: extracts whole-genome regions (dsRID_whole.tsv) from each subsample BAM so
    the full-data model can make transfer predictions
  repo: https://github.com/gxiaolab/dsRID
- name: model_predict.py
  role: scores extracted regions with the full-data-trained model (-i /data/Pacbio_AD_data.tsv)
    and produces feat_importance and cv_scores
  repo: https://github.com/gxiaolab/dsRID
- name: PacBio
  role: source long-read RNA-seq data platform (AD brain library) whose BAM files
    are subsampled at descending depths
- name: Python
  role: runtime (Python 3.x or higher) executing dsRID.sh, extract_whole.py and model_predict.py
    at each subsampling depth
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1093/bioinformatics/btad649
    title: 'dsRID: <i>in silico</i> identification of dsRNA regions using long-read
      RNA-seq data'
  source_content_hash: sha256:4c691b1d88176fc16b161618eadd89c53b45d9520bfc563f736550252b4dfc21
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/gxiaolab/dsRID
  tools:
  - PacBio
  - PacBio long-read RNA-seq
  - Python
  - dsRID
  - extract_whole.py
  - model_predict.py
  edam_operation: http://edamontology.org/operation_2945
  edam_topics:
  - http://edamontology.org/topic_3168
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 15
---

# pacbio-long-read-rnaseq-downsampling

## Summary

A sensitivity-analysis skill that subsamples a PacBio long-read RNA-seq BAM at descending read depths and re-runs the dsRID machine-learning pipeline at each depth to determine how much sequencing depth is required for accurate dsRNA-region prediction, including a plateau estimate.

## When to use

Use when deciding the minimum PacBio long-read RNA-seq depth needed for dsRID training/prediction: you have a full-depth BAM (e.g. 4.27M reads) and must justify how many reads (1.5–2M plateau) suffice.

## When NOT to use

- Input is short-read RNA-seq data — dsRID and this depth analysis are designed for PacBio long-read RNA-seq
- Input is already a feature table such as dsRID_data.tsv or dsRID_whole.tsv (BAM input is required for the training/extraction steps)
- Goal is simply to predict dsRNA regions at full sequencing depth without assessing depth sufficiency — use the standard dsRID workflow instead

## Inputs

- PacBio long-read RNA-seq data in BAM format (e.g. a 4.27M-read PacBio AD brain library)
- dsRID pipeline from https://github.com/gxiaolab/dsRID with requirements.txt packages installed
- Full-data training features file /data/Pacbio_AD_data.tsv for transfer prediction

## Outputs

- Per-subsample dsRID output files: dsRID_train.tsv, dsRID_null.tsv, dsRID_whole.tsv, dsRID_data.tsv, feat_importance.tsv, cv_scores.tsv
- Depth-vs-accuracy table compiling self-trained and transfer accuracies (cv_scores.tsv per subsample)
- Plateau recommendation: minimum read depth (~1.5–2 million reads) at which model accuracy stabilizes

## How to apply

Prepare the PacBio long-read RNA-seq data in BAM format and randomly subsample the full-depth library (here 4.27M reads) at multiple descending depths down to ~0.43M reads. For each subsample, run the dsRID training script (`bash dsRID.sh <input_file> <output_dir>`), which produces dsRID_train.tsv (positive regions), dsRID_null.tsv (negative regions), and their concatenation dsRID_data.tsv used to train the random forest model. Evaluate each self-trained subsample model on a 5% held-out test split repeated 20 times, recording cross-validation accuracy in cv_scores.tsv via model_predict.py. In parallel, run `python src/extract_whole.py -b <input_bam_file> -o <output_dir>/dsRID_whole.tsv` to extract whole-genome regions and score them with the full-data-trained model (`python src/model_predict.py -i /data/Pacbio_AD_data.tsv ...`) to obtain transfer-prediction probabilities. Compile self-trained and transfer accuracies against read depth and compare the resulting curves (cf. Supplementary Fig. S2 trends) to identify the depth at which accuracy plateaus, reported here as roughly 1.5–2 million reads.

## Related tools

- **dsRID** (machine-learning pipeline (dsRID.sh, random forest) re-run at each subsampled depth to generate dsRID_train.tsv, dsRID_null.tsv, dsRID_data.tsv and cv_scores.tsv) — https://github.com/gxiaolab/dsRID
- **extract_whole.py** (extracts whole-genome regions (dsRID_whole.tsv) from each subsample BAM so the full-data model can make transfer predictions) — https://github.com/gxiaolab/dsRID
- **model_predict.py** (scores extracted regions with the full-data-trained model (-i /data/Pacbio_AD_data.tsv) and produces feat_importance and cv_scores) — https://github.com/gxiaolab/dsRID
- **PacBio** (source long-read RNA-seq data platform (AD brain library) whose BAM files are subsampled at descending depths)
- **Python** (runtime (Python 3.x or higher) executing dsRID.sh, extract_whole.py and model_predict.py at each subsampling depth)

## Examples

```
bash dsRID.sh <input_bam> <output_dir>; python src/extract_whole.py -b <input_bam_file> -o <output_dir>/dsRID_whole.tsv; python src/model_predict.py -i /data/Pacbio_AD_data.tsv -p <output_dir>/dsRID_whole.tsv -o <output_dir>
```

## Evaluation signals

- Each subsample run emits the full dsRID output set: dsRID_train.tsv, dsRID_null.tsv, dsRID_whole.tsv, dsRID_data.tsv, feat_importance.tsv, cv_scores.tsv
- Self-trained cross-validation accuracy exceeds 80% even at the lowest subsample depth (~0.43M reads)
- Transfer-prediction accuracy with the full-data-trained model stays above 88% at every subsample depth
- Accuracy increases monotonically with read depth and plateaus at roughly 1.5–2 million reads, matching the Supplementary Fig. S2 trend
- Each model's cross-validation accuracy is the result of 20 repeated 5% held-out train/test splits (cv_scores content)

## Limitations

- No specific dsRID version or package versions are stated — only 'required packages listed in the requirements.txt file'
- Accuracy estimates rely on a small 5% held-out test split repeated 20 times; absolute values may vary with the random subsampling seed
- The 1.5–2M read plateau is calibrated on one PacBio AD brain library (4.27M reads); it may not transfer to libraries with different coverage profiles or organism/tissue backgrounds

## Evidence

- [readme] Prepare the PacBio long-read RNA-seq data in the BAM format.: "Prepare the PacBio long-read RNA-seq data in the BAM format."
- [readme] After running model_predict.py, you should see feat_importance and cv_scores, each represents feature importance results and cross validation accuracy results.: "After running model_predict.py, you should see feat_importance and cv_scores, each represents feature importance results and cross validation accuracy results."
- [methods] python src/extract_whole.py -b <input_bam_file> -o <output_dir>/dsRID_whole.tsv: "python src/extract_whole.py -b <input_bam_file> -o <output_dir>/dsRID_whole.tsv"
- [methods] Run the dsRID script to include the training process with the following command: bash dsRID.sh <input_file> <output_dir>: "Run the dsRID script to include the training process with the following command: bash dsRID.sh <input_file> <output_dir>"
