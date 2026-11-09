---
name: cancer-driver-benchmark-method-comparison
description: Use when comparing predicted pan-cancer driver genes from cTaG (or similar
  RandomForest-based TSG/OG classifiers) against established tools like TUSON, 20/20+
  and DriverNet using curated driver lists such as Bailey et al. and CGC.
when_to_use_negative:
- When a recall or test-set performance comparison against Davoli et al. is required
  — their test set results were not published, so only precision against curated lists
  is feasible.
- When the goal is per-cancer-type (tissue-specific) driver prediction rather than
  pan-cancer method benchmarking — the comparison here uses consolidated pan-cancer
  results.
- When missense/frameshift entropy and frequency features must be computed for Tokheim
  et al. data — these could not be calculated, limiting the comparison overlap.
edam_operation: http://edamontology.org/operation_1812
edam_topics:
- http://edamontology.org/topic_2640
- http://edamontology.org/topic_0622
tools:
- name: cTaG
  role: Source of the predicted driver genes being benchmarked (RandomForest consensus
    pan-cancer model)
  repo: https://github.com/RamanLab/cTaG
- name: TUSON
  role: Comparison method; predictions taken from Tokheim et al. data with q-value
    < 0.05
- name: 20/20+
  role: Comparison method; predictions taken from Tokheim et al. data with q-value
    < 0.05
- name: DriverNet
  role: Comparison method; predictions taken as the union across datasets with p <
    0.05
- name: MutSigCV
  role: Provides mutation rates used for the mutation-rate-based comparison with cTaG
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1038/s41598-021-04015-y
    title: Novel ratio-metric features enable the identification of new driver genes
      across cancer types
  source_content_hash: sha256:23c8876a25a3ec6fe60a45be9fa615d89a1e9fb0cfb3d23b0a57f945bd51f21a
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/RamanLab/cTaG
  tools:
  - 20/20+
  - DriverNet
  - MutSigCV
  - TUSON
  - cTaG
  edam_operation: http://edamontology.org/operation_1812
  edam_topics:
  - http://edamontology.org/topic_2640
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7777777777777778
n_claims: 9
---

# cancer-driver-benchmark-method-comparison

## Summary

A benchmarking skill that reproduces the precision comparison of cTaG's predicted driver genes against TUSON, 20/20+, DriverNet (and MutSigCV-calculated mutation rates) using the Bailey et al. pan-cancer driver list and the CGC driver gene list as reference standards.

## When to use

Use when comparing predicted pan-cancer driver genes from cTaG (or similar RandomForest-based TSG/OG classifiers) against established tools like TUSON, 20/20+ and DriverNet using curated driver lists such as Bailey et al. and CGC.

## When NOT to use

- When a recall or test-set performance comparison against Davoli et al. is required — their test set results were not published, so only precision against curated lists is feasible.
- When the goal is per-cancer-type (tissue-specific) driver prediction rather than pan-cancer method benchmarking — the comparison here uses consolidated pan-cancer results.
- When missense/frameshift entropy and frequency features must be computed for Tokheim et al. data — these could not be calculated, limiting the comparison overlap.

## Inputs

- cTaG consensus RandomForest driver-gene predictions (consolidated pan-cancer results)
- Bailey et al. pan-cancer validated driver gene list
- CGC (Cancer Gene Census) driver gene list
- Tokheim et al. comparison-method prediction data (TUSON, 20/20+, DriverNet)
- MutSigCV-calculated mutation rates

## Outputs

- precision_comparison.csv with one row per method (cTaG, TUSON, 20/20+, DriverNet) per reference list (Bailey et al. 92-gene set, CGC list)
- Per-method precision values TP/(TP+FP)

## How to apply

Load cTaG's consensus driver-gene predictions from the RandomForest output (consolidated pan-cancer results via consolidateModelPredictions.py). Build the evaluation set from the Bailey et al. pan-cancer list by excluding the 136 TSGs and 76 OGs used for training, yielding a 92-gene set, and separately use the CGC driver gene list. Compute precision TP/(TP+FP) for cTaG against both reference lists. For the comparison methods, retrieve predictions from the Tokheim et al. data with thresholds: TUSON q-value < 0.05, 20/20+ q-value < 0.05, and DriverNet taking the union across datasets with p < 0.05; use MutSigCV-calculated mutation rates for the mutation-rate-based comparison. Tabulate per-method precision into precision_comparison.csv and verify the reproduced ranking matches the reported orderings.

## Related tools

- **cTaG** (Source of the predicted driver genes being benchmarked (RandomForest consensus pan-cancer model)) — https://github.com/RamanLab/cTaG
- **TUSON** (Comparison method; predictions taken from Tokheim et al. data with q-value < 0.05)
- **20/20+** (Comparison method; predictions taken from Tokheim et al. data with q-value < 0.05)
- **DriverNet** (Comparison method; predictions taken as the union across datasets with p < 0.05)
- **MutSigCV** (Provides mutation rates used for the mutation-rate-based comparison with cTaG)

## Examples

```
python code/consolidateModelPredictions.py  # consolidate pan-cancer RandomForest predictions, then compute TP/(TP+FP) per method vs Bailey et al. 92-gene and CGC lists into precision_comparison.csv
```

## Evaluation signals

- Against the Bailey et al. 92-gene list, the reproduced ranking is: cTaG best, followed by 20/20+, TUSON, DriverNet.
- Against the CGC driver gene list, the reproduced ranking is: 20/20+ best, followed by TUSON, cTaG, DriverNet.
- precision_comparison.csv exists and contains exactly one row per method per reference list with precision values in [0,1].
- The Bailey et al. evaluation set contains 92 genes after excluding the 136 TSGs and 76 OGs used for training.
- Comparison-method thresholds are correctly applied (q < 0.05 for TUSON and 20/20+; p < 0.05 union across datasets for DriverNet).

## Limitations

- Davoli et al. test set results were not published, so test-set performance could not be compared.
- Missense and frameshift entropy and frequency features could not be calculated for the Tokheim et al. data, limiting the comparison overlap.
- cTaG uses binary classification of TSGs and OGs, which the authors note as a limitation of the model being benchmarked.

## Evidence

- [results] Bailey ranking: "Based on driver genes listed by Bailey et al., cTaG performs best followed by 20/20+, TUSON and DriverNet."
- [results] CGC ranking: "For driver genes listed in CGC 20/20+ performs best followed by TUSON, cTaG and DriverNet"
- [discussion] Tokheim comparison: "We used data from Tokheim et al. undefined19 to compare with other methods, but missense and frameshift entropy and frequency could not be calculated."
- [results] Mutation rate comparison: "We analysed the mutation frequencies of the predicted genes. Mutation rates were calculated using MutSigCV"
- [readme] Consolidation script: "consolidateModelPredictions.py  # Consolidate results for pan-cancer analysis"
