---
name: cytof-dataset-integration
description: Use when combining CyTOF marker-expression matrices from multiple studies
  with dataset-level batch effects, needing harmonized cell-type annotations and preservation
  of dataset-unique populations (e.g. Granulocytes) in a joint embedding.
when_to_use_negative:
- Data are scRNA-seq counts rather than CyTOF marker-expression matrices — scMerge2's
  scRNA-seq workflow (e.g. SEG-based controls) is a separate path
- Only one dataset is present — there is no dataset effect to remove and pseudo-replication
  across conditions is unnecessary
- Joint use of multi-modal information is required — the paper only demonstrates integration
  within each modality and defers multi-modal integration to future work
edam_operation: http://edamontology.org/operation_2935
edam_topics:
- http://edamontology.org/topic_3934
- http://edamontology.org/topic_2229
tools:
- name: scMerge2
  role: Pseudo-bulk construction per sample-by-condition combination, unsupervised
    dataset-effect removal with all features as negative controls, and propagation
    of adjustment factors back to single cells
  repo: https://github.com/SydneyBioX/scMerge
- name: Seurat
  role: Computes the joint UMAP embedding on the scMerge2-adjusted expression matrix
    for visualization
- name: CyTOF
  role: Source measurement technology generating the marker-expression matrices being
    integrated
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1038/s41467-023-39923-2
    title: Atlas-scale single-cell multi-sample multi-condition data integration using
      scMerge2
  source_content_hash: sha256:66573e448301fd8facae82e833b1b0a2a01acc248653027d1ee694a7c2b22039
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/SydneyBioX/scMerge
  tools:
  - CyTOF
  - Seurat
  - scMerge2
  edam_operation: http://edamontology.org/operation_2935
  edam_topics:
  - http://edamontology.org/topic_3934
  - http://edamontology.org/topic_2229
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.75
n_claims: 8
---

# cytof-dataset-integration

## Summary

Integrates CyTOF datasets across studies (e.g. COMBAT and Geanon, >11 million cells from healthy, COVID-19 and sepsis donors) using scMerge2's pseudo-bulk construction and unsupervised dataset-effect removal, verified with joint UMAP embedding and per-marker density overlays.

## When to use

Use when combining CyTOF marker-expression matrices from multiple studies with dataset-level batch effects, needing harmonized cell-type annotations and preservation of dataset-unique populations (e.g. Granulocytes) in a joint embedding.

## When NOT to use

- Data are scRNA-seq counts rather than CyTOF marker-expression matrices — scMerge2's scRNA-seq workflow (e.g. SEG-based controls) is a separate path
- Only one dataset is present — there is no dataset effect to remove and pseudo-replication across conditions is unnecessary
- Joint use of multi-modal information is required — the paper only demonstrates integration within each modality and defers multi-modal integration to future work

## Inputs

- CyTOF marker-expression matrices from COMBAT and Geanon studies with sample and condition metadata
- Per-cell type annotations harmonized to the 18 unified cell types
- scMerge2 data deposit files for the two CyTOF studies

## Outputs

- scMerge2-adjusted single-cell expression matrix with dataset effects removed
- Joint UMAP embedding (Seurat) colored by dataset and unified cell type
- Per-marker expression density distribution plots comparing datasets after integration

## How to apply

Load the COMBAT and Geanon CyTOF marker-expression matrices with sample and condition metadata from the scMerge2 data deposit and harmonize both studies' cell annotations to the 18 unified cell types. Build pseudo-bulk matrices per sample-by-condition combination with scMerge2 to reduce the 11M+ single cells to a tractable pseudo-replicate structure, then run scMerge2 in unsupervised mode with all features as negative controls (no externally supplied control markers) to estimate and remove dataset effects, propagating adjustment factors back to the single-cell level. Compute a joint UMAP with Seurat on the scMerge2-adjusted expression matrix and render it colored by dataset and unified cell type; overlay per-marker expression density distributions across datasets to confirm aligned marker distributions. The rationale is scMerge2's demonstrated ability to remove dataset variability in CyTOF, imaging mass cytometry and CITE-seq experiments at atlas scale (millions of cells) within roughly a day.

## Related tools

- **scMerge2** (Pseudo-bulk construction per sample-by-condition combination, unsupervised dataset-effect removal with all features as negative controls, and propagation of adjustment factors back to single cells) — https://github.com/SydneyBioX/scMerge
- **Seurat** (Computes the joint UMAP embedding on the scMerge2-adjusted expression matrix for visualization)
- **CyTOF** (Source measurement technology generating the marker-expression matrices being integrated)

## Examples

```
BiocManager::install("scMerge"); library(scMerge2)  # then run scMerge2 pseudo-bulk integration with ctl = all features, followed by Seurat UMAP
```

## Evaluation signals

- Post-integration UMAP shows COMBAT and Geanon cells mixed within shared cell types, unlike the raw data where datasets separate
- A clearly separable Granulocyte (Neutrophil/Eosinophil) cluster unique to Geanon is retained after integration
- Per-marker expression density curves overlap across the two datasets after adjustment
- Cell annotations reconcile to exactly 18 unified cell types across both studies
- Composite figure matches FigS7_v1.png qualitatively (mixed datasets, Granulocyte cluster, overlapping densities)

## Limitations

- Integration shown is only within each modality; joint multi-modal integration is deferred to future work
- Integration order is data-guided only (batches within one study or studies of similar size first); knowledge-guided ordering is described but not used
- Embedding-only outputs increase interpretability challenges for downstream features such as cell–cell interactions or pathway information, hence adjusted expression matrices are returned

## Evidence

- [abstract] scMerge2 removes dataset variability in CyTOF experiments: "we demonstrate that scMerge2 can remove dataset variability in CyTOF, imaging mass cytometry and CITE-seq experiments"
- [intro] Applicability of scMerge2 to CyTOF and scalability: "Our scMerge2 algorithm is able to integrate many millions of cells from single-cell studies generated from various single-cell technologies, including scRNA-seq, CyTOF, and imaging mass cytometry"
- [readme] scMerge2 repo for installing the software: "`scMerge` is a R package for merging and normalising single-cell RNA-Seq datasets."
- [discussion] Hierarchical integration and pseudo-bulk innovations: "This was achieved via the use of three essential innovations with hierarchical integration, pseudo-bulk building to minimise processing demand, and pseudo-replication"
