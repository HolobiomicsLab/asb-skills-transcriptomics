---
name: macrophage-marker-gene-enrichment-scoring
description: Use when validating that a cell sketch (e.g., a 20,000-cell Sphetcher-H
  sketch of the 254,941-cell umbilical cord blood dataset) preserved a rare cell population
  by scoring a candidate cluster against known marker genes.
when_to_use_negative:
- When the goal is de novo discovery of cell types without prior marker genes — this
  skill validates recovery of a known rare population against specific markers
- When the input has not been sketched and clustered (e.g., raw full-dataset expression
  matrix without Louvain cluster labels) — cluster-level enrichment cannot be computed
edam_operation: http://edamontology.org/operation_3802
edam_topics:
- http://edamontology.org/topic_3170
tools:
- name: Sphetcher
  role: Produces the 20,000-cell spherical sketch (thresholding-based, Pearson correlation
    distance) whose indicator vector defines the subset of cells searched for the
    macrophage population
  repo: https://github.com/canzarlab/Sphetcher
- name: Louvain
  role: Community-detection clustering of the 20,000 sketched cells to yield the candidate
    macrophage cluster for marker enrichment scoring
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1016/j.isci.2020.101126
    title: 'Sphetcher: Spherical Thresholding Improves Sketching of Single-Cell Transcriptomic
      Heterogeneity'
  source_content_hash: sha256:cdf5639c5950f68dd9e9c0b075a83341fe9e0b6fb015892f61eca08aaff900ea
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/canzarlab/Sphetcher
  tools:
  - Louvain
  - Sphetcher
  edam_operation: http://edamontology.org/operation_3802
  edam_topics:
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9090909090909091
n_claims: 11
---

# macrophage-marker-gene-enrichment-scoring

## Summary

Identify and validate the rare inflammatory macrophage population recovered in a Sphetcher-H spherical sketch by quantifying enrichment of the marker genes CD74, HLA-DRA, B2M, and JUNB within the corresponding Louvain cluster and scoring recovery with marker-gene AUROC (expected > 0.88).

## When to use

Use when validating that a cell sketch (e.g., a 20,000-cell Sphetcher-H sketch of the 254,941-cell umbilical cord blood dataset) preserved a rare cell population by scoring a candidate cluster against known marker genes.

## When NOT to use

- When the goal is de novo discovery of cell types without prior marker genes — this skill validates recovery of a known rare population against specific markers
- When the input has not been sketched and clustered (e.g., raw full-dataset expression matrix without Louvain cluster labels) — cluster-level enrichment cannot be computed

## Inputs

- sketch_indicator_output.csv — binary indicator vector of length n (254,941) with exactly 20,000 ones marking sampled cells
- Umbilical cord blood expression matrix / PC matrix in CSV format (rows = cells, columns = features/PCs)
- Louvain cluster assignments for the 20,000 sketched cells
- Marker-gene set for inflammatory macrophages: CD74, HLA-DRA, B2M, JUNB

## Outputs

- macrophage_marker_auroc.csv — AUROC of the macrophage cluster against marker-gene labels (expected > 0.88)
- Identification of the cluster corresponding to the rare inflammatory macrophage population with quantified marker-gene enrichment

## How to apply

After running Sphetcher-H with sketch_size=20000 (Pearson correlation distance, thresholding-based spherical sketching) on the 254,941-cell umbilical cord blood expression/PC matrix, subset the original matrix to the 20,000 cells whose indicator value is 1 in sketch_indicator_output.csv. Cluster these sketched cells with Louvain community detection, the standard scRNA-seq method noted in the paper. Identify the cluster corresponding to inflammatory macrophages and quantify enrichment of the marker genes CD74, HLA-DRA, B2M, and JUNB within it. Compute the AUROC of the macrophage cluster against the marker-gene labels and write the result to macrophage_marker_auroc.csv; the paper reports AUROC > 0.88, which serves as the validation threshold.

## Related tools

- **Sphetcher** (Produces the 20,000-cell spherical sketch (thresholding-based, Pearson correlation distance) whose indicator vector defines the subset of cells searched for the macrophage population) — https://github.com/canzarlab/Sphetcher
- **Louvain** (Community-detection clustering of the 20,000 sketched cells to yield the candidate macrophage cluster for marker enrichment scoring)

## Examples

```
sphetcher umbilical_cord_blood_pca.csv 20000 sketch_indicator_output.csv
```

## Evaluation signals

- sketch_indicator_output.csv contains exactly 20,000 ones (sketch_size invariant)
- The recovered macrophage cluster reaches AUROC > 0.88 on the marker genes CD74, HLA-DRA, B2M, and JUNB, as reported in the paper
- Marker genes CD74, HLA-DRA, B2M, and JUNB are enriched within the identified cluster relative to other sketched cells
- Exactly one cluster in the Louvain partition corresponds to the rare inflammatory macrophage population
- AUROC result is written to macrophage_marker_auroc.csv

## Limitations

- Sphetcher's model was validated mainly with Louvain and Monocle 2 downstream; compatibility with other clustering algorithms was not tested
- The interplay of sketching distance metric with internal preprocessing of downstream algorithms was not systematically addressed in the study
- No software version numbers were stated for Sphetcher or downstream tools, so exact reproducibility of AUROC values may vary

## Evidence

- [intro] which in turn benefits downstream analyses such as clustering and allows to detect a rare population of inflammatory macrophages: "which in turn benefits downstream analyses such as clustering and allows to detect a rare population of inflammatory macrophages"
- [discussion] Louvain community detection and Monocle 2 are widely used for scRNA-seq clustering: "Louvain community detection and Monocle 2 are widely used for scRNA-seq clustering"
- [methods] sphetcher expression_matrix.csv sketch_size sketch_indicator_output.csv: "sphetcher expression_matrix.csv sketch_size sketch_indicator_output.csv"
- [readme] an indicator vector of n samples where 1 indicates the sample is in the sketch: "an indicator vector of `n` samples where 1 indicates the sample is in the sketch, 0 otherwise (there are `sketch_size` 1's in the vector)."
- [methods] First you need to provide a matrix where rows are samples (cells) and columns are features: "First you need to provide a matrix where rows are samples (cells) and columns are features (genes, transcripts, principal components PCs)."
