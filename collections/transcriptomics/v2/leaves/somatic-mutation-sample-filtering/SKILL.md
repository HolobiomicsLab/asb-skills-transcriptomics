---
name: somatic-mutation-sample-filtering
description: Use when starting from raw COSMIC somatic coding mutation records and
  you need to exclude hyper-mutated samples (>2000 mutations) or restrict to primary
  tissues with >1000 samples before driver-gene modeling.
when_to_use_negative:
- Input is already a pre-processed feature matrix (e.g. data/FeatureMat/FeatureMat.tsv)
  — filtering applies to raw COSMIC mutation records only
- Studying a rare cancer type or cohort with fewer than 1000 samples per tissue, where
  the tissue-size cutoff would exclude the data entirely (use the pan-cancer model
  instead)
- Analyses that intentionally include hyper-mutated samples, e.g. studies of mutator
  phenotypes, since the <2000-mutation cutoff removes them by design
tools:
- name: IdentifyTSGOG
  role: Provides the identifyTSGOG.py feature-building module and TSG_OG_tissueAnalysis.py,
    which operate on the filtered/tissue-split mutation data to generate per-tissue
    feature matrices
  repo: https://github.com/RamanLab/IdentifyTSGOG
- name: cTaG
  role: Downstream random-forest model that consumes the filtered tissue-specific
    feature matrices to classify unlabelled genes as TSG or OG
  repo: https://github.com/RamanLab/cTaG
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1038/s41598-021-04015-y
    title: Novel ratio-metric features enable the identification of new driver genes
      across cancer types
  source_content_hash: sha256:d578993ada1c162336402ac11857e0e9609016a707744fd89e78af5ef9ace36c
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/RamanLab/IdentifyTSGOG
  tools:
  - IdentifyTSGOG
  - cTaG
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.75
n_claims: 12
---

# somatic-mutation-sample-filtering

## Summary

Filter raw COSMIC somatic coding mutation data to samples with fewer than 2000 mutations and, for tissue-specific driver-gene analysis, retain only primary tissues with greater than 1000 samples before building per-tissue feature matrices.

## When to use

Use when starting from raw COSMIC somatic coding mutation records and you need to exclude hyper-mutated samples (>2000 mutations) or restrict to primary tissues with >1000 samples before driver-gene modeling.

## When NOT to use

- Input is already a pre-processed feature matrix (e.g. data/FeatureMat/FeatureMat.tsv) — filtering applies to raw COSMIC mutation records only
- Studying a rare cancer type or cohort with fewer than 1000 samples per tissue, where the tissue-size cutoff would exclude the data entirely (use the pan-cancer model instead)
- Analyses that intentionally include hyper-mutated samples, e.g. studies of mutator phenotypes, since the <2000-mutation cutoff removes them by design

## Inputs

- COSMIC (v79) somatic coding mutation records with per-sample mutation counts and primary-tissue-of-origin annotations
- Cancer Gene Census (CGC) TSG/OG label lists (used downstream, after filtering)

## Outputs

- Filtered COSMIC mutation dataset restricted to samples with <2000 mutations
- 10 tissue-specific sample subsets (breast, central_nervous_system, cervix, endometrium, haematopoietic_and_lymphoid_tissue, kidney, large_intestine, liver, pancreas, prostate), each with >1000 samples
- Per-tissue 37-feature matrices (genes/transcripts x ratio-metric and entropy mutation-type features) written to data/FeatureMat/tissues

## How to apply

Load the unfiltered COSMIC somatic coding mutation data (v79) and compute the number of mutations per sample; drop samples exceeding the 2000-mutation threshold to remove hyper-mutated outliers that would distort ratio-metric and entropy features. For tissue-specific analyses, split the retained samples by their primary tissue of origin as annotated by COSMIC and keep only tissues with greater than 1000 samples — this yields the 10 tissues: breast, central_nervous_system, cervix, endometrium, haematopoietic_and_lymphoid_tissue, kidney, large_intestine, liver, pancreas and prostate. The rationale is that the cTaG random-forest model uses ratio-metric and entropy features (37 features) rather than mutation frequency, but extreme sample mutation burdens still contaminate per-gene feature values; the tissue cutoff ensures each subset has enough samples for stable per-tissue feature matrices. The filtered, tissue-split data then feeds the identifyTSGOG feature-building code (TSG_OG_tissueAnalysis.py), which writes per-tissue feature matrices and predictions under output/TissueSpecificAnalyisis.

## Related tools

- **IdentifyTSGOG** (Provides the identifyTSGOG.py feature-building module and TSG_OG_tissueAnalysis.py, which operate on the filtered/tissue-split mutation data to generate per-tissue feature matrices) — https://github.com/RamanLab/IdentifyTSGOG
- **cTaG** (Downstream random-forest model that consumes the filtered tissue-specific feature matrices to classify unlabelled genes as TSG or OG) — https://github.com/RamanLab/cTaG

## Examples

```
python code/TSG_OG_tissueAnalysis.py  # splits filtered COSMIC samples (<2000 mutations) by primary tissue (>1000 samples) and builds per-tissue feature matrices
```

## Evaluation signals

- Every retained tissue subset has strictly more than 1000 samples; no retained tissue falls below the cutoff
- No sample in the filtered dataset has 2000 or more mutations
- The retained tissue set equals exactly the 10 expected tissues (breast, central_nervous_system, cervix, endometrium, haematopoietic_and_lymphoid_tissue, kidney, large_intestine, liver, pancreas, prostate)
- Each per-tissue feature matrix contains rows of genes/transcripts and exactly 37 ratio-metric/entropy feature columns
- Prediction files exist for all 10 tissues under output/TissueSpecificAnalyisis, each with genes labelled TSG or OG plus consensus scores

## Limitations

- The <2000-mutation cutoff discards hyper-mutated samples, so driver genes that recur mainly in such samples will be missed
- The >1000-sample tissue cutoff restricts tissue-specific analysis to large cohorts; small tissue subsets cannot be analyzed this way
- The cTaG binary classification treats each gene as TSG or OG only, which the authors note as a limitation of the model the filtering feeds

## Evidence

- [methods] sample mutation threshold: "Somatic coding mutation data from COSMIC was considered for samples less than 2000 mutations."
- [methods] tissue sample cutoff: "Tissues with greater than 1000 samples were used."
- [readme] readme tissue split: "The samples were divided based on their primary tissue for origin. Tissues with greater than 1000 samples were used."
- [results] tissue analysis objective: "The objective of predicting genes using a subset of data specific to tumour primary tissue source was to identify genes specific to a cancer type."
