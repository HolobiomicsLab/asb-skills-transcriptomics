---
name: posterior-probability-cutoff-subtype-classification
description: Use when reproducing or comparing cancer molecular subtype classification
  rates on gene expression data, specifically to quantify how many samples remain
  unclassifiable under a posterior probability > 0.5 criterion (e.g., DeepCC vs. the
  273-gene random-forest CMS classifier).
when_to_use_negative:
- When the task is clustering cancer patients into novel subtypes rather than classifying
  samples into an existing subtype system — DeepCC is explicitly developed for classification,
  not clustering.
- When no pre-trained DeepCC model is available and the goal is de novo classifier
  construction — first train via train_DeepCC_model(fs, labels) instead of applying
  a posterior cutoff to a loaded model.
- When input is already a functional spectra / deep-feature table rather than raw
  gene expression profiles — the getFunctionalSpectra transformation step would be
  redundant or invalid.
edam_operation: http://edamontology.org/operation_2990
edam_topics:
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_3474
tools:
- name: DeepCC
  role: R framework providing load_DeepCC_model, getFunctionalSpectra/getFunctionalSpectrum,
    get_DeepCC_prob, and get_DeepCC_label for posterior-probability-based subtype
    classification
  repo: https://github.com/CityUHK-CompBio/DeepCC
- name: msigdbr
  role: R package supplying MSigDBr gene sets (via get_msigdbr()) used to generate
    functional spectra
  repo: https://github.com/CityUHK-CompBio/DeepCC
- name: R
  role: execution environment; the DeepCC model is a Keras model loaded via keras::load_model_hdf5
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1038/s41389-019-0157-8
    title: 'DeepCC: a novel deep learning-based framework for cancer molecular subtype
      classification'
  source_content_hash: sha256:3268128c1ceb898f4e2301f48171a74c397511b368437f0060a3798eee33ec1a
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/CityUHK-CompBio/DeepCC
  tools:
  - DeepCC
  - R
  - msigdbr
  edam_operation: http://edamontology.org/operation_2990
  edam_topics:
  - http://edamontology.org/topic_0203
  - http://edamontology.org/topic_3474
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 6
---

# posterior-probability-cutoff-subtype-classification

## Summary

Classify colorectal cancer samples into CMS subtypes using a pre-trained DeepCC model and flag samples as unclassifiable when the maximum posterior subtype probability falls at or below a 0.5 cutoff, enabling quantitative comparison of unclassifiable fractions across CRCSC data sets and the CIT/GSE39582 cohort.

## When to use

Use when reproducing or comparing cancer molecular subtype classification rates on gene expression data, specifically to quantify how many samples remain unclassifiable under a posterior probability > 0.5 criterion (e.g., DeepCC vs. the 273-gene random-forest CMS classifier).

## When NOT to use

- When the task is clustering cancer patients into novel subtypes rather than classifying samples into an existing subtype system — DeepCC is explicitly developed for classification, not clustering.
- When no pre-trained DeepCC model is available and the goal is de novo classifier construction — first train via train_DeepCC_model(fs, labels) instead of applying a posterior cutoff to a loaded model.
- When input is already a functional spectra / deep-feature table rather than raw gene expression profiles — the getFunctionalSpectra transformation step would be redundant or invalid.

## Inputs

- gene expression profile data.frame for CRC samples (rows = samples, columns = Entrez gene identifiers) from CRCSC-analyzed data sets
- CIT/GSE39582 cohort gene expression profiles (n=557)
- pre-trained CRC DeepCC model files: .RData (label levels) and .hdf5 (Keras model), e.g. loaded as CRC_TCGA or the CRCSC-trained model

## Outputs

- per-sample posterior subtype probability matrix from get_DeepCC_prob()
- per-sample predicted CMS subtype labels from get_DeepCC_label(deepcc_model, new_fs, cutoff = 0.5)
- per-data-set unclassifiable-count table (Fig. 3a equivalent) written to unclassifiable_counts.csv

## How to apply

