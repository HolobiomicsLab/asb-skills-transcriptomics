---
name: reference-mapping-benchmark-comparison
description: Use when choosing a reference-mapping tool for scRNA-seq data and you
  need quantitative evidence on projection fidelity and cell-state classification
  accuracy rather than qualitative UMAP comparisons.
when_to_use_negative:
- You have no labeled reference atlas or no ground-truth cell state annotations, so
  cross-validated accuracy cannot be scored
- You want to project genuinely novel query data rather than benchmark mapping methods
  — use Run.ProjecTILs directly instead of splitting the reference
- Your goal is atlas integration/batch correction rather than projecting new data
  into a fixed reference space (STACAS, Harmony or Seurat integration is the wrong
  comparator here)
edam_operation: http://edamontology.org/operation_2928
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_2229
tools:
- name: ProjecTILs
  role: 'Reference-mapping method under test: projects held-out cells via make.projection
    and classifies states via cellstate.predict'
  repo: https://github.com/carmonalab/ProjecTILs
- name: Seurat
  role: R framework in which ProjecTILs is implemented and through which Azimuth (Seurat
    4) is benchmarked
- name: Azimuth
  role: Alternative reference-based mapping method re-run on the identical cross-validation
    splits as a comparator
- name: scmap
  role: Alternative comparator using scmapCell and scmapCell2Cluster for projection
    and cluster-label transfer on the same splits
- name: R
  role: Language in which the benchmark, classifiers, and accuracy scoring are implemented
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1038/s41467-021-23324-4
    title: Interpretation of T cell states from single-cell transcriptomics data using
      reference atlases
  source_content_hash: sha256:1bc07c1dedcf529a255b8d44e652e0cebff42b38f3d0b62116f29ad3cce44f65
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
  edam_operation: http://edamontology.org/operation_2928
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_2229
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8181818181818182
n_claims: 11
---

# reference-mapping-benchmark-comparison

## Summary

A cross-validation benchmarking protocol that quantifies how accurately ProjecTILs projects and classifies held-out scRNA-seq data onto a reference T cell atlas, and compares it head-to-head against alternative reference-mapping methods (Azimuth/Seurat 4, scmap) using shared projection- and label-accuracy metrics.

## When to use

Use when choosing a reference-mapping tool for scRNA-seq data and you need quantitative evidence on projection fidelity and cell-state classification accuracy rather than qualitative UMAP comparisons.

## When NOT to use

- You have no labeled reference atlas or no ground-truth cell state annotations, so cross-validated accuracy cannot be scored
- You want to project genuinely novel query data rather than benchmark mapping methods — use Run.ProjecTILs directly instead of splitting the reference
- Your goal is atlas integration/batch correction rather than projecting new data into a fixed reference space (STACAS, Harmony or Seurat integration is the wrong comparator here)

## Inputs

- Reference single-cell atlas Seurat object (e.g. mouse TIL atlas, loadable via load.reference.map)
- Held-out query scRNA-seq count matrices with ground-truth cell state labels
- Competing reference-mapping methods: Azimuth/Seurat 4 and scmap (scmapCell, scmapCell2Cluster)

## Outputs

- Per-method cross-validation accuracy table (crossval_accuracy_metrics.tsv)
- ProjecTILs projection accuracy (fraction of cells within 1 UMAP unit and 5 PCA units of original coordinates)
- ProjecTILs classification accuracy (fraction of correctly transferred cell state labels)

## How to apply

Load the TIL reference map with load.reference.map and split its constituent samples into two halves for leave-half-samples-out cross-validation. Project the held-out half with make.projection so query cells are embedded in the reference PCA and UMAP space, then transfer ground-truth cell state labels with cellstate.predict, the nearest-neighbor classifier on the reference embedding. Score projection accuracy as the fraction of cells landing within one UMAP unit (and within five PCA units) of their original coordinates, and classification accuracy as the fraction of correctly assigned cell state labels. Re-run the identical split-based benchmark with Azimuth/Seurat 4 and with scmapCell/scmapCell2Cluster so all methods see the same train/test partitions, then compile per-method accuracies into a crossval_accuracy_metrics.tsv table.

## Related tools

- **ProjecTILs** (Reference-mapping method under test: projects held-out cells via make.projection and classifies states via cellstate.predict) — https://github.com/carmonalab/ProjecTILs
- **Seurat** (R framework in which ProjecTILs is implemented and through which Azimuth (Seurat 4) is benchmarked)
- **Azimuth** (Alternative reference-based mapping method re-run on the identical cross-validation splits as a comparator)
- **scmap** (Alternative comparator using scmapCell and scmapCell2Cluster for projection and cluster-label transfer on the same splits)
- **R** (Language in which the benchmark, classifiers, and accuracy scoring are implemented)

## Examples

```
library(ProjecTILs); ref <- load.reference.map(); query.projected <- make.projection(query, ref=ref); query.projected <- cellstate.predict(query.projected, ref=ref)
```

## Evaluation signals

- ProjecTILs projection accuracy exceeds 90% of held-out cells landing within one UMAP unit of their original coordinates (93.4% within five PCA units)
- ProjecTILs classification accuracy reaches 91.6% correctly assigned cell state labels in cross-validation
- ProjecTILs accuracy is significantly higher than both Azimuth/Seurat 4 and scmapCell/scmapCell2Cluster on the same train/test partitions
- crossval_accuracy_metrics.tsv contains per-method accuracies for all three benchmarked methods
- The same leave-half-samples-out sample partitions were used for every method (no per-method re-splitting)

## Limitations

- Projection accuracy is partly judged in UMAP space, where cell-cell distances and densities are highly distorted and results should be interpreted with caution (README cites Chari and Pachter 2023); PCA-space scoring is more reliable
- Reference atlases do not cover the full diversity of T cell states: CD4+ TILs are under-represented and gamma-delta T cells are not represented at all, limiting generalization of benchmark conclusions
- Benchmark accuracy reflects leave-half-samples-out resampling of the reference itself; accuracy on truly out-of-distribution query data (new tissues, species, perturbations) may be lower

## Evidence

- [results] Cross-validation accuracy thresholds: [evidence span withheld — anchor status: not_found]
- [results] Comparator methods: [evidence span withheld — anchor status: not_found]
- [methods] Accuracy metric definition: [evidence span withheld — anchor status: not_found]
- [methods] Classifier description: "Use a nearest-neighbor algorithm to predict a feature (e.g. the cell state) of the query cells"
- [readme] UMAP caution: [evidence span withheld — anchor status: external_doc]
- [discussion] Integration with Seurat: "Because ProjecTILs is integrated with Seurat, it can be easily combined with other tools for up- and down-stream analyses."
