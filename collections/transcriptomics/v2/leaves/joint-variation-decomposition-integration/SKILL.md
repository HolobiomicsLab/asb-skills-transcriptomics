---
name: joint-variation-decomposition-integration
description: Use when integrating multi-batch or multi-source scRNA-seq data (e.g.,
  Zilionis mouse lung two-batch data) where you need an unsupervised decomposition
  separating shared biological signal from batch effects, and when benchmarking against
  Seurat v5/Harmony.
when_to_use_negative:
- Input is a single-source, single-batch scRNA-seq dataset with no batch structure
  to correct — JIVE's joint/individual decomposition has nothing to separate.
- The original JIVE implementation (r.jive for bulk data) is being applied to large-scale
  single-cell datasets — the paper states this is computationally infeasible; use
  the scJIVE implementation instead.
- Batch sizes are strongly unbalanced — the paper found JIVE performed best in scenarios
  with balanced batch sizes.
edam_operation: http://edamontology.org/operation_3935
edam_topics:
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_3308
tools:
- name: JIVE (scJIVE)
  role: 'Primary decomposition method: `jive` function estimates the joint (shared
    biological) and individual (batch-effect) low-rank structures; scJIVE is the enhanced
    single-cell implementation'
  repo: https://github.com/oconnell-statistics-lab/scJIVE
- name: Seurat v5
  role: Benchmarked alternative integration method compared against JIVE using kBET,
    ASW, and LISI
- name: Harmony
  role: Benchmarked alternative integration method compared against JIVE using kBET,
    ASW, and LISI
- name: r.jive
  role: Original bulk-data R implementation illustrating JIVE usage (`jive`, `showPCA`,
    `jive.predict`, `bic.jive`, `jive.perm`); superseded by scJIVE for single-cell
    scale
- name: R
  role: Analysis language; scJIVE installed via devtools from GitHub
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1093/bioadv/vbae134
    title: Batch-effect correction in single-cell RNA sequencing data using JIVE
  source_content_hash: sha256:3cc899f8cc6fc8eafc2a13583b93ecaed9903af67b40ea8ff64f931f15cb5fd2
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/oconnell-statistics-lab/scJIVE
  tools:
  - Harmony
  - JIVE (scJIVE)
  - R
  - Seurat v5
  - r.jive
  edam_operation: http://edamontology.org/operation_3935
  edam_topics:
  - http://edamontology.org/topic_0203
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8181818181818182
n_claims: 11
---

# joint-variation-decomposition-integration

## Summary

Batch-correct multi-source single-cell RNA-seq data by decomposing it with JIVE (scJIVE) into a joint structure capturing shared biological variability and individual structures capturing per-batch technical variability, then benchmark the integrated embedding against alternatives (Seurat v5, Harmony) with kBET, ASW, and LISI metrics.

## When to use

Use when integrating multi-batch or multi-source scRNA-seq data (e.g., Zilionis mouse lung two-batch data) where you need an unsupervised decomposition separating shared biological signal from batch effects, and when benchmarking against Seurat v5/Harmony.

## When NOT to use

- Input is a single-source, single-batch scRNA-seq dataset with no batch structure to correct — JIVE's joint/individual decomposition has nothing to separate.
- The original JIVE implementation (r.jive for bulk data) is being applied to large-scale single-cell datasets — the paper states this is computationally infeasible; use the scJIVE implementation instead.
- Batch sizes are strongly unbalanced — the paper found JIVE performed best in scenarios with balanced batch sizes.

## Inputs

- Multi-source / multi-batch scRNA-seq expression matrix (genes x cells), e.g. Zilionis mouse lung two-batch data subset to B-cell and T-cell clusters
- Batch labels per cell and cell-type annotations per cell
- Top variable gene subset (e.g., top 2000 HVGs)

## Outputs

- JIVE decomposition: joint low-dimensional representation (batch-corrected embedding) plus individual per-batch structures
- t-SNE embeddings for raw and each integrated representation, colored by batch and cell type
- CSV benchmarking table with kBET acceptance rates, ASW, and cell-type/batch LISI per method
- Benchmarking figure (Figure 5-style PDF)

## How to apply

