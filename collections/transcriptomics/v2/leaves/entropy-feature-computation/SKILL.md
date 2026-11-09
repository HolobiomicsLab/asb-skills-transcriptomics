---
name: entropy-feature-computation
description: Use when raw somatic coding mutation data (e.g. COSMIC v79) must be converted
  into a per-gene/transcript feature matrix with entropy and frequency statistics
  for mutation types, for driver gene classification or feature-importance analysis.
when_to_use_negative:
- A feature matrix already exists (e.g. the published FeatureMat.tsv) and the goal
  is only to train/predict with cTaG via TSG_OG_Feat_final.py
- Working with Tokheim et al. data, where missense and frameshift entropy and frequency
  could not be calculated, limiting comparability
- The analysis requires background mutation rate methods (e.g. MutSigCV) rather than
  mutation-type feature profiles
edam_operation: http://edamontology.org/operation_2945
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_3474
tools:
- name: cTaG
  role: Downstream random-forest classifier that consumes the 37-feature matrix (including
    entropy/frequency features) to classify genes as TSGs or OGs
  repo: https://github.com/RamanLab/cTaG
- name: identifyTSGOG.py
  role: Code module in the cTaG repository for building the feature matrix from mutation
    data
  repo: https://github.com/RamanLab/cTaG
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1038/s41598-021-04015-y
    title: Novel ratio-metric features enable the identification of new driver genes
      across cancer types
  source_content_hash: sha256:042ff08a031a036641ab1e3fb37e87cc0e5c9a515c926adb206dc134074519d6
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/RamanLab/cTaG
  tools:
  - cTaG
  - identifyTSGOG.py
  edam_operation: http://edamontology.org/operation_2945
  edam_topics:
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_3474
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 6
---

# entropy-feature-computation

## Summary

Computes entropy and frequency features of different COSMIC mutation types per gene, combined with ratio-metric features (including PolyPhen2 HiFI/MiFI/LoFI missense bins) to build the 37-feature matrix that cTaG uses to classify genes as TSGs or OGs.

## When to use

Use when raw somatic coding mutation data (e.g. COSMIC v79) must be converted into a per-gene/transcript feature matrix with entropy and frequency statistics for mutation types, for driver gene classification or feature-importance analysis.

## When NOT to use

- A feature matrix already exists (e.g. the published FeatureMat.tsv) and the goal is only to train/predict with cTaG via TSG_OG_Feat_final.py
- Working with Tokheim et al. data, where missense and frameshift entropy and frequency could not be calculated, limiting comparability
- The analysis requires background mutation rate methods (e.g. MutSigCV) rather than mutation-type feature profiles

## Inputs

- Pre-processed COSMIC v79 somatic coding mutation data (Zenodo doi:10.5281/zenodo.4153052), filtered to samples with <2000 mutations
- COSMIC-provided mutation type annotations
- PolyPhen2-based predicted functional impact scores for missense mutations

## Outputs

- Tab-separated 37-feature matrix (FeatureMat.tsv) with one row per gene/transcript plus a gene identifier column
- Tissue-specific 37-feature matrices (for tissues with >1000 samples: breast, central_nervous_system, cervix, endometrium, haematopoietic_and_lymphoid_tissue, kidney, large_intestine, liver, pancreas, prostate)

## How to apply

Load pre-processed COSMIC v79 somatic coding mutation data (deposited at doi:10.5281/zenodo.4153052) and retain only samples with fewer than 2000 mutations, as in the paper's filter. Categorise each mutation using COSMIC-provided mutation-type annotations into the 11 COSMIC categories plus compound, LOF, damaging and benign classes. Bin missense mutations by predicted functional impact into HiFI/MiFI/LoFI classes using PolyPhen2-based scores. For each gene/transcript, compute the entropy and frequency formulas for the different mutation types defined in the Methods, together with the ratio-metric features, yielding exactly 37 features per row. Assemble the tab-separated FeatureMat.tsv with rows = genes/transcripts and 37 feature columns plus a gene identifier; this matrix feeds random-forest training in TSG_OG_Feat_final.py.

## Related tools

- **cTaG** (Downstream random-forest classifier that consumes the 37-feature matrix (including entropy/frequency features) to classify genes as TSGs or OGs) — https://github.com/RamanLab/cTaG
- **identifyTSGOG.py** (Code module in the cTaG repository for building the feature matrix from mutation data) — https://github.com/RamanLab/cTaG

## Examples

```
python code/identifyTSGOG.py
```

## Evaluation signals

- Output file is a tab-separated table with exactly 37 feature columns plus a gene identifier column
- One row per gene retained after the <2000-mutation sample filter; no duplicated gene/transcript rows
- Ratio of entropy/frequency features to ratio-metric features matches the Methods definitions; missense rows carry HiFI/MiFI/LoFI-derived values only
- Regenerated matrix can be compared against the published FeatureMat.tsv at github.com/RamanLab/IdentifyTSGOG/blob/master/data/FeatureMat/FeatureMat.tsv
- Tissue-specific matrices exist only for the ten tissues with greater than 1000 samples listed in the README

## Limitations

- Missense and frameshift entropy/frequency features could not be computed for the Tokheim et al. comparison data, limiting comparison overlap
- The model's binary TSG/OG classification limits interpretation of multi-functional driver genes
- Filtering to samples with fewer than 2000 mutations excludes hypermutated samples from feature computation

## Evidence

- [intro] we define new features that calculate entropy and frequency of different mutation types along with other ratio-metric features: "we define new features that calculate entropy and frequency of different mutation types along with other ratio-metric features."
- [methods] Somatic coding mutation data from COSMIC was considered for samples less than 2000 mutations: "Somatic coding mutation data from COSMIC was considered for samples less than 2000 mutations."
- [readme] The mutation data was used to generate feature matrix were the rows and columns correspond to genes/transcripts and 37 features respectively: "The mutation data was used to generate feature matrix were the rows and columns correspond to genes/transcripts and 37 features respectively."
- [other] Bin missense mutations by predicted functional impact (HiFI/MiFI/LoFI) using PolyPhen2-based scores: [evidence span withheld — anchor status: not_found]
