---
name: pathologist-annotation-accuracy-evaluation
description: Use when validating Cancer-Finder (or similar) spot-level malignancy
  predictions on 10x Visium slides against pathologist annotations, e.g. reproducing
  the reported per-slide accuracy of 82.00–97.37% (Fig. 4b,c).
when_to_use_negative:
- Input is hematologic tumor data — the article states 'We do not therefore recommend
  Cancer-Finder for hematologic tumor data'.
- Slides from cancer types not included in the ST training set — the 82.00–97.37%
  accuracy claim applies only to trained tissues.
- Analysis requires modeling spatial dependencies — 'Cancer-Finder overlooks the spatial
  relationships among spots in ST data'.
tools:
- name: Cancer-Finder
  role: Produces spot-level softmax malignancy scores and binary predictions via its
    infer.py with the ST checkpoint and softmax threshold 0.5.
  repo: https://github.com/Patchouli-M/SequencingCancerFinder
- name: SCANPY
  role: 'Preprocessing: log-transformed global-scaling normalization of Visium count
    matrices (''pp.normalize_total'' + ''log1p'').'
- name: Seurat
  role: 'Alternative preprocessing: ''NormalizeData'' for log-transformed global-scaling
    normalization.'
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1038/s41467-024-46413-6
    title: Domain generalization enables general cancer cell annotation in single-cell
      and spatial transcriptomics
  source_content_hash: sha256:3cda6b9fb0a746f41206aaa9e27ef292964f9be06fcec55b3370513e2ec18b33
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/Patchouli-M/SequencingCancerFinder
  tools:
  - Cancer-Finder
  - SCANPY
  - Seurat
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 9
---

# pathologist-annotation-accuracy-evaluation

## Summary

Evaluates reference-free malignant-spot predictions on 10x Visium spatial transcriptomics slides by comparing binarized model calls to pathologist-derived malignant (1) / non-malignant (0) spot labels, computing per-slide accuracy as (TP+TN)/N and expecting values in the reported 82.00–97.37% range for trained cancer types.

## When to use

Use when validating Cancer-Finder (or similar) spot-level malignancy predictions on 10x Visium slides against pathologist annotations, e.g. reproducing the reported per-slide accuracy of 82.00–97.37% (Fig. 4b,c).

## When NOT to use

- Input is hematologic tumor data — the article states 'We do not therefore recommend Cancer-Finder for hematologic tumor data'.
- Slides from cancer types not included in the ST training set — the 82.00–97.37% accuracy claim applies only to trained tissues.
- Analysis requires modeling spatial dependencies — 'Cancer-Finder overlooks the spatial relationships among spots in ST data'.

## Inputs

- 10x Visium spot expression count matrices (14 slides)
- Pathologist-derived spot labels per slide (malignant=1, non-malignant=0)
- Cancer-Finder ST-pretrained checkpoint

## Outputs

- Spot-level softmax malignancy scores per slide
- Binary malignant/non-malignant spot predictions (threshold 0.5)
- Per-slide accuracy table (accuracy = (TP+TN)/N) over 14 slides

## How to apply

Load 14 10x Visium spot expression matrices with pathologist-derived malignant/non-malignant labels (malignant=1, non-malignant=0) and an ST-pretrained Cancer-Finder checkpoint. Preprocess each count matrix with log-transformed global-scaling normalization (SCANPY 'pp.normalize_total' followed by 'log1p', or Seurat 'NormalizeData'). Run Cancer-Finder's infer.py with the ST checkpoint and the default softmax threshold of 0.5 for 10x Visium data to produce spot-level softmax scores and binary predictions. Binarize at threshold 0.5, compare predictions to pathologist labels per slide, and compute accuracy as (TP+TN)/N. Tabulate all 14 per-slide accuracies and confirm that slides from cancer types included in ST training fall within the reported 82.00–97.37% interval, matching Fig. 4b,c.

## Related tools

- **Cancer-Finder** (Produces spot-level softmax malignancy scores and binary predictions via its infer.py with the ST checkpoint and softmax threshold 0.5.) — https://github.com/Patchouli-M/SequencingCancerFinder
- **SCANPY** (Preprocessing: log-transformed global-scaling normalization of Visium count matrices ('pp.normalize_total' + 'log1p').)
- **Seurat** (Alternative preprocessing: 'NormalizeData' for log-transformed global-scaling normalization.)

## Examples

```
import scanpy as sc; adata = sc.read_visium(slide_dir); sc.pp.normalize_total(adata); sc.pp.log1p(adata); python infer.py --checkpoint ST_pretrained --threshold 0.5
```

## Evaluation signals

- Each per-slide accuracy for cancers included in ST training lies within the reported 82.00–97.37% interval
- Per-slide accuracies match Fig. 4b,c of the article
- Every slide yields both spot-level softmax scores and binarized (threshold 0.5) predictions
- Accuracy is computed as (TP+TN)/N against pathologist labels, not against CNV-based calls
- All 14 slides are processed with log-transformed global-scaling normalization before inference

## Limitations

- Accuracy guarantee (82.00–97.37%) holds only for cancer types represented in the ST training set
- Cancer-Finder is robust only to a certain percentage of mislabeling in the training set (Supplementary Fig. 11)
- Prediction errors may be resolvable only by increasing sequencing depth (Supplementary Fig. 12)
- Cancer-Finder ignores spatial relationships among spots, which could otherwise improve efficacy

## Evidence

- [discussion] Cancer-Finder displays a high level of accuracy (82.00-97.37%) on ST data of trained tissues: "Cancer-Finder displays a high level of accuracy (82.00-97.37%) on ST data of trained tissues"
- [abstract] by replacing the single-cell training data with spatial transcriptomic datasets, Cancer-Finder can accurately identify malignant spots on spatial slides: "by replacing the single-cell training data with spatial transcriptomic datasets, Cancer-Finder can accurately identify malignant spots on spatial slides"
- [results] with per-slide accuracy of 82.00-97.37%: [evidence span withheld — anchor status: not_found]
- [discussion] not recommended for hematologic tumor data: "We do not therefore recommend Cancer-Finder for hematologic tumor data"
- [discussion] overlooks spatial relationships among spots: "Cancer-Finder overlooks the spatial relationships among spots in ST data, a factor with the potential to enhance its overall efficacy"
