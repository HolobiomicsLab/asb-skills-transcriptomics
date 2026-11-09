---
name: functional-spectra-computation
description: Use when raw gene-level expression matrices (rows = samples, columns
  = Entrez IDs) must be converted to pathway-activity features before training or
  applying a DeepCC classifier, or when classifying a single sample that requires
  a reference expression profile.
when_to_use_negative:
- The input is already a functional spectra matrix — proceed directly to train_DeepCC_model
  or get_DeepCC_label instead of recomputing spectra.
- The goal is unsupervised clustering of cancer patients rather than supervised classification
  — DeepCC and its spectra pipeline are 'specifically developed for addressing current
  challenges in classification but not clustering of cancer patients'.
- Only subtype-defining signature genes (no genome-wide expression matrix) are available
  — signature gene lists cannot yield the comprehensive gene-set activities the transformation
  depends on.
edam_topics:
- http://edamontology.org/topic_0203
tools:
- name: DeepCC
  role: Provides getFunctionalSpectra/getFunctionalSpectrum and calcEnrichmentScore
    to compute gene-set activity features, plus pre-defined TCGA reference expression
    profiles for single samples
  repo: https://github.com/CityUHK-CompBio/DeepCC
- name: msigdbr
  role: R package supplying alternative MSigDB gene set collections via get_msigdbr()
    for the geneSets parameter
- name: keras
  role: 'Downstream dependency: the computed functional spectra are the training input
    for the keras ANN (train_DeepCC_model); R keras must be installed and runnable'
provenance:
  source_task_ids:
  - task_003
  - task_004
  - task_002
  source_papers:
  - doi: 10.1038/s41389-019-0157-8
    title: 'DeepCC: a novel deep learning-based framework for cancer molecular subtype
      classification'
  source_content_hash: sha256:1e7b91e073660986b714f6ddaf198525e47420892604525972b841120eb4efb9
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/CityUHK-CompBio/DeepCC
  tools:
  - DeepCC
  - R
  - keras
  - msigdbr
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5555555555555556
n_claims: 9
---

# functional-spectra-computation

## Summary

Transforms gene expression profiles into functional spectra — per-sample quantification of MSigDB gene set (pathway) activities — which serve as the platform-independent feature space that DeepCC trains on, replacing signature-gene inputs.

## When to use

Use when raw gene-level expression matrices (rows = samples, columns = Entrez IDs) must be converted to pathway-activity features before training or applying a DeepCC classifier, or when classifying a single sample that requires a reference expression profile.

## When NOT to use

- The input is already a functional spectra matrix — proceed directly to train_DeepCC_model or get_DeepCC_label instead of recomputing spectra.
- The goal is unsupervised clustering of cancer patients rather than supervised classification — DeepCC and its spectra pipeline are 'specifically developed for addressing current challenges in classification but not clustering of cancer patients'.
- Only subtype-defining signature genes (no genome-wide expression matrix) are available — signature gene lists cannot yield the comprehensive gene-set activities the transformation depends on.

## Inputs

- data.frame of gene expression profiles (rows = patient samples, columns named with Entrez gene identifiers)
- character vector of training labels (NA allowed; DeepCC ignores them automatically)
- geneSets parameter: 'MSigDBv5'/'MSigDBv6'/'MSigDBv7' or MSigDBr object from msigdbr::get_msigdbr()
- reference expression profile for single-sample mode (pre-defined TCGA identifier such as 'COADREAD', or a customized profile)
- getFunctionalSpectrum options: geneSets, refExp, logChange, inverseRescale, filter

## Outputs

- functional spectra matrix (one row per sample; 22,596 gene-set activity columns under default MSigDB v7) consumable by train_DeepCC_model, get_DeepCC_label, and get_DeepCC_features
- single-sample functional spectrum vector for get_DeepCC_label prediction

## How to apply

