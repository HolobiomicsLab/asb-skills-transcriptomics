---
name: pretrained-neural-classifier-inference
description: Use when new gene expression profiles (a batch data.frame with Entrez
  ID columns, or a single-sample profile) must be classified with an already-trained
  DeepCC model without retraining, e.g., assigning CMS labels to colorectal cancer
  samples.
when_to_use_negative:
- You need unsupervised clustering of cancer patients — DeepCC is developed for classification
  but not clustering of cancer patients.
- Input is already a functional-spectrum/enrichment feature matrix rather than raw
  gene expression profiles.
- No reference expression profile of the same or similar cancer type/platform is available
  for a single microarray sample and a customized reference cannot be generated —
  rescaling against the RNA-Seq TCGA reference may be unreliable.
edam_operation: http://edamontology.org/operation_2990
edam_topics:
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_3474
- http://edamontology.org/topic_2640
tools:
- name: DeepCC
  role: supplies load_DeepCC_model, getFunctionalSpectra, getFunctionalSpectrum, get_DeepCC_label,
    get_DeepCC_prob, and get_DeepCC_features for pretrained-model inference
  repo: https://github.com/CityUHK-CompBio/DeepCC
- name: R
  role: runtime environment in which the DeepCC inference functions and Keras model
    loading are executed
- name: msigdbr
  role: provides MSigDB gene sets (via get_msigdbr()) used as an alternative geneSets
    source when computing functional spectra of new samples
- name: keras
  role: loads the pre-trained HDF5 classifier with keras::load_model_hdf5 and runs
    the neural network forward pass
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1038/s41389-019-0157-8
    title: 'DeepCC: a novel deep learning-based framework for cancer molecular subtype
      classification'
  source_content_hash: sha256:de1a00336b46709a32f6bfae873226bd38ff2119404fbc8a6bad55e15c936368
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/CityUHK-CompBio/DeepCC
  tools:
  - DeepCC
  - R
  - keras
  - msigdbr
  edam_operation: http://edamontology.org/operation_2990
  edam_topics:
  - http://edamontology.org/topic_0203
  - http://edamontology.org/topic_3474
  - http://edamontology.org/topic_2640
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.4
n_claims: 10
---

# pretrained-neural-classifier-inference

## Summary

Applies a pre-trained DeepCC Keras classifier to new gene expression data by first computing functional spectra (MSigDB v7, 22,596 gene sets by default) and then emitting subtype labels (e.g., CMS for colorectal cancer) or class probabilities, including a single-sample mode with reference-profile rescaling.

## When to use

Use when new gene expression profiles (a batch data.frame with Entrez ID columns, or a single-sample profile) must be classified with an already-trained DeepCC model without retraining, e.g., assigning CMS labels to colorectal cancer samples.

## When NOT to use

- You need unsupervised clustering of cancer patients — DeepCC is developed for classification but not clustering of cancer patients.
- Input is already a functional-spectrum/enrichment feature matrix rather than raw gene expression profiles.
- No reference expression profile of the same or similar cancer type/platform is available for a single microarray sample and a customized reference cannot be generated — rescaling against the RNA-Seq TCGA reference may be unreliable.

## Inputs

- gene expression profile data.frame (rows = samples, columns = Entrez gene identifiers), same format as training data
- pre-trained DeepCC model object (e.g., CRC_TCGA from prefix.RData + prefix.hdf5)
- reference expression profile for single-sample mode (pre-defined TCGA cancer-type identifier like 'COADREAD' or a customized profile)

## Outputs

