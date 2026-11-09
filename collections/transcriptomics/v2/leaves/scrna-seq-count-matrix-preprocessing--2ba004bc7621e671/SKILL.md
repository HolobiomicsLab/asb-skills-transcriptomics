---
name: scrna-seq-count-matrix-preprocessing
description: Use when the input is a raw scRNA-seq count matrix (cell x gene) that
  must be read into a Seurat/Scanpy-compatible object before rare cell detection with
  GapClust or marker validation with Seurat FindMarkers.
when_to_use_negative:
- Input is already a processed feature/embedding table rather than a raw count matrix
  — GapClust expects expression profiles computed from the count data
- The goal is comprehensive characterisation of major cell types rather than rare-cell
  detection — toolkits such as Seurat or Scater are the article's recommended route
  for that task
edam_operation: http://edamontology.org/operation_2935
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: GapClust
  role: Consumes the preprocessed expression profiles to compute nearest-neighbour
    distances and the second-order derivative gap score for rare cluster detection
  repo: github.com/fabotao/GapClust
- name: Seurat
  role: Provides the R-side object format (Seurat >= 3.1.0 prerequisite) and FindMarkers-style
    differential expression to check marker enrichment (Dclk1, Trpm5) in detected
    rare clusters
- name: Scater
  role: Alternative toolkit mentioned in the article for comprehensively characterising
    major cell types on the same expression matrix
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1038/s41467-021-24489-8
    title: GapClust is a light-weight approach distinguishing rare cells from voluminous
      single cell expression profiles
  source_content_hash: sha256:e1a8844debe0235f34e2d808dd2123087e57c9787509f06b1b05641e13ad138d
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - GapClust
  - Scater
  - Seurat
  edam_operation: http://edamontology.org/operation_2935
  edam_topics:
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.4444444444444444
n_claims: 9
---

# scrna-seq-count-matrix-preprocessing

## Summary

Loads and prepares the raw scRNA-seq count matrix (e.g. the 4491 non-irradiated intestine crypt cells from Arshad et al.) into a pandas/Scanpy- or Seurat-compatible expression object so that rare-cell detection methods such as GapClust can operate on the expression profiles.

## When to use

Use when the input is a raw scRNA-seq count matrix (cell x gene) that must be read into a Seurat/Scanpy-compatible object before rare cell detection with GapClust or marker validation with Seurat FindMarkers.

## When NOT to use

- Input is already a processed feature/embedding table rather than a raw count matrix — GapClust expects expression profiles computed from the count data
- The goal is comprehensive characterisation of major cell types rather than rare-cell detection — toolkits such as Seurat or Scater are the article's recommended route for that task

## Inputs

- Raw scRNA-seq count matrix (cell x gene) of the 4491 non-irradiated intestine crypt cells from Arshad et al.
- Cell barcode and gene symbol identifiers embedded in the matrix headers

## Outputs

- Seurat/Scanpy-compatible expression object suitable as GapClust input
- Rare-cluster membership table (cell barcode, cluster ID, cluster size, marker status), e.g. gapclust_rare_cells.csv

## How to apply

Read the cell x gene count matrix into memory with a pandas/Scanpy-compatible IO path (for R workflows, Seurat >= 3.1.0 objects are the prerequisite format used by GapClust). Preserve cell barcodes and gene symbols as row/column identifiers so that downstream clusters (e.g. the 29-cell R2 tuft-cell cluster, Dclk1/Trpm5+) can be traced back to individual cells. Run GapClust's nearest-neighbour distance computation with a second-order derivative gap score, scanning candidate k and accepting candidates where skewness<k> > 2, on the expression profiles derived from this matrix. Finally, export the rare-cluster membership table (cell barcode, cluster ID, cluster size, marker status) so that marker enrichment can be validated with Seurat/FindMarkers-style differential expression.

## Related tools

- **GapClust** (Consumes the preprocessed expression profiles to compute nearest-neighbour distances and the second-order derivative gap score for rare cluster detection) — github.com/fabotao/GapClust
- **Seurat** (Provides the R-side object format (Seurat >= 3.1.0 prerequisite) and FindMarkers-style differential expression to check marker enrichment (Dclk1, Trpm5) in detected rare clusters)
- **Scater** (Alternative toolkit mentioned in the article for comprehensively characterising major cell types on the same expression matrix)

## Examples

```
import pandas as pd; X = pd.read_csv('intestine_counts.csv', index_col=0); print(X.shape)  # expect (4491, n_genes)
```

## Evaluation signals

- The loaded matrix contains 4491 non-irradiated intestine crypt cells with intact barcodes and gene symbols
- GapClust run on the preprocessed profiles detects rare clusters whose sizes match expectations (an R2 cluster of 29 cells)
- Differential expression on the preprocessed object shows R2 cells enriched for Dclk1 and Trpm5, consistent with Fig. 5a-c
- The membership table gapclust_rare_cells.csv is populated with cell barcode, cluster ID, cluster size, and marker status columns
- Candidate k values reported pass the skewness<k> > 2 acceptance criterion

## Limitations

- No parameter settings for GapClust runs (e.g. K, skewness threshold) are reported in the discussion section, so thresholds must be tuned or taken from the methods of the original workflow
- GapClust requires specific R package versions (Seurat >= 3.1.0, rflann >= 1.8.4, irlba >= 2.3.3, e1071 >= 1.7-0.1) that must be installed beforehand
- To discover additional hidden clusters, users must rerun GapClust with previously identified rare cell types removed

## Evidence

- [abstract] GapClust, a light-weight algorithm to detect rare cell types from ultra-large scRNA-seq datasets with state-of-the-art speed and memory efficiency: "Here, we develop GapClust, a light-weight algorithm to detect rare cell types from ultra-large scRNA-seq datasets"
- [intro] Light-weight tool balancing accuracy and efficiency: "we have developed a light-weight tool, GapClust, which achieves a balance between accuracy and efficiency when searching for the needle (rare cells) in the haystack"
- [readme] R installation prerequisites: "R users  can easily install GapClust by running following code in R console."
