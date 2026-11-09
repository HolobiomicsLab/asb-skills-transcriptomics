---
name: mononuclear-phagocyte-subpopulation-annotation
description: Use when snRNA-seq/scRNA-seq datasets carry conflicting myeloid/MNP annotations
  from independent studies or reference atlases (e.g. Melms/Adams/Travaglini/Wang
  lung labels, MNP-Verse, HLCA) and a high-confidence myeloid subpopulation assignment
  is needed.
when_to_use_negative:
- Only a single consistent myeloid annotation source is available — scTriangulate
  is designed to mix-and-match conflicting clustering results, so with one label there
  is nothing to reconcile
- Input is raw counts without prior annotations and the goal is de novo clustering
  — the workflow starts from scTriangulate only after Leiden/reference labels exist
  ('This step may also include running Leiden clustering sequentially through scTriangulate
  when no prior annotations exist')
- The cell populations of interest are outside the myeloid compartment and no compatible
  reference centroids (MNP-Verse, HLCA, CellRef) exist for them — label transfer quality
  depends on matched reference sets
edam_operation: http://edamontology.org/operation_3432
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: scTriangulate
  role: Decision-level integration of the conflicting MNP label transfers via Shapley
    value and stability metrics (TFIDF5, win_fraction_cutoff 0.4 pruning)
  repo: https://github.com/frankligy/scTriangulate
- name: cellHarmony
  role: 'Two-step supervised label transfer: MarkerFinder top-60 marker centroids
    (Pearson rho > 0.2) from non-diseased controls, then self-reclassification of
    the base dataset'
- name: scanpy
  role: Underlying AnnData framework; normalization to log CPTT and UMAP generation
    through sctriangulate.preprocessing.scanpy_recipe (scanpy 1.7.2)
- name: scikit-learn
  role: Computes Homogeneity, Completeness, and V-Measure of the reconciled MNP labels
    against HLCA/LungMAP reference assignments
- name: ICGS2
  role: MarkerFinder-based centroid generation used upstream in the cellHarmony two-step
    mapping of the lung atlases
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1038/s41467-023-36016-y
    title: Decision level integration of unimodal and multimodal single cell data
      with scTriangulate
  source_content_hash: sha256:12b8aac03ecde77bb6ad662e9f114e98f9b3faa655d5b9d3d35919cdb64b55ef
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/frankligy/scTriangulate
  tools:
  - ICGS2
  - cellHarmony
  - scTriangulate
  - scanpy
  - scikit-learn
  edam_operation: http://edamontology.org/operation_3432
  edam_topics:
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.75
n_claims: 12
---

# mononuclear-phagocyte-subpopulation-annotation

## Summary

Annotates and reconciles mononuclear phagocyte (MNP) subpopulations — dendritic cells, macrophages, and related myeloid states — by mapping query scRNA/snRNA data onto reference atlases and integrating conflicting label transfers at the decision level with scTriangulate, benchmarking against independent references (HLCA, LungMAP CellRef).

## When to use

Use when snRNA-seq/scRNA-seq datasets carry conflicting myeloid/MNP annotations from independent studies or reference atlases (e.g. Melms/Adams/Travaglini/Wang lung labels, MNP-Verse, HLCA) and a high-confidence myeloid subpopulation assignment is needed.

## When NOT to use

- Only a single consistent myeloid annotation source is available — scTriangulate is designed to mix-and-match conflicting clustering results, so with one label there is nothing to reconcile
- Input is raw counts without prior annotations and the goal is de novo clustering — the workflow starts from scTriangulate only after Leiden/reference labels exist ('This step may also include running Leiden clustering sequentially through scTriangulate when no prior annotations exist')
- The cell populations of interest are outside the myeloid compartment and no compatible reference centroids (MNP-Verse, HLCA, CellRef) exist for them — label transfer quality depends on matched reference sets

## Inputs

- Lung cell-population assignments and UMI count matrices from independent studies (GSE161383, GSE171524, GSE136831, EGAS00001004344)
- Wang et al. 9-donor snRNA-Seq dataset selected as the base AnnData
- HLCA reference (Azimuth human-lung-v2) and LungMAP CellRef 1.0 reference assignments
- MNP-Verse mononuclear phagocyte centroids from healthy controls

## Outputs

- Reconciled AnnData with a single high-confidence MNP/DC/macrophage label per cell plus UMAP coordinates
- lung_atlas_agreement_metrics.csv with Homogeneity, Completeness, and V-Measure against HLCA and LungMAP references
- Stability metrics (reassign, TFIDF5, SCCAF scores) and pruned cluster assignments from the scTriangulate run

## How to apply

Select a base dataset (e.g. the Wang 9-donor snRNA-Seq study, EGAS00001004344) and map each query atlas onto it using cellHarmony's two-step strategy: MarkerFinder top-60 marker centroids from non-diseased controls with Pearson correlation rho > 0.2, followed by self-reclassification of the base dataset. Apply the same two-step mapping to the HLCA (Azimuth human-lung-v2) and LungMAP CellRef 1.0 references, and compute Mononuclear phagocyte (MNP-Verse) centroids from healthy controls for the myeloid comparison. Attach all transferred annotations and UMAP coordinates to the base AnnData via scTriangulate.preprocessing.add_annotation and add_umap, normalize raw UMI counts to log CPTT, then run scTriangulate (v0.9.0 with scanpy 1.7.2) with win_fraction_cutoff = 0.4 and add_metrics = 'tfidf5' so each myeloid cell is assigned one label by Shapley-based coalitional iteration. Finally, quantify agreement of the reconciled MNP labels (including subdivided dendritic-cell and macrophage populations) against HLCA/LungMAP reference assignments using Homogeneity, Completeness, and V-Measure (sklearn conventions).

## Related tools

- **scTriangulate** (Decision-level integration of the conflicting MNP label transfers via Shapley value and stability metrics (TFIDF5, win_fraction_cutoff 0.4 pruning)) — https://github.com/frankligy/scTriangulate
- **cellHarmony** (Two-step supervised label transfer: MarkerFinder top-60 marker centroids (Pearson rho > 0.2) from non-diseased controls, then self-reclassification of the base dataset)
- **scanpy** (Underlying AnnData framework; normalization to log CPTT and UMAP generation through sctriangulate.preprocessing.scanpy_recipe (scanpy 1.7.2))
- **scikit-learn** (Computes Homogeneity, Completeness, and V-Measure of the reconciled MNP labels against HLCA/LungMAP reference assignments)
- **ICGS2** (MarkerFinder-based centroid generation used upstream in the cellHarmony two-step mapping of the lung atlases)

## Examples

```
sctri = ScTriangulate(dir='./output', adata=adata, query=['melms','adams','travaglini','wang']); sctri.lazy_run(win_fraction_cutoff=0.4, add_metrics='tfidf5')
```

## Evaluation signals

- V-Measure values in lung_atlas_agreement_metrics.csv must satisfy V = 2*h*c/(h+c) with the computed Homogeneity and Completeness
- All three agreement metrics must lie in [0,1]
- scTriangulate's scores should exceed the ensemble-clustering baseline (ClusterEnsembles) on HLCA/CellRef agreement, confirming the reported relationship
- Cells with winning fraction below win_fraction_cutoff (0.4 here; default 0.25) must be flagged as unstable rather than assigned a definite MNP label
- Base AnnData must contain the transferred annotation columns and adata.obsm['X_umap'] and must not carry adata.raw, per the README requirements

## Limitations

- Integration of cluster predictions from dozens of sources (resolutions, modalities, reference annotations) significantly increases runtime; Shapley selection is auto-replaced with rank-based selection when over 15 annotation sets are provided
- Programmatic default analytical settings may not be optimal for all study designs; win_fraction_cutoff and abs_thresh (minimum cluster size 10) are user-adjustable
- Naming of populations remains largely non-standardized, so reconciled MNP labels still require manual curation against a reference nomenclature
- By default, wfc < 0.25 will be labeled as unstable clusters; the 0.4 cutoff used here is more conservative and prunes more cells into unstable groups

## Evidence

- [methods] The same two-step mapping strategy was applied to HLCA (https://app.azimuth.hubmapconsortium.org/app/human-lung-v2): "The same two-step mapping strategy was applied to HLCA (https://app.azimuth.hubmapconsortium.org/app/human-lung-v2)"
- [methods] add_annotation/add_umap into AnnData: "The obtained annotations and umap coordinates were added to the existing AnnData by scTriangulate.preprocessing.add_annotation and scTriangulate.preprocessing.add_umap functions"
- [readme] scTriangulate reconciles conflicting labels decision-level: "scTriangulate is a Python package to mix-and-match conflicting clustering results in single cell analysis, and generate reconciled clustering solutions."
- [methods] embedded stability scores: "scTriangulate uses four embedded scores: the reassign score, the TFIDF10 score, the TFIDF5 score, and the SCCAF score."
- [discussion] reference-classification scope of the integration: "the integration of diverse cellular annotations from independent studies (i.e., reference-classification), multimodal measurements, distinct clustering algorithms, and software settings"
- [abstract] subdivision to discover subpopulations: "resolve clonal architecture within molecularly defined cell-populations and subdivide clusters to discover splicing-defined"
