---
name: synthetic-sparse-matrix-simulation
description: 'Use when benchmarking a scRNA-seq preprocessing pipeline''s signal detection:
  you need a controlled sparse synthetic matrix with a known TGC bias to test whether
  normalization creates or removes artificial eigenvalue signals.'
when_to_use_negative:
- The input is already real scRNA-seq data and the goal is biological signal detection
  rather than benchmarking normalization-induced artifacts — use the full scLENS pipeline
  instead of the synthetic simulation
- The data lacks a sparse binary block and TGC bias, so eigenvalue counts beyond the
  Marchenko-Pastur bulk would not be attributable to a controlled perturbation
edam_operation: http://edamontology.org/operation_2939
edam_topics:
- http://edamontology.org/topic_3524
- http://edamontology.org/topic_3308
tools:
- name: scLENS
  role: Provides read_file for loading the real count dataset data/Z8eq.csv.gz and
    embodies the log+L2 normalization plus RMT-based noise filtering being benchmarked
    on the synthetic matrices
  repo: github.com/Mathbiomed/scLENS
- name: scLENSpy
  role: Python package (Conda environment sclens-py with PyTorch) that can run the
    scLENS embedding used to validate the normalization effects
  repo: https://github.com/Mathbiomed/scLENSpy
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1038/s41467-024-47884-3
    title: 'scLENS: data-driven signal detection for unbiased scRNA-seq data analysis'
  source_content_hash: sha256:09e9c4ffacb2d4b7c50ad3c06394d735016e1bd46f1b951c796115cae31c801c
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - scLENS
  - scLENSpy
  edam_operation: http://edamontology.org/operation_2939
  edam_topics:
  - http://edamontology.org/topic_3524
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.625
n_claims: 8
---

# synthetic-sparse-matrix-simulation

## Summary

Reproduces the Fig. 2 random-matrix experiment from the scLENS paper, in which a TGC (transcriptional capture) bias is injected into a Poisson-noise sparse matrix to show that log normalization alone manufactures ~400 artificial signals while adding L2 normalization removes them all.

## When to use

Use when benchmarking a scRNA-seq preprocessing pipeline's signal detection: you need a controlled sparse synthetic matrix with a known TGC bias to test whether normalization creates or removes artificial eigenvalue signals.

## When NOT to use

- The input is already real scRNA-seq data and the goal is biological signal detection rather than benchmarking normalization-induced artifacts — use the full scLENS pipeline instead of the synthetic simulation
- The data lacks a sparse binary block and TGC bias, so eigenvalue counts beyond the Marchenko-Pastur bulk would not be attributable to a controlled perturbation

## Inputs

- 2000x2000 dense noise matrix with elements drawn from a Poisson distribution with mean 2
- Sparse variant: 2000x2000 dense block concatenated with a 2000x8000 sparse binary matrix
- TGC-biased variant with the first 400 rows of the dense part divided by two
- Real count dataset data/Z8eq.csv.gz (rows=cells, columns=genes, loaded via scLENS.read_file)

## Outputs

- Signal counts (eigenvalues above the Tracy-Widom 0.05 threshold) for log-normalization-only vs log+L2-normalization conditions
- Fitted Marchenko-Pastur distribution overlaid on the eigenvalue spectrum
- fig2_signal_detection_results.csv recording the signal counts per condition

## How to apply

Generate a 2000x2000 dense matrix with Poisson(mean 2) elements and a sparse variant built by concatenating a 2000x8000 sparse binary matrix. Introduce TGC bias by halving the first 400 rows of the dense part, creating a low-TGC cell group. Preprocess with log normalization (scaling factor L=1) plus per-gene z-score scaling, compute the cell similarity matrix G = X X^T / N, obtain all eigenvalues, fit the Marchenko-Pastur distribution, and count eigenvalues above the Tracy-Widom threshold (0.05 probability cutoff) as signals. Repeat after adding L2 normalization (dividing each cell vector by its Euclidean length) and confirm the artificial signal count drops to zero; validate on the real dataset data/Z8eq.csv.gz where scLENS.read_file is used and signal counts drop from 33 to 6 with L2 normalization.

## Related tools

- **scLENS** (Provides read_file for loading the real count dataset data/Z8eq.csv.gz and embodies the log+L2 normalization plus RMT-based noise filtering being benchmarked on the synthetic matrices) — github.com/Mathbiomed/scLENS
- **scLENSpy** (Python package (Conda environment sclens-py with PyTorch) that can run the scLENS embedding used to validate the normalization effects) — https://github.com/Mathbiomed/scLENSpy

## Examples

```
ndf = scLENS.read_file("data/Z8eq.csv.gz"); pre_df = scLENS.preprocess(ndf); sclens_embedding = scLENS.sclens(pre_df, device_=cur_dev)
```

## Evaluation signals

- TGC-biased sparse matrix under log normalization alone yields approximately 400 artificial signals
- Adding L2 normalization after log normalization reduces the TGC-biased matrix count to 0 signals
- Real dataset data/Z8eq.csv.gz shows signal count dropping from 33 to 6 with L2 normalization
- Eigenvalues exceeding the Marchenko-Pastur fit at the Tracy-Widom 0.05 probability cutoff are counted consistently across conditions
- All counts are written to fig2_signal_detection_results.csv with the expected pattern (≈400 → 0 on synthetic, 33 → 6 on real)

## Limitations

- The RMT-based approach requires computing all eigenvalues of the large cell-to-cell similarity matrix, demanding significant memory resources
- scLENS operation on large data with whole genes still requires a computer with high specifications
- L2 normalization is acknowledged as 'a very simple approach', and median scaling replaces conventional z-score gene scaling in scLENS itself, so the simulation's per-gene z-score step differs from the shipped pipeline

## Evidence

- [abstract] L2 normalization for artificial signal removal: "we identify the primary cause of signal distortion during log normalization and effectively address it by uniformizing cell vector lengths with L2 normalization"
- [discussion] RMT-based noise filtering: "provides a data-driven threshold to differentiate signals from noise by RMT-based noise filtering"
- [discussion] Log normalization scaling factor: "we set a scaling factor of 1 during the first step of log normalization"
- [readme] Loading the real validation dataset: "ndf = scLENS.read_file("data/Z8eq.csv.gz")"
- [discussion] Memory requirement of eigenvalue computation: "it necessitates the computation of all eigenvalues of the large cell-to-cell similarity matrix, requiring significant memory resources"
