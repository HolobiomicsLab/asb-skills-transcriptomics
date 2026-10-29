---
name: scanpy-expression-normalization
description: Use when raw single-cell count matrices must be preprocessed for Cancer-Finder
  inference — e.g. applying the model at scale to external scRNA-seq datasets such
  as the 152 CancerSCEM databases.
when_to_use_negative:
- The input data is hematologic tumor scRNA-seq, where the article does not recommend
  Cancer-Finder
- The input is spatial transcriptomics (ST) data, which requires Cancer-Finder models
  retrained on spatial transcriptomic datasets rather than the scRNA-seq checkpoint
- The matrix is already normalized/log-transformed under a compatible scheme — re-normalizing
  with pp.normalize_total + log1p would double-transform the values
edam_operation: http://edamontology.org/operation_3223
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_3308
tools:
- name: SCANPY
  role: Performs pp.normalize_total and pp.log1p global-scaling, log-transformed normalization
    of raw counts before inference
- name: Cancer-Finder
  role: Consumes the normalized matrix via infer.py with the pre-trained checkpoint
    to classify cells as malignant/non-malignant at softmax threshold 0.5
  repo: https://github.com/Patchouli-M/SequencingCancerFinder
- name: PyTorch
  role: Runtime framework for loading the sc_pretrain_article.pkl checkpoint and running
    Cancer-Finder inference
  repo: https://github.com/Patchouli-M/SequencingCancerFinder
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1038/s41467-024-46413-6
    title: Domain generalization enables general cancer cell annotation in single-cell
      and spatial transcriptomics
  source_content_hash: sha256:9a628f9d38f85c244f44efd60ab55eb1a520311dde32673ee03fa7362c2150fc
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/Patchouli-M/SequencingCancerFinder
  tools:
  - Cancer-Finder
  - PyTorch
  - SCANPY
  edam_operation: http://edamontology.org/operation_3223
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9
n_claims: 10
---

# scanpy-expression-normalization

## Summary

Normalize raw scRNA-seq UMI count matrices with SCANPY's pp.normalize_total followed by log1p (global-scaling, log-transformed normalization) so they conform to the input distribution expected by the Cancer-Finder pre-trained model before running inference with infer.py.

## When to use

Use when raw single-cell count matrices must be preprocessed for Cancer-Finder inference — e.g. applying the model at scale to external scRNA-seq datasets such as the 152 CancerSCEM databases.

## When NOT to use

- The input data is hematologic tumor scRNA-seq, where the article does not recommend Cancer-Finder
- The input is spatial transcriptomics (ST) data, which requires Cancer-Finder models retrained on spatial transcriptomic datasets rather than the scRNA-seq checkpoint
- The matrix is already normalized/log-transformed under a compatible scheme — re-normalizing with pp.normalize_total + log1p would double-transform the values

## Inputs

- Raw scRNA-seq gene-by-cell UMI count matrix (e.g. from CancerSCEM datasets)
- Cancer-Finder pre-trained checkpoint checkpoints/sc_pretrain_article.pkl

## Outputs

- Log-transformed, total-count-normalized expression matrix ready for Cancer-Finder inference
- Per-cell malignant/non-malignant labels at softmax threshold 0.5
- Per-dataset malignant cell percentage for downstream Pearson correlation

## How to apply

Load each raw UMI count matrix and apply SCANPY's pp.normalize_total (global-scaling so each cell's counts sum to a common total), followed by pp.log1p, matching the normalization required by the Cancer-Finder pre-trained scRNA-seq checkpoint (checkpoints/sc_pretrain_article.pkl). Run the normalized matrices through Cancer-Finder's infer.py for direct inference using the default softmax threshold of 0.5 to label cells as malignant or non-malignant. The rationale is that the checkpoint was trained on log-transformed, globally scaled expression, so feeding raw or differently normalized counts would shift the input distribution and degrade malignancy predictions. Aggregating per-cell predictions into per-dataset malignant percentages and correlating them with database-reported values (Pearson r > 0.85, p < 0.0001, n = 152) confirms the normalization matches training conditions.

## Related tools

- **SCANPY** (Performs pp.normalize_total and pp.log1p global-scaling, log-transformed normalization of raw counts before inference)
- **Cancer-Finder** (Consumes the normalized matrix via infer.py with the pre-trained checkpoint to classify cells as malignant/non-malignant at softmax threshold 0.5) — https://github.com/Patchouli-M/SequencingCancerFinder
- **PyTorch** (Runtime framework for loading the sc_pretrain_article.pkl checkpoint and running Cancer-Finder inference) — https://github.com/Patchouli-M/SequencingCancerFinder

## Examples

```
import scanpy as sc; adata = sc.read_h5ad('raw_counts.h5ad'); sc.pp.normalize_total(adata); sc.pp.log1p(adata)
```

## Evaluation signals

- Confirm the matrix passed to Cancer-Finder has been through both pp.normalize_total and pp.log1p (log-scale values, not raw integer counts)
- Per-cell predictions use the default softmax threshold of 0.5
- Per-dataset malignant percentages predicted by Cancer-Finder on 152 CancerSCEM datasets show Pearson correlation > 0.85 with database-reported percentages (p < 0.0001, two-tailed test)
- The correct checkpoint (checkpoints/sc_pretrain_article.pkl) is loaded for direct inference with infer.py
- Predicted per-dataset malignant percentages fall within 0–100% and are computed across all 152 datasets

## Limitations

- Cancer-Finder is not recommended for hematologic tumor data, so this normalization-plus-inference pipeline is inappropriate for those inputs
- Prediction errors may be resolvable only by increasing sequencing depth, not by changing normalization
- The model is robust only to a certain percentage of mislabeling in the training set; normalization variations outside the training distribution may not be tolerated

## Evidence

- [methods] normalization for Cancer-Finder inference: [evidence span withheld — anchor status: not_found]
- [results] Cancer-Finder scale validation on CancerSCEM: [evidence span withheld — anchor status: not_found]
- [abstract] Cancer-Finder accuracy claim: "we present Cancer-Finder, a domain generalization-based deep-learning algorithm that can rapidly identify malignant cells in single-cell data with an average accuracy of 95.16%"
- [discussion] hematologic tumor limitation: "We do not therefore recommend Cancer-Finder for hematologic tumor data"
