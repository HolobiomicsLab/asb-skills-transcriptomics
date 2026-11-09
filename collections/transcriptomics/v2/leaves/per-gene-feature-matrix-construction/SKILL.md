---
name: per-gene-feature-matrix-construction
description: Use when raw pre-processed COSMIC v79 somatic coding mutation records
  must be converted into the paper's 37 ratio-metric, entropy and frequency features
  per gene/transcript for driver-gene classification.
when_to_use_negative:
- The processed feature matrix already exists (RamanLab/IdentifyTSGOG/data/FeatureMat/FeatureMat.tsv)
  and the goal is model training or prediction, not feature construction
- The input mutation dataset lacks the COSMIC mutation type annotations needed for
  the categorisation (e.g. the Tokheim et al. data, for which missense and frameshift
  entropy and frequency could not be calculated)
- The goal is background-mutation-rate-based driver detection (e.g. MutSigCV-style),
  which this feature-matrix approach deliberately avoids
edam_topics: []
tools:
- name: cTaG
  role: Provides the code (identifyTSGOG.py module) that builds the 37-feature matrix
    from COSMIC v79 mutation data
  repo: https://github.com/RamanLab/cTaG
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1038/s41598-021-04015-y
    title: Novel ratio-metric features enable the identification of new driver genes
      across cancer types
  source_content_hash: sha256:554dae597e141c0f6f2f47e2f3dd745de4b2833da886063aa3fa30f8b02320a6
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/RamanLab/cTaG
  tools:
  - cTaG
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9090909090909091
n_claims: 11
---

# per-gene-feature-matrix-construction

## Summary

Regenerates the 37-feature per-gene/transcript matrix from pre-processed COSMIC v79 somatic coding mutation data using cTaG's categorisation of 11 COSMIC mutation types plus compound/LOF/damaging/benign classes, PolyPhen2-based HiFI/MiFI/LoFI missense binning, and entropy/frequency formulas. This matrix is the training input for the cTaG random-forest TSG/OG classifier.

## When to use

Use when raw pre-processed COSMIC v79 somatic coding mutation records must be converted into the paper's 37 ratio-metric, entropy and frequency features per gene/transcript for driver-gene classification.

## When NOT to use

- The processed feature matrix already exists (RamanLab/IdentifyTSGOG/data/FeatureMat/FeatureMat.tsv) and the goal is model training or prediction, not feature construction
- The input mutation dataset lacks the COSMIC mutation type annotations needed for the categorisation (e.g. the Tokheim et al. data, for which missense and frameshift entropy and frequency could not be calculated)
- The goal is background-mutation-rate-based driver detection (e.g. MutSigCV-style), which this feature-matrix approach deliberately avoids

## Inputs

- Pre-processed COSMIC v79 somatic coding mutation data (Zenodo doi:10.5281/zenodo.4153052) with COSMIC mutation type annotations
- PolyPhen2-based functional impact scores for missense variants (HiFI/MiFI/LoFI binning)
- CGC TSG/OG gene label lists (github.com/RamanLab/IdentifyTSGOG/tree/master/data/cgc_genes) for row labelling
- Optional: primary tissue-of-origin annotation for tissue-specific subsetting

## Outputs

- Tab-separated pan-cancer feature matrix FeatureMat.tsv: one row per gene/transcript retained after the <2000-mutation sample filter, 37 feature columns plus gene identifier
- Tissue-specific feature matrices under data/FeatureMat/tissues for tissues with >1000 samples

## How to apply

Load the pre-processed COSMIC v79 somatic coding mutation data (Zenodo doi:10.5281/zenodo.4153052) and retain only samples with fewer than 2000 mutations. Categorise each mutation using the COSMIC-provided mutation type annotations into the 11 COSMIC mutation-type categories plus the compound, LOF, damaging and benign classes. Bin missense mutations by predicted functional impact into HiFI/MiFI/LoFI classes using PolyPhen2-based scores. For each gene/transcript compute the 37 features: ratio-metric features plus the entropy and frequency formulas for the different mutation types defined in the Methods, then assemble rows (genes/transcripts) x 37 columns and write as a tab-separated FeatureMat.tsv, matching the published matrix in RamanLab/IdentifyTSGOG/data/FeatureMat/FeatureMat.tsv. The same procedure, applied per primary tissue for tissues with greater than 1000 samples (breast, central_nervous_system, cervix, endometrium, haematopoietic_and_lymphoid_tissue, kidney, large_intestine, liver, pancreas, prostate), yields the tissue-specific feature matrices.

## Related tools

- **cTaG** (Provides the code (identifyTSGOG.py module) that builds the 37-feature matrix from COSMIC v79 mutation data) — https://github.com/RamanLab/cTaG

## Examples

```
python code/identifyTSGOG.py
```

## Evaluation signals

- Output file exists and is a tab-separated table with exactly 37 feature columns plus a gene identifier column
- One row per gene/transcript retained after the samples-less-than-2000-mutations filter
- Feature values are reproducible from the Zenodo deposit (doi:10.5281/zenodo.4153052) using the paper's categorisation, PolyPhen2 binning, and ratio-metric/entropy/frequency formulas
- Diff against the published FeatureMat.tsv in RamanLab/IdentifyTSGOG/data/FeatureMat/ matches row-wise
- For tissue-specific runs, only the ten tissues with >1000 samples (breast, central_nervous_system, cervix, endometrium, haematopoietic_and_lymphoid_tissue, kidney, large_intestine, liver, pancreas, prostate) produce matrices

## Limitations

- Filtering to samples with fewer than 2000 mutations restricts which genes/transcripts appear as rows
- Entropy/frequency features require specific annotation availability: they could not be computed for the Tokheim et al. data, limiting comparison overlap
- Feature definitions span ratio metrics plus entropy and frequency formulas; any ambiguity in the Methods formulas must be resolved from the deposited data and code

## Evidence

- [methods] Categorise each mutation into the 11 COSMIC mutation-type categories plus compound, LOF, damaging and benign classes, using COSMIC-provided mutation type annotations.: "Somatic coding mutation data from COSMIC was considered for samples less than 2000 mutations."
- [readme] The mutation data was used to generate feature matrix were the rows and columns correspond to genes/transcripts and 37 features respectively.: "The mutation data was used to generate feature matrix were the rows and columns correspond to genes/transcripts and 37 features respectively."
- [intro] The paper defines entropy and frequency features alongside ratio-metric features for TSG/OG classification.: "we define new features that calculate entropy and frequency of different mutation types along with other ratio-metric features"
- [methods] Tissue-specific feature matrices are generated only for tissues with greater than 1000 samples.: "Tissues with greater than 1000 samples were used."
