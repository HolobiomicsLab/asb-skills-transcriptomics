---
name: marchenko-pastur-distribution-fitting
description: Use when preprocessing scRNA-seq count data (log normalization with L=1,
  gene scaling, L2 normalization) and needing a data-driven signal/noise threshold
  from the eigenvalue spectrum of the cell similarity matrix, without subjective dimension
  selection.
when_to_use_negative:
- Input is a similarity matrix whose dimensions do not match the random-matrix regime
  assumptions (e.g. extremely few cells relative to genes may destabilize the MP fit)
- The user has already fixed signal dimensions subjectively or uses a tool requiring
  manual threshold input — RMT filtering exists precisely to avoid such user discretion
- 'Data have not been L2-normalized after log normalization: log normalization alone
  on TGC-biased sparse matrices produces ~400 artificial signals that would be falsely
  passed downstream'
edam_operation: http://edamontology.org/operation_0306
edam_topics:
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_0091
tools:
- name: scLENS
  role: 'Julia package implementing RMT-based noise filtering: EVD of G = XX^T/N,
    MP distribution fit, Tracy-Widom thresholding, and MP-distribution plotting (plot_mpdist)'
  repo: https://github.com/Mathbiomed/scLENS
- name: scLENSpy
  role: Python environment (conda env sclens-py with PyTorch CUDA) providing setup
    for scLENS-based signal detection workflows
  repo: https://github.com/Mathbiomed/scLENSpy
provenance:
  source_task_ids:
  - task_001
  - task_002
  - task_006
  source_papers:
  - doi: 10.1038/s41467-024-47884-3
    title: 'scLENS: data-driven signal detection for unbiased scRNA-seq data analysis'
  source_content_hash: sha256:b648428abae965e6de9dbb233fe7b447b915b57c2f48e020517e8e7184ae7a35
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/Mathbiomed/scLENS
  tools:
  - CUDA
  - Julia
  - scLENS
  - scLENSpy
  edam_operation: http://edamontology.org/operation_0306
  edam_topics:
  - http://edamontology.org/topic_3308
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6363636363636364
n_claims: 11
---

# marchenko-pastur-distribution-fitting

## Summary

RMT-based noise filtering for single-cell similarity matrices: eigenvalues of G = XX^T/N that conform to the Marchenko-Pastur (MP) distribution are classified as noise, while eigenvalues deviating from MP and surpassing a Tracy-Widom (TW) threshold (0.05 probability cutoff) are retained as biological signals.

## When to use

Use when preprocessing scRNA-seq count data (log normalization with L=1, gene scaling, L2 normalization) and needing a data-driven signal/noise threshold from the eigenvalue spectrum of the cell similarity matrix, without subjective dimension selection.

## When NOT to use

- Input is a similarity matrix whose dimensions do not match the random-matrix regime assumptions (e.g. extremely few cells relative to genes may destabilize the MP fit)
- The user has already fixed signal dimensions subjectively or uses a tool requiring manual threshold input — RMT filtering exists precisely to avoid such user discretion
- Data have not been L2-normalized after log normalization: log normalization alone on TGC-biased sparse matrices produces ~400 artificial signals that would be falsely passed downstream

## Inputs

- Preprocessed scRNA-seq expression matrix X (cells x genes) after log normalization with scaling factor L=1, gene scaling, and L2 normalization (e.g. data/Z8eq.csv.gz loaded via scLENS.read_file)
- Raw count matrix (CSV or JLD2, rows = cells, columns = genes) for upstream QC (cells < 200 expressed genes and genes < 15 cells filtered)

## Outputs

- Complete eigenvalue spectrum of G = XX^T/N with eigenvalues exceeding the Tracy-Widom 0.05 cutoff flagged as signals (e.g. signal_eigenvalues.csv)
- Signal eigenvectors V_sig and the resulting low-dimensional embedding / PC score matrix
- MP distribution fit parameters and per-condition detected-signal counts (e.g. fig2_signal_detection_results.csv: ~400 artificial signals -> 0 after L2; ZhengMix: 84 -> 42 -> 13)

## How to apply

Build the cell-to-cell similarity matrix G = XX^T/N from the normalized data matrix X (N = number of genes) after log normalization (scaling factor L=1), gene scaling, and L2 normalization to unit cell-vector length — the L2 step is essential because log normalization alone inflates artificial signals. Compute all eigenvalues of G via eigenvalue decomposition (EVD). Fit the Marchenko-Pastur distribution to the eigenvalue spectrum following the Aparicio et al. procedure, derive the Tracy-Widom threshold as the critical point where an eigenvalue has 0.05 probability under the TW distribution of a random matrix, and flag eigenvalues above the threshold as signals (V_sig). Optionally verify that sqrt(eigenvalue) x eigenvector products equal the corresponding n-PC score matrix, since the RMT signal selection is mathematically equivalent to PCA scores.

## Related tools

- **scLENS** (Julia package implementing RMT-based noise filtering: EVD of G = XX^T/N, MP distribution fit, Tracy-Widom thresholding, and MP-distribution plotting (plot_mpdist)) — https://github.com/Mathbiomed/scLENS
- **scLENSpy** (Python environment (conda env sclens-py with PyTorch CUDA) providing setup for scLENS-based signal detection workflows) — https://github.com/Mathbiomed/scLENSpy

## Examples

```
sclens_embedding = scLENS.sclens(pre_df, device_=cur_dev); scLENS.plot_mpdist(sclens_embedding)
```

## Evaluation signals

- On the Fig. 2 synthetic TGC-biased sparse matrix (first 400 rows of the dense part halved), log normalization alone yields ~400 artificial signals, while adding L2 normalization yields 0 signals.
- On real data (data/Z8eq.csv.gz), signal count drops from 33 (log normalization only) to 6 (log + L2 normalization).
- On ZhengMix PBMC data, detected signals follow the cascade 84 (log) -> 42 (log + L2) -> 13 robust signals after the signal robustness test (robustness value > 0.5 over 10 perturbed datasets).
- PCA-equivalence check: multiplying the n signal eigenvalues' square roots with their eigenvectors must reproduce the n-PC score matrix with near-zero maximum deviation (recorded in equivalence_report.csv).

## Limitations

- The RMT approach requires computing all eigenvalues of the large cell-to-cell similarity matrix, demanding significant memory resources; scLENS replaces z-score gene scaling with median scaling and uses CUDA/GPU to reduce this burden, but large data with whole genes still requires high-specification hardware.
- If CUDA is not available, the package falls back to CPU, which may be slow for full EVD on large matrices.
- Eigenvalues passing the TW threshold are only 'potential biological signals' — low-quality signals arising from stochastic dropout events additionally require the signal robustness test (10 perturbed datasets, sparsity >0.97, robustness > 0.5).

## Evidence

- [abstract] RMT signal/noise criterion: "we utilize random matrix theory-based noise filtering and a signal robustness test to enable data-driven determination of the threshold for signal dimensions"
- [discussion] Data-driven threshold rationale: "provides a data-driven threshold to differentiate signals from noise by RMT-based noise filtering"
- [discussion] Preprocessing rationale for L2 before EVD: "scLENS reduces signal distortion during the preprocessing step by incorporating L2 normalization"
- [discussion] Scaling factor setting: "we set a scaling factor of 1 during the first step of log normalization"
- [readme] MP distribution plot function: "scLENS.plot_mpdist(sclens_embedding)"