Pass a data.frame of gene expression profiles (each row one patient sample, column names Entrez identifiers) to getFunctionalSpectra; by default MSigDB v7 (22,596 gene sets) is used, with geneSets = 'MSigDBv5'/'MSigDBv6'/'MSigDBv7' or MSigDBr from msigdbr::get_msigdbr() selectable for other versions (v5.0: 10,348 sets; v6.0: 17,779 sets). For a single sample, use getFunctionalSpectrum with refExp set to a TCGA cancer-type identifier (e.g., 'COADREAD') or a customized reference profile from the same/similar cancer type and platform; for microarray single samples enable inverseRescale since the TCGA reference is RNA-Seq. Enrichment per gene set is computed via calcEnrichmentScore(geneList, geneSet), and the resulting spectra matrix feeds train_DeepCC_model or get_DeepCC_label. Robustness rationale: functional spectra retain high classification performance (balanced accuracy > 90%) even when computed from as few as 2000 most-variable genes, buffering cross-platform gene-annotation discrepancies.

## Related tools

- **DeepCC** (Provides getFunctionalSpectra/getFunctionalSpectrum and calcEnrichmentScore to compute gene-set activity features, plus pre-defined TCGA reference expression profiles for single samples) — https://github.com/CityUHK-CompBio/DeepCC
- **msigdbr** (R package supplying alternative MSigDB gene set collections via get_msigdbr() for the geneSets parameter)
- **keras** (Downstream dependency: the computed functional spectra are the training input for the keras ANN (train_DeepCC_model); R keras must be installed and runnable)

## Examples

```
fs <- getFunctionalSpectra(eps)
```

## Evaluation signals

- Output spectra dimensionality matches the chosen gene set collection: 22,596 columns for default MSigDB v7, 17,779 for v6.0, 10,348 for v5.0.
- Column names of the input expression data.frame are valid Entrez identifiers; non-Entrez columns cause gene-set mapping failures.
- Downstream validation: a DeepCC model trained on spectra retains balanced accuracy > 90% when spectra are recomputed from random 2000-gene subsets ranked by variance, and error degrades only gradually as subset size shrinks.
- For single-sample microarray input against the RNA-Seq TCGA reference, inverseRescale is enabled; predictions without it should be treated with suspicion due to platform distribution mismatch.
- A single-sample call without refExp (or without a customized same-cancer-type/same-platform reference) is incomplete — getFunctionalSpectrum must return an error or the invocation is wrong.

## Limitations

- Prior gene set knowledge must be comprehensive to avoid potential bias; major characteristic signaling pathways or biological processes for each cancer subtype should be present in the gene set database.
- Single-sample classification against the TCGA RNA-Seq reference from microarray data is imperfect: inverseRescale 'can overcome this distribution differently a little bit, but it's better to generate your customized reference'.
- The transformation does not enable clustering — it serves supervised classification only.

## Evidence

- [methods] By default, DeepCC will use MSigDB v7 (22, 596 gene sets) to generate functional spectra.: "By default, DeepCC will use MSigDB v7 (22, 596 gene sets) to generate functional spectra."
- [methods] getFunctionalSpectrum(expressionProfile, geneSets, refExp, logChange, inverseRescale, filter): "getFunctionalSpectrum(expressionProfile, geneSets, refExp, logChange, inverseRescale, filter)"
- [readme] Add parameter `geneSets = "MSigDBvx"` (x = 5/6/7) or `geneSets = MSigDBr` (MSigDBr attained from R package `msigdbr` with function get_msigdbr() in DeepCC): "Add parameter `geneSets = "MSigDBvx"` (x = 5/6/7) or `geneSets = MSigDBr` (MSigDBr attained from R package `msigdbr` with function get_msigdbr() in DeepCC)"
- [readme] for a given single sample, you have to provide a reference expression profile.: "for a given single sample, you have to provide a reference expression profile."
- [readme] if your single sample is microarray data, we strongly suggest you turn the parameter `inverseRescale` on, since TCGA is RNA-Seq data: "if your single sample is microarray data, we strongly suggest you turn the parameter `inverseRescale` on, since TCGA is RNA-Seq data"