Load the CRCSC-trained pre-trained CRC DeepCC model with load_DeepCC_model(), which restores the .RData label levels and the .hdf5 Keras model via keras::load_model_hdf5. Transform gene expression profiles of each CRCSC data set and the CIT/GSE39582 cohort (n=557) into functional spectra using getFunctionalSpectra/getFunctionalSpectrum with the default MSigDB v7 gene sets (22,596 gene sets). Compute posterior subtype probabilities for every sample with get_DeepCC_prob() and assign labels with get_DeepCC_label() using cutoff = 0.5. Flag samples whose maximum posterior probability is at or below the 0.5 cutoff as unclassifiable and tally unclassifiable counts per data set. Compare DeepCC unclassifiable fractions (~5% across CRCSC sets; 557-531=26 unclassified on CIT) against the 273-gene random-forest CMS classifier result (117 unclassifiable on CIT), writing the per-data-set table to unclassifiable_counts.csv.

## Related tools

- **DeepCC** (R framework providing load_DeepCC_model, getFunctionalSpectra/getFunctionalSpectrum, get_DeepCC_prob, and get_DeepCC_label for posterior-probability-based subtype classification) — https://github.com/CityUHK-CompBio/DeepCC
- **msigdbr** (R package supplying MSigDBr gene sets (via get_msigdbr()) used to generate functional spectra) — https://github.com/CityUHK-CompBio/DeepCC
- **R** (execution environment; the DeepCC model is a Keras model loaded via keras::load_model_hdf5)

## Examples

```
CRC_model <- load_DeepCC_model("CRC_CRCSC"); fs <- getFunctionalSpectra(eps); probs <- get_DeepCC_prob(CRC_model, fs); labels <- get_DeepCC_label(CRC_model, fs, cutoff = 0.5)
```

## Evaluation signals

- On CIT/GSE39582, DeepCC must classify 531 of 557 samples (26 unclassifiable) under the 0.5 cutoff, matching the reported figure.
- Across all CRCSC-analyzed data sets, the unclassifiable fraction must be ~5% under the posterior probability > 0.5 criterion.
- The DeepCC unclassifiable count must be far lower than the 117 unclassifiable samples reported for the 273-gene random-forest CMS classifier on CIT.
- Every sample with maximum posterior probability > 0.5 must receive a label from the model's restored .RData levels; samples at or below 0.5 must be counted as unclassifiable, not force-assigned.
- unclassifiable_counts.csv must contain one row per CRCSC data set plus CIT/GSE39582 with counts summing to the cohort sample sizes.

## Limitations

- Samples whose maximum posterior probability falls at or below the cutoff cannot be assigned a subtype; the cutoff choice (0.5) directly determines the unclassifiable fraction.
- DeepCC requires comprehensive gene set prior knowledge (e.g., full MSigDB v7, 22,596 gene sets) to avoid potential bias in the functional spectra.
- For single microarray samples, the TCGA RNA-Seq-based reference (e.g., COADREAD) may cause distribution mismatch; the README suggests inverseRescale and recommends a customized reference profile.
- DeepCC is designed for classification, not clustering of cancer patients.

## Evidence

- [methods] By default, DeepCC will use MSigDB v7 (22, 596 gene sets) to generate functional spectra.: "By default, DeepCC will use MSigDB v7 (22, 596 gene sets) to generate functional spectra."
- [abstract] deep features learned by DeepCC captured biological characteristics associated with distinct molecular subtypes... and reduced the number of unclassifiable samples: "deep features learned by DeepCC captured biological characteristics associated with distinct molecular subtypes, enabling more compact within-subtype distribution and between-subtype separation of"
- [readme] load_DeepCC_model restores the classifier and levels: "load(file = paste0(prefix, ".RData"))
  classifer <- keras::load_model_hdf5(filepath =paste0(prefix, ".hdf5"))"
- [methods] pred_labels <- get_DeepCC_label(deepcc_model, new_fs): "new_fs <- getFunctionalSpectra(new_eps)
pred_labels <- get_DeepCC_label(deepcc_model, new_fs)"
- [methods] get_DeepCC_label signature with cutoff parameter: "get_DeepCC_label(DeepCCModel, newData, cutoff, prob_mode, prob_raw)"