- predicted subtype label per sample (member of the loaded model's class levels, e.g., CMS classes for CRC)
- class probabilities from get_DeepCC_prob (optional)
- deep features from get_DeepCC_features for visualization via vis_samples

## How to apply

First load the pre-trained model, e.g., load_DeepCC_model('CRC_TCGA'), which restores prefix.RData and the Keras HDF5 classifier via keras::load_model_hdf5, returning the classifier and its class levels. For a batch of new expression profiles, compute functional spectra with getFunctionalSpectra(new_eps) (MSigDB v7 with 22,596 gene sets by default; geneSets = 'MSigDBvx' for v5/v6/v7 or MSigDBr from msigdbr::get_msigdbr() can be passed), then call get_DeepCC_label(deepcc_model, new_fs) for labels or get_DeepCC_prob(deepcc_model, newData) for probabilities. For a single sample, provide a reference expression profile: new_fs_single <- getFunctionalSpectrum(new_ep, refExp = 'COADREAD'), which rescales the sample by linear-model (lm) fitting against the TCGA-derived average expression profile and computes enrichment scores over MSigDB gene sets; use a TCGA cancer-type identifier (COADREAD, BRCA, OV, etc.) or generate a customized reference from same/similar cancer type and platform. If the single sample is microarray data, enable inverseRescale to partially compensate for the distributional difference versus RNA-Seq-based TCGA references. Class labels come from the model's stored class levels; cutoff and prob_mode/prob_raw parameters of get_DeepCC_label control prediction behavior.

## Related tools

- **DeepCC** (supplies load_DeepCC_model, getFunctionalSpectra, getFunctionalSpectrum, get_DeepCC_label, get_DeepCC_prob, and get_DeepCC_features for pretrained-model inference) — https://github.com/CityUHK-CompBio/DeepCC
- **R** (runtime environment in which the DeepCC inference functions and Keras model loading are executed)
- **msigdbr** (provides MSigDB gene sets (via get_msigdbr()) used as an alternative geneSets source when computing functional spectra of new samples)
- **keras** (loads the pre-trained HDF5 classifier with keras::load_model_hdf5 and runs the neural network forward pass)

## Examples

```
CRC_TCGA <- load_DeepCC_model("CRC_TCGA"); new_fs_single <- getFunctionalSpectrum(new_ep, refExp = "COADREAD"); pred_label <- get_DeepCC_label(CRC_TCGA, new_fs_single)
```

## Evaluation signals

- Prediction is a single record (per sample) whose label is a member of the class levels stored in the loaded model (e.g., CRC_TCGA class levels).
- getFunctionalSpectrum output has the expected dimensionality: one enrichment score per MSigDB gene set (22,596 for default v7).
- Batch classification via getFunctionalSpectra + get_DeepCC_label matches the single-sample pathway getFunctionalSpectrum + get_DeepCC_label for the same sample when the reference is appropriate.
- get_DeepCC_prob returns per-class probabilities that are consistent with (argmax =) the emitted label.
- Inference was preceded by loading the model successfully (prefix.RData load plus keras::load_model_hdf5 of prefix.hdf5 without error).

## Limitations

- DeepCC is not applicable to clustering of cancer patients (classification only).
- DeepCC depends on comprehensive gene set prior knowledge; major characteristic signaling pathways for each subtype should be included in the gene set database to avoid potential bias.
- inverseRescale for microarray single samples only overcomes the distributional difference versus RNA-Seq TCGA reference 'a little bit'; a customized reference from the same/similar cancer type and platform is preferred.
- No isolated quantitative accuracy is reported for the single-sample mechanism; validation of single-sample predictions relies on label-level plausibility.
- Gene annotation must be Entrez identifiers, matching the training data format, or functional spectra cannot be computed consistently.

## Evidence

- [methods] classify single sample using reference expression profile: "new_fs_single <- getFunctionalSpectrum(new_ep, refExp = "COADREAD")
pred_label <- get_DeepCC_label(deepcc_model, new_fs_single)"
- [methods] load pre-trained CRC DeepCC model: "CRC_TCGA <- load_DeepCC_model("CRC_TCGA")"
- [readme] single sample rescaling rationale: "if your single sample is microarray data, we strongly suggest you turn the parameter `inverseRescale` on, since TCGA is RNA-Seq data"
- [methods] default gene set database: "By default, DeepCC will use MSigDB v7 (22, 596 gene sets) to generate functional spectra."
- [readme] model loading mechanism: "classifer <- keras::load_model_hdf5(filepath =paste0(prefix, ".hdf5"))
  list(classifier = classifer, levels = levels)"
