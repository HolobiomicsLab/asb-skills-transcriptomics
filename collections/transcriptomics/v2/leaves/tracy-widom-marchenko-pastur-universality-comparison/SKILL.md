---
name: tracy-widom-marchenko-pastur-universality-comparison
description: Use when analyzing a sparse single-cell expression matrix with PCA/RMT
  and you need to separate noise bulk, sparsity-induced artifacts, and true low-rank
  signal via eigenvalue deviations from MP/TW laws.
when_to_use_negative:
- Input data has already been sparsity-filtered or genes causing localization in the
  randomized case have been removed, so upper-edge deviations can be interpreted directly
  without this comparison
- The matrix is dense or the cell-gene matrix is so small that MP/TW asymptotic universality
  assumptions do not plausibly hold
edam_operation: http://edamontology.org/operation_2930
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_3512
tools:
- name: Randomly
  role: Python 3 library implementing the RMT denoising workflow, including covariance
    eigenvalue extraction and comparison to MP/TW universality distributions
  repo: https://github.com/luisaparicio/randomly
- name: Python 3
  role: Runtime required by Randomly (does not work with Python 2)
- name: pip
  role: Installation channel for Randomly from the command line
- name: Jupyter Notebook
  role: Convenient execution environment for running the Randomly workflow interactively
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1016/j.patter.2020.100035
    title: A Random Matrix Theory Approach to Denoise Single-Cell Data
  source_content_hash: sha256:9e81a907fa0f9e6fadb1c0cd702d47c552a30f73ff7edf6deb97a3f87643da65
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-ND-4.0
    url: https://github.com/RabadanLab/randomly
  repo_url: https://github.com/luisaparicio/randomly
  tools:
  - Jupyter Notebook
  - Python 3
  - Randomly
  - pip
  edam_operation: http://edamontology.org/operation_2930
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_3512
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.5555555555555556
n_claims: 9
---

# tracy-widom-marchenko-pastur-universality-comparison

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Compares the bulk eigenvalue spectrum of a single-cell covariance matrix against Marchenko-Pastur (MP) and Tracy-Widom (TW) universality predictions so that large deviations can be interpreted as candidate biological signal while deviations attributable to sparsity are first discounted via randomization.

## When to use

Use when analyzing a sparse single-cell expression matrix with PCA/RMT and you need to separate noise bulk, sparsity-induced artifacts, and true low-rank signal via eigenvalue deviations from MP/TW laws.

## When NOT to use

- Input data has already been sparsity-filtered or genes causing localization in the randomized case have been removed, so upper-edge deviations can be interpreted directly without this comparison
- The matrix is dense or the cell-gene matrix is so small that MP/TW asymptotic universality assumptions do not plausibly hold

## Inputs

- 95%-sparse cell-gene expression matrix (e.g., 6,573 human PBMCs from Kang et al.)
- Randomized version of the matrix (cell labels permuted per gene to preserve sparsity profiles)
- Covariance matrix of the expression matrix with its full eigenvalue/eigenvector spectrum

## Outputs

- Eigenvalue spectrum compared against Marchenko-Pastur and Tracy-Widom universality distributions with quantified upper-edge deviations
- Scree plot exposing the artifactual elbow in randomized sparse data (Figure 2A-2C)
- Classification of spectral variance into random bulk (~95%), sparsity-induced spurious signal (~3%), and true biological signal (~2%)

## How to apply

Model the single-cell dataset as a random matrix (noise) plus a low-rank perturbation (signal), using the ~95% sparse 6,573-cell PBMC matrix of Kang et al. as a reference scale where roughly 95% of information matches MP/TW predictions, about 3% is spurious sparsity-induced signal, and only the last ~2% is true biological signal. Compute the covariance matrix of the cell-gene matrix and extract the full eigenvalue spectrum, then compare the bulk against the MP density and TW-edge predictions. Critically, first randomize the data by permuting cell labels independently for each gene (preserving each gene's sparsity profile but destroying all correlations): upper-edge deviations and an artifactual elbow that persist in the randomized spectrum are sparsity artifacts, not signal. Only eigenvalue deviations that exceed the randomized-data deviations, ideally paired with eigenvector localization statistics distinguishing biological from sparsity-driven localization, are treated as signal for downstream gene selection and low-rank representation. Implementation is via the Randomly Python 3 library (pip install randomly), typically run in a Jupyter Notebook.

## Related tools

- **Randomly** (Python 3 library implementing the RMT denoising workflow, including covariance eigenvalue extraction and comparison to MP/TW universality distributions) — https://github.com/luisaparicio/randomly
- **Python 3** (Runtime required by Randomly (does not work with Python 2))
- **pip** (Installation channel for Randomly from the command line)
- **Jupyter Notebook** (Convenient execution environment for running the Randomly workflow interactively)

## Examples

```
pip install randomly; import randomly
```

## Evaluation signals

- Randomized (label-permuted) data still show localized eigenvectors and upper-edge MP deviations plus an artifactual elbow, reproducing Figure 2A-2C artifacts
- Roughly 95% of the eigenvalue bulk is compatible with MP/TW predictions, about 3% is spurious sparsity signal, and only the last ~2% reflects true biological signal
- Eigenvalue deviations flagged as signal exceed the deviations observed in the randomized sparse matrix
- Genes discarded by the sparsity filter are exactly those responsible for localized eigenvectors in the randomized case

## Limitations

- Sparsity alone induces eigenvector localization and eigenvalue deviations from TW/MP predictions that mimic true signal, so deviations must be benchmarked against randomized data
- No complete quantitative understanding of all noise sources in single-cell measurements exists, and the correct expression distribution or validity of i.i.d. assumptions is unclear
- The article provides no version or release information for the publicly available Randomly code, complicating reproducibility

## Evidence

- [intro] randomizing the PBMC matrix ... data still show localized eigenvectors and upper-edge deviations from the MP distribution: [evidence span withheld — anchor status: not_found]
