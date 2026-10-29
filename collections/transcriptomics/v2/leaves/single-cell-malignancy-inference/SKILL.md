---
name: single-cell-malignancy-inference
description: Use when you need to label individual cells as malignant or non-malignant
  in human tumor scRNA-seq data without a reference, especially across pan-cancer
  TISCH datasets or to benchmark annotation speed against CNV-based tools.
when_to_use_negative:
- The data are hematologic tumor cells — the article explicitly does not recommend
  Cancer-Finder for hematologic tumor data
- The goal is to characterize clonal/copy-number substructure or infer CNV profiles
  (use CopyKAT or SCEVAN instead)
- Spatial-adjacency-aware spot annotation is needed — Cancer-Finder overlooks spatial
  relationships among spots in ST data
edam_operation: http://edamontology.org/operation_1777
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_2640
tools:
- name: Cancer-Finder
  role: Primary domain-generalization classifier; pre-trained checkpoint used with
    infer.py for direct malignant cell inference
  repo: https://github.com/Patchouli-M/SequencingCancerFinder
- name: SCANPY
  role: 'Preprocessing: pp.normalize_total + log1p global-scaling normalization of
    raw counts before inference'
- name: PyTorch
  role: Deep learning framework backing the Cancer-Finder model checkpoint (torch
    1.13.1, CUDA 11.6)
- name: CaSee
  role: Baseline transfer-learning comparator run on GPU for speed/memory benchmarking
  repo: https://github.com/yuansh3354/CaSee
- name: CopyKAT
  role: CNV-based baseline run on CPU with default parameters; batch data above 10,000
    cells per its documentation
  repo: https://github.com/navinlabcode/copykat
- name: SCEVAN
  role: Clonal-structure baseline run on CPU with default parameters in the benchmark
    comparison
  repo: https://github.com/AntonioDeFalco/SCEVAN
- name: ikarus
  role: Machine-learning baseline run on CPU with default parameters in the benchmark
    comparison
  repo: https://github.com/BIMSBbioinfo/ikarus
provenance:
  source_task_ids:
  - task_001
  - task_004
  - task_003
  source_papers:
  - doi: 10.1038/s41467-024-46413-6
    title: Domain generalization enables general cancer cell annotation in single-cell
      and spatial transcriptomics
  source_content_hash: sha256:fd7967890031ef8f148ec4af5b97f021aed6399211ce948215e9f48a804d1a3b
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/Patchouli-M/SequencingCancerFinder
  tools:
  - CaSee
  - Cancer-Finder
  - CopyKAT
  - PyTorch
  - SCANPY
  - SCEVAN
  - ikarus
  edam_operation: http://edamontology.org/operation_1777
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_2640
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8888888888888888
n_claims: 18
---

# single-cell-malignancy-inference

## Summary

Reference-free inference of malignant vs non-malignant cells in scRNA-seq data using the pre-trained Cancer-Finder domain-generalization deep learning model, applied via infer.py to SCANPY-normalized count matrices.

## When to use

Use when you need to label individual cells as malignant or non-malignant in human tumor scRNA-seq data without a reference, especially across pan-cancer TISCH datasets or to benchmark annotation speed against CNV-based tools.

## When NOT to use

- The data are hematologic tumor cells — the article explicitly does not recommend Cancer-Finder for hematologic tumor data
- The goal is to characterize clonal/copy-number substructure or infer CNV profiles (use CopyKAT or SCEVAN instead)
- Spatial-adjacency-aware spot annotation is needed — Cancer-Finder overlooks spatial relationships among spots in ST data

## Inputs

- Raw scRNA-seq gene-by-cell count matrix (tsv/csv) from tumor microenvironment datasets (e.g., TISCH)
- Pre-trained Cancer-Finder checkpoint file checkpoints/sc_pretrain_article.pkl
- Cell malignancy annotations (e.g., TISCH database labels) for accuracy evaluation
- Malignant cell percentages per dataset (e.g., from CancerSCEM, n = 152 datasets) for correlation checks

## Outputs

- Per-cell malignant/non-malignant predictions (softmax score thresholded at 0.5)
- Per-tissue accuracy values ((TP + TN) / N) and cell-count-weighted average accuracy across 13 TISCH tissues
- Inference runtime and peak-memory table per dataset size (e.g., inference_benchmark.csv)

