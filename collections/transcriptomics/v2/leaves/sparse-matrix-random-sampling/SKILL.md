---
name: sparse-matrix-random-sampling
description: 'Use when benchmarking imputation methods like LATE/TRANSLATE: you hold
  a ground-truth matrix (e.g., example.hd5, 10,000 genes x 10,000 cells) and need
  a masked input with a controlled 10% nonzero rate.'
when_to_use_negative:
- The dataset has no ground truth (as in the mouse brain example_mouse_brain_10x.h5
  case, where the ground truth is set to the input itself), so masking would destroy
  real signal with no way to score recovery
- The goal is to distinguish biological zeros from technical zeros — this masking
  assumes injected zeros are technical and LATE/TRANSLATE impute all zeros as missing
  values
- The input is already the masked benchmark file (e.g., example.msk90.hd5) supplied
  by the authors' data deposit
edam_operation: http://edamontology.org/operation_3802
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: numpy
  role: Random number generator used to sample 90% of matrix entries without replacement
    and set them to zero
- name: pandas
  role: Reads and writes the hd5 ground-truth and masked matrices (via tables)
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1007/s40484-019-0192-7
    title: Imputation of single‐cell gene expression with an autoencoder neural network
  source_content_hash: sha256:fec80c3b2121c44aaf2c5e39566ec5a60db587ab70bfdec11933e527b54cd6fc
schema_version: 0.2.0
metadata:
  license_tier: unknown
  tools:
  - Python+
  - numpy
  - pandas
  edam_operation: http://edamontology.org/operation_3802
  edam_topics:
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.4444444444444444
n_claims: 9
---

# sparse-matrix-random-sampling

## Summary

Generates a synthetic sparse scRNA-seq benchmark input by randomly masking 90% of the entries of a ground-truth read-count matrix (e.g., example.hd5) to zero, creating excess 'technical zeros' whose recovery can be scored against the retained ground truth.

## When to use

Use when benchmarking imputation methods like LATE/TRANSLATE: you hold a ground-truth matrix (e.g., example.hd5, 10,000 genes x 10,000 cells) and need a masked input with a controlled 10% nonzero rate.

## When NOT to use

- The dataset has no ground truth (as in the mouse brain example_mouse_brain_10x.h5 case, where the ground truth is set to the input itself), so masking would destroy real signal with no way to score recovery
- The goal is to distinguish biological zeros from technical zeros — this masking assumes injected zeros are technical and LATE/TRANSLATE impute all zeros as missing values
- The input is already the masked benchmark file (e.g., example.msk90.hd5) supplied by the authors' data deposit

## Inputs

- Ground-truth read-count matrix in hd5 format (e.g., example.hd5: 10,000 genes and 10,000 cells from 10x Genomics human PBMCs)
- Masking rate (e.g., 90% of entries to be zeroed)

## Outputs

- Masked sparse input matrix in hd5 (10x Genomics sparse matrix layout), e.g., example.msk90.hd5, with ~90% zero entries and the same row/column identifiers as the input
- Unmodified copy of the original ground-truth matrix retained for downstream imputation evaluation (gtMSE, gtMSEtech)

## How to apply

Load the ground-truth read-count matrix (e.g., the 10x Genomics human PBMC example.hd5 with 10,000 genes and 10,000 cells) using pandas/tables, keeping an unmodified copy as the evaluation ground truth. Use numpy's random number generator to sample 90% of matrix entries without replacement and set them to zero, producing an input with roughly a 10% nonzero rate whose injected zeros are 'technical zeros' quantified by gtMSEtech. Save the masked matrix in hd5 (10x Genomics sparse matrix layout) with the same row and column identifiers as the original — the article's example output is example.msk90.hd5, referred to as PBMC_G949_10K. Rationale: the sampling distribution should be simple and introduce minimal additional uncertainty since the true dropout process is unknown.

## Related tools

- **numpy** (Random number generator used to sample 90% of matrix entries without replacement and set them to zero)
- **pandas** (Reads and writes the hd5 ground-truth and masked matrices (via tables))

## Examples

```
import numpy as np; mask = np.random.choice(X.size, int(0.9 * X.size), replace=False); X.flat[mask] = 0  # mask 90% of ground-truth entries
```

## Evaluation signals

- Masked output file exists and has the same dimensions as the input matrix (e.g., 10,000 genes x 10,000 cells)
- Masked matrix contains approximately 90% zero entries (10% nonzero rate)
- Row and column identifiers match those of the original example.hd5
- An unmodified copy of the original matrix is retained to serve as ground truth for gtMSE/gtMSEtech evaluation
- Masked file is in hd5 (10x Genomics sparse matrix layout) format, loadable by the LATE imputation scripts

## Limitations

- The true dropout process is unknown, so the choice of sampling distribution in simulation may introduce additional uncertainty in performance assessment
- The masking only simulates technical zeros; real scRNA-seq sparsity mixes biological and technical zeros, and LATE/TRANSLATE always impute zeros with a nonzero value
- Hyperparameters and the simulation setup were manually configured rather than systematically tuned

## Evidence
