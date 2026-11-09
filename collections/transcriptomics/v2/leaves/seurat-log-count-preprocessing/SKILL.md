---
name: seurat-log-count-preprocessing
description: Use when raw UMI count matrices from two scRNA-seq batches (e.g., Muraro
  CEL-Seq2 and Segerstolpe Smart-Seq2 pancreas data) must be normalized to log-counts
  before batch correction or before computing Euclidean distances between cells in
  gene space.
when_to_use_negative:
- The input matrices are already log-normalized by a different pipeline and cross-method
  comparability would be broken by re-normalizing only one arm
- The downstream method requires raw counts or its own internal normalization (e.g.,
  some deep-learning integrators), making Seurat log-counts an inappropriate common
  space
- Only one dataset is present — this skill exists to harmonize two batches prior to
  integration
edam_topics:
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_0203
tools:
- name: Seurat V3.0
  role: Performs the pre-processing steps that produce the log-counts consumed by
    all integration methods in the benchmark
- name: BATMAN
  role: Downstream consumer of the log-counts; builds its bipartite graph with Euclidean-distance-weighted
    edges in N-dimensional gene space over the normalized matrices
  repo: https://github.com/mandricigor/batman
- name: Scanorama
  role: Downstream integration method run on the same Seurat log-count data as a comparison
    integrator
- name: MNNcorrect
  role: Downstream integration method run on the same Seurat log-count data as a comparison
    integrator
provenance:
  source_task_ids:
  - task_002
  - task_003
  source_papers:
  - doi: 10.1016/j.isci.2020.101185
    title: 'BATMAN: Fast and Accurate Integration of Single-Cell RNA-Seq Datasets
      via Minimum-Weight Matching'
  source_content_hash: sha256:89be20a20a59c35ad12d58dfdbe98ddc3391a5f2adab17e3ddc54025d41959ab
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
  - MNNcorrect
  - Scanorama
  - Seurat
  - Seurat V3.0
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3308
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 1.0
n_claims: 9
---

# seurat-log-count-preprocessing

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Standardize each input scRNA-seq count matrix with Seurat V3.0 pre-processing to produce log-count matrices that serve as the common input space for BATMAN, Seurat V3.0, MNNcorrect, and Scanorama integration runs, ensuring all methods operate on identically normalized expression values.

## When to use

Use when raw UMI count matrices from two scRNA-seq batches (e.g., Muraro CEL-Seq2 and Segerstolpe Smart-Seq2 pancreas data) must be normalized to log-counts before batch correction or before computing Euclidean distances between cells in gene space.

## When NOT to use

- The input matrices are already log-normalized by a different pipeline and cross-method comparability would be broken by re-normalizing only one arm
- The downstream method requires raw counts or its own internal normalization (e.g., some deep-learning integrators), making Seurat log-counts an inappropriate common space
- Only one dataset is present — this skill exists to harmonize two batches prior to integration

## Inputs

- Raw scRNA-seq UMI count matrix per dataset (genes x cells), e.g., Muraro CEL-Seq2 (2,285 cells) and Segerstolpe Smart-Seq2 (2,394 cells) pancreas matrices
- Splatter-simulated count matrices (1,000 genes per simulated dataset, 100 datasets per scenario) when reproducing the LB simulation benchmark

## Outputs

- Log-count matrix per dataset (genes x cells) suitable for Euclidean-distance graph construction
- Uncorrected (Original) concatenation of the two log-count matrices used as the no-correction baseline

## How to apply

Load the raw count matrix for each dataset (e.g., Muraro CEL-Seq2, 2,285 cells; Segerstolpe Smart-Seq2, 2,394 cells) and apply Seurat V3.0 pre-processing steps to obtain log-counts for each dataset independently. This log-count representation is the fixed common input for the downstream comparison: BATMAN builds a weighted bipartite graph over cells with edges weighted by Euclidean distance in N-dimensional gene space, and MNN-based methods (Seurat V3.0, MNNcorrect, Scanorama) likewise consume the same log-counts. Keep the uncorrected concatenation of log-counts as the Original baseline for iLISI/cLISI and 50-RNN evaluation. Apply the identical pre-processing to every method arm so that differences in integration metrics reflect the integrators, not the normalization.

## Related tools

- **Seurat V3.0** (Performs the pre-processing steps that produce the log-counts consumed by all integration methods in the benchmark)
- **BATMAN** (Downstream consumer of the log-counts; builds its bipartite graph with Euclidean-distance-weighted edges in N-dimensional gene space over the normalized matrices) — https://github.com/mandricigor/batman
- **Scanorama** (Downstream integration method run on the same Seurat log-count data as a comparison integrator)
- **MNNcorrect** (Downstream integration method run on the same Seurat log-count data as a comparison integrator)

## Examples

```
seur <- NormalizeData(seur, normalization.method = "LogNormalize")  # Seurat V3.0 pre-processing to log-counts per dataset
```

## Evaluation signals

- Both datasets emerge with identical normalization provenance (log-counts) so the same preprocessed pair feeds BATMAN, Seurat V3.0, MNNcorrect, Scanorama, and the Original baseline
- Cell and gene dimensions are preserved: Muraro yields 2,285 cells and Segerstolpe yields 2,394 cells post-preprocessing with no cells silently dropped by normalization
- Downstream metric sanity: Original (no-correction) baseline should give mean iLISI 1.82 in the LB simulation and 1.07 on the pancreas all-cell-types task, confirming the preprocessing did not itself mix batches
- Integrated results remain comparable to reported values (e.g., BATMAN mean 50-RNN 0.92 vs Original 1.00 in LB), which requires consistent preprocessing across arms

## Limitations

- The paper does not enumerate the exact Seurat V3.0 pre-processing parameters (e.g., HVG selection or scale factors) beyond 'pre-processing steps to obtain log-counts', so exact parameter reproduction may require consulting the repository code
- BATMAN, the main downstream consumer of these log-counts, currently supports integration of only two datasets, so multi-dataset pipelines cannot directly reuse this two-batch preprocessing pattern

## Evidence
