---
name: gene-set-enrichment-analysis
description: Use when you have CMScaller-classified (CMS1–CMS4) organoid, PDX or cell-line
  expression data and need to confirm the subtypes recapitulate known CMS biology
  (CMS2 HNF4A/MYC/WNT, CMS3 CDX2/metabolic, CMS4 TGFβ/EMT) via gene set enrichment.
when_to_use_negative:
- Samples whose human stromal composition differs from primary CRCs, e.g. metastatic
  CRC biopsies — CMScaller in its current implementation is not recommended for such
  samples
- Datasets with fewer than approximately forty samples — centering/scaling inescapably
  introduces prediction uncertainty in small datasets
- Single-cell RNA-seq data — exceedingly noisy and not what CMScaller/CMSgsa is optimized
  for
edam_operation: http://edamontology.org/operation_2930
edam_topics:
- http://edamontology.org/topic_0203
tools:
- name: CMScaller
  role: Provides CMSgsa (Camera gene set analysis with CMS-informative gene sets)
    executed on classified expression matrices; also supplies the CMS predictions
    used as class labels
  repo: https://github.com/Lothelab/CMScaller
- name: R
  role: Runtime environment in which the CMScaller package, CMSgsa and plotting are
    executed
  repo: https://github.com/Lothelab/CMScaller
- name: Biobase
  role: Used to load and handle expression matrices as ExpressionSet objects (library(Biobase))
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1038/s41598-017-16747-x
    title: 'CMScaller: an R package for consensus molecular subtyping of colorectal
      cancer pre-clinical models'
  source_content_hash: sha256:6502c0e418c396968a318fa66e82338ae1e6088c8c156bdd5411426e4260c6c4
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/Lothelab/CMScaller
  tools:
  - Biobase
  - CMScaller
  - R
  edam_operation: http://edamontology.org/operation_2930
  edam_topics:
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9090909090909091
n_claims: 11
---

# gene-set-enrichment-analysis

## Summary

Runs CMScaller's CMSgsa/Camera gene set analysis — built on the limma camera rotation test — on center-and-scaled expression matrices of CMScaller-classified pre-clinical colorectal cancer models to test enrichment of CMS-informative (cancer cell-intrinsic) gene sets per subtype.

## When to use

Use when you have CMScaller-classified (CMS1–CMS4) organoid, PDX or cell-line expression data and need to confirm the subtypes recapitulate known CMS biology (CMS2 HNF4A/MYC/WNT, CMS3 CDX2/metabolic, CMS4 TGFβ/EMT) via gene set enrichment.

## When NOT to use

- Samples whose human stromal composition differs from primary CRCs, e.g. metastatic CRC biopsies — CMScaller in its current implementation is not recommended for such samples
- Datasets with fewer than approximately forty samples — centering/scaling inescapably introduces prediction uncertainty in small datasets
- Single-cell RNA-seq data — exceedingly noisy and not what CMScaller/CMSgsa is optimized for

## Inputs

- CMScaller-classified gene expression matrix for organoid models (centered and scaled, ExpressionSet-compatible)
- CMScaller-classified gene expression matrix for merged PDX models (centered and scaled)
- Per-sample CMS predictions (e.g. res$prediction from CMScaller) used as class labels
- CMS-informative (cancer cell-intrinsic) gene set collection shipped with CMScaller

## Outputs

- Camera gene set analysis result object with per-subtype frames (cam$CMS1, cam$CMS2, cam$CMS3, cam$CMS4) of enrichment statistics
- cmsgsa_heatmaps.pdf: heatmaps of gene set enrichment for organoid and merged-PDX datasets

## How to apply

Load the CMScaller-classified organoid and merged-PDX expression matrices as ExpressionSet objects using Biobase, then center and scale each expression matrix as required for CMScaller-family inputs. Run the CMSgsa/subCamera gene set analysis — implemented on top of the limma camera rotation test — restricted to the CMS-informative (cancer cell-intrinsic) gene sets, passing the expression matrix and the per-sample CMS predictions, and set RNAseq=TRUE for RNA-seq data. Extract the per-subtype result frames (cam$CMS1 … cam$CMS4), each containing enrichment statistics for the gene sets under that CMS class. Plot heatmaps of gene set enrichment per dataset and judge success by whether the pattern matches expected signatures: CMS1 MSI-like, CMS2 activated HNF4A/MYC/WNT targets, CMS3 up-regulated CDX2 targets and metabolic processes, CMS4 strong TGFβ/EMT activation.

## Related tools

- **CMScaller** (Provides CMSgsa (Camera gene set analysis with CMS-informative gene sets) executed on classified expression matrices; also supplies the CMS predictions used as class labels) — https://github.com/Lothelab/CMScaller
- **R** (Runtime environment in which the CMScaller package, CMSgsa and plotting are executed) — https://github.com/Lothelab/CMScaller
- **Biobase** (Used to load and handle expression matrices as ExpressionSet objects (library(Biobase)))

## Examples

```
library(Biobase); library(CMScaller); cam <- CMSgsa(emat=crcTCGAsubset, class=res$prediction, RNAseq=TRUE); head(cam$CMS4)
```

## Evaluation signals

- cmsgsa_heatmaps.pdf exists for both the organoid and merged-PDX datasets
- CMS2-enriched samples show activated HNF4A, MYC and WNT target gene sets and are MSS-like
- CMS3 samples show up-regulated CDX2 targets and metabolic processes
- CMS4 samples show strong TGFβ and EMT gene set activation
- Per-subtype result frames returned by CMSgsa (e.g. cam$CMS4) are non-empty and contain per-gene-set enrichment statistics

## Limitations

- Input gene expression data must be centered and scaled, which introduces prediction uncertainty for small datasets (fewer than approximately forty samples)
- Caution is warranted for highly selected datasets where molecular distributions deviate from pCRC cohorts, including metastatic samples
- Intra-tumor heterogeneity (immune/stromal infiltration and cancer-cell intrinsic) may explain prediction errors at class boundaries

## Evidence

- [readme] Camera Gene Set Analysis with CMS informative gene sets: "### Camera Gene Set Analysis with CMS informative gene sets
cam <- CMSgsa(emat=crcTCGAsubset, class=res$prediction, RNAseq=TRUE)
head(cam$CMS4)"
- [abstract] CMScaller based on cancer cell-intrinsic subtype-enriched markers recapitulated CMS subtypes in pre-clinical models: "This new classifier, referred to as CMScaller, recapitulated the subtypes in both in vitro and in vivo models (551 in total)."
- [discussion] Input data must be centered and scaled, with uncertainty in small datasets: "input gene expression data should be centered and scaled. The implication is that small datasets inescapably introduce prediction uncertainty"
