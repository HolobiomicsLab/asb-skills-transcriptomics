---
name: cancer-database-annotation-comparison
description: Use when evaluating a pretrained malignant-cell annotation model (Cancer-Finder,
  checkpoints/sc_pretrain_article.pkl) against database-curated tumor annotations
  across many scRNA-seq datasets to test generalization at scale.
when_to_use_negative:
- Input is hematologic tumor scRNA-seq data — the article explicitly does not recommend
  Cancer-Finder for hematologic tumor data
- Evaluating spatial transcriptomic spots rather than single-cell data — ST prediction
  requires spatial-transcriptomics-trained checkpoints and Cancer-Finder overlooks
  spatial relationships among spots
- Only one or a few datasets are available — the correlation-based database-scale
  validation is designed for many datasets (n = 152 in the reference study), not single-sample
  spot checks
edam_operation: http://edamontology.org/operation_3659
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_2640
tools:
- name: Cancer-Finder
  role: Domain generalization-based deep-learning model that predicts malignant vs.
    non-malignant cells per dataset, via the pretrained scRNA-seq checkpoint and infer.py
  repo: https://github.com/Patchouli-M/SequencingCancerFinder
- name: SCANPY
  role: 'Preprocessing: pp.normalize_total followed by log1p (global-scaling, log-transformed
    normalization) of raw counts before inference'
- name: PyTorch
  role: Deep-learning framework used to run Cancer-Finder inference (torch 1.13.1
    in the reference environment)
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1038/s41467-024-46413-6
    title: Domain generalization enables general cancer cell annotation in single-cell
      and spatial transcriptomics
  source_content_hash: sha256:9235153ffa44d35e14b7be4fff7e3b477dabc63dda3140cfbd82e03601010bb1
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/Patchouli-M/SequencingCancerFinder
  tools:
  - Cancer-Finder
  - PyTorch
  - SCANPY
  edam_operation: http://edamontology.org/operation_3659
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_2640
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8333333333333334
n_claims: 12
---

# cancer-database-annotation-comparison

## Summary

Validates Cancer-Finder's malignant cell predictions at database scale by comparing per-dataset malignant percentages predicted on external scRNA-seq databases (e.g., CancerSCEM) against the percentages reported by the database, using a two-tailed Pearson correlation.

## When to use

Use when evaluating a pretrained malignant-cell annotation model (Cancer-Finder, checkpoints/sc_pretrain_article.pkl) against database-curated tumor annotations across many scRNA-seq datasets to test generalization at scale.

## When NOT to use

- Input is hematologic tumor scRNA-seq data — the article explicitly does not recommend Cancer-Finder for hematologic tumor data
- Evaluating spatial transcriptomic spots rather than single-cell data — ST prediction requires spatial-transcriptomics-trained checkpoints and Cancer-Finder overlooks spatial relationships among spots
- Only one or a few datasets are available — the correlation-based database-scale validation is designed for many datasets (n = 152 in the reference study), not single-sample spot checks

## Inputs

- Pretrained scRNA-seq model checkpoint (checkpoints/sc_pretrain_article.pkl) used with infer.py
- Raw count matrices from 152 CancerSCEM scRNA-seq datasets
- Per-dataset malignant cell percentages reported by the CancerSCEM database

## Outputs

- Per-dataset malignant cell percentage predicted by Cancer-Finder for each of the 152 datasets
- Two-tailed Pearson correlation coefficient and p-value comparing predicted vs. database-reported malignant percentages

## How to apply

Load the pretrained single-cell checkpoint (checkpoints/sc_pretrain_article.pkl) used with infer.py for direct inference. For each database dataset (152 CancerSCEM scRNA-seq datasets in the reference study), normalize raw counts with SCANPY's pp.normalize_total followed by log1p (global-scaling, log-transformed normalization), as required by the model. Run Cancer-Finder inference on each normalized matrix with the default softmax threshold of 0.5 to label cells malignant or non-malignant, then aggregate per-cell predictions into a per-dataset malignant percentage. Compare predicted vs. database-reported percentages across all datasets with a two-tailed Pearson correlation; in the article this yielded r > 0.85 (p < 0.0001, n = 152), which is the acceptance criterion.

## Related tools

- **Cancer-Finder** (Domain generalization-based deep-learning model that predicts malignant vs. non-malignant cells per dataset, via the pretrained scRNA-seq checkpoint and infer.py) — https://github.com/Patchouli-M/SequencingCancerFinder
- **SCANPY** (Preprocessing: pp.normalize_total followed by log1p (global-scaling, log-transformed normalization) of raw counts before inference)
- **PyTorch** (Deep-learning framework used to run Cancer-Finder inference (torch 1.13.1 in the reference environment))

## Examples

```
import scanpy as sc; adata = sc.read_h5ad('cancerscem_dataset.h5ad'); sc.pp.normalize_total(adata); sc.pp.log1p(adata); python infer.py --checkpoint checkpoints/sc_pretrain_article.pkl --input normalized.h5ad --threshold 0.5
```

## Evaluation signals

- Pearson correlation between predicted and CancerSCEM-reported malignant percentages exceeds 0.85 with p < 0.0001 (n = 152, two-tailed)
- Every dataset's raw counts were normalized with pp.normalize_total + log1p before inference (no unnormalized input)
- Per-cell labels assigned with the default softmax threshold of 0.5; per-dataset percentages computed as malignant cells / total cells
- Inference uses the same pretrained checkpoint (sc_pretrain_article.pkl) as the article; accuracy on golden standard scRNA-seq datasets should be in the ~98.30% regime as a sanity reference

## Limitations

- Cancer-Finder is not recommended for hematologic tumor data
- Cancer-Finder overlooks spatial relationships among spots in ST data, limiting ST application
- Robust only to a certain percentage of mislabeling in the training set
- Some prediction errors may only be resolvable by increasing sequencing depth

## Evidence

- [abstract] rapidly identify malignant cells in single-cell data with an average accuracy of 95.16%: "rapidly identify malignant cells in single-cell data with an average accuracy of 95.16%"
- [discussion] not recommended for hematologic tumor data: "We do not therefore recommend Cancer-Finder for hematologic tumor data"
- [discussion] robust to a certain percentage of mislabeling in training set: "it is robust to a certain percentage of mislabeling in training set (Supplementary Fig. 11)"
- [discussion] sequencing depth limitation: "a problem that can be resolved by increasing the sequencing depth, Supplementary Fig. 12"
- [readme] pretrained checkpoint and infer.py: [evidence span withheld — anchor status: not_found]
