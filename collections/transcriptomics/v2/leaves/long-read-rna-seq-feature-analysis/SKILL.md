---
name: long-read-rna-seq-feature-analysis
description: Use when a trained random forest model (e.g., dsRID trained on /data/Pacbio_AD_data.tsv)
  needs feature-level interpretation, specifically to rank features like Skip_ratio
  and Len_skip by permutation contribution score.
when_to_use_negative:
- You have raw PacBio BAM alignments rather than an extracted feature table — run
  extract_whole.py first to produce dsRID_whole.tsv before feature contribution analysis.
- You want to train a new model rather than interpret an existing one — use the full
  dsRID.sh training workflow instead of the opt-out prediction path.
- Your data are short-read RNA-seq — dsRID is designed for long-read RNA-seq data
  such as PacBio.
edam_topics:
- http://edamontology.org/topic_3474
- http://edamontology.org/topic_3170
tools:
- name: dsRID
  role: Supplies the pretrained random forest model and model_predict.py script that
    computes feature importance (feat_importance) and cross-validation scores (cv_scores)
  repo: https://github.com/gxiaolab/dsRID
- name: Python 3.x
  role: Runtime for executing model_predict.py and the permutation-based feature contribution
    analysis
- name: PacBio long-read RNA-seq
  role: Source of the AD brain training features (/data/Pacbio_AD_data.tsv) on which
    contribution scores are computed
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1093/bioinformatics/btad649
    title: 'dsRID: <i>in silico</i> identification of dsRNA regions using long-read
      RNA-seq data'
  source_content_hash: sha256:9947e8a852b8d7d3745ea8510f4b6659b916f23aaf437a2cfefdbfe852ce5ff7
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/gxiaolab/dsRID
  tools:
  - PacBio long-read RNA-seq
  - Python
  - Python 3.x
  - dsRID
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3474
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5555555555555556
n_claims: 9
---

# long-read-rna-seq-feature-analysis

## Summary

Quantifies which molecular features of candidate dsRNA regions drive the dsRID random forest predictions, using permutation-based feature contribution analysis on PacBio long-read RNA-seq-derived features such as skipping ratio (Skip_ratio) and length of skipping (Len_skip).

## When to use

Use when a trained random forest model (e.g., dsRID trained on /data/Pacbio_AD_data.tsv) needs feature-level interpretation, specifically to rank features like Skip_ratio and Len_skip by permutation contribution score.

## When NOT to use

- You have raw PacBio BAM alignments rather than an extracted feature table — run extract_whole.py first to produce dsRID_whole.tsv before feature contribution analysis.
- You want to train a new model rather than interpret an existing one — use the full dsRID.sh training workflow instead of the opt-out prediction path.
- Your data are short-read RNA-seq — dsRID is designed for long-read RNA-seq data such as PacBio.

## Inputs

- PacBio AD feature table (/data/Pacbio_AD_data.tsv, TSV)
- Whole-genome scan predictions dsRID_whole.tsv produced by extract_whole.py
- Pretrained dsRID random-forest model (trained on dsRID_data.tsv, concatenation of dsRID_train positives and dsRID_null negatives)

## Outputs

- feat_importance.tsv (permutation-based feature contribution scores)
- cv_scores.tsv (cross-validation accuracy results, including R2 on the held-out validation split)

## How to apply

Run model_predict.py with the pretrained dsRID random-forest model, passing the PacBio AD feature table (-i /data/Pacbio_AD_data.tsv) and the whole-genome scan predictions dsRID_whole.tsv produced by extract_whole.py (-p <output_dir>/dsRID_whole.tsv), with results written to the output directory (-o). The script produces feat_importance.tsv (feature importance results) and cv_scores.tsv (cross validation accuracy results). For the permutation contribution analysis, evaluate the model with R2 on a held-out validation split of the AD data, then permute each feature vector individually to quantify its contribution to the model's predictions. Validate the reproduction by checking that Skip_ratio accounts for ~75.4% and Len_skip for ~15.9% of the contribution, as reported in Fig. 2E–F and Suppl. Fig. S4.

## Related tools

- **dsRID** (Supplies the pretrained random forest model and model_predict.py script that computes feature importance (feat_importance) and cross-validation scores (cv_scores)) — https://github.com/gxiaolab/dsRID
- **Python 3.x** (Runtime for executing model_predict.py and the permutation-based feature contribution analysis)
- **PacBio long-read RNA-seq** (Source of the AD brain training features (/data/Pacbio_AD_data.tsv) on which contribution scores are computed)

## Examples

```
python src/model_predict.py -i /data/Pacbio_AD_data.tsv -p <output_dir>/dsRID_whole.tsv -o <output_dir>
```

## Evaluation signals

- feat_importance.tsv and cv_scores.tsv both exist in the output directory after running model_predict.py
- Reproduced permutation contributions match the reported values: Skip_ratio ≈ 75.4% and Len_skip ≈ 15.9%
- Skip_ratio is ranked as the highest-contributing feature and Len_skip as second, consistent with Fig. 2E–F and Suppl. Fig. S4
- cv_scores.tsv contains R2 evaluation on the held-out validation split of the AD data
- dsRID_whole.tsv input contains a column of predicted probability from the random forest model trained on dsRID_data.tsv

## Limitations

- No specific version of dsRID or package versions are stated; required packages are only listed in requirements.txt
- Permutation contribution scores are specific to the PacBio AD-trained model; results may differ for models trained on other datasets
- The opt-out path assumes the feature table (/data/Pacbio_AD_data.tsv) and dsRID_whole.tsv already exist; running model_predict.py directly on raw BAM input is not supported

## Evidence

- [other] Permutation contribution ranking: [evidence span withheld — anchor status: not_found]
- [readme] model_predict.py outputs: "After running model_predict.py, you should see feat_importance and cv_scores, each represents feature importance results and cross validation accuracy results."
- [methods] prediction invocation: "python src/model_predict.py -i /data/Pacbio_AD_data.tsv -p <output_dir>/dsRID_whole.tsv -o <output_dir>"
- [readme] dsRID_whole content: "dsRID_whole will contain column of predicted probability to be a candidate dsRNA region from the random forest model trained on dsRID_data.tsv."
