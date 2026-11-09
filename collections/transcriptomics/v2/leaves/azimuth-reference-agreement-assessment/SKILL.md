---
name: azimuth-reference-agreement-assessment
description: Use when scTriangulate final labels need external validation against
  a supervised reference annotation (Azimuth L2/L2-level atlas labels), or when a
  reference label-transfer annotation should be jointly integrated as a fourth conflicting
  annotation set.
when_to_use_negative:
- When no reference atlas annotations exist for the tissue/technology — the agreement
  assessment requires Azimuth (or equivalent supervised) labels for the same cells
- When the goal is data-level integration via a joint latent space or geometric graph
  (e.g., Seurat-WNN, TotalVI) rather than decision-level label reconciliation
- When reference annotations should be the sole ground truth overwriting all unsupervised
  solutions — scTriangulate weighs the reference as one player among many, not as
  an oracle
edam_operation: http://edamontology.org/operation_3432
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_2229
tools:
- name: scTriangulate
  role: Decision-level integration engine (lazy_run) that mixes the Azimuth reference
    annotation with Leiden solutions and exports reconciled labels for agreement scoring;
    version 0.9.0 used
  repo: https://github.com/frankligy/scTriangulate
- name: Azimuth
  role: Supervised reference-mapping app supplying L2-level cell-type annotations
    used both as an integrated player and as an external validation benchmark
- name: scanpy
  role: Preprocessing and Leiden clustering via the scanpy_recipe wrapper (version
    1.7.2) that generates the conflicting annotations to compare against the reference
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1038/s41467-023-36016-y
    title: Decision level integration of unimodal and multimodal single cell data
      with scTriangulate
  source_content_hash: sha256:fd6fda26090bdff22c52e7fe7f8bf766a624138fb6cf9b805ca66f1fa5acca49
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/frankligy/scTriangulate
  tools:
  - Azimuth
  - scTriangulate
  - scanpy
  edam_operation: http://edamontology.org/operation_3432
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_2229
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 6
---

# azimuth-reference-agreement-assessment

## Summary

Assess the agreement between scTriangulate decision-level integrated cluster labels and Azimuth reference-derived cell-type annotations (e.g., L2-level PBMC or human-lung atlas labels), treating the reference annotation as an additional conflicting 'player' and as an external validation benchmark.

## When to use

Use when scTriangulate final labels need external validation against a supervised reference annotation (Azimuth L2/L2-level atlas labels), or when a reference label-transfer annotation should be jointly integrated as a fourth conflicting annotation set.

## When NOT to use

- When no reference atlas annotations exist for the tissue/technology — the agreement assessment requires Azimuth (or equivalent supervised) labels for the same cells
- When the goal is data-level integration via a joint latent space or geometric graph (e.g., Seurat-WNN, TotalVI) rather than decision-level label reconciliation
- When reference annotations should be the sole ground truth overwriting all unsupervised solutions — scTriangulate weighs the reference as one player among many, not as an oracle

## Inputs

- AnnData (adata.X log-normalized CPTT for RNA) with at least two conflicting annotation columns in adata.obs (e.g., sctri_rna_leiden_1/2/3 from scanpy_recipe)
- Azimuth reference annotations (L2-level) for the same cell barcodes, obtained from the Azimuth R-Shiny app
- adata.obsm['X_umap'] coordinates for visualization (generated via scanpy_recipe)

## Outputs

- Per-cell pruned scTriangulate labels reconciled with the reference annotation as one input player (e.g., exported to sctriangulate_pbmc10k_labels.csv)
- Per-cell winning-fraction confidence values
- Per-cluster stability metrics (reassign, TFIDF5, TFIDF10, SCCAF)
- Agreement measure of final labels vs. Azimuth L2 reference annotations

## How to apply

Fetch reference annotations from the Azimuth R-Shiny app for the same cells (e.g., Azimuth PBMC L2 labels for pbmc_10k_v3, or human-lung-v2 for HLCA) and add them to the AnnData via scTriangulate.preprocessing.add_annotation so they participate as an additional annotation set alongside unsupervised Leiden solutions (resolutions 1, 2, 3; n_neighbors=15, n_pcs=50). scTriangulate's Shapley-based selection can then weigh the supervised reference against unsupervised solutions, since it 'cannot jointly consider prior annotations from label transfer' is precisely the gap it fills. After pruning (win_fraction_cutoff=0.25, abs_thresh=10 cells with nearest-neighbor reclassification), export final per-cell labels and compute agreement of the pruned labels against the Azimuth L2 reference as an independent accuracy signal. Because scTriangulate integrates at the decision level, no joint embedding of the reference labels is required — the reference only enters as discrete per-cell labels.

## Related tools

- **scTriangulate** (Decision-level integration engine (lazy_run) that mixes the Azimuth reference annotation with Leiden solutions and exports reconciled labels for agreement scoring; version 0.9.0 used) — https://github.com/frankligy/scTriangulate
- **Azimuth** (Supervised reference-mapping app supplying L2-level cell-type annotations used both as an integrated player and as an external validation benchmark)
- **scanpy** (Preprocessing and Leiden clustering via the scanpy_recipe wrapper (version 1.7.2) that generates the conflicting annotations to compare against the reference)

## Examples

```
sctri = ScTriangulate(dir='./output', adata=adata, query=['sctri_rna_leiden_1','sctri_rna_leiden_2','sctri_rna_leiden_3','azimuth_l2']); sctri.lazy_run()
```

## Evaluation signals

- Final pruned labels exported with per-cell winning-fraction values; clusters with winning fraction < 0.25 or < 10 cells must have been reclassified by nearest-neighbor
- Agreement of final labels with Azimuth L2 reference annotations is computed and reported (e.g., confusion-matrix-based concordance per cell type)
- Every cell receives exactly one definitive annotation (binary assignment matrix X of shape I * N); no cell left unassigned
- Shapley Phi values show no single annotation set (including the reference) always drives the decision — near-tie ranks offset by 0.01 with all_or_none winner-takes-all adjustment
- Per-cluster stability metrics (reassign, TFIDF5, TFIDF10, SCCAF) are computed for every cluster of every annotation set, including the Azimuth reference column

## Limitations

- Population naming remains largely non-standardized, complicating direct label agreement assessment
- Integration of cluster predictions from dozens of sources significantly increases runtime; beyond 15 annotation sets Shapley selection is substituted with combined rank-based selection
- Default analytical settings (win_fraction_cutoff=0.25, abs_thresh=10) may not be optimal for all study designs

## Evidence

- [intro] scTriangulate integrates results at a decision level to reconcile conflicting cluster-label assignments, rather than at the data level: "While other multimodal methods integrate at the data level (through either a low-dimensional latent space6 or through a geometric graph5), scTriangulate integrates results at a decision level to"
- [methods] add_annotation imports reference labels into AnnData: "The obtained annotations and UMAP coordinates24 were added to the existing AnnData by the scTriangulate.preprocessing.add_annotation and scTriangulate.preprocessing.add_umap functions"
- [discussion] Reference-classification integration is a supported use case: "the integration of diverse cellular annotations from independent studies (i.e., reference-classification), multimodal measurements, distinct clustering algorithms, and software settings"
- [methods] Pruning thresholds for unstable clusters: "By default, wfc < 0.25 will be labeled as unstable clusters."
- [methods] Azimuth app provides reference annotations used in the workflow: "The same two-step mapping strategy was applied to HLCA (https://app.azimuth.hubmapconsortium.org/app/human-lung-v2)"
