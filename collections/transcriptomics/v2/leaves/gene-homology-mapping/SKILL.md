---
name: gene-homology-mapping
description: Use when preparing single-cell RNA-seq count matrices from two or more
  species for integration and you must decide which homologous gene sets (O2O vs in-paralog-including
  HE/SH) define the shared feature space.
when_to_use_negative:
- Genes have no annotated homology and the downstream algorithm cannot exploit unshared
  features (only LIGER UINMF takes unshared features)
- The analysis targets within-species batch integration rather than cross-species
  integration, where orthology-based gene translation is unnecessary
- Cell type homology rather than gene homology is the mapping target — the article
  notes flat gene/cell-type mappings do not comprehensively describe cross-species
  correspondence and needs specialised metrics
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: BENGAL
  role: Pipeline that implements the three gene homology mapping approaches (O2O,
    HE, SH) and builds the concatenated cross-species feature space
- name: biomaRt
  role: Queries the ENSEMBL v106 database to retrieve orthology relationships between
    species' genes
- name: LIGER UINMF
  role: Downstream integrative NMF method that additionally consumes unshared (non-homologous)
    features appended to the concatenated matrix
  repo: https://github.com/welch-lab/liger
- name: SAMap
  role: Alternative whole-body atlas integrator that builds its own BLAST-based gene
    homology mapping between species with challenging gene homology annotation
  repo: https://github.com/atarashansky/SAMap
- name: scOntoMatch
  role: Companion package used in BENGAL to trim and match cell type ontology annotations
    across datasets after gene-space construction
  repo: https://github.com/Papatheodorou-Group/scOntoMatch
provenance:
  source_task_ids:
  - task_001
  - task_007
  - task_005
  source_papers:
  - doi: 10.1038/s41467-023-41855-w
    title: Benchmarking strategies for cross-species integration of single-cell RNA
      sequencing data
  source_content_hash: sha256:de32b1301bb9558b587dd7b7554320167bba6624a7035e44e85eaa0f26b9da77
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/welch-lab/liger
  tools:
  - BENGAL
  - Harmony
  - LIGER
  - LIGER UINMF
  - SAMap
  - Scanorama
  - SeuratV4
  - biomaRt
  - fastMNN
  - scANVI
  - scIB
  - scOntoMatch
  - scVI
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.875
n_claims: 16
---

# gene-homology-mapping

## Summary

Constructs the cross-species gene feature space for scRNA-seq integration by grouping genes into one-to-one orthologs (O2O), one-to-many (HE, in-paralogs) and many-to-many (SH, co-orthologs) homology sets, then subsetting and concatenating raw counts across species. This is the pivotal BENGAL preprocessing step that determines which genes integration algorithms can align across species.

## When to use

Use when preparing single-cell RNA-seq count matrices from two or more species for integration and you must decide which homologous gene sets (O2O vs in-paralog-including HE/SH) define the shared feature space.

## When NOT to use

- Genes have no annotated homology and the downstream algorithm cannot exploit unshared features (only LIGER UINMF takes unshared features)
- The analysis targets within-species batch integration rather than cross-species integration, where orthology-based gene translation is unnecessary
- Cell type homology rather than gene homology is the mapping target — the article notes flat gene/cell-type mappings do not comprehensively describe cross-species correspondence and needs specialised metrics

## Inputs

- Raw single-cell RNA-seq count matrices per species (Seurat objects/RDS or AnnData)
- Cell type annotations and species metadata per cell
- ENSEMBL v106 orthology annotations retrieved via biomaRt (or curated sequence-based orthology annotations)

## Outputs

- One AnnData h5ad file per task-homology combination (o2o, he, sh, and unshared-feature counterparts)
- Concatenated cross-species raw count matrix restricted to the specified homology group, with species and cell-type metadata

## How to apply

Query the ENSEMBL v106 database through biomaRt to retrieve orthology relationships for the genes in each species' raw count matrix, then assemble gene groups classified as one-to-one orthologs (O2O), one-to-many (HE, containing in-paralogs) and many-to-many (SH, out-paralogs/co-orthologs). Apply the O2O/HE/SH selection rules to subset each species' expression matrix to the corresponding homologous feature set, and concatenate the subsetted raw count matrices across species into a single matrix carrying species and cell-type metadata. For the LIGER UINMF variant, additionally append each species' unshared (non-homologous) features to the concatenated matrix, since UINMF explicitly exploits unshared features. Write one AnnData h5ad file per task-homology combination (o2o, he, sh, and their unshared-feature counterparts) so downstream integration (scVI, scANVI, SeuratV4) can be run per homology strategy. Decide between O2O and in-paralog-including strategies based on evolutionary distance: for evolutionarily distant species pairs (e.g. Embryo_xt_dr, Xenopus tropicalis vs Danio rerio), including in-paralogs (HE/SH) is beneficial for biology conservation.

## Related tools

- **BENGAL** (Pipeline that implements the three gene homology mapping approaches (O2O, HE, SH) and builds the concatenated cross-species feature space)
- **biomaRt** (Queries the ENSEMBL v106 database to retrieve orthology relationships between species' genes)
- **LIGER UINMF** (Downstream integrative NMF method that additionally consumes unshared (non-homologous) features appended to the concatenated matrix) — https://github.com/welch-lab/liger
- **SAMap** (Alternative whole-body atlas integrator that builds its own BLAST-based gene homology mapping between species with challenging gene homology annotation) — https://github.com/atarashansky/SAMap
- **scOntoMatch** (Companion package used in BENGAL to trim and match cell type ontology annotations across datasets after gene-space construction) — https://github.com/Papatheodorou-Group/scOntoMatch

## Examples

```
library(biomaRt); ensembl <- useEnsembl(biomart = "genes", version = 106); ortho <- getBM(attributes = c("external_gene_name", "mmusculus_homolog_associated_gene_name"), mart = ensembl)
```

## Evaluation signals

- Each output h5ad contains only genes belonging to the specified homology group (O2O, HE or SH)
- Per-species cell metadata is retained after subsetting and concatenation
- The set of output files covers O2O, HE and SH with and without unshared features
- In the Embryo_xt_dr task, in-paralog-including strategies (HE/SH) outperform O2O on the biology-conservation score, reproducing the reported direction
- Downstream integrated scores (e.g. Pancreas_hs_mm 0.33–0.94, Hippocampus_hs_mu_ss 0.31–0.89) are computable from every homology variant without feature-space errors

## Limitations

- Cross-species integration relies heavily on curated sequence-based gene orthology and ignores species-specific genes
- Functional diversification between orthologs may exceed that between paralogs: out-paralogs can exhibit higher expression similarity than their corresponding orthologs through gain, loss and partition of function
- Expression similarities across species may arise from convergence or concerted evolution rather than homology, and no current algorithm disentangles these sources

## Evidence

- [abstract] For evolutionarily distant species, including in-paralogs is beneficial.: "For evolutionarily distant species, including in-paralogs is beneficial."
- [discussion] cross-species integration still heavily relies on curated sequence-based gene orthology: "cross-species integration still heavily relies on curated gene orthology based on sequence"
- [discussion] out-paralogs can exhibit higher expression similarity compared to their corresponding orthologs: "out-paralogs can exhibit higher expression similarity compared to their corresponding orthologs through gain, loss and partition of function"
- [readme] UINMF mosaic integration using unshared features: "UINMF performs mosaic integration of single-cell multi-omic datasets using nonnegative matrix factorization"
- [readme] SAMap BLAST homology mapping requirement: [evidence span withheld — anchor status: external_doc]
