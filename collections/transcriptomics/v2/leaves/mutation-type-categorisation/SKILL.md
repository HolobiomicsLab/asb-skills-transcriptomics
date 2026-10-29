---
name: mutation-type-categorisation
description: Use when raw COSMIC somatic coding mutation data must be converted into
  per-gene mutation-type counts and missense functional-impact bins to construct the
  37-feature matrix for TSG/OG classification.
when_to_use_negative:
- Input is already the processed feature matrix (e.g. FeatureMat.tsv from the IdentifyTSGOG
  repository)
- Analysis requires background mutation rate modelling (e.g. MutSigCV-style) rather
  than mutation-type profile features
- Data is from Tokheim et al. style sources where missense and frameshift entropy/frequency
  features cannot be computed
edam_operation: http://edamontology.org/operation_0362
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_2640
tools:
- name: cTaG
  role: Downstream random-forest classifier that consumes the categorisation-derived
    37-feature matrix to classify genes as TSG or OG
  repo: https://github.com/RamanLab/cTaG
- name: identifyTSGOG.py
  role: Repository code module used for building the feature matrix from the categorised
    mutation data
  repo: https://github.com/RamanLab/cTaG
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1038/s41598-021-04015-y
    title: Novel ratio-metric features enable the identification of new driver genes
      across cancer types
  source_content_hash: sha256:266df41fb023251ce8bd1b043cf60994be20f8b695839aba9f1f735e70df0924
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/RamanLab/cTaG
  tools:
  - cTaG
  - identifyTSGOG.py
  edam_operation: http://edamontology.org/operation_0362
  edam_topics:
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_2640
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8571428571428571
n_claims: 14
---

# mutation-type-categorisation

## Summary

Categorises somatic coding mutations from COSMIC v79 into the paper's mutation-type classes (11 COSMIC mutation-type categories plus compound, LOF, damaging and benign) and bins missense mutations by predicted functional impact (HiFI/MiFI/LoFI via PolyPhen2), as the first step toward building the 37-feature per-gene matrix used by cTaG.

## When to use

Use when raw COSMIC somatic coding mutation data must be converted into per-gene mutation-type counts and missense functional-impact bins to construct the 37-feature matrix for TSG/OG classification.

## When NOT to use

- Input is already the processed feature matrix (e.g. FeatureMat.tsv from the IdentifyTSGOG repository)
- Analysis requires background mutation rate modelling (e.g. MutSigCV-style) rather than mutation-type profile features
- Data is from Tokheim et al. style sources where missense and frameshift entropy/frequency features cannot be computed

## Inputs

- Pre-processed COSMIC v79 somatic coding mutation data (Zenodo doi:10.5281/zenodo.4153052)
- COSMIC-provided mutation type annotations per mutation
- PolyPhen2-based predicted functional impact scores for missense mutations

## Outputs

- Per-gene/transcript mutation-type categorisation feeding the 37-feature matrix
- FeatureMat.tsv (tab-separated feature matrix: rows = genes/transcripts, 37 feature columns plus gene identifier)

## How to apply

Load the pre-processed COSMIC v79 somatic coding mutation data (deposited at Zenodo, doi:10.5281/zenodo.4153052) and retain only samples with fewer than 2000 mutations, as in the paper's Methods. Categorise each mutation using the COSMIC-provided mutation type annotations into the 11 COSMIC mutation-type categories plus compound, LOF, damaging and benign classes. Bin missense mutations by predicted functional impact into HiFI/MiFI/LoFI using PolyPhen2-based scores. For each gene/transcript, feed these categorised counts into the ratio-metric, entropy and frequency formulas defined in Methods to yield the 37 features, implemented in the repository's identifyTSGOG.py module ('Module for building feature matrix'). The rationale is that distinct mutation-type profiles distinguish TSGs from OGs, with nonsense mutations among the most important features for classification.

## Related tools

- **cTaG** (Downstream random-forest classifier that consumes the categorisation-derived 37-feature matrix to classify genes as TSG or OG) — https://github.com/RamanLab/cTaG
- **identifyTSGOG.py** (Repository code module used for building the feature matrix from the categorised mutation data) — https://github.com/RamanLab/cTaG

## Examples

```
python code/identifyTSGOG.py --input cosmic_v79_somatic.tsv --output data/FeatureMat/FeatureMat.tsv
```

## Evaluation signals

- Output FeatureMat.tsv exists, is tab-separated, and has exactly 37 feature columns plus a gene identifier column
- One row per gene/transcript retained after the <2000-mutation-per-sample filter
- Every mutation is assigned to one of the 11 COSMIC mutation-type categories or compound, LOF, damaging, benign classes
- All missense mutations fall into a HiFI/MiFI/LoFI bin derived from PolyPhen2 scores
- Regenerated feature values match the published FeatureMat.tsv in the IdentifyTSGOG repository when run on the Zenodo deposit

## Limitations

- Depends on COSMIC-provided mutation type annotations; alternative mutation callers' annotations may not map directly
- Missense and frameshift entropy/frequency features could not be computed for some external datasets (e.g. Tokheim et al. data), limiting comparisons
- Feature definitions must follow the paper's Methods exactly; the README notes the feature-matrix builder lives in the IdentifyTSGOG repository, which has been moved to cTaG

## Evidence

- [methods] Somatic coding mutation data from COSMIC was considered for samples less than 2000 mutations.: "Somatic coding mutation data from COSMIC was considered for samples less than 2000 mutations."
- [other] Categorise each mutation into 11 COSMIC categories plus compound/LOF/damaging/benign: [evidence span withheld — anchor status: not_found]
- [readme] Mutation type annotations used were given by COSMIC.: "The data was filtered. Mutation type annotations used were given by COSMIC."
- [readme] identifyTSGOG.py is the module for building the feature matrix: "identifyTSGOG.py                # Module for building feature matrix"
- [abstract] certain mutation types, such as nonsense mutations, are more important for classification: "We overcome overfitting and show that certain mutation types, such as nonsense mutations, are more important for classification."
