---
name: louvain-community-clustering
description: Use when a scRNA-seq matrix (cells × features/PCs) has been reduced to
  a Sphetcher spherical sketch (indicator vector with sketch_size ones) and the sketched
  cells must be clustered into cell types.
when_to_use_negative:
- The dataset is small enough to cluster in full — sketching plus Louvain is meant
  to bridge scalability gaps for datasets whose enormous size exceeds the capability
  of current methods
- The downstream algorithm makes assumptions incompatible with Sphetcher's underlying
  model — the paper notes compatibility was tested only with Louvain and Monocle 2
- You need trajectory inference rather than discrete cell-type clustering — Monocle
  2 is the paired method for inferring single-cell trajectories
edam_operation: http://edamontology.org/operation_0362
edam_topics:
- http://edamontology.org/topic_3512
- http://edamontology.org/topic_0203
tools:
- name: Louvain
  role: Community-detection clustering applied to the Sphetcher-sketched cells to
    recover cell-type populations
- name: Sphetcher
  role: Produces the sketch_indicator_output.csv that defines which cells are clustered
    by Louvain
  repo: https://github.com/canzarlab/Sphetcher
- name: SC3
  role: Computationally expensive consensus-clustering alternative that may become
    practical on a spherical sketch but is avoided for large data
provenance:
  source_task_ids:
  - task_005
  - task_006
  source_papers:
  - doi: 10.1016/j.isci.2020.101126
    title: 'Sphetcher: Spherical Thresholding Improves Sketching of Single-Cell Transcriptomic
      Heterogeneity'
  source_content_hash: sha256:d12c09903cfb0776ed54f8a81005dd385e0567ab2b3b5e2ecd3365544978b44a
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/canzarlab/Sphetcher
  tools:
  - Louvain
  - SC3
  - Sphetcher
  - git
  edam_operation: http://edamontology.org/operation_0362
  edam_topics:
  - http://edamontology.org/topic_3512
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6363636363636364
n_claims: 11
---

# louvain-community-clustering

## Summary

Downstream scRNA-seq clustering of Sphetcher spherical sketches: subset the original expression matrix to the sketch_indicator_output.csv cells and partition them into cell-type clusters with Louvain community detection, scoring recovery via ARI against reference labels or marker-gene AUROC.

## When to use

Use when a scRNA-seq matrix (cells × features/PCs) has been reduced to a Sphetcher spherical sketch (indicator vector with sketch_size ones) and the sketched cells must be clustered into cell types.

## When NOT to use

- The dataset is small enough to cluster in full — sketching plus Louvain is meant to bridge scalability gaps for datasets whose enormous size exceeds the capability of current methods
- The downstream algorithm makes assumptions incompatible with Sphetcher's underlying model — the paper notes compatibility was tested only with Louvain and Monocle 2
- You need trajectory inference rather than discrete cell-type clustering — Monocle 2 is the paired method for inferring single-cell trajectories

## Inputs

- expression_matrix.csv — scRNA-seq expression matrix in CSV format, rows are cells, columns are features (genes, transcripts, PCs)
- sketch_indicator_output.csv — binary indicator vector from Sphetcher with exactly sketch_size 1's marking the sketched cells
- reference cell labels (ground-truth cell types) for ARI computation
- sketch_size — number of cells to sample (e.g., 20000)

## Outputs

- Cluster assignments (community labels) for each sketched cell
- ari_scores.csv — Adjusted Rand Index table across datasets and distance metrics (Euclidean, Manhattan, cosine, Pearson)
- macrophage_marker_auroc.csv — AUROC of the rare inflammatory macrophage cluster against marker genes CD74, HLA-DRA, B2M, JUNB

## How to apply

Run Sphetcher to obtain sketch_indicator_output.csv, verifying it contains exactly sketch_size 1's (e.g., 20,000 ones for the 254,941-cell umbilical cord blood dataset), then subset the original CSV matrix (rows = cells, columns = features/PCs) to cells with indicator = 1. Build a similarity graph over the sketched cells and run Louvain community detection, a standard scRNA-seq clustering method widely used alongside Monocle 2. Judge success by comparing clusterings against reference cell labels with the Adjusted Rand Index (tabulated as ari_scores.csv) — on dataset chen, sketches computed with Euclidean distance yield the highest ARI relative to Manhattan, cosine, and Pearson metrics. For rare-population recovery, identify the Louvain cluster enriched for the marker genes CD74, HLA-DRA, B2M, and JUNB and require AUROC > 0.88. Louvain is preferred over computationally expensive alternatives such as SC3 consensus clustering, although SC3 'might become practical again on a spherical sketch'.

## Related tools

- **Louvain** (Community-detection clustering applied to the Sphetcher-sketched cells to recover cell-type populations)
- **Sphetcher** (Produces the sketch_indicator_output.csv that defines which cells are clustered by Louvain) — https://github.com/canzarlab/Sphetcher
- **SC3** (Computationally expensive consensus-clustering alternative that may become practical on a spherical sketch but is avoided for large data)

## Evaluation signals

- sketch_indicator_output.csv contains exactly sketch_size 1's (e.g., exactly 20,000 ones for the umbilical cord blood run)
- ARI of the Louvain clustering against reference cell labels is computed and tabulated; Euclidean-metric spherical sketches give the highest ARI on dataset chen
- Recovered rare inflammatory macrophage cluster reaches AUROC > 0.88 on the marker genes CD74, HLA-DRA, B2M, and JUNB
- Marker genes CD74, HLA-DRA, B2M, and JUNB are measurably enriched within the identified macrophage cluster
- Every sketched cell (indicator = 1) receives exactly one community assignment from Louvain

## Limitations

- The interplay of sketching with respect to a given distance metric and subsequent algorithmic analysis was not systematically addressed in the study
- Sphetcher's underlying model might be less compatible with assumptions made by algorithms other than Louvain and Monocle 2
- No software version numbers are stated for Sphetcher or the downstream tools, complicating exact reproduction

## Evidence

- [discussion] Louvain is widely used for downstream scRNA-seq clustering of sketched cells: "Louvain community detection and Monocle 2 are widely used for scRNA-seq clustering"
- [readme] Sphetcher output defines which cells are passed to Louvain: "an indicator vector of `n` samples where 1 indicates the sample is in the sketch, 0 otherwise (there are `sketch_size` 1's in the vector)"
- [readme] Sphetcher CLI generates the indicator vector that precedes clustering: "sphetcher expression_matrix.csv sketch_size sketch_indicator_output.csv"
- [discussion] SC3 is a costlier alternative that becomes feasible only on sketches: "consensus clustering by SC3 (Kiselev et al., 2017) might become practical again on a spherical sketch"
- [discussion] Metric-to-clustering interplay not systematically evaluated: "This interplay of sketching with respect to a given distance metric and subsequent algorithmic analysis was not systematically addressed"
