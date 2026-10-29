---
name: rna-seq-count-matrix-preparation
description: Use when fitting SNP–gene eQTL or differential expression models with
  kimma's kmFit/kmFit_eQTL and you must assemble expression counts, sample metadata,
  and a kinship matrix into the expected dat/kin/counts/meta format.
when_to_use_negative:
- Input is already fitted model output or a summarised DEG/eQTL results table — use
  summarise_kmFit instead of re-preparing matrices
- You only need standard paired/unpaired differential expression without kinship or
  covariance structure, where limma or dream-style pipelines may suffice
edam_operation: http://edamontology.org/operation_3563
edam_topics:
- http://edamontology.org/topic_3517
- http://edamontology.org/topic_0203
tools:
- name: kimma
  role: R package whose example datasets (example.count, example.dat, example.kin)
    define the input schema for count matrix preparation and downstream kmFit_eQTL
    modeling
  repo: https://github.com/BIGslu/kimma
- name: R
  role: runtime environment in which kimma is installed via devtools and count matrices
    are prepared
- name: devtools
  role: used to install kimma from GitHub before loading the example datasets
- name: lme4qtl
  role: mixed model backend underlying kimma's kinship-aware linear mixed effects
    fits
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1093/bioinformatics/btad279
    title: 'Kimma: flexible linear mixed effects modeling with kinship covariance
      for RNA-seq data'
  source_content_hash: sha256:36d8886f3d3fedf83fd3eb30297b0a7b6d6a6da4ddf87cbb5104be608816ec20
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/BIGslu/kimma
  tools:
  - R
  - devtools
  - kimma
  - lme4qtl
  edam_operation: http://edamontology.org/operation_3563
  edam_topics:
  - http://edamontology.org/topic_3517
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 8
---

# rna-seq-count-matrix-preparation

## Summary

Prepare RNA-seq expression count matrices and associated sample metadata for linear mixed effects modeling in kimma, loading the package's example.count, example.dat, and example.kin datasets as the canonical input schema for kmFit_eQTL.

## When to use

Use when fitting SNP–gene eQTL or differential expression models with kimma's kmFit/kmFit_eQTL and you must assemble expression counts, sample metadata, and a kinship matrix into the expected dat/kin/counts/meta format.

## When NOT to use

- Input is already fitted model output or a summarised DEG/eQTL results table — use summarise_kmFit instead of re-preparing matrices
- You only need standard paired/unpaired differential expression without kinship or covariance structure, where limma or dream-style pipelines may suffice

## Inputs

- RNA-seq gene expression count matrix (genes x samples), e.g. example.count
- Sample metadata table with patient and library identifiers, e.g. example.dat
- Genetic kinship covariance matrix relating samples, e.g. example.kin
- Optional gene weights (e.g. voom weights, example.voom)
- For eQTL: SNP genotype table (dat_snp) and SNP-to-gene map (dat_map) linking genotypeID to geneID

## Outputs

- kimma-ready expression counts, metadata, and kinship objects consumable by kmFit/kmFit_eQTL
- For eQTL: paired dat_snp and dat_map inputs keyed by genotypeID/geneID

## How to apply

Install kimma from GitHub with devtools::install_github("BIGslu/kimma") and load the example datasets example.count (expression counts), example.dat (sample metadata), and example.kin (genetic kinship covariance matrix) shipped with the package. Structure your own data to match this schema: a genes-by-samples count matrix, a metadata table whose rows key to the count matrix columns, and a symmetric kinship covariance matrix over the patient identifier used as patientID. For eQTL use, additionally build dat_snp (genotypes) and dat_map (a SNP-to-gene map linking each genotypeID to its geneID). These prepared objects are then passed to kmFit_eQTL together with kin, patientID, and libraryID so each SNP–gene pair can be modeled with a genotype-by-condition interaction and a kinship random effect; optionally provide normalization weights such as the example.voom object.

## Related tools

- **kimma** (R package whose example datasets (example.count, example.dat, example.kin) define the input schema for count matrix preparation and downstream kmFit_eQTL modeling) — https://github.com/BIGslu/kimma
- **R** (runtime environment in which kimma is installed via devtools and count matrices are prepared)
- **devtools** (used to install kimma from GitHub before loading the example datasets)
- **lme4qtl** (mixed model backend underlying kimma's kinship-aware linear mixed effects fits)

## Examples

```
install.packages("devtools"); devtools::install_github("BIGslu/kimma"); data(example.count); data(example.dat); data(example.kin)
```

## Evaluation signals

- Count matrix dimensions match the metadata rows and kinship matrix patient IDs (patientID/libraryID alignment).
- dat_map links every genotypeID in dat_snp to a valid geneID present in the expression count matrix.
- Kinship matrix is symmetric with diagonal consistent with a covariance structure over the related cohort.
- A test call to kmFit_eQTL with the prepared objects runs without missing-sample errors and returns per-SNP–gene-pair rows.

## Limitations

- kimma is designed for RNA-seq linear mixed effects modeling with kinship/covariance structure; counts must come from an RNA-seq experiment with identifiable patient and library identifiers.
- Kinship-informed modeling requires an actual genetic relationship (kinship) matrix; without related samples, the kinship random effect is inapplicable.
- SNP–gene pairing requires a pre-specified dat_map; kmFit_eQTL does not map SNPs to genes itself.

## Evidence

- [intro] kimma (Kinship In Mixed Model Analysis), an open-source R package for flexible linear mixed effects modeling of RNA-seq: "kimma (Kinship In Mixed Model Analysis), an open-source R package for flexible linear mixed effects modeling of RNA-seq"
- [methods] kmFit_eQTL signature listing dat_snp, dat_map, geneID, genotypeID, dat, kin, patientID: "kmFit_eQTL(dat_snp, dat_map, geneID, genotypeID, dat, kin, patientID, libraryID, counts"
- [methods] Example datasets enumerated in the methods section: "example.count  example.dat  example.kin"
- [readme] README installation instructions for kimma: "install.packages("devtools")
devtools::install_github("BIGslu/kimma")"
