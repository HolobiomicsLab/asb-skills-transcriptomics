---
name: limma-camera-rotational-gene-set-testing
description: Use when you have a CMScaller-classified expression matrix (organoids,
  PDXs, cell lines) and need per-subtype gene set enrichment statistics to confirm
  the subtypes recapitulate canonical CMS pathway signatures.
when_to_use_negative:
- Samples with a different human stromal component than primary CRCs, e.g. metastatic
  CRC biopsies — CMScaller is not recommended there
- Very small datasets (fewer than ~40 samples), where centering/scaling inescapably
  introduces prediction uncertainty
- Single-cell RNA-seq data, which is exceedingly noisy and for which CMScaller is
  not optimized
edam_operation: null
edam_topics:
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_2640
tools:
- name: CMScaller
  role: Provides CMSgsa/subCamera camera gene set analysis with CMS-informative gene
    sets on classified pre-clinical model expression data
  repo: https://github.com/Lothelab/CMScaller
- name: limma
  role: Underlying camera rotation gene set test used by CMSgsa
- name: Biobase
  role: Loads the organoid/merged-PDX expression matrices as ExpressionSet objects
    before CMSgsa
- name: R
  role: Runtime environment for the CMScaller package and CMSgsa analysis
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1038/s41598-017-16747-x
    title: 'CMScaller: an R package for consensus molecular subtyping of colorectal
      cancer pre-clinical models'
  source_content_hash: sha256:1e0c783c2c33d41581f86518e07cba357a47117197d8c1cf0586424a6fe960b0
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/Lothelab/CMScaller
  tools:
  - Biobase
  - CMScaller
  - R
  - limma
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0203
  - http://edamontology.org/topic_2640
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6
n_claims: 5
---

# limma-camera-rotational-gene-set-testing

## Summary

Performs camera rotation-based gene set analysis (from limma) with CMS-informative, cancer cell-intrinsic gene sets via CMScaller's CMSgsa/subCamera function, to verify that CMScaller-assigned subtypes in organoids and PDXs recapitulate known CMS biology (e.g., CMS2 HNF4A/MYC/WNT, CMS4 TGF-beta/EMT).

## When to use

Use when you have a CMScaller-classified expression matrix (organoids, PDXs, cell lines) and need per-subtype gene set enrichment statistics to confirm the subtypes recapitulate canonical CMS pathway signatures.

## When NOT to use

- Samples with a different human stromal component than primary CRCs, e.g. metastatic CRC biopsies — CMScaller is not recommended there
- Very small datasets (fewer than ~40 samples), where centering/scaling inescapably introduces prediction uncertainty
- Single-cell RNA-seq data, which is exceedingly noisy and for which CMScaller is not optimized

## Inputs

- CMScaller-classified gene expression matrix (organoid, PDX, or cell line; ExpressionSet from Biobase or matrix), centered and scaled
- Per-sample CMS class predictions from CMScaller (e.g., res$prediction)
- CMS-informative cancer cell-intrinsic gene sets bundled with CMScaller

## Outputs

- Per-subtype camera gene set enrichment statistics (e.g., cam$CMS1 ... cam$CMS4 data frames from CMSgsa)
- Gene set enrichment heatmaps (cmsgsa_heatmaps.pdf) showing subtype-specific pathway patterns

## How to apply

Load the CMScaller-classified organoid or merged-PDX expression matrices as ExpressionSet objects with Biobase. Center and scale the expression matrix, as CMScaller input requires this and camera tests are sensitive to expression scale. Call CMSgsa (subCamera), which wraps the limma camera rotation test, passing the expression matrix, the CMScaller class predictions, and RNAseq=TRUE for RNA-seq data; it uses the CMS-informative (cancer cell-intrinsic) gene sets since stromal/immune signatures are absent from pre-clinical models. Extract per-subtype gene set statistics from the returned list (e.g., cam$CMS4) and plot heatmaps of enrichment across subtypes. Interpret the pattern against expected signatures: CMS1 MSI-like, CMS2 HNF4A/MYC/WNT target activation, CMS3 CDX2 targets and metabolic processes, CMS4 TGF-beta/EMT activation.

## Related tools

- **CMScaller** (Provides CMSgsa/subCamera camera gene set analysis with CMS-informative gene sets on classified pre-clinical model expression data) — https://github.com/Lothelab/CMScaller
- **limma** (Underlying camera rotation gene set test used by CMSgsa)
- **Biobase** (Loads the organoid/merged-PDX expression matrices as ExpressionSet objects before CMSgsa)
- **R** (Runtime environment for the CMScaller package and CMSgsa analysis)

## Examples

```
cam <- CMSgsa(emat=crcTCGAsubset, class=res$prediction, RNAseq=TRUE); head(cam$CMS4)
```

## Evaluation signals

- cmsgsa_heatmaps.pdf exists and its enrichment pattern matches expected subtype signatures: CMS1 MSI-like, CMS2 HNF4A/MYC/WNT, CMS3 CDX2/metabolic, CMS4 TGF-beta/EMT
- head(cam$CMS4) returns a data frame of gene set statistics with sensible FDR/rank columns
- Expression matrix was centered and scaled before CMSgsa, as required for CMScaller input
- All CMScaller class predictions map to exactly one of CMS1–CMS4 with no missing classes

## Limitations

- Input gene expression data must be centered and scaled, which introduces prediction uncertainty for small datasets (fewer than approximately forty samples)
- CMScaller in its current implementation is not recommended for samples with a different human stromal component than primary CRCs, e.g. metastatic CRC biopsies
- Intra-tumor heterogeneity (immune/stromal infiltration and cancer-cell intrinsic) may explain prediction errors at class boundaries

## Evidence

- [readme] Camera Gene Set Analysis with CMS informative gene sets
cam <- CMSgsa(emat=crcTCGAsubset, class=res$prediction, RNAseq=TRUE)
head(cam$CMS4): "cam <- CMSgsa(emat=crcTCGAsubset, class=res$prediction, RNAseq=TRUE)"
- [abstract] This new classifier, referred to as CMScaller, recapitulated the subtypes in both in vitro and in vivo models (551 in total).: "This new classifier, referred to as CMScaller, recapitulated the subtypes in both in vitro and in vivo models (551 in total)."
- [discussion] input gene expression data should be centered and scaled. The implication is that small datasets inescapably introduce prediction uncertainty: "input gene expression data should be centered and scaled. The implication is that small datasets inescapably introduce prediction uncertainty"
