---
name: negative-control-feature-selection-for-batch-correction
description: Use when integrating atlas-scale multi-sample multi-condition single-cell
  studies (e.g. >1M cells across cohorts like COMBAT and Geanon CyTOF) and you need
  control features to guide unsupervised batch/dataset-effect removal.
when_to_use_negative:
- When a joint embedding alone suffices — rapid embedding-only methods (deepMNN, BBKNN,
  Harmony, scVI, scANVI, DESC) do not return adjusted expression matrices, but this
  skill is specifically for producing one
- When the goal is joint integration across modalities — the article's integration
  was performed within each modality only, with multi-modal incorporation deferred
  to future work
- When control features cannot be trusted and data is too small to support all-features-as-controls
  estimation — the unsupervised mode assumes most features are stable across batches
edam_topics:
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_0091
tools:
- name: scMerge2
  role: Estimates and removes dataset effects using negative-control features (SEGs
    or all features in unsupervised mode) via hierarchical integration and pseudo-bulk
    pseudo-replication
  repo: https://github.com/SydneyBioX/scMerge
- name: scMerge
  role: R/Bioconductor package supplying the segList of stably expressed genes (human_scSEG,
    mouse_scSEG) used as negative controls
  repo: https://github.com/SydneyBioX/scMerge
- name: Seurat
  role: Computes the joint UMAP embedding on the scMerge2-adjusted expression matrix
    to visually verify batch removal
- name: CyTOF
  role: 'Target data modality: CyTOF marker-expression matrices from COMBAT and Geanon
    are integrated to demonstrate dataset-effect removal on non-transcriptomic data'
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1038/s41467-023-39923-2
    title: Atlas-scale single-cell multi-sample multi-condition data integration using
      scMerge2
  source_content_hash: sha256:597e2dc04e593ec585c60c7d9a5ee43c334c00b43537ca7588e182c6226d5958
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/SydneyBioX/scMerge
  tools:
  - CyTOF
  - Seurat
  - scMerge
  - scMerge2
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0203
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.75
n_claims: 8
---

# negative-control-feature-selection-for-batch-correction

## Summary

Selects stably expressed genes (SEGs) or, in unsupervised mode, treats all features as negative controls so scMerge2 can estimate and remove batch/dataset effects from multi-sample single-cell data (scRNA-seq, CyTOF, imaging mass cytometry, CITE-seq) without external control markers.

## When to use

Use when integrating atlas-scale multi-sample multi-condition single-cell studies (e.g. >1M cells across cohorts like COMBAT and Geanon CyTOF) and you need control features to guide unsupervised batch/dataset-effect removal.

## When NOT to use

- When a joint embedding alone suffices — rapid embedding-only methods (deepMNN, BBKNN, Harmony, scVI, scANVI, DESC) do not return adjusted expression matrices, but this skill is specifically for producing one
- When the goal is joint integration across modalities — the article's integration was performed within each modality only, with multi-modal incorporation deferred to future work
- When control features cannot be trusted and data is too small to support all-features-as-controls estimation — the unsupervised mode assumes most features are stable across batches

## Inputs

- Marker-expression or gene-count matrix from multiple single-cell studies (e.g. COMBAT and Geanon CyTOF matrices, >11M cells)
- Sample and condition metadata per cell (study/batch, donor, disease condition such as healthy control, COVID-19, sepsis)
- Cell-type annotations harmonized to a unified cell-type scheme
- Optionally, curated negative-control features (e.g. segList$human$human_scSEG); otherwise none required in unsupervised mode

## Outputs

- scMerge2-adjusted single-cell expression matrix with dataset effects removed
- Estimated dataset-effect adjustment factors learned on the pseudo-bulk representation
- Integration artifacts such as a joint Seurat UMAP embedding colored by dataset and cell type

## How to apply

Harmonize cell annotations across studies to a unified cell-type scheme (e.g. 18 unified types) before selecting controls. Build pseudo-bulk matrices per sample-by-condition combination with scMerge2 to reduce millions of cells to a tractable pseudo-replicate structure. Either supply curated negative controls — the scMerge package ships segList with human_scSEG and mouse_scSEG SEGs loadable via data(segList) — or run scMerge2 in unsupervised mode configuring all features as negative controls when no validated control markers exist for the platform (e.g. CyTOF marker panels). Estimate dataset effects on the pseudo-bulk representation, then propagate the adjustment factors back to the single-cell level to obtain an adjusted expression matrix. Rationale: scMerge2's hierarchical integration, pseudo-bulk building, and pseudo-replication make control-based factor estimation scalable to millions of cells while still returning an adjusted matrix for downstream analyses.

## Related tools

- **scMerge2** (Estimates and removes dataset effects using negative-control features (SEGs or all features in unsupervised mode) via hierarchical integration and pseudo-bulk pseudo-replication) — https://github.com/SydneyBioX/scMerge
- **scMerge** (R/Bioconductor package supplying the segList of stably expressed genes (human_scSEG, mouse_scSEG) used as negative controls) — https://github.com/SydneyBioX/scMerge
- **Seurat** (Computes the joint UMAP embedding on the scMerge2-adjusted expression matrix to visually verify batch removal)
- **CyTOF** (Target data modality: CyTOF marker-expression matrices from COMBAT and Geanon are integrated to demonstrate dataset-effect removal on non-transcriptomic data)

## Examples

```
library(scMerge); data(segList); ctrl <- segList$human$human_scSEG
```

## Evaluation signals

- Post-integration UMAP colored by dataset shows datasets mixed within shared cell types compared to raw data
- Batch-unique biology is retained: the Granulocyte (Neutrophil/Eosinophil) cluster unique to the Geanon CyTOF dataset remains clearly separable
- Per-marker expression density distributions overlap across datasets after adjustment
- Cell-type-specific marker expression distributions confirm effective dataset-effect removal
- An adjusted expression matrix (not just an embedding) is returned for downstream sample-level analyses

## Limitations

- Integration demonstrated only within each modality; incorporating multi-modal information to better identify pseudo-replicates is future work
- Integration order is data-guided (batches within one study or similarly sized studies first); knowledge-guided ordering is described as an unused alternative
- Embedding-only outputs would limit interpretable downstream features such as cell–cell interactions or pathway information — hence an adjusted matrix is produced
- The concatenating integration approach is more computationally intensive than the balanced tree approach

## Evidence

- [abstract] scMerge2 removes dataset variability in CyTOF, imaging mass cytometry and CITE-seq: "we demonstrate that scMerge2 can remove dataset variability in CyTOF, imaging mass cytometry and CITE-seq experiments"
- [intro] Scalability across single-cell technologies: "Our scMerge2 algorithm is able to integrate many millions of cells from single-cell studies generated from various single-cell technologies, including scRNA-seq"
- [discussion] Three innovations enabling control-based integration: "This was achieved via the use of three essential innovations with hierarchical integration, pseudo-bulk building to minimise processing demand, and pseudo-replication"
- [readme] SEG negative controls shipped with scMerge: [evidence span withheld — anchor status: external_doc]
- [readme] SEG evaluation reference: [evidence span withheld — anchor status: external_doc]
