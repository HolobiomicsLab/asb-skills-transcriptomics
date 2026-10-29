---
name: reference-label-transfer-evaluation
description: Use when prior reference-annotations (e.g., Azimuth label transfer, Seurat
  label transfer, cellHarmony) must be reconciled with Leiden clusters from one or
  more modalities, or when benchmarking clustering solutions against a reference atlas
  with Homogeneity/Completeness/V-Measure.
when_to_use_negative:
- When a joint latent-space integration is required (e.g., building a continuous multimodal
  embedding for trajectory analysis) — scTriangulate integrates at the decision level
  and makes no use of joint embeddings
- When only a single clustering solution with no conflicting annotations or reference
  labels exists — there is nothing to mix-and-match and Shapley-based reconciliation
  adds no value
- When integrating multiple samples of the same modality or across technologies is
  the goal — this support is planned but not yet available
edam_operation: http://edamontology.org/operation_0337
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: scTriangulate
  role: 'Core engine: add_annotation/add_umap preprocessing, Shapley-value decision-level
    reconciliation of reference and clustering labels, stability scoring'
  repo: https://github.com/frankligy/scTriangulate
- name: scanpy
  role: Generates per-modality normalized AnnData, Leiden clusterings, and UMAP coordinates
    consumed by scTriangulate (scanpy_recipe)
- name: ClusterEnsembles
  role: Baseline ensemble comparison (HGPA, MCLA, HBGF) against which reference-reconciled
    scTriangulate solutions are evaluated
  repo: https://github.com/tsano430/ClusterEnsembles
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1038/s41467-023-36016-y
    title: Decision level integration of unimodal and multimodal single cell data
      with scTriangulate
  source_content_hash: sha256:0a39796e6a6e850444bb0ac2d0d6db8a6a10629c1d182ee8b0c4bcb1cdf27ed8
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/frankligy/scTriangulate
  tools:
  - ClusterEnsembles
  - TotalVI
  - scTriangulate
  - scanpy
  edam_operation: http://edamontology.org/operation_0337
  edam_topics:
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9285714285714286
n_claims: 14
---

# reference-label-transfer-evaluation

## Summary

A scTriangulate-based decision-level strategy for integrating and evaluating reference-derived cell labels (e.g., Azimuth PBMC level-2 transfer) against unsupervised clustering results across modalities, scoring cluster stability and agreement rather than merging latent spaces.

## When to use

Use when prior reference-annotations (e.g., Azimuth label transfer, Seurat label transfer, cellHarmony) must be reconciled with Leiden clusters from one or more modalities, or when benchmarking clustering solutions against a reference atlas with Homogeneity/Completeness/V-Measure.

## When NOT to use

- When a joint latent-space integration is required (e.g., building a continuous multimodal embedding for trajectory analysis) — scTriangulate integrates at the decision level and makes no use of joint embeddings
- When only a single clustering solution with no conflicting annotations or reference labels exists — there is nothing to mix-and-match and Shapley-based reconciliation adds no value
- When integrating multiple samples of the same modality or across technologies is the goal — this support is planned but not yet available

## Inputs

- AnnData (adata.X) with properly normalized values (log CPTT for RNA, CLR for ADT) per modality
- adata.obs columns holding conflicting annotations (unsupervised Leiden labels and reference-derived label-transfer annotations)
- adata.obsm['X_umap'] coordinates for automatic visualization (no adata.raw attribute present)
- Reference atlas annotations (e.g., Azimuth human-pbmc level-2 labels derived from post-QC raw UMI counts)

## Outputs

- Final per-cell population assignments reconciling reference and clustering solutions
- Per-annotation winning-fraction and stability metrics (reassign, TFIDF5, TFIDF10, SCCAF scores)
- UMAP visualizations of the reconciled annotations written to the specified output directory
- Homogeneity, Completeness, and V-Measure table comparing all solutions against the Azimuth L2 reference

## How to apply

