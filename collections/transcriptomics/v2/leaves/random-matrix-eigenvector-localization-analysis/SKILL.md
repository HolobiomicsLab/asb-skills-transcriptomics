---
name: random-matrix-eigenvector-localization-analysis
description: Use when denoising a sparse single-cell gene expression matrix (e.g.
  95%-sparse scRNA-seq) and you need to decide which eigenvectors/gene sets reflect
  signal versus sparsity artifacts.
when_to_use_negative:
- Input is dense/low-sparsity data where sparsity-induced eigenvector localization
  is not a concern (the method targets ~95%-sparse single-cell matrices).
- You need imputation or zero-inflated modeling (e.g. MAGIC, scImpute, ZIFA, ZIMB-WaVE,
  scVI/DCA) rather than RMT-based feature selection and denoising.
- The dataset is too small for a meaningful covariance spectrum, so Tracy-Widom/Marchenko-Pastur
  universality comparisons are unreliable.
edam_operation: http://edamontology.org/operation_2939
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_3523
tools:
- name: Randomly
  role: 'Python 3 library implementing the RMT denoising workflow: randomization,
    eigenvector localization tests, sparsity feature filtering, and low-rank representation'
  repo: https://github.com/luisaparicio/randomly
- name: Python 3
  role: Runtime required for Randomly (it does not work with Python 2)
- name: Jupyter Notebook
  role: Recommended environment for running the Randomly workflow interactively
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1016/j.patter.2020.100035
    title: A Random Matrix Theory Approach to Denoise Single-Cell Data
  source_content_hash: sha256:0b013a24bf7bc82bde5b094bbbce47c15b36bad45ed5bcb4e31355f68b07c530
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
  edam_operation: http://edamontology.org/operation_2939
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_3523
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.5714285714285714
n_claims: 7
---

# random-matrix-eigenvector-localization-analysis

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Analyzes eigenvalue/eigenvector spectra of a single-cell expression matrix via random matrix theory (RMT), using eigenvector localization as a phase-transition marker to separate noise, sparsity-induced spurious signal (~3%), and true biological signal (~2%) from the ~95% RMT-compatible bulk.

## When to use

Use when denoising a sparse single-cell gene expression matrix (e.g. 95%-sparse scRNA-seq) and you need to decide which eigenvectors/gene sets reflect signal versus sparsity artifacts.

## When NOT to use

- Input is dense/low-sparsity data where sparsity-induced eigenvector localization is not a concern (the method targets ~95%-sparse single-cell matrices).
- You need imputation or zero-inflated modeling (e.g. MAGIC, scImpute, ZIFA, ZIMB-WaVE, scVI/DCA) rather than RMT-based feature selection and denoising.
- The dataset is too small for a meaningful covariance spectrum, so Tracy-Widom/Marchenko-Pastur universality comparisons are unreliable.

## Inputs

- Cell-gene expression matrix, e.g. 95%-sparse matrix of 6,573 human PBMCs (Kang et al., GSE96583) or 3,005 mouse cortex cells (Zeisel et al., GSE60361)
- Randomized control matrix obtained by permuting cell labels per gene

## Outputs

- List of genes discarded by the sparsity filter (e.g. sparsity_filtered_genes.csv) plus the filtered matrix
- Set of genes responsible for potentially interesting biological signal (localized eigenvectors in the raw data)
- Low-rank representation of the single-cell data for downstream interpretation

## How to apply

Model the cell-gene matrix as a random matrix (noise) plus a low-rank perturbation (signal). First, randomize the dataset by independently permuting cell labels for each gene to build a sparsity-only control that preserves marginal expression but destroys inter-gene correlation. Decompose both matrices and test gene projections onto eigenvectors for deviations from the RMT universality distributions (TW/MP); large deviations or localization statistics indicate signal. Discard genes responsible for localized eigenvectors in the randomized case — this is the first algorithm step that suppresses spurious sparsity effects — then select genes behind localized eigenvectors in the raw data as candidate biological signal and produce a low-rank representation for interpretation.

## Related tools

- **Randomly** (Python 3 library implementing the RMT denoising workflow: randomization, eigenvector localization tests, sparsity feature filtering, and low-rank representation) — https://github.com/luisaparicio/randomly
- **Python 3** (Runtime required for Randomly (it does not work with Python 2))
- **Jupyter Notebook** (Recommended environment for running the Randomly workflow interactively)

## Examples

```
pip install randomly; import randomly  # run in Python 3 / Jupyter Notebook on the 95%-sparse PBMC matrix
```

## Evaluation signals

- The removed-gene list written to sparsity_filtered_genes.csv contains exactly the genes whose projections onto eigenvectors of the randomized matrix deviate from normality.
- After the sparsity filter, localized eigenvectors attributable to the randomized (sparsity-only) matrix are suppressed.
- Roughly 95% of the dataset information should be compatible with RMT predictions, ~3% flagged as sparsity-induced spurious signal, and ~2% retained as candidate biological signal.
- The permuted control matrix preserves each gene's marginal expression distribution while destroying inter-gene correlations (sanity check on randomization).
- Final output is a low-rank representation whose dominant eigenvectors correspond to groups of correlated cells.

## Limitations

- No predictive quantitative models of gene expression exist, so the correct distribution and i.i.d. assumption for the noise model are unclear.
- There is a lack of complete quantitative understanding of the different sorts of noise in single-cell measurements.
- No version or release information is provided for the publicly available algorithm code.
- Randomly only works with Python 3 (not Python 2).

## Evidence
