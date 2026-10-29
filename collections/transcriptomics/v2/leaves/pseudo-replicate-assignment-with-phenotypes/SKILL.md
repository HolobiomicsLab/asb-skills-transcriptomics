---
name: pseudo-replicate-assignment-with-phenotypes
description: Use when integrating multi-sample, multi-condition single-cell collections
  (e.g. millions of cells across many donors/studies) where batch effects must be
  removed without confounding biological differences between phenotypes or severity
  groups.
when_to_use_negative:
- The data lack cell-type annotations — annotations are required as the pseudo-replicate
  information for pseudo-bulk construction
- The collection has no phenotype/condition structure (e.g. a single condition, single-sample
  study), so condition-aware pseudo-replication has no groups to respect
- Only a low-dimensional embedding is needed and no adjusted gene expression matrix
  is required for downstream sample-level analysis
edam_topics:
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_0203
tools:
- name: scMerge2
  role: Scalable integration algorithm that implements pseudo-bulk building and condition-aware
    pseudo-replication for atlas-scale multi-sample multi-condition single-cell data
    integration
  repo: https://github.com/SydneyBioX/scMerge
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1038/s41467-023-39923-2
    title: Atlas-scale single-cell multi-sample multi-condition data integration using
      scMerge2
  source_content_hash: sha256:5bf8ab6413642b4e7c15786d0e277cc007ca17717c0ca43f7ef2265eae3e57f4
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/SydneyBioX/scMerge
  tools:
  - scMerge2
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3308
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8
n_claims: 10
---

# Pseudo-replicate assignment with phenotypes

## Summary

A key scMerge2 innovation in which pseudo-replicates are assigned within phenotype/condition groups after building pseudo-bulk profiles per annotated cell type per sample, enabling condition-aware atlas-scale integration of multi-sample single-cell data.

## When to use

Use when integrating multi-sample, multi-condition single-cell collections (e.g. millions of cells across many donors/studies) where batch effects must be removed without confounding biological differences between phenotypes or severity groups.

## When NOT to use

- The data lack cell-type annotations — annotations are required as the pseudo-replicate information for pseudo-bulk construction
- The collection has no phenotype/condition structure (e.g. a single condition, single-sample study), so condition-aware pseudo-replication has no groups to respect
- Only a low-dimensional embedding is needed and no adjusted gene expression matrix is required for downstream sample-level analysis

## Inputs

- Single-cell count data from multi-study collections (e.g. the ~5M-cell COVID-19 PBMC collection: 20 studies, 963 donors, 1298 samples)
- Refined cell-type annotations per cell (serving as pseudo-replicate information)
- Sample-level metadata with phenotype/condition labels (e.g. COVID-19 severity) and technical labels (dataset, protocol, technology)

## Outputs

- Pseudo-replicate assignments within phenotype/condition groups
- Pseudo-bulk profiles per cell type per sample
- scMerge2-adjusted expression matrix for the integrated collection

## How to apply

First build pseudo-bulk profiles for each annotated cell type within each sample to minimise the processing demand of atlas-scale data. Then assign pseudo-replicates within phenotype/condition groups, so that pseudo-replication accounts for circumstances with phenotypes and the adjustment does not erase condition-level biology such as COVID-19 severity states. Use the refined cell-type annotations as the pseudo-replicate information underlying the assignment, and combine this with hierarchical integration in a data-guided order (batches within one study or similarly sized studies first) to produce an adjusted expression matrix for the full collection. The rationale is that pseudo-replication grouped by condition preserves phenotype discrimination while removing dataset-, protocol- and technology-driven variation.

## Related tools

- **scMerge2** (Scalable integration algorithm that implements pseudo-bulk building and condition-aware pseudo-replication for atlas-scale multi-sample multi-condition single-cell data integration) — https://github.com/SydneyBioX/scMerge

## Examples

```
BiocManager::install("scMerge")
```

## Evaluation signals

- Pseudo-replicates are assigned only within phenotype/condition groups — no pseudo-replicates pair samples from different conditions/severities
- Pseudo-bulk profiles exist for each annotated cell type within each sample before pseudo-replication is performed
- Post-integration, PCA on technical labels (dataset, protocol, technology) shows diminished separation compared with raw data
- Biological cell-state discrimination across COVID-19 severity levels is preserved after adjustment, with improved cell type identification
- The output is a full adjusted expression matrix, not an embedding-only result

## Limitations

- Integration was performed within each modality; joint use of multi-modal information (e.g. CITE-seq, CyTOF) for identifying pseudo-replicates is deferred to future work
- The integration order used is data-guided only (batches within one study or similarly sized studies first); a knowledge-guided order is described as an alternative that was not used
- The concatenating integration approach is more computationally intensive than the balanced tree approach

## Evidence

- [discussion] Three key innovations in pseudo-replication: "This was achieved via the use of three essential innovations with hierarchical integration, pseudo-bulk building to minimise processing demand, and pseudo-replication that accounts for circumstances"
- [abstract] scMerge2 scope: "Here, we present scMerge2, a scalable algorithm that allows data integration of atlas-scale multi-sample multi-condition single-cell studies."
- [readme] Package availability: [evidence span withheld — anchor status: external_doc]