Load each modality as a normalized AnnData (log-CPTT with top-3000 HVGs and 15 PCs for RNA; log-CPTT with top-60000 peaks and 100 PCs for ATAC; CLR-normalized ADT with 15 PCs), run per-modality Leiden at resolutions 1, 2, 3, then merge via scTriangulate.preprocessing.concat_rna_and_other. Add reference-derived labels and UMAP coordinates to the existing AnnData with scTriangulate.preprocessing.add_annotation and scTriangulate.preprocessing.add_umap, then instantiate ScTriangulate with the reference annotation and the conflicting Leiden solutions passed to the query argument. The Shapley value from coalitional game theory, aided by reassign, TFIDF5, TFIDF10, and SCCAF stability scores, mixes-and-matches clusters (including reference labels) into a single per-cell assignment; final assignments with winning fraction < 0.25 (win_fraction_cutoff default) are pruned as unstable and small clusters (< abs_thresh=10 cells) reclassified by nearest neighbors. Rationale: scTriangulate integrates at the decision level so it can jointly consider prior label-transfer annotations, which joint-embedding methods like Seurat-WNN and TotalVI cannot. Evaluate by computing Homogeneity, Completeness, and V-Measure of every solution against the reference (e.g., Azimuth PBMC L2) and checking recovery of rare populations.

## Related tools

- **scTriangulate** (Core engine: add_annotation/add_umap preprocessing, Shapley-value decision-level reconciliation of reference and clustering labels, stability scoring) — https://github.com/frankligy/scTriangulate
- **scanpy** (Generates per-modality normalized AnnData, Leiden clusterings, and UMAP coordinates consumed by scTriangulate (scanpy_recipe))
- **ClusterEnsembles** (Baseline ensemble comparison (HGPA, MCLA, HBGF) against which reference-reconciled scTriangulate solutions are evaluated) — https://github.com/tsano430/ClusterEnsembles

## Examples

```
sctri = ScTriangulate(dir='./output', adata=adata, query=['azimuth_l2_ref', 'sctri_rna_leiden_1', 'sctri_rna_leiden_2', 'sctri_rna_leiden_3']); sctri.lazy_run()
```

## Evaluation signals

- scTriangulate attains the highest V-Measure (plus Homogeneity/Completeness) relative to Seurat-WNN, TotalVI, and HGPA/MCLA/HBGF baselines against the Azimuth L2 reference
- Rare populations (NK CD56+ bright, B-memory, HSCP) are recovered in the final annotation set
- Cells with winning fraction < 0.25 are flagged as unstable clusters rather than silently assigned
- Final clusters contain at least the minimum cluster size (default abs_thresh = 10 cells); smaller clusters are reclassified via nearest neighbors
- Output directory contains the reconciled annotation matrix and UMAP plots; adata.raw was deleted beforehand so marker-gene calculation is not interfered with

## Limitations

- With more than 15 annotation sets, Shapley selection is automatically substituted with a combined rank-based selection, which changes the reconciliation behavior
- Integration of predictions from dozens of sources (resolutions, modalities, reference annotations) significantly increases runtime
- Programmatic default analytical settings (e.g., win_fraction_cutoff 0.25, abs_thresh 10) may not be optimal for all study designs
- Population naming remains largely non-standardized; reference-based labels do not resolve this naming challenge

## Evidence

- [intro] Decision-level vs data-level integration rationale: "While other multimodal methods integrate at the data level (through either a low-dimensional latent space6 or through a geometric graph5), scTriangulate integrates results at a decision level"
- [methods] Adding reference annotations and UMAP to AnnData: "The obtained annotations and umap coordinates were added to the existing AnnData by scTriangulate.preprocessing.add_annotation and scTriangulate.preprocessing.add_umap functions"
- [methods] Embedded stability scores: "scTriangulate uses four embedded scores: the reassign score, the TFIDF10 score, the TFIDF5 score, and the SCCAF score."
- [methods] Unstable-cluster pruning threshold: "By default, wfc < 0.25 will be labeled as unstable clusters."
- [readme] Reference-atlas integration use case: "3. Integrate results from different reference atlases."
- [readme] Supervised label-transfer integration use case: "Integrate results from both unsupervised and supervised (i.e. cellHarmony, Seurat label transfer) clustering algorithms."
