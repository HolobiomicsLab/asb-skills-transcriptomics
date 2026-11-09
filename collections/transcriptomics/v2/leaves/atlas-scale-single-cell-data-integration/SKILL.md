---
name: atlas-scale-single-cell-data-integration
description: Use when integrating thousands of samples and millions of cells from
  multiple studies/technologies (scRNA-seq, CyTOF, CITE-seq) where reduced technical
  variation AND an adjusted expression matrix for downstream sample-level analysis
  are required.
when_to_use_negative:
- Joint multi-modal integration is required — the paper's integration was done within
  each modality only (scRNA-seq, CyTOF, imaging mass cytometry, CITE-seq separately)
- A knowledge-guided integration order is required rather than the data-guided order
  (batches within one study or studies of similar size first)
- Downstream analyses rely on interpretable features like cell–cell interactions or
  pathway information and only an embedding output would be returned
edam_topics:
- http://edamontology.org/topic_3170
tools:
- name: scMerge
  role: R/Bioconductor package providing scMerge2 for merging and normalising single-cell
    RNA-seq datasets; performs the hierarchical integration and returns adjusted expression
    matrices
  repo: https://github.com/SydneyBioX/scMerge
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1038/s41467-023-39923-2
    title: Atlas-scale single-cell multi-sample multi-condition data integration using
      scMerge2
  source_content_hash: sha256:98847eab3fb123bbd428ed6359a6d8bcc0e767c9370d01e86d04f8cb982c1e21
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/SydneyBioX/scMerge
  tools:
  - scMerge
  - scMerge2
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9
n_claims: 10
---

# atlas-scale-single-cell-data-integration

## Summary

scMerge2-based integration of atlas-scale multi-sample multi-condition single-cell studies (e.g. ~5 million COVID-19 PBMC cells from 1298 samples across 20 studies) via hierarchical integration, pseudo-bulk building, and pseudo-replication, producing adjusted expression matrices rather than embeddings only.

## When to use

Use when integrating thousands of samples and millions of cells from multiple studies/technologies (scRNA-seq, CyTOF, CITE-seq) where reduced technical variation AND an adjusted expression matrix for downstream sample-level analysis are required.

## When NOT to use

- Joint multi-modal integration is required — the paper's integration was done within each modality only (scRNA-seq, CyTOF, imaging mass cytometry, CITE-seq separately)
- A knowledge-guided integration order is required rather than the data-guided order (batches within one study or studies of similar size first)
- Downstream analyses rely on interpretable features like cell–cell interactions or pathway information and only an embedding output would be returned

## Inputs

- Multi-study single-cell count matrices (scRNA-seq, CyTOF, CITE-seq) with sample/study/condition metadata (e.g. ~5M cells, 1298 samples, 963 donors, 20 studies)
- Refined cell-type annotations used as pseudo-replicate information (e.g. scClassify-refined labels)
- Technical labels (dataset, protocol, technology) and phenotype/condition groupings for pseudo-replicate assignment and evaluation

## Outputs

- Adjusted (integrated) gene expression matrix for the full collection
- Pseudo-bulk profiles per annotated cell type per sample
- Pseudo-replicate assignments within phenotype/condition groups
- PCA scores on technical labels and UMAP integration plots for raw vs adjusted data

## How to apply

Load the multi-study collection with refined cell-type annotations, then build pseudo-bulk profiles per annotated cell type within each sample to minimise processing demand of atlas-scale data. Assign pseudo-replicates within phenotype/condition groups so pseudo-replication accounts for circumstances with phenotypes. Run hierarchical scMerge2 integration in a data-guided order — batches within one study or studies of similar size first — to produce the adjusted expression matrix for the full collection. Validate by computing PCA scores on technical labels (dataset, protocol, technology) for raw vs scMerge2-adjusted data: accept integration when technical-label PCA separation diminishes post-integration (as in Fig. 3a-b) while biological discrimination of COVID-19 severity levels is preserved. scMerge2 demonstrated this on a COVID-19 collection of five million cells from 20 studies within a day.

## Related tools

- **scMerge** (R/Bioconductor package providing scMerge2 for merging and normalising single-cell RNA-seq datasets; performs the hierarchical integration and returns adjusted expression matrices) — https://github.com/SydneyBioX/scMerge

## Examples

```
BiocManager::install("scMerge"); library(scMerge); data(segList); head(segList$human$human_scSEG)
```

## Evaluation signals

- Raw-vs-scMerge2 metrics table shows reduced technical variation caused by dataset, protocol and technology
- PCA separation on technical labels (dataset, protocol, technology) is diminished post-integration relative to raw data
- Biological cell-state discrimination across COVID-19 severity levels is preserved after integration
- Improved cell type identification in the integrated COVID-19 collection
- Atlas-scale runtimes remain tractable (five million cells from 20 studies integrated within a day)

## Limitations

- Multi-modal integration was only performed within each modality; joint use of multi-modal information is deferred to future work
- The current integration order is data-guided only (batches within one study or studies of similar size first); a knowledge-guided order is described as an alternative not used
- The concatenating integration approach is more computationally intensive than the balanced tree approach
- Embedding-only outputs (as from rapid methods) increase interpretability challenges for downstream features such as cell–cell interactions or pathway information

## Evidence

- [abstract] Here, we present scMerge2, a scalable algorithm that allows data integration of atlas-scale multi-sample multi-condition single-cell studies.: "Here, we present scMerge2, a scalable algorithm that allows data integration of atlas-scale multi-sample multi-condition single-cell studies."
- [discussion] Three essential innovations of scMerge2: "This was achieved via the use of three essential innovations with hierarchical integration, pseudo-bulk building to minimise processing demand, and pseudo-replication"
- [intro] scMerge2 runtime and scale on COVID-19 collection: "scMerge2 is able to integrate five million cells from a large COVID-19 data collection with over 1000 samples from 20 studies globally within a day"
- [readme] README role of the scMerge package: "`scMerge` is a R package for merging and normalising single-cell RNA-Seq datasets."
