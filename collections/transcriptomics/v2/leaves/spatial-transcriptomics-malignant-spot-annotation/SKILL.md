---
name: spatial-transcriptomics-malignant-spot-annotation
description: Use when you have 10x Visium spot-level count matrices (slides with suspected
  malignant regions) and need reference-free malignant/non-malignant spot calls without
  a reference or spatial deconvolution.
when_to_use_negative:
- Input is hematologic tumor data — the article explicitly does not recommend Cancer-Finder
  for hematologic tumor data
- A reference-based annotation with known cell-type profiles is required, e.g. via
  deconvolution tools such as cell2location or CARD
- Accurate copy-number/clonal structure characterization (as from inferCNV, CopyKAT,
  or SCEVAN) is the goal rather than binary malignant/non-malignant spot calls
edam_topics:
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_0091
tools:
- name: Cancer-Finder
  role: Domain-generalization deep-learning model (via infer.py with the ST checkpoint)
    that produces spot-level malignant/non-malignant predictions
  repo: https://github.com/Patchouli-M/SequencingCancerFinder
- name: SCANPY
  role: 'Preprocessing: pp.normalize_total + log1p for log-transformed global-scaling
    normalization of Visium count matrices'
- name: Seurat
  role: Alternative preprocessing via NormalizeData for log-transformed global-scaling
    normalization
- name: PyTorch
  role: Deep-learning framework that loads and runs the pre-trained ST checkpoint
    for inference
provenance:
  source_task_ids:
  - task_006
  - task_007
  source_papers:
  - doi: 10.1038/s41467-024-46413-6
    title: Domain generalization enables general cancer cell annotation in single-cell
      and spatial transcriptomics
  source_content_hash: sha256:435ab89656a7db3a1ff59e3472b4dce2cf49fd76c33fdcf4e54b4d52f2281ac4
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/Patchouli-M/SequencingCancerFinder
  tools:
  - Cancer-Finder
  - PyTorch
  - SCANPY
  - Seurat
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3308
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8
n_claims: 10
---

# spatial-transcriptomics-malignant-spot-annotation

## Summary

Reference-free malignant spot annotation on 10x Visium spatial-transcriptomics (ST) slides using Cancer-Finder's domain-generalization deep-learning model retrained on spatial transcriptomic data, replacing CNV-based tools like inferCNV/CopyKAT or deconvolution methods like cell2location/CARD.

## When to use

Use when you have 10x Visium spot-level count matrices (slides with suspected malignant regions) and need reference-free malignant/non-malignant spot calls without a reference or spatial deconvolution.

## When NOT to use

- Input is hematologic tumor data — the article explicitly does not recommend Cancer-Finder for hematologic tumor data
- A reference-based annotation with known cell-type profiles is required, e.g. via deconvolution tools such as cell2location or CARD
- Accurate copy-number/clonal structure characterization (as from inferCNV, CopyKAT, or SCEVAN) is the goal rather than binary malignant/non-malignant spot calls

## Inputs

- 10x Visium spot expression count matrices (gene-by-spot)
- Pre-trained Cancer-Finder ST checkpoint (feature-extraction network sized for 5000 ST input genes)
- Optional pathologist-derived spot labels (malignant=1, non-malignant=0) for validation

## Outputs

- Spot-level softmax malignant-probability scores
- Binary spot predictions (malignant/non-malignant) at softmax threshold 0.5
- Per-slide accuracy values computed as (TP+TN)/N against pathologist annotations

## How to apply

Load the ST pre-trained Cancer-Finder checkpoint and the Visium spot expression matrices with pathologist-derived labels (malignant=1, non-malignant=0). Preprocess each count matrix with log-transformed global-scaling normalization (SCANPY 'pp.normalize_total' + 'log1p', or Seurat 'NormalizeData'). Run malignant-spot inference with Cancer-Finder's infer.py using the ST checkpoint and the default softmax threshold of 0.5 for 10x Visium data, writing spot-level softmax scores and binary predictions. Compare binarized predictions to pathologist annotations per slide, computing per-slide accuracy as (TP+TN)/N; for cancer types included in ST training, per-slide accuracy should fall in the reported 82.00–97.37% range (Fig. 4b,c). The model ignores spatial relationships among spots, so it operates spot-by-spot on expression only.

## Related tools

- **Cancer-Finder** (Domain-generalization deep-learning model (via infer.py with the ST checkpoint) that produces spot-level malignant/non-malignant predictions) — https://github.com/Patchouli-M/SequencingCancerFinder
- **SCANPY** (Preprocessing: pp.normalize_total + log1p for log-transformed global-scaling normalization of Visium count matrices)
- **Seurat** (Alternative preprocessing via NormalizeData for log-transformed global-scaling normalization)
- **PyTorch** (Deep-learning framework that loads and runs the pre-trained ST checkpoint for inference)

## Examples

```
python infer.py --checkpoint cancer_finder_st.pt --input visium_counts.h5ad --threshold 0.5
```

## Evaluation signals

- Each per-slide accuracy for cancers included in ST training lies within the reported 82.00–97.37% interval, matching Fig. 4b,c
- Per-slide accuracy computed as (TP+TN)/N against pathologist-derived labels (malignant=1, non-malignant=0) reproduces reported values across the 14 10x Visium slides
- Binarized predictions use the default softmax threshold of 0.5 for 10x Visium data
- Preprocessing applied log-transformed global-scaling normalization (SCANPY pp.normalize_total + log1p or Seurat NormalizeData) before inference

## Limitations

- Cancer-Finder is not recommended for hematologic tumor data
- Cancer-Finder overlooks the spatial relationships among spots in ST data, a factor that could otherwise enhance overall efficacy
- The model is robust only to a certain percentage of mislabeling in the training set (Supplementary Fig. 11)
- Some prediction errors may be resolvable only by increasing sequencing depth (Supplementary Fig. 12)

## Evidence

- [abstract] by replacing the single-cell training data with spatial transcriptomic datasets, Cancer-Finder can accurately identify malignant spots on spatial slides: "by replacing the single-cell training data with spatial transcriptomic datasets, Cancer-Finder can accurately identify malignant spots on spatial slides"
- [discussion] Cancer-Finder displays a high level of accuracy (82.00-97.37%) on ST data of trained tissues: "Cancer-Finder displays a high level of accuracy (82.00-97.37%) on ST data of trained tissues"
- [discussion] We do not therefore recommend Cancer-Finder for hematologic tumor data: "We do not therefore recommend Cancer-Finder for hematologic tumor data"
- [discussion] Cancer-Finder overlooks the spatial relationships among spots in ST data: "Cancer-Finder overlooks the spatial relationships among spots in ST data, a factor with the potential to enhance its overall efficacy"
- [readme] a domain generalization-based deep learning algorithm that can rapidly identify malignant cells/spots in single-cell and spatial transcriptomics data: "we present Cancer-Finder, a domain generalization-based deep learning algorithm that can rapidly identify malignant cells/spots in single-cell and spatial transcriptomics data"
