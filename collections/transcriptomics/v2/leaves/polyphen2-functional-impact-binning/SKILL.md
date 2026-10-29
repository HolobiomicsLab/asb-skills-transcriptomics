---
name: polyphen2-functional-impact-binning
description: Use when building mutation-type feature matrices from COSMIC somatic
  mutation data where missense variants must be stratified by predicted functional
  impact (HiFI/MiFI/LoFI) before computing per-gene ratio-metric, entropy and frequency
  features for driver-gene classification.
when_to_use_negative:
- The input is already a processed feature table (e.g. the released FeatureMat.tsv
  from RamanLab/IdentifyTSGOG) — binning is only needed when regenerating features
  from raw mutation records
- Working with datasets that lack the fields needed to compute missense entropy/frequency
  features (as happened with the Tokheim et al. comparison data)
- The analysis requires background-mutation-rate-based driver detection rather than
  mutation-type-profile features (the cTaG feature approach is explicitly not a MutSigCV-style
  mutation-rate method)
edam_operation: http://edamontology.org/operation_0282
edam_topics:
- http://edamontology.org/topic_0622
tools:
- name: cTaG
  role: Downstream pan-cancer random-forest model whose 37-feature matrix depends
    on the HiFI/MiFI/LoFI missense binning
  repo: https://github.com/RamanLab/cTaG
- name: identifyTSGOG.py
  role: Module in the cTaG repository that builds the feature matrix, including the
    PolyPhen2-based missense impact classes
  repo: https://github.com/RamanLab/cTaG
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1038/s41598-021-04015-y
    title: Novel ratio-metric features enable the identification of new driver genes
      across cancer types
  source_content_hash: sha256:ada403727b6cb94ab55de9ab47d67080e9eb1075f6928a2f6703d3e1583a4848
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/RamanLab/cTaG
  tools:
  - cTaG
  - identifyTSGOG.py
  edam_operation: http://edamontology.org/operation_0282
  edam_topics:
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 6
---

# polyphen2-functional-impact-binning

## Summary

Stratifies missense mutations from COSMIC v79 somatic coding mutation data into predicted functional-impact classes (HiFI/MiFI/LoFI) using PolyPhen2-based scores. These binned classes feed the cTaG 37-feature per-gene/transcript matrix (ratio-metric plus entropy and frequency features) used to classify tumour suppressor genes (TSGs) and oncogenes (OGs).

## When to use

Use when building mutation-type feature matrices from COSMIC somatic mutation data where missense variants must be stratified by predicted functional impact (HiFI/MiFI/LoFI) before computing per-gene ratio-metric, entropy and frequency features for driver-gene classification.

## When NOT to use

- The input is already a processed feature table (e.g. the released FeatureMat.tsv from RamanLab/IdentifyTSGOG) — binning is only needed when regenerating features from raw mutation records
- Working with datasets that lack the fields needed to compute missense entropy/frequency features (as happened with the Tokheim et al. comparison data)
- The analysis requires background-mutation-rate-based driver detection rather than mutation-type-profile features (the cTaG feature approach is explicitly not a MutSigCV-style mutation-rate method)

## Inputs

- Pre-processed COSMIC v79 somatic coding mutation data (Zenodo doi:10.5281/zenodo.4153052), filtered to samples with fewer than 2000 mutations
- COSMIC-provided mutation type annotations per mutation
- PolyPhen2-based predicted functional-impact scores for missense variants

## Outputs

- Binned missense classes (HiFI/MiFI/LoFI) per gene/transcript feeding the 37-feature matrix
- FeatureMat.tsv — tab-separated table with one row per gene/transcript and 37 feature columns plus a gene identifier column
- Tissue-specific feature matrices (e.g. data/FeatureMat/tissues) for tissues with >1000 samples

## How to apply

Load the pre-processed COSMIC v79 somatic coding mutation data from the Zenodo deposit (doi:10.5281/zenodo.4153052) and retain only samples with fewer than 2000 mutations. Use the COSMIC-provided mutation type annotations to categorise each mutation into the 11 COSMIC mutation-type categories plus compound, LOF, damaging and benign classes. For missense mutations specifically, bin each variant by predicted functional impact into HiFI, MiFI, or LoFI using PolyPhen2-based scores; this binning is what distinguishes high- from low-impact missense in downstream features. Feed the binned missense classes together with the other mutation-type counts into the Methods formulas for ratio-metric, entropy, and frequency features to compute the 37 per-gene/transcript features (as implemented in the identifyTSGOG.py feature-matrix module). Write the result as a tab-separated FeatureMat.tsv with one row per retained gene/transcript; tissue-specific matrices (tissues with >1000 samples, e.g. breast, large_intestine, pancreas) are generated the same way via TSG_OG_tissueAnalysis.py.

## Related tools

- **cTaG** (Downstream pan-cancer random-forest model whose 37-feature matrix depends on the HiFI/MiFI/LoFI missense binning) — https://github.com/RamanLab/cTaG
- **identifyTSGOG.py** (Module in the cTaG repository that builds the feature matrix, including the PolyPhen2-based missense impact classes) — https://github.com/RamanLab/cTaG

## Examples

```
python code/identifyTSGOG.py
```

## Evaluation signals

- FeatureMat.tsv exists, is tab-separated, and contains exactly 37 feature columns plus a gene identifier column
- One row per gene/transcript retained after the <2000-mutation sample filter; no genes dropped or duplicated silently
- Every missense mutation in the filtered COSMIC v79 data is assigned to exactly one of the three impact classes (HiFI, MiFI, or LoFI)
- Recomputed feature values match the publicly released FeatureMat.tsv (github.com/RamanLab/IdentifyTSGOG/data/FeatureMat/FeatureMat.tsv) built from the same Zenodo input
- Tissue-specific matrices (for tissues with >1000 samples, e.g. breast, kidney, pancreas) reproduce the same 37-column schema

## Limitations

- The README does not restate the exact PolyPhen2 score thresholds defining HiFI/MiFI/LoFI; consult the paper's Methods formulas before reproducing
- Missense (and frameshift) entropy and frequency features could not be computed for the Tokheim et al. comparison data, limiting cross-dataset comparability
- Binning is tied to COSMIC v79-vintage data and COSMIC-provided mutation type annotations; other annotation sources may not map cleanly to the 11 categories plus compound/LOF/damaging/benign classes

## Evidence

- [methods] Bin missense mutations by predicted functional impact (HiFI/MiFI/LoFI) using PolyPhen2-based scores.: [evidence span withheld — anchor status: not_found]
- [readme] Feature matrix schema: "The mutation data was used to generate feature matrix were the rows and columns correspond to genes/transcripts and 37 features respectively."
- [readme] Sample filter: "Somatic coding mutation data from COSMIC was considered for samples less than 2000 mutations."
- [discussion] Feature scope limitation: "We used data from Tokheim et al. undefined19 to compare with other methods, but missense and frameshift entropy and frequency could not be calculated."
