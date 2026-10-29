---
name: gene-set-enrichment-scoring
description: Use when raw gene expression profiles (rows = samples, columns = Entrez
  gene identifiers) must be transformed into pathway activity features — e.g., to
  train or apply a DeepCC subtype classifier, or to make batch/platform-robust single-sample
  predictions.
when_to_use_negative:
- The goal is clustering of cancer patients rather than supervised classification
  — DeepCC is specifically developed for classification but not clustering.
- The input is already a pathway-level/functional spectrum feature table — recomputing
  enrichment scores would be redundant.
- Comprehensive gene set prior knowledge (e.g., subtype-characteristic pathways) is
  unavailable, since biased or incomplete gene sets introduce potential bias.
edam_operation: http://edamontology.org/operation_2436
edam_topics:
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_3523
tools:
- name: DeepCC
  role: Provides getFunctionalSpectra/getFunctionalSpectrum and calcEnrichmentScore
    to compute gene-set enrichment-based functional spectra
  repo: https://github.com/CityUHK-CompBio/DeepCC
- name: msigdbr
  role: R package supplying MSigDB gene sets (get_msigdbr()) as an alternative to
    bundled MSigDB versions
- name: R
  role: Language in which the enrichment scoring and DeepCC workflow are implemented
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1038/s41389-019-0157-8
    title: 'DeepCC: a novel deep learning-based framework for cancer molecular subtype
      classification'
  source_content_hash: sha256:3baa2c34fa4ff6eeaed3c103a90ec2bbe2142fb5248befdf196c643bf04980d8
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/CityUHK-CompBio/DeepCC
  tools:
  - DeepCC
  - R
  - msigdbr
  edam_operation: http://edamontology.org/operation_2436
  edam_topics:
  - http://edamontology.org/topic_0203
  - http://edamontology.org/topic_3523
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.4444444444444444
n_claims: 9
---

# gene-set-enrichment-scoring

## Summary

Compute functional spectra by scoring a gene expression profile against MSigDB gene sets (e.g., MSigDB v7 with 22,596 sets), converting gene-level expression into pathway-level enrichment features that feed DeepCC's classifier. This pathway activity transformation underpins platform independence and single-sample prediction.

## When to use

Use when raw gene expression profiles (rows = samples, columns = Entrez gene identifiers) must be transformed into pathway activity features — e.g., to train or apply a DeepCC subtype classifier, or to make batch/platform-robust single-sample predictions.

## When NOT to use

- The goal is clustering of cancer patients rather than supervised classification — DeepCC is specifically developed for classification but not clustering.
- The input is already a pathway-level/functional spectrum feature table — recomputing enrichment scores would be redundant.
- Comprehensive gene set prior knowledge (e.g., subtype-characteristic pathways) is unavailable, since biased or incomplete gene sets introduce potential bias.

## Inputs

- data.frame of gene expression profiles (rows = patient samples, column names = Entrez identifiers)
- Single-sample expression profile (new_ep) for single-sample mode
- Reference expression profile: TCGA cancer-type identifier (e.g., "COADREAD", "BRCA", "OV") or a customized reference from similar cancer type/platform
- Gene sets: default MSigDB v7 (22,596 sets), or MSigDB v5.0 (10,348 sets) / v6.0 / v7.0 (17,779 sets), or MSigDBr from R package msigdbr

## Outputs

- Functional spectra (fs) matrix: samples x gene-set enrichment scores
- Single-sample functional spectrum (new_fs_single) usable by get_DeepCC_label

## How to apply

Prepare a data.frame of gene expression profiles with column names as Entrez identifiers and call getFunctionalSpectra(eps) for a batch of samples, or getFunctionalSpectrum(expressionProfile, refExp = "COADREAD") for a single sample. By default DeepCC uses MSigDB v7 (22,596 gene sets); you may instead pass geneSets = "MSigDBv5"/"MSigDBv6"/"MSigDBv7" or MSigDBr obtained from the R package msigdbr via get_msigdbr(). For single samples, the profile is rescaled by linear-model (lm) fitting against a predefined TCGA average reference expression profile (e.g., the COADREAD average), and enrichment scores (calcEnrichmentScore over each gene set) are computed on the residuals; the inverseRescale parameter should be enabled for microarray inputs since the TCGA reference is RNA-Seq. Customized references from the same/similar cancer type and platform can be supplied instead of predefined TCGA references.

## Related tools

- **DeepCC** (Provides getFunctionalSpectra/getFunctionalSpectrum and calcEnrichmentScore to compute gene-set enrichment-based functional spectra) — https://github.com/CityUHK-CompBio/DeepCC
- **msigdbr** (R package supplying MSigDB gene sets (get_msigdbr()) as an alternative to bundled MSigDB versions)
- **R** (Language in which the enrichment scoring and DeepCC workflow are implemented)

## Examples

```
library(DeepCC); fs <- getFunctionalSpectra(eps); new_fs_single <- getFunctionalSpectrum(new_ep, refExp = "COADREAD", inverseRescale = TRUE)
```

## Evaluation signals

- The computed functional spectrum has one enrichment score per gene set in the selected MSigDB version (22,596 columns for default MSigDB v7).
- For single samples, the rescaling residuals against the reference profile (e.g., TCGA-COADREAD average) are finite and non-degenerate before enrichment scoring.
- The resulting fs object is accepted by train_DeepCC_model / get_DeepCC_label without dimension mismatch, and predicted labels fall within the model's class levels.
- Switching geneSets (v5.0/v6.0/v7.0/MSigDBr) changes the number of enrichment features to the documented set counts (10,348 / 17,779 / 22,596).

## Limitations

- Prior knowledge of gene sets must be comprehensive to avoid potential bias; major characteristic signaling pathways of each subtype should be included in the gene set database.
- For microarray single samples, inverseRescale only partially compensates for the distributional difference versus the RNA-Seq-based TCGA reference; a customized reference from the same platform is better.
- Robustness depends on sufficient gene overlap between the sample and the gene sets; simulation showed robustness to random missing genes but severe platform mismatch can degrade scores.

## Evidence

- [readme] By default, DeepCC will use MSigDB v7 (22, 596 gene sets) to generate functional spectra.: "By default, DeepCC will use MSigDB v7 (22, 596 gene sets) to generate functional spectra. You can also use MSigDB v5.0 (10, 348 gene sets) and MSigDB v7.0 (17, 779 gene sets)."
- [readme] Single-sample classification requires a reference expression profile.: "for a given single sample, you have to provide a reference expression profile.
new_fs_single <- getFunctionalSpectrum(new_ep, refExp = "COADREAD")"
- [methods] Enrichment score computed on residuals of lm rescaling against the reference.: [evidence span withheld — anchor status: not_found]
- [abstract] DeepCC's pathway activity transformation addresses platform dependency.: "DeepCC provides a novel cancer classification framework that is platform independent, robust to missing data, and can be used for single sample prediction"
- [readme] Microarray single samples should enable inverseRescale.: "if your single sample is microarray data, we strongly suggest you turn the parameter `inverseRescale` on, since TCGA is RNA-Seq data, compared with microarray."
