---
name: umap-embedding-overlay-visualization
description: Use when per-cell scores (DA-seq differential abundance scores, cluster
  or severity annotations) need to be visualized in low-dimensional space on a UMAP
  built from scMerge2-adjusted single-cell expression data.
when_to_use_negative:
- The expression data have not been integrated across samples/conditions (batch variability
  will dominate the UMAP), so first apply scMerge2 integration
- Only embedding outputs (no adjusted expression matrix) are available and interpretable
  per-gene features are needed downstream — the article notes 'a low-dimensional representation
  does not naturally lend itself to the development of interpretable features'
edam_operation: http://edamontology.org/operation_0570
tools:
- name: scMerge2
  role: Produces the adjusted expression matrix from which the 30-PC embedding and
    UMAP are derived before overlaying DA scores
  repo: https://github.com/SydneyBioX/scMerge
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1038/s41467-023-39923-2
    title: Atlas-scale single-cell multi-sample multi-condition data integration using
      scMerge2
  source_content_hash: sha256:b9b0d7687f2525a59c2242e9f585e6d423e8810ef40583a722bbe97cae5a9d20
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/SydneyBioX/scMerge
  tools:
  - scMerge2
  edam_operation: http://edamontology.org/operation_0570
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8571428571428571
n_claims: 7
---

# umap-embedding-overlay-visualization

## Summary

Overlays per-cell quantitative scores (e.g., DA-seq differential abundance scores) onto a UMAP embedding computed from the scMerge2-adjusted single-cell expression matrix, enabling visual localization of severity-associated cell states in the COVID-19 atlas.

## When to use

Use when per-cell scores (DA-seq differential abundance scores, cluster or severity annotations) need to be visualized in low-dimensional space on a UMAP built from scMerge2-adjusted single-cell expression data.

## When NOT to use

- The expression data have not been integrated across samples/conditions (batch variability will dominate the UMAP), so first apply scMerge2 integration
- Only embedding outputs (no adjusted expression matrix) are available and interpretable per-gene features are needed downstream — the article notes 'a low-dimensional representation does not naturally lend itself to the development of interpretable features'

## Inputs

- scMerge2-adjusted single-cell expression matrix (e.g., SingleCellExperiment object)
- Top-30 principal component embedding of the adjusted data
- Per-cell DA-seq differential abundance scores (k from 50 to 500)
- Cell type annotations (colData of the SingleCellExperiment)

## Outputs

- UMAP embedding of the adjusted data colored by per-cell DA score
- UMAP overlay figures of severity-associated DA regions and cell type labels
- Per-cell DA score file used for downstream export

## How to apply

Compute the UMAP on the top principal components of the scMerge2-adjusted expression matrix (the DA-seq workflow used a 30-PC embedding of the COVID-19 data integrating ~5 million cells from 20 studies). Align per-cell DA-seq differential abundance scores — computed by sweeping neighborhood size k from 50 to 500 — with the UMAP coordinates by cell identity, then overlay the scores as a continuous color scale on the UMAP to reveal spatial localization of severe-outcome-associated DA regions. Overlay the annotated cell type labels on the same embedding to cross-check that the highest-scoring regions correspond to neutrophils. The overlay is a verification step for the aggregation step in which per-cell DA scores are summarized by cell type into a DA-proportion table.

## Related tools

- **scMerge2** (Produces the adjusted expression matrix from which the 30-PC embedding and UMAP are derived before overlaying DA scores) — https://github.com/SydneyBioX/scMerge

## Examples

```
BiocManager::install("scMerge")
```

## Evaluation signals

- UMAP coordinates are computed from the top 30 PCs of the scMerge2-adjusted matrix (not raw counts)
- Every cell in the UMAP has a defined DA score value (no missing join between score file and embedding)
- The colored UMAP shows the DA-score regions consistent with the cell-type DA-proportion table where neutrophils hold the highest proportion of severe-outcome-associated cells
- Overlay is reproduced across the swept neighborhood sizes (k = 50 to 500) to confirm score regions are stable
- The per-cell DA score file exported alongside the visualization matches the cells represented in the UMAP

## Limitations

- Embedding-only outputs increase interpretability challenges for downstream features such as cell-cell interactions or pathway information (article discussion)
- The visualization reflects the scMerge2 data-guided integration order and quality; misintegration of samples would distort the DA-score overlay
- UMAP overlay is qualitative support for the DA-seq quantification, not a substitute for the per-cell-type aggregation step

## Evidence

- [intro] integration improves discrimination of cell states: "the integration using scMerge2 improves the performance of discriminating distinct cell states in COVID-19 patients with varying degrees of severity"
- [abstract] scMerge2 is a scalable integration algorithm: "Here, we present scMerge2, a scalable algorithm that allows data integration of atlas-scale multi-sample multi-condition single-cell studies."
- [discussion] embedding interpretability caveat: "an embedding output by itself increases interpretability challenges since a low-dimensional representation does not naturally lend itself to the development of interpretable features"
- [readme] scMerge package availability: "scMerge is a R package for merging and normalising single-cell RNA-Seq datasets."
