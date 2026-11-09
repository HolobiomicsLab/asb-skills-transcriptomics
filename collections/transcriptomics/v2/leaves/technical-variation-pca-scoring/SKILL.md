---
name: technical-variation-pca-scoring
description: Use when evaluating whether a batch-correction/integration method (e.g.
  scMerge2) actually reduced technical variation across datasets, protocols or technologies,
  or when comparing raw vs adjusted matrices for label-mixing in PCA space.
when_to_use_negative:
- The goal is joint multi-modal integration — the scMerge2 paper states integration
  was performed within each modality only, deferring multi-modal incorporation to
  future work
- The input is a low-dimensional embedding without an adjusted expression matrix —
  the paper notes lack of adjusted expression matrices restricts sample-level analysis
  and interpretability
- Data is small with a single batch/protocol, where technical-variation scoring across
  datasets/protocols/technologies has nothing to separate
edam_topics:
- http://edamontology.org/topic_0091
tools:
- name: scMerge2
  role: Produces the adjusted expression matrix whose raw-vs-adjusted PCA scores are
    compared to quantify technical-variation reduction
  repo: https://github.com/SydneyBioX/scMerge
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1038/s41467-023-39923-2
    title: Atlas-scale single-cell multi-sample multi-condition data integration using
      scMerge2
  source_content_hash: sha256:6569b35d62417ae295687e8a0a3a04b05682ee9b8c5d726842696735c7d8a6aa
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/SydneyBioX/scMerge
  tools:
  - scMerge2
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.875
n_claims: 8
---

# technical-variation-pca-scoring

## Summary

Quantifies technical variation in single-cell data integration by computing PCA scores coloured by technical labels (dataset, protocol, technology) on both raw and integrated (e.g. scMerge2-adjusted) expression matrices, as used to evaluate the ~5M-cell COVID-19 PBMC collection (1298 samples, 20 studies).

## When to use

Use when evaluating whether a batch-correction/integration method (e.g. scMerge2) actually reduced technical variation across datasets, protocols or technologies, or when comparing raw vs adjusted matrices for label-mixing in PCA space.

## When NOT to use

- The goal is joint multi-modal integration — the scMerge2 paper states integration was performed within each modality only, deferring multi-modal incorporation to future work
- The input is a low-dimensional embedding without an adjusted expression matrix — the paper notes lack of adjusted expression matrices restricts sample-level analysis and interpretability
- Data is small with a single batch/protocol, where technical-variation scoring across datasets/protocols/technologies has nothing to separate

## Inputs

- Raw single-cell expression matrix (e.g. SingleCellExperiment with counts/logcounts)
- Integrated/adjusted expression matrix from scMerge2 (adjusted assay)
- Per-cell metadata with technical labels: dataset/study, protocol, technology (scRNA-seq, CyTOF, imaging mass cytometry, CITE-seq)
- Biological labels (cell types, COVID-19 severity) for preservation checks

## Outputs

- PCA scores for raw and adjusted data coloured by technical labels (dataset, protocol, technology)
- Raw-vs-adjusted technical-variation metrics table showing diminished PCA separation
- UMAP integration plots (analogous to Fig. 3a-b)

## How to apply

Take the raw single-cell expression matrix (e.g. SingleCellExperiment counts/logcounts) and the integration-adjusted matrix produced by scMerge2, plus per-cell technical metadata (study/dataset, protocol, technology such as scRNA-seq, CyTOF, imaging mass cytometry, CITE-seq). Run PCA (e.g. prcomp in R) separately on raw and adjusted data and score how strongly cells separate by each technical label in the leading PCs. Generate UMAP integration plots analogous to Fig. 3a-b of the scMerge2 paper. Accept the integration when technical-label PCA separation diminishes post-adjustment while biological discrimination (e.g. COVID-19 severity levels, cell-type expression signatures) is preserved. The reduction in separation is the quantitative evidence that scMerge2 'reduces the technical variation caused by dataset, protocol and technology'.

## Related tools

- **scMerge2** (Produces the adjusted expression matrix whose raw-vs-adjusted PCA scores are compared to quantify technical-variation reduction) — https://github.com/SydneyBioX/scMerge

## Examples

```
pca_raw <- prcomp(t(logcounts(sce_raw))); pca_adj <- prcomp(t(assay(sce_adj, "scMerge2"))); plot(pca_adj$x[,1], pca_adj$x[,2], col = sce_adj$technology)
```

## Evaluation signals

- Technical-label PCA separation (dataset, protocol, technology) is visibly diminished in the scMerge2-adjusted data compared to raw data
- Biological cell-state discrimination across COVID-19 severity levels is preserved post-integration
- Cell-type identification in the integrated COVID-19 collection improves relative to raw data
- The raw-vs-scMerge2 metrics table confirms reduced technical variation before acceptance of the integration
- Adjustment succeeds across modalities: dataset variability is removed in CyTOF, imaging mass cytometry and CITE-seq experiments

## Limitations

- Integration is data-guided only (batches within one study or similarly sized studies first); knowledge-guided ordering is described as an unused alternative
- Embedding-only outputs increase interpretability challenges for downstream features such as cell–cell interactions or pathway information
- The concatenating integration approach is more computationally intensive than the balanced tree approach

## Evidence

- [abstract] scMerge2 removes dataset variability across modalities: "we demonstrate that scMerge2 can remove dataset variability in CyTOF, imaging mass cytometry and CITE-seq experiments"
- [discussion] Three innovations behind the integration being scored: "This was achieved via the use of three essential innovations with hierarchical integration, pseudo-bulk building to minimise processing demand, and pseudo-replication"
- [intro] Multi-technology scope of the integration evaluated: "Our scMerge2 algorithm is able to integrate many millions of cells from single-cell studies generated from various single-cell technologies, including scRNA-seq, CyTOF, and imaging mass cytometry"
- [readme] Tool source for the adjusted matrices: [evidence span withheld — anchor status: not_found]
