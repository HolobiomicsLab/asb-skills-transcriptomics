---
name: scree-plot-elbow-artifact-identification
description: Use when a scree plot of a highly sparse cell-gene matrix (e.g. ~95%
  zeros) shows an elbow or upper-edge eigenvalue deviations, and you must determine
  whether these are real signal or sparsity artifacts before interpreting PCs.
when_to_use_negative:
- The matrix is dense or only mildly sparse — sparsity-induced localization and artifactual
  elbows may not arise, so randomization controls add little
- The goal is to extract biological signal from the top eigenvectors rather than to
  diagnose sparsity artifacts — use the feature-selection/denoising skill instead
edam_operation: http://edamontology.org/operation_2928
edam_topics:
- http://edamontology.org/topic_3170
tools:
- name: Randomly
  role: Python 3 library used to compute covariance eigenvalues/eigenvectors and RMT-based
    diagnostics on the randomized single-cell matrix
  repo: https://github.com/luisaparicio/randomly
- name: Python 3
  role: Runtime environment — Randomly only works with Python 3 (not Python 2)
  repo: https://github.com/RabadanLab/randomly
- name: pip
  role: Installation method for the Randomly package
  repo: https://github.com/luisaparicio/randomly
- name: Jupyter Notebook
  role: Recommended environment for running Randomly and generating scree plots
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1016/j.patter.2020.100035
    title: A Random Matrix Theory Approach to Denoise Single-Cell Data
  source_content_hash: sha256:a5113988c14d1845534a65acee84bee29e1b6128c2c7b3e72f0c704d68b96e92
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
  edam_operation: http://edamontology.org/operation_2928
  edam_topics:
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.4
n_claims: 10
---

# scree-plot-elbow-artifact-identification

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

This skill exposes artifactual elbows in scree plots of sparse single-cell data by randomizing the matrix (permuting cell labels per gene), recomputing the covariance eigenvalue spectrum, and showing that the elbow persists even when all biological correlation is destroyed — demonstrating it is sparsity-induced rather than true signal.

## When to use

Use when a scree plot of a highly sparse cell-gene matrix (e.g. ~95% zeros) shows an elbow or upper-edge eigenvalue deviations, and you must determine whether these are real signal or sparsity artifacts before interpreting PCs.

## When NOT to use

- The matrix is dense or only mildly sparse — sparsity-induced localization and artifactual elbows may not arise, so randomization controls add little
- The goal is to extract biological signal from the top eigenvectors rather than to diagnose sparsity artifacts — use the feature-selection/denoising skill instead

## Inputs

- 95%-sparse cell-gene expression matrix of 6,573 human PBMCs (Kang et al.)
- Any highly sparse single-cell count matrix (cell x gene) loadable in Python 3

## Outputs

- Scree plot of eigenvalues of the covariance matrix of the randomized data showing the artifactual elbow (Figure 2A-2C style)
- Eigenvalue spectrum with quantified deviations from Tracy-Widom / Marchenko-Pastur predictions
- Eigenvector localization statistics for the randomized eigenvectors

## How to apply

Load the sparse cell-gene expression matrix (e.g. the 95%-sparse matrix of 6,573 human PBMCs from Kang et al.) into Python 3 with the Randomly library in a Jupyter Notebook. Randomize the dataset by permuting cell labels independently for each gene, which destroys all biological correlation while preserving each gene's sparsity profile. Compute the covariance matrix of the randomized matrix and extract its full eigenvalue and eigenvector spectrum. Compare the bulk of the randomized eigenvalue spectrum against the Marchenko-Pastur (MP) and Tracy-Widom (TW) universality distributions and quantify deviations; compute eigenvector localization statistics to show localization arises purely from sparsity. Finally, generate the scree plot of the randomized eigenvalues: an elbow that survives randomization is an artifactual elbow, since roughly 95% of the information in single-cell data is compatible with random matrix theory and about 3% is spurious sparsity-induced signal.

## Related tools

- **Randomly** (Python 3 library used to compute covariance eigenvalues/eigenvectors and RMT-based diagnostics on the randomized single-cell matrix) — https://github.com/luisaparicio/randomly
- **Python 3** (Runtime environment — Randomly only works with Python 3 (not Python 2)) — https://github.com/RabadanLab/randomly
- **pip** (Installation method for the Randomly package) — https://github.com/luisaparicio/randomly
- **Jupyter Notebook** (Recommended environment for running Randomly and generating scree plots)

## Examples

```
pip install randomly; python -c "import randomly"
```

## Evaluation signals

- The randomized (label-permuted) scree plot still exhibits an elbow and upper-edge eigenvalue deviations, reproducing Figure 2A-2C
- Eigenvector localization statistics are nonzero in the randomized data despite zero preserved correlation between cells
- The bulk of the randomized eigenvalue spectrum matches the MP/TW universality bulk, with deviations confined to the upper edge
- Per-gene sparsity profiles are preserved after permuting cell labels independently for each gene (invariant check on column sparsity)

## Limitations

- Artifactual elbow and localization in sparse randomized data can be falsely interpreted as true signal if the randomization control is not run
- Only ~2% of eigenvalue signal reflects true biology; ~3% is spurious sparsity-induced signal, so elbow position alone cannot separate biology from artifacts
- No version or release information is provided for the publicly available algorithm code

## Evidence

- [results] Sparsity-only artifacts reproduce localized eigenvectors, TW/MP deviations, and elbow: [evidence span withheld — anchor status: not_found]
