---
name: gene-wise-permutation-randomization
description: Use when analyzing a sparse single-cell expression matrix where you must
  separate sparsity-induced spurious eigenvector localization from true biological
  signal before RMT denoising.
when_to_use_negative:
- The dataset has already undergone sparsity-based feature filtering, so permuting
  again would not isolate new sparsity effects
- The analysis goal is to detect biological signal (localized eigenvectors in the
  raw data), not to suppress sparsity-induced effects — randomization destroys exactly
  the inter-gene correlations such signal depends on
- Working in Python 2 — Randomly only works with Python 3
tools:
- name: Randomly
  role: Python 3 library used to load, randomize (gene-wise permutation), decompose,
    and filter the sparse single-cell expression matrix
  repo: github.com/RabadanLab/randomly
- name: Python 3
  role: Runtime environment required by Randomly (not Python 2)
- name: pip
  role: Installer used to install Randomly from the command line
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1016/j.patter.2020.100035
    title: A Random Matrix Theory Approach to Denoise Single-Cell Data
  source_content_hash: sha256:37c6cedc2e3ee1944bb1af59cb94a546161661c219f764de428b4f7519885b61
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-ND-4.0
    url: https://github.com/RabadanLab/randomly
  tools:
  - Python 3
  - Randomly
  - pip
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.8181818181818182
n_claims: 11
---

# gene-wise-permutation-randomization

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

A randomization-based control from the Randomly library (Aparicio et al., Patterns 2020) that permutes cell labels independently for each gene to destroy inter-gene correlations while preserving marginal expression, exposing sparsity-induced signal in single-cell data.

## When to use

Use when analyzing a sparse single-cell expression matrix where you must separate sparsity-induced spurious eigenvector localization from true biological signal before RMT denoising.

## When NOT to use

- The dataset has already undergone sparsity-based feature filtering, so permuting again would not isolate new sparsity effects
- The analysis goal is to detect biological signal (localized eigenvectors in the raw data), not to suppress sparsity-induced effects — randomization destroys exactly the inter-gene correlations such signal depends on
- Working in Python 2 — Randomly only works with Python 3

## Inputs

- Sparse cell-gene expression matrix (~95% sparse), e.g., 6,573 human PBMCs (Kang et al., GSE96583)
- Sparse cell-gene expression matrix, e.g., 3,005 mouse cortex cells (Zeisel et al., GSE60361)
- Python 3 environment with the Randomly library installed

## Outputs

- Sparsity-only randomized control matrix (marginals preserved, inter-gene correlations destroyed)
- List of genes deviating from RMT-universality normal projections in the randomized data
- sparsity_filtered_genes.csv (removed-gene list) and the filtered expression matrix

## How to apply

Load the ~95%-sparse cell-gene expression matrix (e.g., 6,573 human PBMCs from Kang et al., GSE96583, or 3,005 mouse cortex cells from Zeisel et al., GSE60361) using the Randomly library in Python 3. Randomize the dataset by independently permuting the cell labels for each gene, producing a sparsity-only control matrix that preserves each gene's marginal expression distribution but destroys inter-gene correlations. Decompose the randomized matrix and test each gene's projections onto eigenvectors for deviations from the normal distribution expected under RMT universality (TW/MP statistics); genes showing such deviations are responsible for localized eigenvectors and are flagged as introducing spurious sparsity effects. The rationale is that only sparsity—not biology—survives randomization, so localization in the randomized matrix marks genes whose discreteness creates spurious eigenvector localization. The flagged genes are then discarded from the raw matrix as the first algorithm step, with the removed-gene list written to sparsity_filtered_genes.csv along with the filtered matrix.

## Related tools

- **Randomly** (Python 3 library used to load, randomize (gene-wise permutation), decompose, and filter the sparse single-cell expression matrix) — github.com/RabadanLab/randomly
- **Python 3** (Runtime environment required by Randomly (not Python 2))
- **pip** (Installer used to install Randomly from the command line)

## Examples

```
pip install randomly; import randomly
```

## Evaluation signals

- The randomized control matrix preserves each gene's marginal expression distribution (permutation within gene) while inter-gene correlations are destroyed
- Flagged genes correspond to projections onto eigenvectors that deviate from the normal distribution expected under RMT universality (TW/MP statistics)
- The set of discarded genes matches the localized-eigenvector-associated genes of the randomized (sparsity-only) matrix, and removal suppresses spurious eigenvector localization in the raw data
- Output files exist with expected content: sparsity_filtered_genes.csv lists removed genes and the filtered matrix excludes exactly those genes
- Quantitatively, the spurious sparsity component removed should be roughly ~3% of the information in the dataset (per the paper's decomposition: ~95% random-matrix-compatible, ~3% sparsity-induced, ~2% biological)

## Limitations

- Only ~3% of the information in single-cell data is spurious sparsity signal, so this filter removes a small fraction; the remaining biological signal (~2%) must be identified separately via RMT universality deviations
- The paper does not name specific statistical thresholds (p-values or cutoffs) for flagging deviations from normality of gene projections; the article's public code page provides no version or release information
- Randomly only works with Python 3 (not Python 2)

## Evidence
