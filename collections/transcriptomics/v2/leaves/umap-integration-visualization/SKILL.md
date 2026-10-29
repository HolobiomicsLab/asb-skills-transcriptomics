---
name: umap-integration-visualization
description: Use when you have raw and scMerge2-adjusted expression matrices from
  a multi-sample, multi-condition single-cell atlas (e.g. 20 COVID-19 studies, 1298
  samples, 963 donors) and must demonstrate integration quality via UMAP embedding
  plots and technical-label PCA metrics.
when_to_use_negative:
- Data has not been integrated (no adjusted expression matrix) — UMAP on raw data
  will show technical separation and cannot serve as integration evidence
- Analysis goal is joint multi-modal integration — the paper notes integration was
  only performed within each modality and multi-modal incorporation is deferred to
  future work
- The embedding alone is the deliverable for interpretable downstream features — the
  article warns an embedding output by itself increases interpretability challenges
  for features like cell–cell interactions or pathway information
edam_operation: http://edamontology.org/operation_0570
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: scMerge2
  role: Produces the scMerge2-adjusted expression matrix whose raw-vs-adjusted UMAP
    and technical-label PCA scores are visualized; installed via the scMerge Bioconductor
    package
  repo: https://github.com/SydneyBioX/scMerge
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1038/s41467-023-39923-2
    title: Atlas-scale single-cell multi-sample multi-condition data integration using
      scMerge2
  source_content_hash: sha256:f4c33da899cbc6d6f7a896eb1862436a9bf10c43a09da5ca7556e6ba55cdca37
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/SydneyBioX/scMerge
  tools:
  - scMerge2
  edam_operation: http://edamontology.org/operation_0570
  edam_topics:
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 6
---

# umap-integration-visualization

## Summary

Visualize atlas-scale scMerge2-integrated single-cell data by computing PCA scores on technical labels (dataset, protocol, technology) and generating UMAP integration plots, as done for the ~5M-cell COVID-19 PBMC collection in Fig. 3a-b, to confirm reduced technical variation with preserved biological cell-state separation.

## When to use

Use when you have raw and scMerge2-adjusted expression matrices from a multi-sample, multi-condition single-cell atlas (e.g. 20 COVID-19 studies, 1298 samples, 963 donors) and must demonstrate integration quality via UMAP embedding plots and technical-label PCA metrics.

## When NOT to use

- Data has not been integrated (no adjusted expression matrix) — UMAP on raw data will show technical separation and cannot serve as integration evidence
- Analysis goal is joint multi-modal integration — the paper notes integration was only performed within each modality and multi-modal incorporation is deferred to future work
- The embedding alone is the deliverable for interpretable downstream features — the article warns an embedding output by itself increases interpretability challenges for features like cell–cell interactions or pathway information

## Inputs

- raw (pre-integration) single-cell expression matrix from a multi-study atlas (e.g. SingleCellExperiment objects from 20 COVID-19 studies)
- scMerge2-adjusted expression matrix produced by hierarchical integration
- cell-level metadata with technical labels (dataset/study, protocol, technology) and biological labels (cell type, COVID-19 severity)

## Outputs

- UMAP integration plots comparing raw and scMerge2-adjusted data (Fig. 3a-b style)
- PCA scores computed on technical labels (dataset, protocol, technology) for raw and adjusted data
- raw-vs-scMerge2 metrics table quantifying reduction of technical variation

## How to apply

After running hierarchical scMerge2 (with pseudo-bulk construction and pseudo-replication within phenotype/condition groups), compute PCA on cells colored or stratified by technical labels — dataset, protocol and technology — for both raw and adjusted expression matrices. Generate UMAP integration plots in the style of Fig. 3a-b, comparing raw versus scMerge2-adjusted embeddings of the ~5M-cell COVID-19 collection. Accept the integration when technical-label PCA separation diminishes post-integration while biological discrimination of COVID-19 severity levels is preserved. The comparison table of raw-vs-scMerge2 metrics is the quantitative validation artefact; UMAP plots are the visual companion evidence.

## Related tools

- **scMerge2** (Produces the scMerge2-adjusted expression matrix whose raw-vs-adjusted UMAP and technical-label PCA scores are visualized; installed via the scMerge Bioconductor package) — https://github.com/SydneyBioX/scMerge

## Examples

```
BiocManager::install("scMerge"); library(scMerge); data(segList)
```

## Evaluation signals

- Technical-label PCA separation (dataset, protocol, technology) is visibly diminished in the scMerge2-adjusted data compared to raw data
- Biological cell-state discrimination across COVID-19 severity levels is preserved in the adjusted-data UMAP
- UMAP integration plots reproduce the qualitative pattern of Fig. 3a-b (raw vs adjusted comparison)
- The raw-vs-scMerge2 metrics table shows reduced technical variation while retaining biological signal
- Visualization is computed on the full atlas output of hierarchical integration (~5 million cells, 1298 samples, 20 studies), not a subsample inconsistent with the reported metrics

## Limitations

- Integration (and hence these evaluation visuals) was performed within each modality only; multi-modal information is deferred to future work
- The current integration order is data-guided (batches within one study or studies of similar size first); a knowledge-guided order is described as an unused alternative
- Embedding-only outputs increase interpretability challenges for downstream features such as cell–cell interactions or pathway information — pair UMAP evidence with the adjusted expression matrix

## Evidence

- [abstract] Atlas-scale COVID-19 integration demonstrated on >5M cells: "Using a large COVID-19 data collection with over five million cells from 1000+ individuals, we demonstrate that scMerge2 enables multi-sample multi-condition scRNA-seq data integration from"
- [intro] Downstream utility of integration for severity discrimination: "the integration using scMerge2 improves the performance of discriminating distinct cell states in COVID-19 patients with varying degrees of severity"
- [discussion] Three innovations underpinning the integrated data being visualized: "This was achieved via the use of three essential innovations with hierarchical integration, pseudo-bulk building to minimise processing demand, and pseudo-replication"
- [discussion] Embedding-only interpretability caveat: "an embedding output by itself increases interpretability challenges since a low-dimensional representation does not naturally lend itself to the development of interpretable features"
- [readme] Package installation path for scMerge/scMerge2 tooling: [evidence span withheld — anchor status: not_found]
