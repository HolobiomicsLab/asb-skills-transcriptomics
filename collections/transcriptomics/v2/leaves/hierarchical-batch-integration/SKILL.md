---
name: hierarchical-batch-integration
description: Use when integrating atlas-scale scRNA-seq collections (many studies,
  thousands of samples, millions of cells) where rapid methods like Harmony or scVI
  return only embeddings, and an adjusted expression matrix plus preserved biological
  condition signal is required.
when_to_use_negative:
- When only a low-dimensional joint embedding is needed and adjusted expression is
  not required — embedding-focused methods (deepMNN, BBKNN, Harmony, scVI, scANVI,
  DESC) are faster alternatives
- When joint cross-modality integration is required — scMerge2 integration as described
  was performed within each modality (scRNA-seq, CyTOF, CITE-seq, imaging mass cytometry
  separately), with multi-modal incorporation deferred to future work
edam_operation: http://edamontology.org/operation_3435
edam_topics:
- http://edamontology.org/topic_3170
tools:
- name: scMerge2
  role: Scalable algorithm performing the hierarchical integration, pseudo-bulk construction,
    and pseudo-replication to produce the adjusted expression matrix
  repo: https://github.com/SydneyBioX/scMerge
- name: scMerge
  role: Bioconductor R package providing scMerge2 vignette and the SEG (stably expressed
    genes) lists used for merging single-cell datasets
  repo: https://github.com/SydneyBioX/scMerge
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1038/s41467-023-39923-2
    title: Atlas-scale single-cell multi-sample multi-condition data integration using
      scMerge2
  source_content_hash: sha256:c3561338fc29faf61df3ac5615e19944af9ea60d8c48c01d85ff5fd77236f4a1
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/SydneyBioX/scMerge
  tools:
  - scMerge
  - scMerge2
  edam_operation: http://edamontology.org/operation_3435
  edam_topics:
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7777777777777778
n_claims: 9
---

# hierarchical-batch-integration

## Summary

Atlas-scale integration of multi-sample, multi-condition single-cell data via scMerge2's hierarchical integration: pseudo-bulk construction minimises processing demand, pseudo-replication within phenotype/condition groups supplies the merging design, and batches/studies are merged in a data-guided order to yield an adjusted expression matrix.

## When to use

Use when integrating atlas-scale scRNA-seq collections (many studies, thousands of samples, millions of cells) where rapid methods like Harmony or scVI return only embeddings, and an adjusted expression matrix plus preserved biological condition signal is required.

## When NOT to use

- When only a low-dimensional joint embedding is needed and adjusted expression is not required — embedding-focused methods (deepMNN, BBKNN, Harmony, scVI, scANVI, DESC) are faster alternatives
- When joint cross-modality integration is required — scMerge2 integration as described was performed within each modality (scRNA-seq, CyTOF, CITE-seq, imaging mass cytometry separately), with multi-modal incorporation deferred to future work

## Inputs

- Multi-study scRNA-seq collection (e.g. ~5M cells, 20 studies, 963 donors, 1298 samples), e.g. as SingleCellExperiment objects
- Cell-type annotations per cell (pseudo-replicate information; scClassify-refined annotations used in the COVID-19 PBMC collection)
- Technical/covariate labels per sample: dataset, protocol, technology, condition/severity

## Outputs

- Adjusted (batch-corrected) expression matrix for the full collection
- PCA scores on technical labels for raw vs integrated data; UMAP integration plots
- Downstream signatures derived from cell-type expression for disease-progression discrimination

## How to apply

Load the multi-study collection with cell-type annotations (refined via scClassify where available) as pseudo-replicate information. Build pseudo-bulk profiles per annotated cell type within each sample — this is one of scMerge2's three key innovations and minimises the processing demand of atlas-scale data. Assign pseudo-replicates within phenotype/condition groups so pseudo-replication accounts for circumstances with phenotypes. Run scMerge2 hierarchically in a data-guided order (batches within one study, or studies of similar size, first) to produce the adjusted expression matrix for the full collection; note the alternative knowledge-guided order and that the concatenating approach is more computationally intensive than the balanced tree approach. Validate by comparing PCA scores on technical labels (dataset, protocol, technology) before and after integration, plus UMAP integration plots, accepting only if technical separation is diminished while biological cell-state discrimination (e.g. COVID-19 severity) is preserved.

## Related tools

- **scMerge2** (Scalable algorithm performing the hierarchical integration, pseudo-bulk construction, and pseudo-replication to produce the adjusted expression matrix) — https://github.com/SydneyBioX/scMerge
- **scMerge** (Bioconductor R package providing scMerge2 vignette and the SEG (stably expressed genes) lists used for merging single-cell datasets) — https://github.com/SydneyBioX/scMerge

## Examples

```
BiocManager::install("scMerge"); library(scMerge); vignettes: https://sydneybiox.github.io/scMerge/articles/scMerge2.html
```

## Evaluation signals

- Technical-label PCA separation (dataset, protocol, technology) is diminished in scMerge2-adjusted data relative to raw data
- Biological cell-state discrimination across COVID-19 severity levels is preserved post-integration
- Integration of the 5M-cell / 1298-sample COVID-19 collection completes within a day, demonstrating scalability
- Adjusted expression matrix (not just an embedding) is returned for downstream sample-level analysis
- UMAP integration plots show mixing by technical labels consistent with Fig. 3a-b of the paper

## Limitations

- Integration order is data-guided only (batches within one study or studies of similar size first); knowledge-guided ordering is described as an alternative not used
- Multi-modal integration was only performed within each modality; joint use of multi-modal information is deferred to future work
- The concatenating integration approach is more computationally intensive than the balanced tree approach
- Embedding-only outputs (from other methods) increase interpretability challenges for downstream features such as cell-cell interactions or pathway information

## Evidence

- [discussion] three key innovations with hierarchical integration: "This was achieved via the use of three essential innovations with hierarchical integration, pseudo-bulk building to minimise processing demand, and pseudo-replication"
- [discussion] data-guided integration order: "Our current method is based on a data-guided order, in which we integrate batches within one study or studies with similar size first."
- [intro] COVID-19 atlas-scale demonstration: "scMerge2 is able to integrate five million cells from a large COVID-19 data collection with over 1000 samples from 20 studies globally within a day"
- [intro] adjusted expression matrices as key output: "the next generation of atlas-scale integration algorithms should be capable of integrating a large number of studies and producing consensus cell type maps as well as adjusted expression matrix for"
- [readme] vignette and Bioconductor installation: [evidence span withheld — anchor status: external_doc]
- [discussion] within-modality scope limitation: "Currently, the integration illustrated in this paper was done within each modality."
