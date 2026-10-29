---
name: single-cell-cluster-stability-integration
description: Use when an AnnData object carries ≥2 conflicting cell-cluster annotation
  columns (e.g. Leiden at different resolutions, Seurat3/Monocle3/ICGS2, cellHarmony
  label transfers, or RNA/ADT modality labels) and a single high-confidence reconciled
  assignment is needed.
when_to_use_negative:
- When a single well-justified clustering resolution already resolves all cell states
  of interest — scTriangulate addresses the case where no single resolution identifies
  the most discrete cell states across modalities
- When the goal is a joint latent-space or graph-based multimodal embedding (e.g.
  Seurat-WNN or TotalVI) rather than reconciling pre-computed cluster annotations
  at the decision level
- 'When cell populations still need to be named: population naming remains largely
  non-standardized and is an explicitly unmet challenge of this approach'
edam_operation: http://edamontology.org/operation_3432
edam_topics:
- http://edamontology.org/topic_3366
tools:
- name: scTriangulate
  role: 'Core framework: ScTriangulate + lazy_run performs Shapley-based mix-and-match
    of conflicting annotations with stability metrics and pruning (win_fraction_cutoff,
    abs_thresh)'
  repo: https://github.com/frankligy/scTriangulate
- name: scanpy 1.7.2
  role: Preprocessing wrapper (scanpy_recipe) for QC, normalization to log CPTT, and
    UMAP generation feeding the triangulation
  repo: https://github.com/frankligy/scTriangulate
- name: cellHarmony
  role: Supervised two-step label transfer (MarkerFinder top-60 marker centroids,
    Pearson rho > 0.2, self-reclassification) producing reference annotations that
    serve as triangulation inputs
- name: ClusterEnsembles
  role: Ensemble-clustering baseline (default options) against which scTriangulate's
    agreement with independent references is compared
  repo: https://github.com/tsano430/ClusterEnsembles
- name: scikit-learn
  role: Computes Homogeneity, Completeness, and V-Measure agreement of the triangulated
    solution with HLCA/LungMAP references
provenance:
  source_task_ids:
  - task_004
  - task_005
  source_papers:
  - doi: 10.1038/s41467-023-36016-y
    title: Decision level integration of unimodal and multimodal single cell data
      with scTriangulate
  source_content_hash: sha256:2658ba051c1afc5273d86fec7f7c863e9189d1285c15a13d1f48c524b8b72a30
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/frankligy/scTriangulate
  tools:
  - ClusterEnsembles
  - cellHarmony
  - scTriangulate
  - scanpy
  - scanpy 1.7.2
  - scikit-learn
  edam_operation: http://edamontology.org/operation_3432
  edam_topics:
  - http://edamontology.org/topic_3366
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9
n_claims: 10
---

# single-cell-cluster-stability-integration

## Summary

A decision-level integration approach (scTriangulate) that mixes and matches conflicting single-cell clustering results from multiple algorithms, modalities, resolutions, and reference annotations using the Shapley Value from coalitional game theory plus cluster stability metrics (reassign, TFIDF5/TFIDF10, SCCAF), rather than selecting an ensemble consensus.

## When to use

Use when an AnnData object carries ≥2 conflicting cell-cluster annotation columns (e.g. Leiden at different resolutions, Seurat3/Monocle3/ICGS2, cellHarmony label transfers, or RNA/ADT modality labels) and a single high-confidence reconciled assignment is needed.

## When NOT to use

- When a single well-justified clustering resolution already resolves all cell states of interest — scTriangulate addresses the case where no single resolution identifies the most discrete cell states across modalities
- When the goal is a joint latent-space or graph-based multimodal embedding (e.g. Seurat-WNN or TotalVI) rather than reconciling pre-computed cluster annotations at the decision level
- When cell populations still need to be named: population naming remains largely non-standardized and is an explicitly unmet challenge of this approach

## Inputs

- AnnData (.h5ad) with adata.X normalized to log CPTT (RNA) or CLR (ADT), no adata.raw attribute
- ≥2 conflicting cluster-annotation columns in adata.obs (query columns) from unsupervised clustering, supervised label transfer (e.g. cellHarmony), or independent atlases
- adata.obsm['X_umap'] for automatic visualization
- Raw UMI count matrices for normalization (e.g. via scTriangulate.preprocessing.scanpy_recipe)

## Outputs

- Final mix-and-match cluster assignment per cell (stability-aware reconciled annotation)
- Confidence/stability scores per cluster (winning fraction, reassign/TFIDF/SCCAF metrics)
- UMAP visualizations colored by triangulated annotations
- Agreement metric tables (Homogeneity, Completeness, V-Measure) against independent references, e.g. lung_atlas_agreement_metrics.csv

