---
name: tpm-log2-count-transformation
description: Use when raw UMI count matrices (e.g., 10x Genomics h5 count data) must
  be converted into a stable, log-scaled input for training the VASC variational autoencoder
  or similar dropout-aware deep models.
when_to_use_negative:
- Input data is already normalized and log-transformed (e.g., a log-TPM or log-CPM
  matrix) — re-applying the transformation would distort the distribution
- The downstream method expects raw integer UMI counts rather than continuous normalized
  values
edam_operation: http://edamontology.org/operation_3223
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_3308
tools:
- name: h5py
  role: Loads the raw UMI count matrix from the deposited 10x Genomics data within
    the Anaconda environment
  repo: https://github.com/wang-research/VASC
- name: numpy
  role: Performs the UMI-to-TPM-like normalization and log2 transformation of the
    expression matrix
- name: VASC
  role: Consumes the log2-transformed matrix for training the deep variational autoencoder
    (class VASC in vasc.py)
  repo: https://github.com/wang-research/VASC
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1016/j.gpb.2018.08.003
    title: 'VASC: Dimension Reduction and Visualization of Single-Cell RNA-Seq Data
      by Deep Variational Autoencoder'
  source_content_hash: sha256:f0c403a3788a2a6a2295319597be62bdba7dbef495eab734bfdad75be66418dd
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/wang-research/VASC
  tools:
  - Anaconda
  - Python
  - VASC
  - h5py
  - keras
  - numpy
  - sklearn
  - tensorflow
  edam_operation: http://edamontology.org/operation_3223
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.4166666666666667
n_claims: 12
---

# TPM-like normalization and log2 transformation of UMI counts for VASC

## Summary

Preprocesses raw scRNA-seq UMI count matrices (e.g., 2700 cells x 1158 genes from 10x Genomics PBMC3k) by a UMI-to-TPM-like per-cell normalization followed by log2 transformation, stabilizing the dropout-dominated expression distribution before VASC autoencoder training.

## When to use

Use when raw UMI count matrices (e.g., 10x Genomics h5 count data) must be converted into a stable, log-scaled input for training the VASC variational autoencoder or similar dropout-aware deep models.

## When NOT to use

- Input data is already normalized and log-transformed (e.g., a log-TPM or log-CPM matrix) — re-applying the transformation would distort the distribution
- The downstream method expects raw integer UMI counts rather than continuous normalized values

## Inputs

- Raw scRNA-seq UMI count matrix (e.g., 10x Genomics PBMC3k: 2700 cells x 1158 selected genes, loaded from deposited 10x data via h5py)

## Outputs

- Log2-transformed, TPM-like normalized expression matrix suitable as VASC autoencoder input
- Downstream: 2D latent embedding of all cells from the trained VASC encoder for scatter visualization

## How to apply

Load the raw UMI count matrix (e.g., PBMC3k: 2700 cells x 1158 selected genes) from the deposited 10x data using h5py. Apply a UMI-to-TPM-like normalization that converts raw UMI counts into TPM-like expression values per cell, so cells become comparable despite differences in sequencing depth. Then log2-transform the normalized matrix to stabilize the dropout-dominated distribution prior to model training — this is the input format the VASC class (vasc.py, Gumbel-distribution modeling of zero and near-zero dropout events) expects when run via demo.py/config.py with Keras and TensorFlow. Train the VASC model on the log2-transformed matrix and extract the low-dimensional (2D) latent embedding for visualization.

## Related tools

- **h5py** (Loads the raw UMI count matrix from the deposited 10x Genomics data within the Anaconda environment) — https://github.com/wang-research/VASC
- **numpy** (Performs the UMI-to-TPM-like normalization and log2 transformation of the expression matrix)
- **VASC** (Consumes the log2-transformed matrix for training the deep variational autoencoder (class VASC in vasc.py)) — https://github.com/wang-research/VASC

## Examples

```
import h5py, numpy as np; umi = np.array(h5py.File('pbmc3k.h5','r')['matrix']).sum(axis=0); tpm = umi / umi.sum(axis=1, keepdims=True) * 1e6; log2_tpm = np.log2(tpm + 1)
```

## Evaluation signals

- The normalized matrix contains continuous TPM-like (non-integer) values per cell rather than raw integer UMI counts
- The transformed matrix is strictly non-negative after log2 transform (values >= 0, zeros retained as zeros)
- VASC training on the transformed matrix converges and yields a 2D embedding that separates major cell types (e.g., B cells, CD4+ T cells, CD8+ T cells, NK cells on PBMC3k)
- The resulting 2D visualization resolves sub-populations within cell types (e.g., FCGR3A+ vs CD14+ monocytes) when re-run on the monocyte subset, consistent with Figure 6

## Limitations

- VASC model learning on the transformed data takes several hours on a single-GPU desktop for datasets with several thousand cells
- Stochastic optimization in VASC introduces run-to-run variation in the dimension reduction; repeated runs are recommended for consensus
- Determining the optimal latent dimension is difficult without prior knowledge; for small sample sizes it is better to fix the co-variance matrix

## Evidence

- [other] UMI-to-TPM-like normalization and log2 transformation: [evidence span withheld — anchor status: not_found]
- [other] Log2-transform prior to model training: [evidence span withheld — anchor status: not_found]
- [intro] Gumbel dropout modeling motivates the transform: "In addition, it uses the Gumbel distribution to better model the zero and near-zero dropout events."
- [readme] VASC python environment prerequisites: "+ Python 3.5+ + numpy 1.12.1 + h5py 2.7.0 + sklearn 0.18.1 + tensorflow 1.1.0 + keras 2.0.6"
