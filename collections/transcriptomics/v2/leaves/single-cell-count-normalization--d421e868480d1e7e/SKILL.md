---
name: single-cell-count-normalization
description: Use when raw single-cell RNA-seq count data (e.g., CELLxGENE census 2023-05-15
  extracts) must be converted to the scTab sf-log1p representation — 10,000-count
  size factors plus log1p over 19,331 protein-coding genes — before model training,
  augmentation-vector computation, or inference.
when_to_use_negative:
- Input is already size-factor normalized and log1p transformed (e.g., loading the
  merlin_cxg sf-log1p store directly) — reapplying the transform would distort the
  values.
- The analysis requires features outside the 19,331 protein-coding gene space (e.g.,
  non-coding RNAs or full-transcriptome coverage).
- Input is not raw per-cell count data (e.g., bulk RNA-seq or already TPM/CPM-scaled
  values), where per-cell size factors to 10,000 counts are not meaningful.
edam_topics:
- http://edamontology.org/topic_0203
tools:
- name: NumPy
  role: Provides the sf_normalize routine that scales each cell to 10,000 counts before
    the log1p transformation.
- name: scTab
  role: Defines the normalization convention and supplies the store-creation notebooks
    (01_download_data, 03_write_store_merlin) that produce the sf-log1p training data.
  repo: theislab/scTab
provenance:
  source_task_ids:
  - task_008
  - task_006
  source_papers:
  - doi: 10.1038/s41467-024-51059-5
    title: 'scTab: Scaling cross-tissue single-cell annotation models'
  source_content_hash: sha256:7c11934c67d6520ef23841d243c44cc43192bf7ae3562541762fbe488fef38b5
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/theislab/scTab
  tools:
  - NumPy
  - cellxgene-census
  - scTab
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5555555555555556
n_claims: 9
---

# single-cell-count-normalization

## Summary

Preprocesses raw scRNA-seq count matrices in the scTab convention: per-cell size-factor normalization to 10,000 counts via the NumPy sf_normalize routine followed by log1p transformation, restricted to a fixed 19,331 protein-coding gene space. This guarantees that TabNet training and inference operate on identically scaled tabular inputs.

## When to use

Use when raw single-cell RNA-seq count data (e.g., CELLxGENE census 2023-05-15 extracts) must be converted to the scTab sf-log1p representation — 10,000-count size factors plus log1p over 19,331 protein-coding genes — before model training, augmentation-vector computation, or inference.

## When NOT to use

- Input is already size-factor normalized and log1p transformed (e.g., loading the merlin_cxg sf-log1p store directly) — reapplying the transform would distort the values.
- The analysis requires features outside the 19,331 protein-coding gene space (e.g., non-coding RNAs or full-transcriptome coverage).
- Input is not raw per-cell count data (e.g., bulk RNA-seq or already TPM/CPM-scaled values), where per-cell size factors to 10,000 counts are not meaningful.

## Inputs

- raw cell-by-gene count matrix from a CELLxGENE census (CxG) query (census version 2023-05-15)
- gene feature list with the 19,331 protein-coding genes (notebooks/store_creation/features.parquet)

## Outputs

- size-factor normalized (10,000 counts per cell) and log1p-transformed cell-by-gene matrix in the 19,331-gene space
- on-disk Merlin-dataloader-compatible training store (e.g., merlin_cxg_2023_05_15_sf-log1p.tar.gz, 164GB full / 0.5GB minimal subset)

## How to apply

Scale every cell so its raw counts sum to exactly 10,000 using the NumPy sf_normalize routine published in the scTab methods, then apply np.log1p to the size-factor-normalized matrix. Restrict the gene axis to the 19,331 protein-coding genes enumerated in notebooks/store_creation/features.parquet so feature order matches the training store. Persist the result into the on-disk Nvidia Merlin dataloader format (e.g., the merlin_cxg_2023_05_15_sf-log1p store) via the 03_write_store_merlin.ipynb workflow. Apply the identical sf_normalize + log1p pipeline at inference time on holdout cells (donor-based split, test fraction 0.15), since the classifier was trained on exactly this representation and any train/inference preprocessing skew degrades evaluation. The rationale is that tabular models such as TabNet are sensitive to feature scaling, so preprocessing must be deterministic and identical across all pipeline stages.

## Related tools

- **NumPy** (Provides the sf_normalize routine that scales each cell to 10,000 counts before the log1p transformation.)
- **scTab** (Defines the normalization convention and supplies the store-creation notebooks (01_download_data, 03_write_store_merlin) that produce the sf-log1p training data.) — theislab/scTab

## Examples

```
X_log1p = np.log1p(sf_normalize(X_counts))
```

## Evaluation signals

- After sf_normalize (before log1p), every cell's counts sum to exactly 10,000.
- All stored matrix values are non-negative and non-integer (consistent with log1p of fractional size-factor-scaled counts).
- The gene dimension of the output equals 19,331 and the gene identifiers match the list in features.parquet.
- Cells preprocessed at inference time (e.g., the 0.15 test-fraction donor-holdout split) pass through the identical sf_normalize + log1p steps used at training, yielding distributions comparable to the training store.
- The produced store filename/manifest carries the sf-log1p marker (e.g., merlin_cxg_2023_05_15_sf-log1p), matching the released data package naming.

## Limitations

- This preprocessing does not resolve cross-protocol generalization problems: scTab trained on 10X-only data reaches macro F1 ~0.4 on non-10X protocols (STRT-seq, Smart-seq2, BD Rhapsody Targeted mRNA) versus ~0.8 on holdout 10X data.
- Restricting to 19,331 protein-coding genes discards non-protein-coding features present in the raw census data.
- The pipeline assumes primary raw counts (is_primary_data == True); applying it to already normalized matrices double-transforms the data and breaks comparability with the trained checkpoints.

## Evidence

- [methods] Normalize each cell to have `10000` counts (size factor normalization): "1. Normalize each cell to have `10000` counts (size factor normalization)"
- [methods] Gene space subset to `19331` protein coding genes (see `notebooks/store_creation/features.parquet` for full list): "* Gene space subset to `19331` protein coding genes (see `notebooks/store_creation/features.parquet` for full list)"
- [readme] Training data (compatible with Merlin Dataloader infrastructure): https://pklab.med.harvard.edu/felix/data/merlin_cxg_2023_05_15_sf-log1p.tar.gz (164GB): "Training data (compatible with Merlin Dataloader infrastructure): https://pklab.med.harvard.edu/felix/data/merlin_cxg_2023_05_15_sf-log1p.tar.gz (164GB)"