## How to apply

First attach each conflicting annotation and UMAP coordinates to the AnnData via scTriangulate.preprocessing.add_annotation and add_umap, ensure adata.X is properly normalized (log CPTT for RNA, CLR for ADT) and adata.raw is deleted, then instantiate ScTriangulate with the query column names and run lazy_run(). The method treats each annotation set as a 'player' and applies the Shapley Value with embedded stability scores — the reassign score, TFIDF5/TFIDF10, and SCCAF — to mix-and-match clusters at the decision level (no joint latent space or geometric graph). Pruning applies a winning-fraction cutoff (win_fraction_cutoff; default wfc < 0.25 labeled unstable; 0.4 used for the four-lung-atlas case, 0.3 for the bone-marrow atlas, with add_metrics='tfidf5') plus a minimum cluster size (abs_thresh, default 10), after which pruned cells are reclassified by nearest neighbor. When more than 15 annotation sets are supplied, the program automatically substitutes a combined rank-based selection for Shapley selection to reduce runtime; results should be benchmarked against an ensemble-clustering baseline (e.g. ClusterEnsembles) or independent references such as the HLCA/Azimuth human-lung-v2 and LungMAP CellRef using Homogeneity, Completeness, and V-Measure (sklearn conventions).

## Related tools

- **scTriangulate** (Core framework: ScTriangulate + lazy_run performs Shapley-based mix-and-match of conflicting annotations with stability metrics and pruning (win_fraction_cutoff, abs_thresh)) — https://github.com/frankligy/scTriangulate
- **scanpy 1.7.2** (Preprocessing wrapper (scanpy_recipe) for QC, normalization to log CPTT, and UMAP generation feeding the triangulation) — https://github.com/frankligy/scTriangulate
- **cellHarmony** (Supervised two-step label transfer (MarkerFinder top-60 marker centroids, Pearson rho > 0.2, self-reclassification) producing reference annotations that serve as triangulation inputs)
- **ClusterEnsembles** (Ensemble-clustering baseline (default options) against which scTriangulate's agreement with independent references is compared) — https://github.com/tsano430/ClusterEnsembles
- **scikit-learn** (Computes Homogeneity, Completeness, and V-Measure agreement of the triangulated solution with HLCA/LungMAP references)

## Examples

```
sctri = ScTriangulate(dir='./output', adata=adata, query=['sctri_rna_leiden_1','sctri_rna_leiden_2','sctri_rna_leiden_3']); sctri.lazy_run()
```

## Evaluation signals

- V-Measure values in the output table must satisfy V = 2*h*c/(h+c) and all three metrics (Homogeneity, Completeness, V-Measure) must lie in [0,1]
- scTriangulate's agreement scores against independent references (Azimuth CellRef, HLCA) should exceed the ensemble-clustering (ClusterEnsembles) baseline
- All input annotation sets must be consumed without error (e.g. ICGS2, Monocle3, Seurat3 on the bone-marrow atlas), and rare transitional states such as HSC-cycle, MEP, and LMPP should be identified
- Clusters with winning fraction below the cutoff (default wfc < 0.25) must be flagged as unstable and pruned cells reclassified by nearest neighbor
- All metrics/scores fall in expected ranges and the final assignment assigns exactly one annotation per cell (binary I*N assignment matrix)

## Limitations

- Integration of cluster predictions from dozens of sources (resolutions, modalities, reference annotations) significantly increases runtime; over 15 annotation sets triggers rank-based selection instead of Shapley
- Programmatic default analytical settings (e.g. win_fraction_cutoff, abs_thresh=10) may not be optimal for all study designs
- Population naming remains largely non-standardized, an unmet challenge of such analyses

## Evidence

- [abstract] Shapley-based mix-and-match rationale: "Rather than ensemble approaches which select the "consensus", scTriangulate picks the most stable solution through coalitional iteration."
- [intro] Decision-level integration: "While other multimodal methods integrate at the data level (through either a low-dimensional latent space6 or through a geometric graph5), scTriangulate integrates results at a decision level to"
- [methods] Versioned preprocessing stack: "All the preprocessing and QC steps were conducted using scTriangulate version 0.9.0 and scanpy version 1.7.2."
- [methods] Pruning cutoff and instability rule: "By default, wfc < 0.25 will be labeled as unstable clusters."
- [methods] Adding prior annotations to AnnData: "The obtained annotations and UMAP coordinates24 were added to the existing AnnData by the scTriangulate.preprocessing.add_annotation and scTriangulate.preprocessing.add_umap functions"
- [readme] AnnData prerequisites in README: [evidence span withheld — anchor status: external_doc]
- [readme] AnnData prerequisite for marker genes: [evidence span withheld — anchor status: external_doc]
