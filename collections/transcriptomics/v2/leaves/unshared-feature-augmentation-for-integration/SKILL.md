---
name: unshared-feature-augmentation-for-integration
description: Use when integrating scRNA-seq count matrices from multiple species via
  BENGAL/LIGER UINMF and genes lacking annotated ENSEMBL homology should be retained
  as unshared features instead of being dropped.
when_to_use_negative:
- If the downstream integration method does not accept unshared features (e.g. standard
  LIGER iNMF, fastMNN, Harmony, SeuratV4, scVI/scANVI which rely only on shared homologous
  features)
- If the goal is strict orthology-based matching only — cross-species integration
  in this benchmark relies on curated sequence-based gene orthology, so sequence-similarity
  heuristics like SAMap are a different strategy
- If cell-type (rather than gene) correspondence is one-to-many or flat mappings are
  insufficient — the benchmark is limited to one-to-one cell type mappings
edam_topics:
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_0196
tools:
- name: LIGER UINMF
  role: Integration method whose iNMF variant consumes the unshared (non-homologous)
    features appended to the concatenated matrix
  repo: https://github.com/welch-lab/liger
- name: BENGAL
  role: Benchmarking pipeline that builds the homology-grouped, unshared-feature-augmented
    count matrices and writes the per-combination h5ad files
- name: biomaRt
  role: Queries ENSEMBL v106 for gene orthology relationships used to define shared
    vs unshared gene sets
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1038/s41467-023-41855-w
    title: Benchmarking strategies for cross-species integration of single-cell RNA
      sequencing data
  source_content_hash: sha256:80a064b59a6d9e3ed33c18a598939e9d88b84f13573e0568b9e97abfd347cf40
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/welch-lab/liger
  tools:
  - BENGAL
  - LIGER UINMF
  - biomaRt
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3308
  - http://edamontology.org/topic_0196
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 8
---

# unshared-feature-augmentation-for-integration

## Summary

Augments cross-species single-cell integration by appending each species' non-homologous (unshared) genes to the concatenated homologous feature matrix, enabling the LIGER UINMF integrative non-negative matrix factorization to use dataset-specific signals that one-to-one/one-to-many/many-to-many orthology mapping alone would discard.

## When to use

Use when integrating scRNA-seq count matrices from multiple species via BENGAL/LIGER UINMF and genes lacking annotated ENSEMBL homology should be retained as unshared features instead of being dropped.

## When NOT to use

- If the downstream integration method does not accept unshared features (e.g. standard LIGER iNMF, fastMNN, Harmony, SeuratV4, scVI/scANVI which rely only on shared homologous features)
- If the goal is strict orthology-based matching only — cross-species integration in this benchmark relies on curated sequence-based gene orthology, so sequence-similarity heuristics like SAMap are a different strategy
- If cell-type (rather than gene) correspondence is one-to-many or flat mappings are insufficient — the benchmark is limited to one-to-one cell type mappings

## Inputs

- Per-species raw scRNA-seq count matrices with cell metadata (species, cell type)
- ENSEMBL v106 orthology relationships retrieved via biomaRt (O2O, HE/in-paralog, SH/co-ortholog gene groups)
- Species-specific genes without annotated homology (unshared features)

## Outputs

- One AnnData h5ad file per task-homology combination (o2o, he, sh, each with and without unshared features)
- Concatenated cross-species count matrix with per-species cell-type metadata for LIGER UINMF input

## How to apply

In the BENGAL pipeline, first query ENSEMBL v106 via biomaRt to classify each species' genes into one-to-one orthologs (O2O), one-to-many orthologs with in-paralogs (HE), and many-to-many orthologs/co-orthologs (SH), then subset each species' raw count matrix to the selected homologous feature set and concatenate matrices across species with species and cell-type metadata. For the UINMF variant, additionally append each species' unshared (non-homologous) features to the concatenated matrix, writing one AnnData h5ad per task-homology combination (e.g. o2o, he, sh and their unshared-feature counterparts). Rationale: LIGER UINMF's integrative non-negative matrix factorization explicitly models dataset-specific factors, so unshared genes carry biology that shared-gene-only integration loses. Validation requires that each h5ad contains only genes belonging to the specified homology group, retains per-species cell metadata, and that the file set covers O2O, HE and SH both with and without unshared features.

## Related tools

- **LIGER UINMF** (Integration method whose iNMF variant consumes the unshared (non-homologous) features appended to the concatenated matrix) — https://github.com/welch-lab/liger
- **BENGAL** (Benchmarking pipeline that builds the homology-grouped, unshared-feature-augmented count matrices and writes the per-combination h5ad files)
- **biomaRt** (Queries ENSEMBL v106 for gene orthology relationships used to define shared vs unshared gene sets)

## Examples

```
devtools::install_github("welch-lab/liger"); library(rliger)  # LIGER UINMF consumes the concatenated matrix with species' unshared features appended
```

## Evaluation signals

- Each output h5ad contains only genes belonging to the specified homology group (O2O, HE or SH)
- The set of combination files covers O2O, HE and SH both with and without unshared features
- Each h5ad retains per-species cell metadata alongside species and cell-type annotations
- Unshared features are present only in the UINMF-variant matrices, never in shared-only counterparts
- Gene-to-gene translation across species matches the ENSEMBL v106 / biomaRt orthology annotations queried from the raw count matrices

## Limitations

- Cross-species integration here relies on curated sequence-based gene orthology, ignoring species-specific biology beyond simply appending unshared genes
- Functional diversification between orthologs may exceed that between paralogs, so orthology-based shared-feature matching can mislead even with unshared augmentation
- The benchmark's quantitative scoring assumes accurate cell type annotations and confident known cell type homology, and is limited to one-to-one cell type mappings; valid non-one-to-one cell type mappings across species are not assessed

## Evidence

- [full_text] genes without annotated homology are added as unshared features for LIGER UINMF: [evidence span withheld — anchor status: not_found]
- [methods] Query the ENSEMBL v106 database through biomaRt to retrieve orthology relationships: [evidence span withheld — anchor status: not_found]
- [intro] LIGER UINMF intro evidence: "and its recent upgrade LIGER UINMF that also takes unshared features23;"
- [readme] UINMF method reference: [evidence span withheld — anchor status: external_doc]
- [readme] Unshared-features vignette: [evidence span withheld — anchor status: external_doc]
