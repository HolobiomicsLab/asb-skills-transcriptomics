---
name: scrna-seq-minmax-cell-normalization
description: Use when a raw gene-by-cell scRNA-seq count matrix is about to be fed
  into scDHA's non-negative kernel autoencoder or any model that expects per-cell
  expression profiles scaled to [0, 1] with non-negative values.
when_to_use_negative:
- Data is already normalized or is a feature-level table (e.g. gene statistics, not
  a cell-by-gene matrix)
- Downstream method explicitly requires library-size normalization or z-scored values
  (e.g. tools like SC3/Scanpy pipelines that assume different preprocessing) rather
  than [0,1]-bounded, non-negative input
- The matrix contains negative values (e.g. imputed or log-difference data), where
  non-negativity required by the kernel autoencoder would be violated by construction
edam_operation: http://edamontology.org/operation_2939
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: scDHA
  role: Target framework whose non-negative kernel autoencoder consumes the min-max
    scaled cell profiles; installed from CRAN or GitHub, uses Torch (no Python environment
    required)
  repo: github.com/duct317/scDHA
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1038/s41467-021-21312-2
    title: Fast and precise single-cell data analysis using a hierarchical autoencoder
  source_content_hash: sha256:1266baad95cf3f63223fa29f6c15dfa8b73803da7d8b691daa8046c9dd0cd95a
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - scDHA
  edam_operation: http://edamontology.org/operation_2939
  edam_topics:
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.42857142857142855
n_claims: 7
---

# scrna-seq-minmax-cell-normalization

## Summary

Normalizes each cell's scRNA-seq expression profile by min-max scaling to the [0, 1] range (optionally after log2(data + 1) transformation), preparing the count matrix for scDHA's non-negative kernel autoencoder gene-filtering module, which assumes non-negative, bounded inputs to produce a part-based representation.

## When to use

Use when a raw gene-by-cell scRNA-seq count matrix is about to be fed into scDHA's non-negative kernel autoencoder or any model that expects per-cell expression profiles scaled to [0, 1] with non-negative values.

## When NOT to use

- Data is already normalized or is a feature-level table (e.g. gene statistics, not a cell-by-gene matrix)
- Downstream method explicitly requires library-size normalization or z-scored values (e.g. tools like SC3/Scanpy pipelines that assume different preprocessing) rather than [0,1]-bounded, non-negative input
- The matrix contains negative values (e.g. imputed or log-difference data), where non-negativity required by the kernel autoencoder would be violated by construction

## Inputs

- Raw scRNA-seq count matrix (genes x cells) loaded from CSV/TSV or an R data object (e.g. Goolam$data)
- Optional log-transformed expression matrix (log2(counts + 1))

## Outputs

- Per-cell min-max normalized expression matrix with all values in [0, 1]
- Normalized matrix formatted as samples x genes for direct input to scDHA (e.g. scdha_module1_representation.csv downstream)

## How to apply

Load the scRNA-seq count matrix (genes x cells; scDHA's own functions expect rows as samples and columns as genes, so transpose accordingly) and optionally apply a log2(x + 1) transformation as shown in the scDHA Goolam example to compress the count dynamic range. Then apply min-max scaling independently to each cell's expression profile so every value falls in [0, 1]. This bounded, non-negative scaling is required because the downstream one-layer autoencoder with a 50-node bottleneck clamps encoder weights to be non-negative to yield a part-based representation, from which genes with insignificant encoder-weight contributions are filtered (scDHA retains a top gene set, ~5000 in the source workflow). Decision point: cells where all genes are zero have an undefined range and must be handled (drop or leave as zeros) before dividing by (max - min).

## Related tools

- **scDHA** (Target framework whose non-negative kernel autoencoder consumes the min-max scaled cell profiles; installed from CRAN or GitHub, uses Torch (no Python environment required)) — github.com/duct317/scDHA

## Examples

```
data <- t(Goolam$data); data <- log2(data + 1); data <- apply(data, 1, function(x) (x - min(x)) / (max(x) - min(x))); result <- scDHA(data, seed = 1)
```

## Evaluation signals

- All values in the normalized matrix fall within [0, 1] (min = 0, max = 1 per cell)
- No negative values remain, satisfying the non-negativity requirement of the kernel autoencoder
- Matrix dimensions are preserved: one row/column per cell and one per gene (with the expected samples x genes orientation for scDHA input)
- Feeding the normalized matrix into the 50-node bottleneck autoencoder yields decreasing reconstruction loss over training and a 50-column compressed representation with one row per cell

## Limitations

- The scDHA README's worked example log-transforms data (log2(data + 1)) but the min-max scaling detail comes from the paper's workflow; practitioners must confirm the exact normalization order for their data
- Cells with zero total expression have an undefined min-max range and require special handling to avoid division by zero
- scDHA previously required a Python/TensorFlow environment; current versions use Torch called directly from R, so setup instructions differ across package versions

## Evidence

- [abstract] non-negative kernel autoencoder preprocessing: "The first module is a non-negative kernel autoencoder able to remove genes or components that have insignificant contributions to the part-based representation of the data."
- [intro] encoder-weight-based gene filtering: "Based on the weight distribution of the encoder, scDHA removes genes or components that have insignificant contributions to the representation"
- [readme] scDHA input orientation and log transform: [evidence span withheld — anchor status: external_doc]
- [readme] log2 transform example: "Log transform the data: `data <- log2(data + 1)`"
