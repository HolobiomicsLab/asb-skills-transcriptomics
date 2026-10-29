---
name: pancreas-scrnaseq-preprocessing
description: Use when raw pancreas scRNA-seq count matrices (Muraro CEL-Seq2, Segerstolpe
  Smart-Seq2) must be converted to Seurat V3.0 log-counts before integration with
  BATMAN, Scanorama, Seurat V3.0, or MNNcorrect, or when building the 1-held-out condition.
when_to_use_negative:
- The datasets are already in log-count form after Seurat V3.0 pre-processing and
  only integration or scoring is required
- More than two datasets must be integrated — BATMAN's current implementation works
  only with two datasets, so this two-dataset pre-processing setup does not extend
  directly
- The analysis targets a tissue other than the Muraro/Segerstolpe pancreas pair, since
  the alpha-cell exclusion fraction (37% of CEL-Seq2 cells) is dataset-specific
edam_operation: http://edamontology.org/operation_3223
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: Seurat
  role: Performs the V3.0 pre-processing steps that convert raw pancreas counts into
    log-counts for each dataset
- name: BATMAN
  role: Consumes the preprocessed log-count matrices; builds a bipartite graph with
    edges weighted by Euclidean distance and applies minimum-weight matching for batch
    correction
  repo: https://github.com/mandricigor/batman
- name: Scanorama
  role: Comparison integration method run on the same preprocessed dataset pair
- name: MNNcorrect
  role: MNN-based comparison integration method run on the same preprocessed dataset
    pair
provenance:
  source_task_ids:
  - task_004
  - task_008
  source_papers:
  - doi: 10.1016/j.isci.2020.101185
    title: 'BATMAN: Fast and Accurate Integration of Single-Cell RNA-Seq Datasets
      via Minimum-Weight Matching'
  source_content_hash: sha256:bdb4fc709c5194009e482148fcb63b9284bb9866da38bc90c808d378b18aa5fc
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
  - BERMUDA
  - MNNcorrect
  - Scanorama
  - Seurat
  edam_operation: http://edamontology.org/operation_3223
  edam_topics:
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.5714285714285714
n_claims: 14
---

# pancreas-scrnaseq-preprocessing

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Seurat V3.0 pre-processing of the Muraro CEL-Seq2 (2,285 cells) and Segerstolpe Smart-Seq2 (2,394 cells) human pancreas scRNA-seq datasets into log-count matrices, plus optional exclusion of alpha cells from CEL-Seq2 to set up the 1-held-out integration condition used in the BATMAN benchmark.

## When to use

Use when raw pancreas scRNA-seq count matrices (Muraro CEL-Seq2, Segerstolpe Smart-Seq2) must be converted to Seurat V3.0 log-counts before integration with BATMAN, Scanorama, Seurat V3.0, or MNNcorrect, or when building the 1-held-out condition.

## When NOT to use

- The datasets are already in log-count form after Seurat V3.0 pre-processing and only integration or scoring is required
- More than two datasets must be integrated — BATMAN's current implementation works only with two datasets, so this two-dataset pre-processing setup does not extend directly
- The analysis targets a tissue other than the Muraro/Segerstolpe pancreas pair, since the alpha-cell exclusion fraction (37% of CEL-Seq2 cells) is dataset-specific

## Inputs

- Raw UMI count matrix for the Muraro et al. CEL-Seq2 pancreas dataset (2,285 cells)
- Raw count matrix for the Segerstolpe et al. Smart-Seq2 pancreas dataset (2,394 cells)
- Cell-type annotations for each dataset (needed to identify alpha cells)

## Outputs

- Log-count matrices for each dataset (Seurat V3.0 pre-processing)
- Alpha-cell-excluded CEL-Seq2 log-count matrix for the 1-held-out condition
- Pre/post-integration expression matrices ready for k-RNN and iLISI/cLISI scoring (krnn_scores.csv, pancreas_1heldout_results.csv)

## How to apply

Load the raw count matrices for Muraro et al. CEL-Seq2 (2,285 cells) and Segerstolpe et al. Smart-Seq2 (2,394 cells) and apply the Seurat V3.0 pre-processing steps to obtain log-counts for each dataset. For the 1-held-out experiment, exclude the most abundant cell type — alpha cells, 37% of cells in the CEL-Seq2 dataset (42% in Smart-Seq2) — from the CEL-Seq2 matrix only, for maximum stringency. Feed the resulting log-count matrices directly into integration methods (BATMAN's minimum-weight matching on Euclidean-distance-weighted bipartite graphs, Scanorama, Seurat V3.0, MNNcorrect) and into the 50-RNN local-structure evaluation, which computes k=50 nearest neighbors in the high-dimensional gene space before and after correction. Correctness is judged by whether downstream metrics reproduce the reported pattern (e.g., mean 50-RNN of 0.92 for BATMAN versus 0.14 for Scanorama, and alpha cells left unmatched by all methods in the 1-held-out run).

## Related tools

- **Seurat** (Performs the V3.0 pre-processing steps that convert raw pancreas counts into log-counts for each dataset)
- **BATMAN** (Consumes the preprocessed log-count matrices; builds a bipartite graph with edges weighted by Euclidean distance and applies minimum-weight matching for batch correction) — https://github.com/mandricigor/batman
- **Scanorama** (Comparison integration method run on the same preprocessed dataset pair)
- **MNNcorrect** (MNN-based comparison integration method run on the same preprocessed dataset pair)

## Examples

```
library(Seurat); seu <- NormalizeData(CreateSeuratObject(counts)); logcounts <- GetAssayData(seu, assay = "RNA")
```

## Evaluation signals

- Cell counts after loading match the article: 2,285 cells (CEL-Seq2) and 2,394 cells (Smart-Seq2)
- Expression matrices are on the log-count scale produced by Seurat V3.0 pre-processing
- In the 1-held-out condition, alpha cells (37% of CEL-Seq2 cells) are removed and all integration methods leave Smart-Seq2 alpha cells unmatched
- Downstream 50-RNN values reproduce the reported pattern: Original 1.00, BATMAN 0.92, Seurat V3.0 0.58, MNNcorrect 0.51, Scanorama 0.14
- Mean iLISI in the 1-held-out experiment shows BATMAN best at 1.39 with MNNcorrect and Scanorama worst

## Limitations

- The 1-held-out exclusion is deliberately stringent: alpha cells are the most abundant type (37% of CEL-Seq2 cells, 42% of Smart-Seq2 cells), so removing them changes dataset composition substantially
- This pre-processing setup supports the two-dataset case only; BATMAN's current implementation works only with two datasets
- All methods fail to match alpha cells from the Smart-Seq2 dataset in the 1-held-out experiment, so downstream cell-type matching is incomplete for that type

## Evidence
