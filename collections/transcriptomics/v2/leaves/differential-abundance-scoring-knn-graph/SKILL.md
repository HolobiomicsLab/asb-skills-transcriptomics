---
name: differential-abundance-scoring-knn-graph
description: Use when comparing cell abundance between patient conditions on an integrated
  single-cell dataset — e.g. finding which cell types are enriched in severe versus
  moderate COVID-19 samples.
when_to_use_negative:
- Input is an unintegrated or batch-effect-confounded expression matrix — kNN-graph
  DA scores on such embeddings reflect dataset variability rather than biological
  condition differences.
- The analysis goal is joint use of multi-modal information — the paper's integration
  was done within each modality, with joint multi-modal use deferred to future work.
- Only an embedding (low-dimensional representation) is available without an adjusted
  expression matrix — embedding-only outputs increase interpretability challenges
  for downstream features.
edam_operation: http://edamontology.org/operation_3223
edam_topics:
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_0091
tools:
- name: scMerge2
  role: Produces the scMerge2-adjusted expression matrix and embedding on which the
    30-PC kNN graph and DA-seq differential abundance scoring are performed
  repo: https://github.com/SydneyBioX/scMerge
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1038/s41467-023-39923-2
    title: Atlas-scale single-cell multi-sample multi-condition data integration using
      scMerge2
  source_content_hash: sha256:ab63a673cd9b4274770ca18e50a48e65f4e98e28ad3464b8564ef16a115a6d98
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/SydneyBioX/scMerge
  tools:
  - scMerge2
  edam_operation: http://edamontology.org/operation_3223
  edam_topics:
  - http://edamontology.org/topic_0203
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8333333333333334
n_claims: 12
---

# Differential-abundance-scoring-kNN-graph

## Summary

Quantifies cell-type abundance shifts between disease conditions by computing per-cell differential abundance (DA-seq) scores over a kNN graph built on a scMerge2-integrated single-cell embedding, then aggregating scores per annotated cell type. It identifies which cell types (e.g. neutrophils in COVID-19) are enriched in severe versus moderate patients.

## When to use

Use when comparing cell abundance between patient conditions on an integrated single-cell dataset — e.g. finding which cell types are enriched in severe versus moderate COVID-19 samples.

## When NOT to use

- Input is an unintegrated or batch-effect-confounded expression matrix — kNN-graph DA scores on such embeddings reflect dataset variability rather than biological condition differences.
- The analysis goal is joint use of multi-modal information — the paper's integration was done within each modality, with joint multi-modal use deferred to future work.
- Only an embedding (low-dimensional representation) is available without an adjusted expression matrix — embedding-only outputs increase interpretability challenges for downstream features.

## Inputs

- scMerge2-adjusted single-cell expression matrix (e.g. COVID-19 collection, ~5 million cells, 963 donors, 1298 samples from 20 studies)
- Per-cell patient condition labels (moderate vs severe COVID-19)
- Per-cell annotated cell type labels (e.g. neutrophils)

## Outputs

- Per-cell DA-seq differential abundance score file (overlayable on UMAP)
- Cell-type DA-proportion table ranking cell types by proportion of severe-outcome-associated cells
- UMAP embedding with per-cell DA scores overlaid

## How to apply

Load the scMerge2-adjusted single-cell expression matrix, which produces a consensus embedding suitable for downstream analysis across cohorts, and reduce it to the top 30 principal components. Build a k-nearest-neighbour graph on this 30-PC embedding and run DA-seq comparing moderate versus severe patients, sweeping the neighbourhood size k from 50 to 500 so per-cell differential abundance scores are robust to neighbourhood scale. Identify severity-associated DA regions and overlay the per-cell DA scores on the UMAP of the adjusted data to visually confirm concordance. Aggregate per-cell DA scores by annotated cell type to compute, for each cell type, the proportion of cells in severe-outcome-associated regions; a higher proportion marks a more severity-associated cell state. The integration step matters because many rapid integration methods do not return adjusted expression matrices, which restricts sample-level analysis; scMerge2 supplies the adjusted matrix that this abundance analysis consumes.

## Related tools

- **scMerge2** (Produces the scMerge2-adjusted expression matrix and embedding on which the 30-PC kNN graph and DA-seq differential abundance scoring are performed) — https://github.com/SydneyBioX/scMerge

## Evaluation signals

- Neutrophils hold the highest proportion of severe-outcome-associated cells in the aggregated cell-type DA-proportion table, consistent with neutrophil accumulation marking critical COVID-19 illness.
- Per-cell DA scores are computed across the full k sweep from 50 to 500 and remain stable, rather than being driven by a single neighbourhood size.
- The DA-score overlay on the UMAP of the scMerge2-adjusted data places severe-associated regions in spatially coherent neighbourhoods rather than scattered isolated cells.
- The per-cell DA score file covers all cells from the moderate-versus-severe comparison and joins cleanly to cell type annotations.

## Limitations

- DA results depend on integration quality: the interpretation of cell states relies on the scMerge2 consensus embedding, and multi-modal joint integration was not performed (only within each modality).
- Embedding-only integration outputs would be insufficient because a low-dimensional representation does not naturally lend itself to interpretable downstream features.
- Computationally heavier integration orderings exist; the concatenating approach is more computationally intensive than the balanced tree approach used for atlas-scale data.

## Evidence

- [abstract] scMerge2 enables multi-sample multi-condition integration on COVID-19 data: "Here, we present scMerge2, a scalable algorithm that allows data integration of atlas-scale multi-sample multi-condition single-cell studies."
- [intro] Integration improves discrimination of severity-associated cell states, enabling this downstream abundance analysis: "improves the performance of discriminating distinct cell states in COVID-19 patients with varying degrees of severity"
- [discussion] Key innovations underlying the adjusted matrix the DA-seq workflow consumes: "This was achieved via the use of three essential innovations with hierarchical integration, pseudo-bulk building to minimise processing demand"
- [readme] Repository providing the scMerge/scMerge2 software: [evidence span withheld — anchor status: external_doc]
