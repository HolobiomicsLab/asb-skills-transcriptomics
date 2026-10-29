---
name: scrna-count-normalization-log1p-transform
description: Use when raw single-cell RNA-seq count matrices must be converted to
  the sf-log1p representation expected by scTab/TabNet-style cell type classifiers
  trained on CELLxGENE census data.
when_to_use_negative:
- Input is already normalized or log-transformed (e.g. a preprocessed feature table)
  — re-normalizing would distort library-size information
- Analysis targets the 256-component PCA embeddings used for model training baselines
  rather than the full sf-log1p gene-space input
- Modality is not transcriptomic single-cell counts (scTab currently uses only transcriptomic
  input)
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: NumPy
  role: Implements the sf_normalize function that performs size-factor normalization
    to 10,000 counts and the log1p transform on the count matrix
- name: scTab
  role: Defines the sf-log1p preprocessing convention and applies it within the model
    input pipeline before feature transformer blocks
  repo: https://github.com/theislab/scTab
- name: cellxgene-census
  role: Source of the raw Cell-by-Gene (CxG) count data that is subset and then sf-log1p
    normalized
  repo: https://chanzuckerberg.github.io/cellxgene-census/python-api.html
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1038/s41467-024-51059-5
    title: 'scTab: Scaling cross-tissue single-cell annotation models'
  source_content_hash: sha256:990c798ef80467716f7ef4aac04cddd0f1d6aac78dcec32187406eaf408bce86
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/theislab/scTab
  tools:
  - NumPy
  - TabNet
  - cellxgene-census
  - scTab
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.4444444444444444
n_claims: 9
---

# scrna-count-normalization-log1p-transform

## Summary

Applies the scTab preprocessing scheme for single-cell RNA-seq counts: per-cell size-factor normalization to 10,000 counts followed by a log1p transform, producing the sf-log1p feature space that feeds the scTab TabNet classifier's input normalization stack.

## When to use

Use when raw single-cell RNA-seq count matrices must be converted to the sf-log1p representation expected by scTab/TabNet-style cell type classifiers trained on CELLxGENE census data.

## When NOT to use

- Input is already normalized or log-transformed (e.g. a preprocessed feature table) — re-normalizing would distort library-size information
- Analysis targets the 256-component PCA embeddings used for model training baselines rather than the full sf-log1p gene-space input
- Modality is not transcriptomic single-cell counts (scTab currently uses only transcriptomic input)

## Inputs

- Raw cell-by-gene UMI count matrix (single-cell RNA-seq), restricted to the 19,331 protein-coding gene space
- Cell counts from CELLxGENE census version 2023-05-15 in Cell-by-Gene (CxG) format

## Outputs

- Size-factor-normalized, log1p-transformed expression matrix (sf-log1p representation)
- On-disk Merlin-compatible data store named with the sf-log1p suffix (e.g. merlin_cxg_2023_05_15_sf-log1p)

## How to apply

Normalize each cell to a fixed total of 10,000 counts (size factor normalization) so expression values are comparable across cells with differing library sizes, then apply the log1p transformation to stabilize variance and compress the heavy-tailed count distribution. In the scTab implementation this is encapsulated in an `sf_normalize` function built on NumPy and applied before the model's batch-norm-based input normalization inside the forward pass. The result is the 'sf-log1p' on-disk feature store (e.g. merlin_cxg_2023_05_15_sf-log1p) consumed by the Nvidia Merlin dataloader. Do not apply additional scaling per gene beyond what the model's internal batch norm handles, and apply the identical transform to train, validation, and test splits.

## Related tools

- **NumPy** (Implements the sf_normalize function that performs size-factor normalization to 10,000 counts and the log1p transform on the count matrix)
- **scTab** (Defines the sf-log1p preprocessing convention and applies it within the model input pipeline before feature transformer blocks) — https://github.com/theislab/scTab
- **cellxgene-census** (Source of the raw Cell-by-Gene (CxG) count data that is subset and then sf-log1p normalized) — https://chanzuckerberg.github.io/cellxgene-census/python-api.html

## Examples

```
import numpy as np; X_norm = np.log1p(X / X.sum(axis=1, keepdims=True) * 10000)  # sf_normalize(X)
```

## Evaluation signals

- After size-factor normalization, each cell's total counts sum to exactly 10,000 (within floating-point tolerance)
- All transformed values are non-negative, and zeros in the raw counts remain exactly 0 after log1p
- Transformed matrix shape matches the input cell-by-gene dimensions over the 19,331 protein-coding gene space
- The sf-log1p values in the rebuilt store match the reference dataset merlin_cxg_2023_05_15_sf-log1p on a sampled batch
- Downstream scTab forward pass on the transformed batch yields finite class-probability outputs of the correct shape

## Limitations

- The 10,000-count target and log1p choice are tied to the scTab/TabNet pipeline trained on CELLxGENE 2023-05-15; other models may expect different normalization (e.g. 1e4 vs 1e6 CPM)
- Coarsely annotated or non-primary cells passing through the pipeline still inherit normalization but may carry label noise; filters like is_primary_data == True must be applied separately

## Evidence

- [methods] Normalize each cell to have `10000` counts (size factor normalization): "1. Normalize each cell to have `10000` counts (size factor normalization)"
- [methods] Log1p after size factor normalization: "2. After size factor normalization the data is Log1p transformed"
- [methods] sf_normalize NumPy implementation: "import numpy as np
      
   def sf_normalize(X):"
- [readme] sf-log1p training data naming: "https://pklab.med.harvard.edu/felix/data/merlin_cxg_2023_05_15_sf-log1p.tar.gz (164GB)"
