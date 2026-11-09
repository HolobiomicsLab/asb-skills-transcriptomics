---
name: uci-benchmark-dataset-preparation
description: Use when benchmarking filter FS algorithms (mRMR, CFS, JMI, RELIEF, RRCT,
  etc.) and you need the study's real-world datasets — Spambase, SPECTF, SRBCT, Ovarian
  cancer, Cardiotocography, Handwriting, Relathe, Lymph — loaded in matrix form.
when_to_use_negative:
- The data are already local prepared (X, y) matrices and the task is running the
  19 FS algorithms or the RF evaluation rather than sourcing the benchmark data
- The task requires image or text deep-learning benchmarks rather than the tabular
  UCI/ASU FS Repository datasets used by this paper
- A dataset exceeds memory constraints — 'fat' datasets like Relathe (1,427 × 4,322)
  and Ovarian cancer (72 × 592) still fit in the paper's 64 GB RAM setup, but very
  large raw downloads may not
edam_operation: http://edamontology.org/operation_0224
edam_topics:
- http://edamontology.org/topic_0203
tools:
- name: MATLAB 2021b
  role: Runtime environment in which the prepared benchmark matrices are loaded and
    the UTS-style FS interface and random forest evaluation are executed
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1016/j.patter.2022.100471
    title: 'Relevance, redundancy, and complementarity trade-off (RRCT): A principled,
      generic, robust feature-selection tool'
  source_content_hash: sha256:74f79f6f01cf19f273330f368d386b1d35430876106ecb41432fbfb460b7b151
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - MATLAB
  - MATLAB 2021b
  - uts
  edam_operation: http://edamontology.org/operation_0224
  edam_topics:
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8571428571428571
n_claims: 14
---

# uci-benchmark-dataset-preparation

## Summary

Prepares the twelve benchmark datasets (four synthetic, eight real-world) used in the RRCT filter feature-selection study by downloading them from the UCI Machine Learning Repository and the ASU FS Repository and loading them as data matrices (X) with responses (y) for MATLAB 2021b experiments.

## When to use

Use when benchmarking filter FS algorithms (mRMR, CFS, JMI, RELIEF, RRCT, etc.) and you need the study's real-world datasets — Spambase, SPECTF, SRBCT, Ovarian cancer, Cardiotocography, Handwriting, Relathe, Lymph — loaded in matrix form.

## When NOT to use

- The data are already local prepared (X, y) matrices and the task is running the 19 FS algorithms or the RF evaluation rather than sourcing the benchmark data
- The task requires image or text deep-learning benchmarks rather than the tabular UCI/ASU FS Repository datasets used by this paper
- A dataset exceeds memory constraints — 'fat' datasets like Relathe (1,427 × 4,322) and Ovarian cancer (72 × 592) still fit in the paper's 64 GB RAM setup, but very large raw downloads may not

## Inputs

- UCI Machine Learning Repository downloads (Spambase, Cardiotocography, Lymph, etc.)
- ASU FS Repository downloads (Relathe, SPECTF, SRBCT, Ovarian cancer, Handwriting, etc.)
- Data matrix X and response y pairs, MATLAB-compatible (column-major numeric)

## Outputs

- Twelve prepared benchmark datasets: 8 real-world (X, y) matrices with documented dimensions and 4 synthetic datasets
- Train/test splits preserved for SPECTF [80 × 44; 187 × 44] and SRBCT [63 × 2,308; 25 × 2,308]

## How to apply

Download the eight real-world datasets from the UCI Machine Learning Repository (http://archive.ics.uci.edu/ml/datasets.html) and the ASU FS Repository (https://jundongl.github.io/scikit-feature/datasets.html), noting the published dimensions: Spambase [4,601 × 57], SPECTF [80 × 44; 187 × 44], Relathe [1,427 × 4,322], Ovarian cancer [72 × 592], Cardiotocography [2,129 × 21], Handwriting [2,000 × 649], Lymph [148 × 18], and SRBCT [63 × 2,308; 25 × 2,308] (train; test). Structure each dataset as a feature matrix X and response y, so that a UTS-style interface can be called with K set to M, or K=30 when the dataset has more than 30 features, returning features in descending order of importance. Keep the SPECTF and SRBCT train/test splits intact since out-of-sample misclassification percentage via the random forest (RF) classifier depends on them. All experiments in the paper were run on a desktop Windows 10 machine with an Intel i9-9900K CPU at 3.6 GHz with 64 GB RAM using MATLAB 2021b, so ensure the prepared matrices are MATLAB-compatible (column-major, numeric).

## Related tools

- **MATLAB 2021b** (Runtime environment in which the prepared benchmark matrices are loaded and the UTS-style FS interface and random forest evaluation are executed)

## Examples

```
X = readmatrix('spambase.data'); y = X(:,end); X(:,end) = [];  %% Spambase [4,601 × 57], ready for UTS-style FS calls
```

## Evaluation signals

- Dataset dimensions match the published shapes: Spambase 4,601 × 57, Relathe 1,427 × 4,322, Ovarian cancer 72 × 592, Cardiotocography 2,129 × 21, Handwriting 2,000 × 649, Lymph 148 × 18
- SPECTF and SRBCT retain their train/test structure (80/187 × 44 and 63/25 × 2,308) for out-of-sample RF evaluation
- Each prepared dataset yields a numeric (X, y) pair loadable in MATLAB 2021b and consumable by the UTS interface with K = min(M, 30)
- Downstream spot-check values reproduce the paper's figures (e.g. HITON 4.26% on Spambase, LOGO 0.90% on Handwriting), confirming the prepared inputs are the correct ones

## Limitations

- FEAST-based algorithms (mRMR, CMIM, JMI, DISR, CIFE) expect discrete inputs: continuous values must be discretised before use (10 equal-width bins is suggested, though this is data-set-size dependent), so the raw UCI/ASU matrices cannot be fed directly
- SPECTF and SRBCT require the exact train/test split; conflating them would break out-of-sample misclassification measurement
- UCI and ASU mirror availability and formatting can drift over time, so dimension checks against the paper's reported sizes are the main correctness safeguard

## Evidence

- [results] Downloaded from the UCI Machine Learning Repository (http://archive.ics.uci.edu/ml/datasets.html): "Downloaded from the UCI Machine Learning Repository (http://archive.ics.uci.edu/ml/datasets.html)"
- [results] ASU FS Repository source: "Downloaded from the ASU FS Repository (https://jundongl.github.io/scikit-feature/datasets.html"
- [results] Dataset dimensions: "Spambase [4,601 × 57]"
- [results] MATLAB environment: "All experiments were run on a desktop Windows 10 machine with an Intel i9-9900K CPU at 3.6 GHz with 64 GB RAM using MATLAB 2021b."
- [readme] FEAST discretisation requirement: "FEAST works on discrete inputs, and all continuous values **must** be
discretised before use with FEAST."
