---
name: pearson-correlation-benchmarking
description: Use when a model predicts per-cell malignant labels on many externally
  curated scRNA-seq datasets and you must verify that aggregated per-dataset malignant
  fractions match those reported by the source database (e.g., CancerSCEM) using Pearson
  correlation.
when_to_use_negative:
- Hematologic tumor data — the article explicitly does not recommend Cancer-Finder
  for hematologic tumor data
- Spatial transcriptomics spots — this single-cell workflow requires the spatial-trained
  variant; applying the scRNA-seq checkpoint to ST slides is out of scope
- When per-cell CNV profiles or clonal structure are the target output (e.g., CopyKAT
  or SCEVAN use cases) rather than dataset-level percentage validation
edam_operation: http://edamontology.org/operation_2945
edam_topics:
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_3308
tools:
- name: Cancer-Finder
  role: Domain generalization deep-learning model run via infer.py with the sc_pretrain_article.pkl
    checkpoint to label cells as malignant (softmax >= 0.5) before percentage aggregation
  repo: https://github.com/Patchouli-M/SequencingCancerFinder
- name: SCANPY
  role: 'Performs the required preprocessing: pp.normalize_total followed by pp.log1p
    global-scaling, log-transformed normalization of raw counts'
- name: PyTorch
  role: Deep-learning framework backing Cancer-Finder inference (torch 1.13.1, CUDA
    11.6 per README prerequisites)
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1038/s41467-024-46413-6
    title: Domain generalization enables general cancer cell annotation in single-cell
      and spatial transcriptomics
  source_content_hash: sha256:a91fd8fec7d72daf9aba98c38170e0ccfff5663e559beff3e1d1b50e98331e56
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/Patchouli-M/SequencingCancerFinder
  tools:
  - Cancer-Finder
  - PyTorch
  - SCANPY
  edam_operation: http://edamontology.org/operation_2945
  edam_topics:
  - http://edamontology.org/topic_0203
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 8
---

# pearson-correlation-benchmarking

## Summary

A validation pattern that aggregates Cancer-Finder's per-cell malignant predictions into per-dataset malignant percentages across 152 CancerSCEM scRNA-seq datasets and checks them against database-reported percentages via a two-tailed Pearson correlation, accepting results with r > 0.85 and p < 0.0001.

## When to use

Use when a model predicts per-cell malignant labels on many externally curated scRNA-seq datasets and you must verify that aggregated per-dataset malignant fractions match those reported by the source database (e.g., CancerSCEM) using Pearson correlation.

## When NOT to use

- Hematologic tumor data — the article explicitly does not recommend Cancer-Finder for hematologic tumor data
- Spatial transcriptomics spots — this single-cell workflow requires the spatial-trained variant; applying the scRNA-seq checkpoint to ST slides is out of scope
- When per-cell CNV profiles or clonal structure are the target output (e.g., CopyKAT or SCEVAN use cases) rather than dataset-level percentage validation

## Inputs

- Raw scRNA-seq count matrices from 152 CancerSCEM datasets
- Pre-trained Cancer-Finder checkpoint (checkpoints/sc_pretrain_article.pkl)
- Per-dataset malignant cell percentages reported by the CancerSCEM database

## Outputs

- Per-cell malignant/non-malignant labels (softmax score >= 0.5)
- Per-dataset malignant cell percentages (n = 152)
- Two-tailed Pearson correlation coefficient r and p-value comparing predicted vs. database-reported percentages

## How to apply

Load the pre-trained single-cell checkpoint (checkpoints/sc_pretrain_article.pkl) and run Cancer-Finder inference with infer.py, normalizing each dataset's raw counts with SCANPY's pp.normalize_total followed by pp.log1p (global-scaling, log-transformed normalization) as required by the model. Apply the default softmax threshold of 0.5 to label each cell as malignant or non-malignant, then aggregate per-cell predictions into a per-dataset malignant percentage for all 152 datasets. Compute a two-tailed Pearson correlation between the predicted percentages and the percentages reported by the CancerSCEM database across the 152 datasets. Accept the benchmark when r exceeds 0.85 with p < 0.0001 (n = 152, two-tailed test), matching the correlation reported in the article.

## Related tools

- **Cancer-Finder** (Domain generalization deep-learning model run via infer.py with the sc_pretrain_article.pkl checkpoint to label cells as malignant (softmax >= 0.5) before percentage aggregation) — https://github.com/Patchouli-M/SequencingCancerFinder
- **SCANPY** (Performs the required preprocessing: pp.normalize_total followed by pp.log1p global-scaling, log-transformed normalization of raw counts)
- **PyTorch** (Deep-learning framework backing Cancer-Finder inference (torch 1.13.1, CUDA 11.6 per README prerequisites))

## Examples

```
sc.pp.normalize_total(adata); sc.pp.log1p(adata); labels = infer(adata, ckpt='checkpoints/sc_pretrain_article.pkl', thresh=0.5); r, p = pearsonr(pred_pct, db_pct)
```

## Evaluation signals

- Pearson correlation coefficient between predicted and CancerSCEM-reported percentages exceeds 0.85
- Correlation p-value < 0.0001 in a two-tailed test
- All 152 CancerSCEM datasets contribute one paired (predicted, reported) percentage each
- Per-dataset malignant percentages fall in the valid 0-100% range and are computed after the same normalize_total + log1p preprocessing and 0.5 softmax threshold used in the article

## Limitations

- Cancer-Finder is not recommended for hematologic tumor data, so dataset-level percentage correlations on such samples are unreliable
- Cancer-Finder is robust only to a certain percentage of mislabeling in the training set (Supplementary Fig. 11)
- Some prediction errors may be resolvable only by increasing sequencing depth (Supplementary Fig. 12)
- The correlation is a dataset-level aggregate metric; it does not certify per-cell classification accuracy

## Evidence

- [other] CancerSCEM percentage correlation finding: [evidence span withheld — anchor status: not_found]
- [other] Workflow comparison step: [evidence span withheld — anchor status: not_found]
- [readme] Cancer-Finder description from repo README: "we present Cancer-Finder, a domain generalization-based deep learning algorithm that can rapidly identify malignant cells/spots in single-cell and spatial transcriptomics data"
- [abstract] Accuracy benchmark: "rapidly identify malignant cells in single-cell data with an average accuracy of 95.16%"
