---
name: projection-accuracy-metrics-evaluation
description: Use when you need to quantitatively validate how faithfully a method
  projects held-out scRNA-seq data onto a reference atlas and how accurately it transfers
  cell-state labels, relative to alternative mapping tools.
when_to_use_negative:
- The goal is annotating a genuinely new query dataset rather than benchmarking projection
  fidelity — use Run.ProjecTILs label transfer directly instead of cross-validation
  scoring.
- The query contains cell states absent from the reference (e.g. γδ T cells, under-represented
  CD4+ TILs), where projection-accuracy metrics against reference coordinates are
  not meaningful.
edam_operation: http://edamontology.org/operation_2939
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_0091
tools:
- name: ProjecTILs
  role: Provides load.reference.map, make.projection and cellstate.predict used to
    embed held-out samples and transfer labels for accuracy scoring
  repo: https://github.com/carmonalab/ProjecTILs
- name: Seurat
  role: Underlying framework for PCA/UMAP embedding of the reference; also used via
    Azimuth/Seurat 4 as a competing mapping method in the benchmark
- name: Azimuth
  role: Alternative reference-mapping method re-run on identical cross-validation
    splits to compare accuracies
- name: scmap
  role: Alternative method (scmapCell/scmapCell2Cluster) benchmarked against ProjecTILs
    on the same splits
- name: R
  role: Language in which ProjecTILs and the cross-validation benchmark are implemented
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1038/s41467-021-23324-4
    title: Interpretation of T cell states from single-cell transcriptomics data using
      reference atlases
  source_content_hash: sha256:7458bd905b47d826f2f33f34867a296e7bda703ac45b92c7463f91184ca37a74
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
  edam_operation: http://edamontology.org/operation_2939
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8888888888888888
n_claims: 9
---

# projection-accuracy-metrics-evaluation

## Summary

Benchmarks a reference-mapping method by leave-half-samples-out cross-validation on a reference TIL atlas, scoring projection accuracy (fraction of cells landing within one UMAP unit / five PCA units of original coordinates) and classification accuracy of cell-state labels against competing methods such as Azimuth/Seurat 4 and scmap.

## When to use

Use when you need to quantitatively validate how faithfully a method projects held-out scRNA-seq data onto a reference atlas and how accurately it transfers cell-state labels, relative to alternative mapping tools.

## When NOT to use

- The goal is annotating a genuinely new query dataset rather than benchmarking projection fidelity — use Run.ProjecTILs label transfer directly instead of cross-validation scoring.
- The query contains cell states absent from the reference (e.g. γδ T cells, under-represented CD4+ TILs), where projection-accuracy metrics against reference coordinates are not meaningful.

## Inputs

- TIL reference map loaded via load.reference.map (mouse TIL atlas, 16,803 cells / 25 samples)
- Held-out half of reference samples as Seurat query objects with raw counts and ground-truth cell-state labels
- Competing method outputs from Azimuth/Seurat 4 and scmapCell/scmapCell2Cluster runs on identical splits

## Outputs

- Projection accuracy: fraction of cells within one UMAP unit and within five PCA units of original coordinates
- Classification accuracy: fraction of correctly assigned cell-state labels (e.g. 91.6% for ProjecTILs)
- Per-method accuracy table (crossval_accuracy_metrics.tsv)

## How to apply

Load the TIL reference map with load.reference.map and split its constituent samples into two halves for leave-half-samples-out cross-validation. Project the held-out half with make.projection, embedding query cells in the reference PCA and UMAP space, then transfer ground-truth labels with cellstate.predict, the nearest-neighbor classifier on the reference embedding. Score projection accuracy as the fraction of cells landing within one UMAP unit (and within five PCA units) of their original coordinates, and classification accuracy as the fraction of correctly assigned cell-state labels. Re-run the identical benchmark with Azimuth/Seurat 4 and scmapCell/scmapCell2Cluster, then compile per-method accuracies (e.g. into crossval_accuracy_metrics.tsv) and accept if projection exceeds 90% within one UMAP unit (93.4% within five PCA units) and classification reaches 91.6%.

## Related tools

- **ProjecTILs** (Provides load.reference.map, make.projection and cellstate.predict used to embed held-out samples and transfer labels for accuracy scoring) — https://github.com/carmonalab/ProjecTILs
- **Seurat** (Underlying framework for PCA/UMAP embedding of the reference; also used via Azimuth/Seurat 4 as a competing mapping method in the benchmark)
- **Azimuth** (Alternative reference-mapping method re-run on identical cross-validation splits to compare accuracies)
- **scmap** (Alternative method (scmapCell/scmapCell2Cluster) benchmarked against ProjecTILs on the same splits)
- **R** (Language in which ProjecTILs and the cross-validation benchmark are implemented)

## Examples

```
ref <- load.reference.map(); query.projected <- make.projection(query, ref=ref); query.projected <- cellstate.predict(query.projected, ref=ref)
```

## Evaluation signals

- ProjecTILs projection accuracy exceeds 90% of cells within one UMAP unit of original coordinates
- Projection accuracy within five PCA units reaches 93.4%
- Cell-state classification accuracy reaches 91.6% in leave-half-samples-out cross-validation
- ProjecTILs accuracies significantly exceed those of Azimuth/Seurat 4 and scmapCell/scmapCell2Cluster on identical splits
- Per-method metrics compiled without alteration into crossval_accuracy_metrics.tsv

## Limitations

- Projection accuracy is assessed against the reference embedding itself (held-out samples from the atlas), so it measures consistency with the reference rather than performance on wholly independent data.
- UMAP distances are distorted; the README notes cell-cell distances and densities in UMAP are highly distorted, so the one-UMAP-unit metric should be read alongside PCA-space accuracy.
- Reference atlases do not cover all states — γδ T cells were not represented at all and CD4+ TILs were under-represented, limiting accuracy estimates for such query cells.

## Evidence

- [other] Score projection accuracy as the fraction of cells landing within one UMAP unit (and within five PCA units) of their original coordinates, and classification accuracy as the fraction of correctly assigned cell state labels.: [evidence span withheld — anchor status: not_found]
- [other] Cross-validation finding: [evidence span withheld — anchor status: not_found]
- [methods] make.projection function description: "`make.projection`   Project a single-cell RNA-seq dataset onto a reference map of cellular states"
- [methods] cellstate.predict function description: "Use a nearest-neighbor algorithm to predict a feature (e.g. the cell state) of the query cells"
- [abstract] ProjecTILs embedding claim: "ProjecTILs allows not only accurate embedding of new scRNA-seq data into a reference without altering its structure"
- [readme] README note on UMAP distortion: [evidence span withheld — anchor status: external_doc]