## How to apply

Load the pre-trained checkpoint (checkpoints/sc_pretrain_article.pkl) and run infer.py (or the 'infering' function on an AnnData object) with the --ckp parameter. First normalize raw UMI counts with SCANPY's pp.normalize_total followed by log1p (global-scaling, log-transformed normalization), as required by the model. Classify each cell as malignant/non-malignant using the default softmax threshold of 0.5 (c = 2 classes). Compare predictions against TISCH database malignancy annotations and compute per-tissue accuracy as (TP + TN) / N; the internal validation on 328,230 cells (20% fold of the 4:1 train/validation split of 13 TISCH tissues) should reproduce a cell-count-weighted average accuracy of 95.16%. For scale benchmarks, subsample to 100–1,000,000 cells and record wall-clock time from data reading to output on an NVIDIA RTX 3090 GPU platform.

## Related tools

- **Cancer-Finder** (Primary domain-generalization classifier; pre-trained checkpoint used with infer.py for direct malignant cell inference) — https://github.com/Patchouli-M/SequencingCancerFinder
- **SCANPY** (Preprocessing: pp.normalize_total + log1p global-scaling normalization of raw counts before inference)
- **PyTorch** (Deep learning framework backing the Cancer-Finder model checkpoint (torch 1.13.1, CUDA 11.6))
- **CaSee** (Baseline transfer-learning comparator run on GPU for speed/memory benchmarking) — https://github.com/yuansh3354/CaSee
- **CopyKAT** (CNV-based baseline run on CPU with default parameters; batch data above 10,000 cells per its documentation) — https://github.com/navinlabcode/copykat
- **SCEVAN** (Clonal-structure baseline run on CPU with default parameters in the benchmark comparison) — https://github.com/AntonioDeFalco/SCEVAN
- **ikarus** (Machine-learning baseline run on CPU with default parameters in the benchmark comparison) — https://github.com/BIMSBbioinfo/ikarus

## Examples

```
python infer.py --ckp checkpoints/sc_pretrain_article.pkl --input normalized_counts.h5ad
```

## Evaluation signals

- Weighted average accuracy across the 13 TISCH internal-validation tissues reproduces 95.16%, with 11 of 13 tissues exceeding 90% malignant-status prediction accuracy
- Accuracy in golden standard scRNA-seq datasets reaches 98.30% and similarity in silver standard datasets reaches 90.89%
- Pearson correlation between predicted and CancerSCEM-reported malignant cell percentages across 152 datasets exceeds 0.85 (p < 0.0001, two-tailed)
- Inference times on 100/1,000/10,000/100,000/1,000,000 cells match 1.39 s / 2.53 s / 58.65 s / 3,903.41 s / 38,903.92 s, faster than CaSee, CopyKAT, SCEVAN and ikarus with lower memory
- Binary-stored matrices at 10,000 and 100,000 cells show csv-parsing overhead reduction to ~4.15 s and ~39.46 s

## Limitations

- Not recommended for hematologic tumor data
- Overlooks spatial relationships among spots in ST data, which could otherwise improve efficacy
- Robust only to a certain percentage of mislabeling in the training set (Supplementary Fig. 11)
- Prediction errors may be resolvable only by increasing sequencing depth (Supplementary Fig. 12)

## Evidence

- [abstract] rapidly identify malignant cells in single-cell data with an average accuracy of 95.16%: "we present Cancer-Finder, a domain generalization-based deep-learning algorithm that can rapidly identify malignant cells in single-cell data with an average accuracy of 95.16%"
- [discussion] not recommended for hematologic tumor data: "We do not therefore recommend Cancer-Finder for hematologic tumor data"
- [discussion] overlooks spatial relationships among spots: "Cancer-Finder overlooks the spatial relationships among spots in ST data, a factor with the potential to enhance its overall efficacy"
- [readme] environment prerequisites from README: "System: Ubuntu 18.04
python: 3.9.16
CUDA: 11.6
torch: 1.13.1"
- [intro] copykat as CNV-based comparator: "Representative tools identifying malignant cells/spots in this manner are inferCNV7,12 and CopyKAT13."