Load the multi-batch scRNA-seq matrix (e.g., Zilionis mouse lung subset to B-cell and T-cell clusters, two batches), select the top ~2000 highly variable genes, and organize data as a multi-source expression matrix with batch labels and cell-type annotations. Run the scJIVE decomposition via the `jive` function (optionally selecting joint/individual ranks with the permutation-based `jive.perm` or BIC-motivated `bic.jive` approaches, with row-orthogonality enforced between joint and individual estimates), and extract the joint low-dimensional representation as the integrated embedding. In parallel, run Seurat v5 and Harmony integrations plus raw uncorrected data as a baseline. Compute t-SNE embeddings colored by batch and cell type, then quantify batch mixing and cell-type preservation with kBET acceptance rates (bounded 0–1), ASW, and batch/cell-type LISI, assembling metric values into a CSV comparison table. Interpretation follows the paper's finding: JIVE preserves cell-type effects best with balanced batch sizes, but can fail cell-type preservation on some datasets, so compare LISI cell-type values against the raw baseline before adopting it.

## Related tools

- **JIVE (scJIVE)** (Primary decomposition method: `jive` function estimates the joint (shared biological) and individual (batch-effect) low-rank structures; scJIVE is the enhanced single-cell implementation) — https://github.com/oconnell-statistics-lab/scJIVE
- **Seurat v5** (Benchmarked alternative integration method compared against JIVE using kBET, ASW, and LISI)
- **Harmony** (Benchmarked alternative integration method compared against JIVE using kBET, ASW, and LISI)
- **r.jive** (Original bulk-data R implementation illustrating JIVE usage (`jive`, `showPCA`, `jive.predict`, `bic.jive`, `jive.perm`); superseded by scJIVE for single-cell scale)
- **R** (Analysis language; scJIVE installed via devtools from GitHub)

## Examples

```
devtools::install_github("oconnell-statistics-lab/scJIVE"); res <- jive(data, rank.est = TRUE, rank.search = TRUE); joint_embedding <- res$joint
```

## Evaluation signals

- kBET acceptance rates for each method's embedding fall within 0–1 and the CSV contains kBET, ASW, and LISI fields
- The benchmarking figure file (Figure 5-style PDF) exists and t-SNE panels render for raw, JIVE, Seurat v5, and Harmony embeddings colored by both batch and cell type
- Batch-mixing LISI improves over the raw uncorrected baseline for JIVE (paper: JIVE and Harmony were best performers on batch-mixing LISI)
- Cell-type LISI is checked against the raw baseline to detect methods that over-mix (paper: JIVE and Seurat v5 had considerably worse LISI cell-type values than raw on Zilionis mouse lung)
- Rank selection (e.g., estimated joint rank such as 2 on BRCA data, with per-source individual ranks) is reported via permutation-based (`jive.perm`) or BIC-motivated (`bic.jive`) approaches

## Limitations

- JIVE and Seurat v5 failed to preserve cell type distinctions on the Zilionis mouse lung data, with considerably worse cell-type LISI values than the raw data — over-correction is a real failure mode
- The original JIVE implementation was designed for bulk sequencing data and is computationally infeasible for large-scale single-cell datasets; only the scJIVE enhanced implementation scales
- JIVE performed best when batch sizes are balanced; performance degrades in unbalanced scenarios

## Evidence

- [abstract] JIVE can be used to extract shared biological patterns from multi-source sequencing data while adjusting for individual non-biological variations (i.e., batch effect).: "scJIVE can be used to extract shared biological patterns from multi-source sequencing data while adjusting for individual non-biological variations (i.e., batch effect)."
- [abstract] JIVE performed best in terms of preserving cell-type effects and in scenarios in which the batch sizes are balanced.: "JIVE performed best in terms of preserving cell-type effects and in scenarios in which the batch sizes are balanced."
- [methods] The default function to estimate the JIVE decomposition is `jive`: "The default function to estimate the JIVE decomposition is `jive`"
- [methods] One is a permutation-based approach described in @Lock, the other is a BIC-motivated approach.: "One is a permutation-based approach described in @Lock, the other is a BIC-motivated approach."
- [results] On the Zilionis mouse lung data, JIVE and Seurat v5 failed to preserve cell type distinctions, with both methods having considerably worse LISI cell type values than the raw data: [evidence span withheld — anchor status: not_found]
- [abstract] its current implementation is originally designed for bulk sequencing data, making it computationally infeasible for large-scale single-cell sequencing datasets: "However, its current implementation is originally designed for bulk sequencing data, making it computationally infeasible for large-scale single-cell sequencing datasets."
- [readme] You can install the development version of scJIVE from GitHub with: devtools::install_github("oconnell-statistics-lab/scJIVE"): "You can install the development version of scJIVE from GitHub with:

``` r
# install.packages("devtools")
devtools::install_github("oconnell-statistics-lab/scJIVE")
```"
