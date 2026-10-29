---
name: per-dataset-malignant-fraction-aggregation
description: Use when you have per-cell malignant-cell predictions from a scRNA-seq
  annotation model across many datasets and need to compare dataset-level malignant
  fractions against reference percentages (e.g., CancerSCEM database values).
when_to_use_negative:
- Input data are from hematologic tumors — the article explicitly does not recommend
  Cancer-Finder for hematologic tumor data
- Single-cell datasets with no reference malignant-cell percentage available (e.g.,
  unannotated cohorts), since the skill's validation step requires database-reported
  fractions
- Spatial transcriptomics spots where spatial relationships among spots matter — Cancer-Finder
  overlooks spatial relationships in ST data
edam_operation: http://edamontology.org/operation_3664
edam_topics:
- http://edamontology.org/topic_0091
- http://edamontology.org/topic_0203
tools:
- name: Cancer-Finder
  role: Deep-learning model whose per-cell malignant predictions (softmax threshold
    0.5) are aggregated into per-dataset malignant fractions
  repo: https://github.com/Patchouli-M/SequencingCancerFinder
- name: SCANPY
  role: Performs the required per-dataset normalization (pp.normalize_total + log1p)
    of raw counts before Cancer-Finder inference
- name: PyTorch
  role: Deep-learning framework used to load the checkpoint (sc_pretrain_article.pkl)
    and run inference
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1038/s41467-024-46413-6
    title: Domain generalization enables general cancer cell annotation in single-cell
      and spatial transcriptomics
  source_content_hash: sha256:2c44883c12522804a691031344a8f78ff4b6541c81a940f04f793ece4434aad3
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/Patchouli-M/SequencingCancerFinder
  tools:
  - Cancer-Finder
  - PyTorch
  - SCANPY
  edam_operation: http://edamontology.org/operation_3664
  edam_topics:
  - http://edamontology.org/topic_0091
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 10
---

# per-dataset-malignant-fraction-aggregation

## Summary

Aggregates per-cell malignant/non-malignant predictions from Cancer-Finder into per-dataset malignant cell percentages across 152 CancerSCEM scRNA-seq datasets, then validates the predicted percentages against the database-reported percentages via a two-tailed Pearson correlation.

## When to use

Use when you have per-cell malignant-cell predictions from a scRNA-seq annotation model across many datasets and need to compare dataset-level malignant fractions against reference percentages (e.g., CancerSCEM database values).

## When NOT to use

- Input data are from hematologic tumors — the article explicitly does not recommend Cancer-Finder for hematologic tumor data
- Single-cell datasets with no reference malignant-cell percentage available (e.g., unannotated cohorts), since the skill's validation step requires database-reported fractions
- Spatial transcriptomics spots where spatial relationships among spots matter — Cancer-Finder overlooks spatial relationships in ST data

## Inputs

- Raw count matrices (UMI) from the 152 CancerSCEM scRNA-seq datasets
- Pre-trained Cancer-Finder single-cell checkpoint: checkpoints/sc_pretrain_article.pkl (used with infer.py)
- Per-dataset malignant cell percentages reported by the CancerSCEM database (n = 152)

## Outputs

- Per-cell malignant/non-malignant labels (softmax score thresholded at 0.5)
- Per-dataset malignant cell percentage for each of the 152 datasets
- Two-tailed Pearson correlation coefficient and p-value comparing predicted vs. database-reported percentages

## How to apply

For each of the 152 CancerSCEM scRNA-seq datasets, normalize raw counts with SCANPY's 'pp.normalize_total' followed by 'log1p' (global-scaling, log-transformed normalization), which is the input format the Cancer-Finder model requires. Run Cancer-Finder inference on each normalized matrix using the pre-trained single-cell checkpoint (checkpoints/sc_pretrain_article.pkl with infer.py) and the default softmax threshold of 0.5 to label individual cells as malignant or non-malignant. Aggregate the per-cell labels into a per-dataset percentage of malignant cells. Compute a two-tailed Pearson correlation across all 152 datasets between predicted percentages and the percentages reported by the CancerSCEM database. Accept the result when the Pearson correlation exceeds 0.85 with p < 0.0001 (n = 152, two-tailed test), matching the correlation reported in the article.

## Related tools

- **Cancer-Finder** (Deep-learning model whose per-cell malignant predictions (softmax threshold 0.5) are aggregated into per-dataset malignant fractions) — https://github.com/Patchouli-M/SequencingCancerFinder
- **SCANPY** (Performs the required per-dataset normalization (pp.normalize_total + log1p) of raw counts before Cancer-Finder inference)
- **PyTorch** (Deep-learning framework used to load the checkpoint (sc_pretrain_article.pkl) and run inference)

## Examples

```
sc.pp.normalize_total(adata); sc.pp.log1p(adata); python infer.py --checkpoint checkpoints/sc_pretrain_article.pkl --softmax_threshold 0.5
```

## Evaluation signals

- All 152 CancerSCEM datasets are processed and each yields exactly one per-dataset malignant percentage
- Every dataset is normalized with pp.normalize_total followed by log1p before inference (correct order and transform)
- Cell-level labeling uses the default softmax threshold of 0.5
- Two-tailed Pearson correlation across the 152 datasets exceeds 0.85 with p < 0.0001
- Per-dataset malignant percentages fall within valid bounds (0–100% or 0–1 depending on representation)

## Limitations

- Cancer-Finder is not recommended for hematologic tumor data
- The model is robust only to a certain percentage of mislabeling in the training set
- Prediction errors may require increased sequencing depth to resolve (Supplementary Fig. 12)

## Evidence

- [results] malignant-fraction correlation validation: [evidence span withheld — anchor status: not_found]
- [readme] readme-model-scope: "we present Cancer-Finder, a domain generalization-based deep learning algorithm that can rapidly identify malignant cells/spots in single-cell and spatial transcriptomics data"
- [abstract] abstract-accuracy: "rapidly identify malignant cells in single-cell data with an average accuracy of 95.16%"
