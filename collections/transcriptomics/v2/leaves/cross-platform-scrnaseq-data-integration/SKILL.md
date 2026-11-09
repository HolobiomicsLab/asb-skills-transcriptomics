---
name: cross-platform-scrnaseq-data-integration
description: Decision support for choosing between a unified processing pipeline and
  platform-specific pipelines when integrating scRNA-seq data generated on different
  UMI-based technologies.
when_to_use_negative:
- Datasets all generated on the same platform with the same processing pipeline —
  platform-specific batch effects are not the confounder
- 5' scRNA-seq technologies other than Chromium, which are not supported by UniverSC
- Non-UMI-based single-cell RNA-seq platforms, which UniverSC does not support
edam_operation: http://edamontology.org/operation_3432
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_3308
tools:
- name: UniverSC
  role: Universal pipeline used to process both Chromium and SmartSeq2 datasets uniformly
  repo: https://github.com/minoda-lab/universc
- name: zUMIs
  role: Platform-specific pipeline used to process SmartSeq2 data as the baseline
    comparison
- name: Seurat
  role: R workflow used to integrate each SmartSeq2 GBM independently with the reference
    Chromium GBM
- name: kBET
  role: R package (v0.99.6) used to quantify batch-effect removal after integration
- name: cluster
  role: R package (v2.1.3) used to compute Silhouette scores for cluster distinctness
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1038/s41467-022-34681-z
    title: A flexible cross-platform single-cell data processing pipeline
  source_content_hash: sha256:23de363ec9cf5948d0683e24a642298ef1862dd1c9a8abf3f44c4265d97dad3f
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/minoda-lab/universc
  tools:
  - Seurat
  - UniverSC
  - cluster
  - kBET
  - zUMIs
  edam_operation: http://edamontology.org/operation_3432
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8333333333333334
n_claims: 12
---

# cross-platform-scrnaseq-data-integration

## Summary

Evaluates whether processing multiple scRNA-seq datasets (e.g., Chromium and SmartSeq2) with a single universal pipeline (UniverSC) improves cross-platform integration versus platform-specific pipelines (zUMIs), quantified with kBET and Silhouette scores after Seurat integration.

## When to use

Decision support for choosing between a unified processing pipeline and platform-specific pipelines when integrating scRNA-seq data generated on different UMI-based technologies.

## When NOT to use

- Datasets all generated on the same platform with the same processing pipeline — platform-specific batch effects are not the confounder
- 5' scRNA-seq technologies other than Chromium, which are not supported by UniverSC
- Non-UMI-based single-cell RNA-seq platforms, which UniverSC does not support

## Inputs

- Chromium reference gene-by-barcode matrix (GBM) from mouse primary cells (published scRNA-seq benchmarking study)
- SmartSeq2 gene-by-barcode matrices processed with UniverSC and with zUMIs against GRCm39

## Outputs

- integration_scores.csv with kBET and Silhouette values for UniverSC and zUMIs integrations
- Seurat-integrated cross-platform objects (one per SmartSeq2 processing variant)

## How to apply

Process the reference dataset (Chromium) once with UniverSC and the comparison dataset (SmartSeq2) twice — once with UniverSC and once with zUMIs — against the full mouse genome (GRCm39) with no downsampling. Harmonise the two SmartSeq2 gene-by-barcode matrices so they share identical gene and barcode sets: keep only intersecting barcodes and add missing genes with 0 UMIs, then format all GBMs to share identical genes. Independently integrate each SmartSeq2 GBM with the reference Chromium GBM using the Seurat integration workflow in R. Compute kBET (R package kBET v0.99.6) and Silhouette score (R package cluster v2.1.3) for each integration; the rationale is that processing all platforms with one pipeline removes batch effects attributable to the processing pipeline itself rather than the biology.

## Related tools

- **UniverSC** (Universal pipeline used to process both Chromium and SmartSeq2 datasets uniformly) — https://github.com/minoda-lab/universc
- **zUMIs** (Platform-specific pipeline used to process SmartSeq2 data as the baseline comparison)
- **Seurat** (R workflow used to integrate each SmartSeq2 GBM independently with the reference Chromium GBM)
- **kBET** (R package (v0.99.6) used to quantify batch-effect removal after integration)
- **cluster** (R package (v2.1.3) used to compute Silhouette scores for cluster distinctness)

## Evaluation signals

- UniverSC integration yields lower kBET (~0.06) than zUMIs (~0.11), reproducing Fig. 3B direction
- UniverSC integration yields higher Silhouette score (~0.43) than zUMIs (~0.36), reproducing Fig. 3C direction
- Both SmartSeq2 GBMs share identical gene and barcode sets after harmonisation (intersecting barcodes kept, missing genes filled with 0 UMIs)
- All three GBMs (Chromium reference, UniverSC-SmartSeq2, zUMIs-SmartSeq2) share identical gene sets before integration

## Limitations

- UniverSC wraps Cell Ranger 3.0.2 due to licensing; later Cell Ranger versions exist but core updates do not majorly affect scRNA-seq processing
- 5' scRNA-seq technologies other than Chromium are not supported by UniverSC at the time of writing
- Only UMI-based platforms are supported; SmartSeq2 comparison relies on the cross-platform benchmarking study's mouse primary cell datasets

## Evidence

- [methods] kBET and Silhouette score evaluation of integration quality: "To evaluate the quality of integration, kBET12 and Silhouette score13 were calculated for each case"
- [methods] Data integration with Seurat: "each version of SmartSeq2 GBM was integrated with the reference GBMs independently using Seurat"
- [methods] GBM harmonisation filter step: "only barcodes found in both GBMs were kept, and genes only found in one GBM were added to the other with 0 UMIs assigned."
- [abstract] UniverSC purpose: "a universal single-cell RNA-seq data processing tool that supports any unique molecular identifier-based platform"
- [readme] UniverSC package version: [evidence span withheld — anchor status: external_doc]
