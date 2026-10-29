---
name: harmony-integration-baseline-execution
description: Use when benchmarking a new batch-integration method (e.g., BATMAN) against
  Harmony on two single-cell datasets projected into a shared PCA space, and you need
  mixing (iLISI) and local-structure (RNN) metrics plus runtimes.
when_to_use_negative:
- The current implementation of BATMAN works only with two datasets, and this comparison
  is designed for exactly two 10X PBMC datasets — do not use it as a baseline for
  multi-dataset integration tasks
- When comparing methods that operate in high-dimensional gene space, running Harmony
  in PCA space is not an apples-to-apples comparison unless both tools are given the
  same PCA embeddings
edam_operation: http://edamontology.org/operation_2935
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: Harmony
  role: PCA-space batch-integration baseline against which BATMAN's iLISI, 10-RNN,
    and runtime are compared
- name: BATMAN
  role: Method under evaluation; its results are contrasted with Harmony's on the
    same PCA embeddings
  repo: https://github.com/mandricigor/batman
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1016/j.isci.2020.101185
    title: 'BATMAN: Fast and Accurate Integration of Single-Cell RNA-Seq Datasets
      via Minimum-Weight Matching'
  source_content_hash: sha256:75206efaccfb7693f4128710b5f26db69823474b6fe2f22a6450aa19dbaad74b
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-ND-4.0
    url: https://github.com/mandricigor/batman
  repo_url: https://github.com/mandricigor/batman
  tools:
  - BATMAN
  - Harmony
  edam_operation: http://edamontology.org/operation_2935
  edam_topics:
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 1.0
n_claims: 11
---

# harmony-integration-baseline-execution

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Runs Harmony, a PCA-space scRNA-seq batch-integration method, as a baseline against BATMAN on the two 10X Genomics PBMC datasets (8k and Pan T cells), scoring integration with mean iLISI, 10-RNN, and wall-clock runtime.

## When to use

Use when benchmarking a new batch-integration method (e.g., BATMAN) against Harmony on two single-cell datasets projected into a shared PCA space, and you need mixing (iLISI) and local-structure (RNN) metrics plus runtimes.

## When NOT to use

- The current implementation of BATMAN works only with two datasets, and this comparison is designed for exactly two 10X PBMC datasets — do not use it as a baseline for multi-dataset integration tasks
- When comparing methods that operate in high-dimensional gene space, running Harmony in PCA space is not an apples-to-apples comparison unless both tools are given the same PCA embeddings

## Inputs

- 10X Genomics PBMC 8k count matrix (8,381 cells)
- 10X Genomics Pan T cells count matrix (3,555 T cells)
- Log-count matrices after Seurat V3.0 pre-processing
- Shared PCA embeddings of both datasets

## Outputs

- Harmony-corrected PCA embeddings of the two datasets
- Mean iLISI and 10-RNN scores for the Harmony integration
- Wall-clock runtime measurement
- Metric and runtime comparison rows (batman_harmony_pbmc_comparison.csv)

## How to apply

Preprocess the 10X PBMC 8k (8,381 cells) and Pan T cells (3,555 T cells) count matrices with Seurat V3.0-style pre-processing to obtain log-counts, then project both datasets into a shared PCA space so each cell is a point in N-dimensional space. Run Harmony on these PCA embeddings with default settings, since Harmony performs integration in PCA space rather than full gene space. Compute mean iLISI to quantify batch mixing and 10-RNN to quantify retention of local cell-to-cell structure, and record wall-clock runtime. In this article's comparison Harmony completed in about 14 s and showed slightly better iLISI and 10-RNN than BATMAN, so treat sub-15-s runtimes and slightly higher metrics as the expected baseline outcome.

## Related tools

- **Harmony** (PCA-space batch-integration baseline against which BATMAN's iLISI, 10-RNN, and runtime are compared)
- **BATMAN** (Method under evaluation; its results are contrasted with Harmony's on the same PCA embeddings) — https://github.com/mandricigor/batman

## Examples

```
python3.7 batman.py -data pbmc8k.csv,panTcells.csv -output integrated.csv; # then run Harmony on the same PCA embeddings and record iLISI, 10-RNN, and runtime
```

## Evaluation signals

- Harmony's mean iLISI on the 10X PBMC pair is slightly higher than BATMAN's, matching the paper's reported outcome
- Harmony's 10-RNN score is slightly higher than BATMAN's on the same PCA embeddings
- Wall-clock runtimes of the two tools are comparable (~14 s each)
- Both datasets were pre-processed identically with Seurat V3.0 steps before PCA projection
- Metric and runtime values are written to batman_harmony_pbmc_comparison.csv

## Limitations

- The article's runtime and metric comparison is specific to the two 10X Genomics PBMC datasets in PCA space; it does not generalize to other data modalities
- Harmony was used only as the PCA-space comparison baseline; the BATMAN paper's headline claim is that BATMAN outperforms state-of-the-art tools by up to 80% on batch-effect metrics elsewhere
- BATMAN's current implementation supports only two datasets, limiting the scope of this two-tool comparison

## Evidence

- [results] the "Pan T cells" dataset (3,555 T cells) from 10X Genomics' official website: [evidence span withheld — anchor status: not_found]
