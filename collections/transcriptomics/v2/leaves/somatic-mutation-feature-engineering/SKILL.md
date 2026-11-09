---
name: somatic-mutation-feature-engineering
description: Use when raw somatic coding mutation data (e.g., COSMIC v79) must be
  converted into a gene-by-feature matrix for driver-gene classification, and feature
  definitions (ratio-metrics, entropy, frequency) need to be reproduced.
when_to_use_negative:
- The input is already a gene-by-feature matrix (e.g., the deposited FeatureMat.tsv)
  — use it directly for modelling instead of re-engineering features
- The downstream method relies on background mutation rate modelling (e.g., MutSigCV-style)
  rather than mutation-type feature profiles — cTaG's features deliberately avoid
  mutation-rate bias
- Mutations come from a source without COSMIC-style mutation type annotations or PolyPhen2
  impact predictions, so the 11 categories and HiFI/MiFI/LoFI bins cannot be defined
edam_operation: http://edamontology.org/operation_3936
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_2640
tools:
- name: cTaG
  role: Repository tool whose identifyTSGOG.py module builds the 37-feature matrix
    from somatic mutation data
  repo: https://github.com/RamanLab/cTaG
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1038/s41598-021-04015-y
    title: Novel ratio-metric features enable the identification of new driver genes
      across cancer types
  source_content_hash: sha256:781cdf2c62dd9c1ff6271a00aeedc24e85fbdf94ad42b819887b1ba0ac6e881a
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/RamanLab/cTaG
  tools:
  - cTaG
  edam_operation: http://edamontology.org/operation_3936
  edam_topics:
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_2640
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8
n_claims: 5
---

# somatic-mutation-feature-engineering

## Summary

Generates a per-gene 37-feature matrix from COSMIC v79 somatic coding mutation data using ratio-metric features plus entropy and frequency formulas over 11 COSMIC mutation-type categories (with compound, LOF, damaging, benign, and PolyPhen2-binned missense classes), as used to train the cTaG TSG/OG classifier.

## When to use

Use when raw somatic coding mutation data (e.g., COSMIC v79) must be converted into a gene-by-feature matrix for driver-gene classification, and feature definitions (ratio-metrics, entropy, frequency) need to be reproduced.

## When NOT to use

- The input is already a gene-by-feature matrix (e.g., the deposited FeatureMat.tsv) — use it directly for modelling instead of re-engineering features
- The downstream method relies on background mutation rate modelling (e.g., MutSigCV-style) rather than mutation-type feature profiles — cTaG's features deliberately avoid mutation-rate bias
- Mutations come from a source without COSMIC-style mutation type annotations or PolyPhen2 impact predictions, so the 11 categories and HiFI/MiFI/LoFI bins cannot be defined

## Inputs

- Pre-processed COSMIC v79 somatic coding mutation data (Zenodo doi:10.5281/zenodo.4153052)
- COSMIC mutation type annotations per mutation
- PolyPhen2-based predicted functional impact scores for missense mutations
- Gene/transcript identifiers

## Outputs

- FeatureMat.tsv — tab-separated matrix with one row per gene/transcript retained after the <2000-mutation sample filter and exactly 37 feature columns plus gene identifier column
- Tissue-specific feature matrices under data/FeatureMat/tissues for the 10 tissues with >1000 samples

## How to apply

Load the pre-processed COSMIC v79 somatic coding mutation data (Zenodo doi:10.5281/zenodo.4153052) and retain only samples with fewer than 2000 mutations, as per the paper's filter. Categorise each mutation using COSMIC-provided mutation type annotations into the 11 mutation-type categories plus compound, LOF, damaging and benign classes, and bin missense mutations by PolyPhen2-predicted functional impact into HiFI/MiFI/LoFI. For each gene/transcript, compute the 37 features from the ratio-metric features plus the entropy and frequency formulas for the different mutation types defined in the paper's Methods. Assemble rows as genes/transcripts and 37 columns as features, writing a tab-separated FeatureMat.tsv. For tissue-specific extensions, split samples by primary tissue of origin and keep only tissues with greater than 1000 samples (breast, central_nervous_system, cervix, endometrium, haematopoietic_and_lymphoid_tissue, kidney, large_intestine, liver, pancreas, prostate).

## Related tools

- **cTaG** (Repository tool whose identifyTSGOG.py module builds the 37-feature matrix from somatic mutation data) — https://github.com/RamanLab/cTaG

## Examples

```
python code/identifyTSGOG.py  # builds the 37-feature FeatureMat.tsv from filtered COSMIC v79 mutation data
```

## Evaluation signals

- Output FeatureMat.tsv exists, is tab-separated, and has exactly 37 feature columns plus a gene identifier column
- One row per gene retained after the samples-with-fewer-than-2000-mutations filter
- Ratio-metric, entropy and frequency feature values recomputed from the Zenodo deposit match the published FeatureMat.tsv values
- For tissue-specific matrices, each corresponds to one of the 10 tissues with >1000 samples (breast, central_nervous_system, cervix, endometrium, haematopoietic_and_lymphoid_tissue, kidney, large_intestine, liver, pancreas, prostate)
- Mutation categorisation covers the 11 COSMIC mutation-type categories plus compound, LOF, damaging and benign classes

## Limitations

- Only samples with fewer than 2000 mutations are considered; highly mutated samples are excluded
- Missense and frameshift entropy/frequency features could not be computed for some external data (e.g., Tokheim et al.), limiting comparison overlap
- The 37-feature definitions depend on COSMIC mutation type annotations and PolyPhen2 binning, so they are not directly portable to annotation schemes lacking these fields

## Evidence

- [methods] Somatic coding mutation data from COSMIC was considered for samples less than 2000 mutations.: "Somatic coding mutation data from COSMIC was considered for samples less than 2000 mutations."
- [intro] we define new features that calculate entropy and frequency of different mutation types along with other ratio-metric features: "we define new features that calculate entropy and frequency of different mutation types along with other ratio-metric features."
- [readme] The mutation data was used to generate feature matrix were the rows and columns correspond to genes/transcripts and 37 features respectively.: "The mutation data was used to generate feature matrix were the rows and columns correspond to genes/transcripts and 37 features respectively."
- [methods] Tissues with greater than 1000 samples were used.: "Tissues with greater than 1000 samples were used."
