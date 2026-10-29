---
name: ortholog-group-construction-o2o-he-sh
description: Use when preparing raw scRNA-seq count matrices from multiple species
  for integration and you must define homologous gene groups (one-to-one, one-to-many,
  many-to-many) before concatenating counts into a shared feature matrix.
when_to_use_negative:
- The input is already a homologous-gene-aligned or integrated feature matrix rather
  than per-species raw counts
- The goal is species-specific gene analysis where dropping unshared genes (in the
  non-UINMF variants) is not acceptable and UINMF is not used
- Species lack curated sequence-based gene orthology in ENSEMBL, making orthology-based
  feature construction unreliable
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: BENGAL
  role: Pipeline that implements the O2O/HE/SH homology grouping, matrix concatenation,
    and per-combination h5ad output
- name: biomaRt
  role: Queries ENSEMBL v106 to retrieve orthology relationships for genes of each
    species' raw count matrix
- name: LIGER UINMF
  role: Downstream integrator whose ability to 'take unshared features' motivates
    appending each species' non-homologous genes to the concatenated matrix
  repo: https://github.com/welch-lab/liger
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1038/s41467-023-41855-w
    title: Benchmarking strategies for cross-species integration of single-cell RNA
      sequencing data
  source_content_hash: sha256:fb1e6f980a9e076d6227ea9d10114898157e87b07d6499743da6325897df1f04
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
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 5
---

# ortholog-group-construction-o2o-he-sh

## Summary

Constructs the cross-species gene feature space for the BENGAL benchmarking pipeline by grouping genes into one-to-one (O2O), one-to-many in-paralog (HE), and many-to-many out-paralog/co-ortholog (SH) orthology classes from ENSEMBL v106 via biomaRt, then subsetting and concatenating raw count matrices per homology class, optionally appending unshared features for LIGER UINMF.

## When to use

Use when preparing raw scRNA-seq count matrices from multiple species for integration and you must define homologous gene groups (one-to-one, one-to-many, many-to-many) before concatenating counts into a shared feature matrix.

## When NOT to use

- The input is already a homologous-gene-aligned or integrated feature matrix rather than per-species raw counts
- The goal is species-specific gene analysis where dropping unshared genes (in the non-UINMF variants) is not acceptable and UINMF is not used
- Species lack curated sequence-based gene orthology in ENSEMBL, making orthology-based feature construction unreliable

## Inputs

- Raw scRNA-seq count matrix per species with per-cell species and cell-type metadata
- ENSEMBL v106 orthology annotations retrieved via biomaRt
- Target homology class specification (o2o, he, or sh) and whether unshared features should be appended

## Outputs

- Concatenated cross-species raw count matrix restricted to the selected homologous gene group, with species and cell-type metadata
- One AnnData (.h5ad) file per task-homology combination, including variants with appended unshared features for LIGER UINMF

## How to apply

Query the ENSEMBL v106 database through biomaRt to retrieve orthology relationships for the genes in each species' raw count matrix, assembling gene groups classified as one-to-one orthologs (O2O), one-to-many orthologs containing in-paralogs (HE), and many-to-many out-paralogs/co-orthologs (SH). Apply the chosen selection rule (O2O-only, or HE/SH genes selected by high average expression level or strong homology confidence) to subset each species' expression matrix to the corresponding homologous feature set. Concatenate the subsetted raw count matrices across species into a single matrix carrying species and cell-type metadata so downstream integrators such as LIGER UINMF, which 'also takes unshared features', can operate on it. For the UINMF variant, append each species' unshared (non-homologous) genes to the concatenated matrix. Write one AnnData h5ad file per task-homology combination (e.g. o2o, he, sh, and their unshared-feature counterparts). The rationale is that homology-class choice is a benchmarked variable — 28 homology-mapping/integration combinations were tested, and including in-paralogs was found beneficial for evolutionarily distant species.

## Related tools

- **BENGAL** (Pipeline that implements the O2O/HE/SH homology grouping, matrix concatenation, and per-combination h5ad output)
- **biomaRt** (Queries ENSEMBL v106 to retrieve orthology relationships for genes of each species' raw count matrix)
- **LIGER UINMF** (Downstream integrator whose ability to 'take unshared features' motivates appending each species' non-homologous genes to the concatenated matrix) — https://github.com/welch-lab/liger

## Evaluation signals

- Each output h5ad contains only genes belonging to the specified homology group (O2O, HE, or SH) — no genes from other classes leak in
- Per-species cell metadata is retained in every concatenated matrix
- The set of combination files covers O2O, HE and SH, each with and without unshared features
- Gene groups are derived from ENSEMBL v106 orthology queried via biomaRt, so gene-to-ortholog assignments should be reproducible against that database version
- Non-homologous genes appear only in the UINMF variants, appended as unshared features

## Limitations

- Cross-species integration relies on curated gene orthology based on sequence, ignoring species-specific genes except in the UINMF unshared-feature variants
- Functional diversification between orthologs may exceed that between paralogs: out-paralogs can exhibit higher expression similarity than their corresponding orthologs, undermining orthology-based matching
- Whether including in-paralogs (HE) helps depends on divergence time — it was beneficial for evolutionarily distant species

## Evidence

- [intro] BENGAL pipeline and benchmark framing: "we develop the BENchmarking strateGies for cross-species integrAtion of singLe-cell RNA sequencing data (BENGAL) pipeline"
- [abstract] 28 homology/integration combinations benchmarked: "Here, we benchmark 28 combinations of gene homology mapping methods and data integration algorithms in a variety of biological settings."
- [abstract] in-paralogs benefit for distant species: "For evolutionarily distant species, including in-paralogs is beneficial."
- [intro] UINMF takes unshared features: "and its recent upgrade LIGER UINMF that also takes unshared features23;"
- [discussion] reliance on curated orthology: "cross-species integration still heavily relies on curated gene orthology based on sequence"
- [readme] LIGER cross-species use case: [evidence span withheld — anchor status: external_doc]
