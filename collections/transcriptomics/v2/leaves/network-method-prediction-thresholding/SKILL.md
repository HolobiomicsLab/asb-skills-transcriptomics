---
name: network-method-prediction-thresholding
description: Use when benchmarking pan-cancer driver-gene predictions against published
  methods (TUSON, 20/20+, DriverNet) using precision TP/(TP+FP) against reference
  driver lists such as Bailey et al. or CGC.
when_to_use_negative:
- The goal is driver discovery on new data rather than benchmarking already-generated
  predictions against published driver lists
- Recall or sensitivity comparison is required — the workflow defines performance
  solely as precision TP/(TP+FP) against reference lists
- Comparing test-set performance against Davoli et al., whose test set results were
  not published
edam_topics:
- http://edamontology.org/topic_0622
tools:
- name: cTaG
  role: Random-forest model whose consolidated pan-cancer driver predictions are the
    method under evaluation
  repo: https://github.com/RamanLab/cTaG
- name: MutSigCV
  role: Provides the mutation rates used for the mutation-rate-based comparison of
    predicted genes
- name: TUSON
  role: Feature-based comparison method; predictions taken at q-value < 0.05 from
    the Tokheim et al. data
- name: 20/20+
  role: Feature-based comparison method; predictions taken at q-value < 0.05 from
    the Tokheim et al. data
- name: DriverNet
  role: Network-based comparison method; predictions taken as the union across datasets
    with p < 0.05
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1038/s41598-021-04015-y
    title: Novel ratio-metric features enable the identification of new driver genes
      across cancer types
  source_content_hash: sha256:7ec2e134fd69ffe3f8424d1a7c88c155be8a8d11adf3371711f174cfdc38d908
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
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9230769230769231
n_claims: 13
---

# network-method-prediction-thresholding

## Summary

Reproduces the precision comparison of cTaG's predicted driver genes against the feature-based methods TUSON and 20/20+ and the network-based method DriverNet (with MutSigCV-calculated mutation rates), evaluated against the Bailey et al. pan-cancer driver list and the CGC driver gene list.

## When to use

Use when benchmarking pan-cancer driver-gene predictions against published methods (TUSON, 20/20+, DriverNet) using precision TP/(TP+FP) against reference driver lists such as Bailey et al. or CGC.

## When NOT to use

- The goal is driver discovery on new data rather than benchmarking already-generated predictions against published driver lists
- Recall or sensitivity comparison is required — the workflow defines performance solely as precision TP/(TP+FP) against reference lists
- Comparing test-set performance against Davoli et al., whose test set results were not published

## Inputs

- cTaG consensus pan-cancer driver-gene predictions (RandomForest consolidated output)
- Bailey et al. pan-cancer driver gene list (minus 136 training TSGs and 76 training OGs, leaving 92 evaluation genes)
- Cancer Gene Census (CGC) driver gene list
- Tokheim et al. comparison-method prediction data (TUSON, 20/20+, DriverNet)
- MutSigCV-calculated mutation rates

## Outputs

- Per-method, per-reference-list precision values (TP/(TP+FP))
- Tabulated comparison table (e.g. precision_comparison.csv) with one row per method per reference list
- Method ranking against Bailey et al. and CGC reference lists

## How to apply

Load the cTaG consensus driver-gene predictions from the package's RandomForest output (consolidated pan-cancer results via consolidateModelPredictions.py). Build the evaluation sets: take the Bailey et al. pan-cancer driver list and exclude the 136 TSGs and 76 OGs used for training, yielding a 92-gene evaluation set, and load the CGC driver gene list. Compute precision TP/(TP+FP) for cTaG against both reference lists. Retrieve the comparison-method predictions from the Tokheim et al. data using the paper's thresholds: TUSON with q-value < 0.05, 20/20+ with q-value < 0.05, and DriverNet as the union across datasets with p < 0.05, using MutSigCV-calculated mutation rates for the mutation-rate-based comparison. Compute precision for each comparison method against the same two reference lists and tabulate per-method, per-list results (e.g. precision_comparison.csv). Validation is a ranking check: against Bailey et al., cTaG should rank best followed by 20/20+, TUSON and DriverNet; against CGC, 20/20+ should rank best followed by TUSON, cTaG and DriverNet.

## Related tools

- **cTaG** (Random-forest model whose consolidated pan-cancer driver predictions are the method under evaluation) — https://github.com/RamanLab/cTaG
- **MutSigCV** (Provides the mutation rates used for the mutation-rate-based comparison of predicted genes)
- **TUSON** (Feature-based comparison method; predictions taken at q-value < 0.05 from the Tokheim et al. data)
- **20/20+** (Feature-based comparison method; predictions taken at q-value < 0.05 from the Tokheim et al. data)
- **DriverNet** (Network-based comparison method; predictions taken as the union across datasets with p < 0.05)

## Examples

```
python code/consolidateModelPredictions.py  # consolidate pan-cancer RandomForest consensus, then precision = TP/(TP+FP) per method vs Bailey 92-gene and CGC lists
```

## Evaluation signals

- Against the Bailey et al. pan-cancer list, the reproduced ranking must be: cTaG best, followed by 20/20+, TUSON, DriverNet
- Against the CGC driver list, the reproduced ranking must be: 20/20+ best, followed by TUSON, cTaG, DriverNet
- The Bailey evaluation set must contain 92 genes after excluding the 136 training TSGs and 76 training OGs
- Each precision value equals TP/(TP+FP) computed only against the specified reference list and method threshold (q < 0.05 for TUSON and 20/20+; union p < 0.05 for DriverNet)
- Output table contains exactly one row per method per reference list (e.g. precision_comparison.csv)

## Limitations

- Comparison coverage is limited: missense and frameshift entropy and frequency features could not be computed for the Tokheim et al. data
- Performance is assessed only as precision against two reference lists (Bailey et al. pan-cancer and CGC); recall is not part of this comparison
- Davoli et al. test-set performance could not be compared because their test set results were not published

## Evidence

- [results] Bailey ranking: "Based on driver genes listed by Bailey et al., cTaG performs best followed by 20/20+, TUSON and DriverNet."
- [results] CGC ranking: "For driver genes listed in CGC 20/20+ performs best followed by TUSON, cTaG and DriverNet"
- [discussion] Comparison source data: "We used data from Tokheim et al. undefined19 to compare with other methods"
- [readme] Feature matrix and comparison workflow: "The mutation data was used to generate feature matrix were the rows and columns correspond to genes/transcripts and 37 features respectively."
- [readme] Consolidated predictions code: "consolidateModelPredictions.py  # Consolidate results for pan-cancer analysis"
