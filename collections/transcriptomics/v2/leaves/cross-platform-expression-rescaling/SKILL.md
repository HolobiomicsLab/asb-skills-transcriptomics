---
name: cross-platform-expression-rescaling
description: Use when classifying a single gene expression sample with a pre-trained
  DeepCC model, or when the input sample's platform differs from the RNA-Seq-based
  TCGA training reference (e.g., microarray data).
when_to_use_negative:
- Classifying a batch of samples — use getFunctionalSpectra (plural) on the expression
  profile set instead of single-sample rescaling
- Clustering cancer patients — DeepCC is specifically developed for classification,
  not clustering
- The input is already a functional spectrum / enrichment score table — rescaling
  operates on raw gene expression profiles only
edam_operation: http://edamontology.org/operation_0314
edam_topics:
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_3518
tools:
- name: DeepCC
  role: Provides getFunctionalSpectrum with refExp and inverseRescale parameters,
    pre-defined TCGA average reference profiles, and the rescaling/enrichment machinery
  repo: https://github.com/CityUHK-CompBio/DeepCC
- name: R
  role: Language in which DeepCC's rescaling (lm fitting) and enrichment score computation
    are implemented
- name: msigdbr
  role: Supplies MSigDBr gene sets via get_msigdbr() used for functional spectrum
    generation after rescaling
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1038/s41389-019-0157-8
    title: 'DeepCC: a novel deep learning-based framework for cancer molecular subtype
      classification'
  source_content_hash: sha256:47210b1248a4f7845c5c1cbb262b4873b03d0229c88576f481a88784ee29899e
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/CityUHK-CompBio/DeepCC
  tools:
  - DeepCC
  - R
  - msigdbr
  edam_operation: http://edamontology.org/operation_0314
  edam_topics:
  - http://edamontology.org/topic_0203
  - http://edamontology.org/topic_3518
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 6
---

# cross-platform-expression-rescaling

## Summary

Rescales a single-sample gene expression profile against a predefined or user-defined reference average expression profile (e.g., TCGA-derived 'COADREAD') before pathway enrichment, enabling DeepCC to generate a functional spectrum from one sample regardless of platform.

## When to use

Use when classifying a single gene expression sample with a pre-trained DeepCC model, or when the input sample's platform differs from the RNA-Seq-based TCGA training reference (e.g., microarray data).

## When NOT to use

- Classifying a batch of samples — use getFunctionalSpectra (plural) on the expression profile set instead of single-sample rescaling
- Clustering cancer patients — DeepCC is specifically developed for classification, not clustering
- The input is already a functional spectrum / enrichment score table — rescaling operates on raw gene expression profiles only

## Inputs

- Single-sample gene expression profile (data.frame, row/column genes as Entrez identifiers)
- Reference expression profile, either a predefined TCGA cancer-type identifier string (e.g., 'COADREAD', 'BRCA', 'OV') or a user-generated customized profile
- Optional MSigDB gene sets: 'MSigDBv7' default (22,596 sets), 'MSigDBv5'/'MSigDBv6', or MSigDBr from msigdbr::get_msigdbr()

## Outputs

- Single-sample functional spectrum (new_fs_single) of enrichment scores over the selected gene sets
- Predicted molecular subtype label via get_DeepCC_label(deepcc_model, new_fs_single), optionally class probabilities via get_DeepCC_prob

## How to apply

Overlap the single sample's genes with a TCGA average reference expression profile — DeepCC pre-defines references for each TCGA cancer type, invoked by TCGA identifier (COADREAD, BRCA, OV, etc.) via getFunctionalSpectrum(expressionProfile, refExp = "COADREAD"). The sample is rescaled by linear-model (lm) fitting against the reference, and enrichment scores are computed on the residuals over MSigDB v7 gene sets (22,596 sets by default; alternatively MSigDB v5.0 with 10,348 or MSigDBr from the R package msigdbr). If the input sample is microarray data, turn on the inverseRescale parameter to partially compensate for distributional differences versus RNA-Seq TCGA data, though generating a customized reference from same/similar cancer type and platform is preferred. The resulting single-sample functional spectrum (new_fs_single) can then be passed to get_DeepCC_label for subtype prediction.

## Related tools

- **DeepCC** (Provides getFunctionalSpectrum with refExp and inverseRescale parameters, pre-defined TCGA average reference profiles, and the rescaling/enrichment machinery) — https://github.com/CityUHK-CompBio/DeepCC
- **R** (Language in which DeepCC's rescaling (lm fitting) and enrichment score computation are implemented)
- **msigdbr** (Supplies MSigDBr gene sets via get_msigdbr() used for functional spectrum generation after rescaling)

## Examples

```
new_fs_single <- getFunctionalSpectrum(new_ep, refExp = "COADREAD", inverseRescale = TRUE); pred_label <- get_DeepCC_label(deepcc_model, new_fs_single)
```

## Evaluation signals

- getFunctionalSpectrum returns a single functional-spectrum record for the one input sample with length matching the selected gene set collection (22,596 for default MSigDB v7)
- The predicted label from get_DeepCC_label is a member of the class levels stored in the loaded model (e.g., levels returned by load_DeepCC_model("CRC_TCGA"))
- Reference identifier resolves against DeepCC's pre-defined TCGA average expression profiles (TCGA cancer-type identifiers such as COADREAD, BRCA, OV)
- For microarray input, confirm inverseRescale was enabled; residuals from lm rescaling feed the enrichment score computation without errors from gene overlap

## Limitations

- inverseRescale only partially compensates for the distributional difference between microarray data and the RNA-Seq-based TCGA reference; a customized reference from the same/similar cancer type and platform is better
- Depends on comprehensive gene set prior knowledge to avoid potential bias; major characteristic signaling pathways or biological processes for each cancer subtype should be included in the gene set database
- Batch effects and platform differences are the motivation for the reference-based rescaling, but robustness to missing data was only demonstrated via random gene subsampling simulations

## Evidence

- [methods] Single-sample classification via predefined reference: "new_fs_single <- getFunctionalSpectrum(new_ep, refExp = "COADREAD")
pred_label <- get_DeepCC_label(deepcc_model, new_fs_single)"
- [readme] Customized references encouraged over inverseRescale: "inverseRescale can overcome this distribution differently a little bit, but it's better to generate your customized reference."
- [readme] Predefined TCGA references: "In DeepCC we prepared average expression profiles of each cancer types in TCGA project as reference. To use them, just use the TCGA identifier (COADREAD, BRCA, OV, etc.) to indicate the cancer type."
- [abstract] Platform-independent single-sample prediction motivation: "DeepCC provides a novel cancer classification framework that is platform independent, robust to missing data, and can be used for single sample prediction"
