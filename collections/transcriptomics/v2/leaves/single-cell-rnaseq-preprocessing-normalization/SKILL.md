---
name: single-cell-rnaseq-preprocessing-normalization
description: Use when raw single-cell RNA-seq counts must be prepared as a log-normalized
  (genes x cells) matrix for full-dimensionality kernel methods (e.g. q-diffusion)
  that require no feature selection or dimensionality reduction.
when_to_use_negative:
- When the analysis pipeline requires feature selection, dimensionality reduction,
  or marker-gene identification — q-diffusion depends on the full gene library and
  those steps discard the interactions it captures
- When input is already a log-normalized expression matrix (e.g. an atlas reference's
  supplied dimensionality reduction such as scVI) — re-normalizing or re-reducing
  it is unnecessary
edam_operation: http://edamontology.org/operation_2935
edam_topics:
- http://edamontology.org/topic_3170
tools:
- name: q-Diffusion.jl
  role: Julia package supplying `preprocess_cell_counts(data)` for log-normalizing
    the (genes x cells) count matrix and `estimate_inner_bandwidth(data)` for downstream
    bandwidth estimation on the normalized matrix
  repo: github.com/marmarelis/QDiffusion.jl
- name: q-diffusion
  role: Framework for which the normalized full-dimensionality matrix is the direct
    input (kernel construction, NMF, qPHATE, LDS all consume it)
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1038/s42003-024-06104-w
    title: q-Diffusion leverages the full dimensionality of gene coexpression in single-cell
      transcriptomics
  source_content_hash: sha256:78d1db2a269ca66b97ebc0959dbd5b54180ed84c1432849c3b47a8624d067a2f
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - Monocle
  - PHATE
  - SC3
  - Scanpy
  - Seurat
  - q-Diffusion.jl
  - q-diffusion
  edam_operation: http://edamontology.org/operation_2935
  edam_topics:
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.75
n_claims: 8
---

# single-cell-rnaseq-preprocessing-normalization

## Summary

Log-normalize a raw (genes x cells) scRNAseq count matrix — without feature selection or dimensionality reduction — to prepare full-dimensionality inputs for q-diffused kernel construction, using QDiffusion.jl's `preprocess_cell_counts`.

## When to use

Use when raw single-cell RNA-seq counts must be prepared as a log-normalized (genes x cells) matrix for full-dimensionality kernel methods (e.g. q-diffusion) that require no feature selection or dimensionality reduction.

## When NOT to use

- When the analysis pipeline requires feature selection, dimensionality reduction, or marker-gene identification — q-diffusion depends on the full gene library and those steps discard the interactions it captures
- When input is already a log-normalized expression matrix (e.g. an atlas reference's supplied dimensionality reduction such as scVI) — re-normalizing or re-reducing it is unnecessary

## Inputs

- Raw single-cell RNA-seq count matrix, shaped (genes x cells) (e.g. the in vitro PBMC dataset, IFN-γ stimulated vs control cells)

## Outputs

- Log-normalized (genes x cells) expression matrix (the `.normalized` element of the named tuple returned by `preprocess_cell_counts`)
- Named tuple of alternative preprocessed matrix versions from common preprocessing pipelines (for reference)

## How to apply

Start from a raw `(genes x cells)` count matrix and apply log-normalization before passing it to q-diffusion methods. QDiffusion.jl provides `preprocess_cell_counts(data)`, a convenience method returning a named tuple with several matrix versions from common preprocessing pipelines; use the `*.normalized` element, which is simply log-normalized with no feature selection or dimensionality reduction, matching the authors' study protocol. The resulting normalized matrix is then fed directly to bandwidth estimation (`estimate_inner_bandwidth`) and kernel construction (e.g. AdaptiveKernel wrapping DeformedKernel). Preserving all genes is intentional: q-diffusion captures high-order gene coexpression interactions that aggressive feature selection, dimensionality reduction, or marker-gene identification would discard.

## Related tools

- **q-Diffusion.jl** (Julia package supplying `preprocess_cell_counts(data)` for log-normalizing the (genes x cells) count matrix and `estimate_inner_bandwidth(data)` for downstream bandwidth estimation on the normalized matrix) — github.com/marmarelis/QDiffusion.jl
- **q-diffusion** (Framework for which the normalized full-dimensionality matrix is the direct input (kernel construction, NMF, qPHATE, LDS all consume it))

## Examples

```
using QDiffusion; preprocessed = preprocess_cell_counts(data); normalized = preprocessed.normalized
```

## Evaluation signals

- Output matrix shape must remain (genes x cells) — the number of genes is unchanged, confirming no feature selection was applied
- `preprocess_cell_counts(data)` must return a named tuple whose `.normalized` element is log-transformed only (no dimensionality reduction)
- The same normalized matrix passed to `estimate_inner_bandwidth` should yield a finite inner bandwidth (median of per-gene average pairwise squared distances) suitable for DeformedKernel construction
- Downstream q-diffused kernel evaluation should be numerically stable; the README notes log-transformation (`in_log=true`) is necessary for numerical stability in kernel construction

## Limitations

- No batch-effect correction or alignment of disparate datasets was considered — normalization alone will not harmonize batches
- q-diffusion makes parametric assumptions on the data that may be restrictive in some use cases
- The normalized full matrix leads to suboptimal runtime: evaluating the q-diffused kernel across all pairs of cells takes 20 minutes to 3 hours for one affinity matrix

## Evidence

- [methods] convenience method log-normalization: "There is also a convenience mathod called `preprocess_cell_counts(data)` that returns a named tuple containing different versions of the matrix"
- [readme] normalized element definition: "The `*.normalized` element is simply log-normalized, sans feature selection or dimensionality reduction, like we do in our study."
- [intro] full dimensionality rationale: "toolkits in the state of the art rely on combinations of aggressive feature selection5,6, dimensionality reduction7–9, or marker-gene identification10"
- [methods] bandwidth estimation on normalized data: "inner_bandwidth = estimate_inner_bandwidth(data)"
