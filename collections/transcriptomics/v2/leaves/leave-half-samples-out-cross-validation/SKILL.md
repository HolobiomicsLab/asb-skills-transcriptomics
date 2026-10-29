---
name: leave-half-samples-out-cross-validation
description: Use when you must quantify how accurately a reference-mapping method
  (ProjecTILs, Azimuth/Seurat 4, scmap) projects and classifies held-out scRNA-seq
  samples onto a reference atlas, by a sample-level split so no query cell was seen
  during reference construction.
when_to_use_negative:
- The query dataset was part of the reference integration — sample-level split is
  required, so self-referential evaluation would inflate accuracy
- The biological goal is exploring a new query in the reference space rather than
  benchmarking mapping accuracy — use Run.ProjecTILs in standard mode instead
- The cell types of interest are absent from the reference atlas (e.g. γδ T cells,
  which were not represented at all) — classification benchmarks would be meaningless
edam_operation: http://edamontology.org/operation_0282
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: ProjecTILs
  role: 'Method under test: projects query cells into the reference PCA/UMAP space
    (make.projection) and classifies cell states via nearest-neighbor label transfer
    (cellstate.predict)'
  repo: https://github.com/carmonalab/ProjecTILs
- name: Seurat
  role: Foundation object model in which the reference map and projected query are
    stored; ProjecTILs is integrated with Seurat for upstream/downstream analysis
- name: Azimuth
  role: Alternative reference-mapping method re-run on the identical cross-validation
    split as a performance comparator
- name: scmap
  role: Alternative reference-based method (scmapCell/scmapCell2Cluster) benchmarked
    under the same leave-half-samples-out scheme
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1038/s41467-021-23324-4
    title: Interpretation of T cell states from single-cell transcriptomics data using
      reference atlases
  source_content_hash: sha256:d058b23df926552ca87081d673bea8a5b83feacad5ba471cea076c7ff3fc7c59
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/carmonalab/ProjecTILs
  tools:
  - Azimuth
  - ProjecTILs
  - R
  - Seurat
  - scmap
  edam_operation: http://edamontology.org/operation_0282
  edam_topics:
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8888888888888888
n_claims: 9
---

# leave-half-samples-out-cross-validation

## Summary

Benchmark protocol that partitions a reference single-cell atlas's constituent samples into two halves, projects the held-out half onto the retained reference with ProjecTILs, and scores projection and cell-state classification accuracy — with the identical scheme re-run for Azimuth/Seurat 4 and scmap for comparison.

## When to use

Use when you must quantify how accurately a reference-mapping method (ProjecTILs, Azimuth/Seurat 4, scmap) projects and classifies held-out scRNA-seq samples onto a reference atlas, by a sample-level split so no query cell was seen during reference construction.

## When NOT to use

- The query dataset was part of the reference integration — sample-level split is required, so self-referential evaluation would inflate accuracy
- The biological goal is exploring a new query in the reference space rather than benchmarking mapping accuracy — use Run.ProjecTILs in standard mode instead
- The cell types of interest are absent from the reference atlas (e.g. γδ T cells, which were not represented at all) — classification benchmarks would be meaningless

## Inputs

- Reference single-cell atlas (TIL reference map) with per-sample metadata and ground-truth cell state labels, loaded via load.reference.map
- Held-out half of reference samples as a query Seurat object with scRNA-seq raw counts (internally log-normalized by ProjecTILs)

## Outputs

- Projected query object embedded in the reference PCA and UMAP space
- Predicted cell state labels per query cell from cellstate.predict
- Per-method projection accuracy (within 1 UMAP unit; within 5 PCA units) and classification accuracy, compiled into crossval_accuracy_metrics.tsv

## How to apply

Load the TIL reference map with load.reference.map and partition its constituent samples into two halves for leave-half-samples-out cross-validation. Project the held-out half with make.projection (Run.ProjecTILs), embedding query cells in the reference PCA and UMAP space, then transfer ground-truth cell state labels with cellstate.predict, the nearest-neighbor classifier on the reference embedding. Score projection accuracy as the fraction of cells landing within one UMAP unit (and within five PCA units) of their original coordinates, and classification accuracy as the fraction of correctly assigned cell state labels. Re-run the identical benchmark with Azimuth/Seurat 4 and with scmapCell/scmapCell2Cluster so methods are compared under the same sample split. Accept the result if ProjecTILs projection accuracy exceeds 90% within one UMAP unit (93.4% within five PCA units) and classification accuracy reaches 91.6%, and compile per-method accuracies into crossval_accuracy_metrics.tsv.

## Related tools

- **ProjecTILs** (Method under test: projects query cells into the reference PCA/UMAP space (make.projection) and classifies cell states via nearest-neighbor label transfer (cellstate.predict)) — https://github.com/carmonalab/ProjecTILs
- **Seurat** (Foundation object model in which the reference map and projected query are stored; ProjecTILs is integrated with Seurat for upstream/downstream analysis)
- **Azimuth** (Alternative reference-mapping method re-run on the identical cross-validation split as a performance comparator)
- **scmap** (Alternative reference-based method (scmapCell/scmapCell2Cluster) benchmarked under the same leave-half-samples-out scheme)

## Examples

```
ref <- load.reference.map(); query.projected <- Run.ProjecTILs(query_example_seurat, ref=ref)
```

## Evaluation signals

- Projection accuracy exceeds 90% of held-out cells landing within one UMAP unit of their original coordinates
- Projection accuracy is 93.4% within five PCA units of original coordinates
- Classification accuracy reaches 91.6% of correctly assigned cell state labels
- Both comparator methods (Azimuth/Seurat 4 and scmap) are evaluated under the identical sample split and show significantly lower accuracy than ProjecTILs
- Per-method accuracy values are written as a complete table to crossval_accuracy_metrics.tsv

## Limitations

- Reference atlases do not yet encompass the full diversity of T cell transcriptional states, particularly for CD4+ TILs (under-represented compared to CD8+), and γδ T cells were not represented at all
- UMAP distances and densities are highly distorted; cell-state quantification is performed in the PCA latent space and UMAP is for visual exploration only
- Accuracy metrics are tied to the specific mouse TIL reference atlas and sample composition; no high-quality human T cell reference atlas existed at publication time

## Evidence

- [methods] Projection accuracy metric: "`make.projection`   Project a single-cell RNA-seq dataset onto a reference map of cellular states"
- [methods] Classification metric: "`cellstate.predict`   Use a nearest-neighbor algorithm to predict a feature (e.g. the cell state) of the query cells"
- [abstract] Reference construction context: "we generate reference T cell atlases for cancer and viral infection by multi-study integration, and develop ProjecTILs, an algorithm for reference atlas projection"
- [intro] Rationale for ProjecTILs over competitors: "these methods define a new embedding space specific for the query, not preserving the integrity of the reference atlas space upon mapping"
- [readme] PCA vs UMAP scoring basis: [evidence span withheld — anchor status: external_doc]
- [readme] Representative invocation: [evidence span withheld — anchor status: external_doc]
