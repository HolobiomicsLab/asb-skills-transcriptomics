---
name: driver-gene-precision-metric-computation
description: Use when you have driver-gene prediction lists from one or more methods
  and need to quantify and rank their precision against a curated reference driver
  set such as the Bailey et al. pan-cancer list or CGC.
when_to_use_negative:
- Your method predictions were trained on or derived from the reference list itself
  without excluding the 136 TSGs and 76 OGs — precision would be inflated and the
  comparison invalid
- You lack the Tokheim et al. comparison predictions at the specified thresholds (q
  < 0.05 for TUSON and 20/20+; union p < 0.05 for DriverNet), so no apples-to-apples
  precision comparison is possible
- The goal is recall or F1 rather than precision — this workflow computes TP/(TP+FP)
  only
edam_operation: http://edamontology.org/operation_3501
edam_topics:
- http://edamontology.org/topic_2640
- http://edamontology.org/topic_0622
tools:
- name: cTaG
  role: Source of the consensus pan-cancer driver-gene predictions whose precision
    is being evaluated
  repo: https://github.com/RamanLab/cTaG
- name: MutSigCV
  role: Calculates mutation rates used as the mutation-rate-based comparison method
    in the precision comparison
- name: TUSON
  role: Comparison feature-based method; predictions taken at q-value < 0.05 from
    Tokheim et al. data
- name: 20/20+
  role: Comparison feature-based method; predictions taken at q-value < 0.05 from
    Tokheim et al. data
- name: DriverNet
  role: Comparison network-based method; union across datasets at p < 0.05 from Tokheim
    et al. data
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1038/s41598-021-04015-y
    title: Novel ratio-metric features enable the identification of new driver genes
      across cancer types
  source_content_hash: sha256:ec7b74a3560606f7994b7003c9238d15ce0e2bf61281baaca60d6f63a7e2b6e8
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
  edam_operation: http://edamontology.org/operation_3501
  edam_topics:
  - http://edamontology.org/topic_2640
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9
n_claims: 10
---

# driver-gene-precision-metric-computation

## Summary

Computes precision TP/(TP+FP) of predicted driver genes from multiple methods (cTaG, TUSON, 20/20+, DriverNet, MutSigCV-based rates) against curated reference lists (Bailey et al. pan-cancer drivers, CGC) to rank methods by prediction quality.

## When to use

Use when you have driver-gene prediction lists from one or more methods and need to quantify and rank their precision against a curated reference driver set such as the Bailey et al. pan-cancer list or CGC.

## When NOT to use

- Your method predictions were trained on or derived from the reference list itself without excluding the 136 TSGs and 76 OGs — precision would be inflated and the comparison invalid
- You lack the Tokheim et al. comparison predictions at the specified thresholds (q < 0.05 for TUSON and 20/20+; union p < 0.05 for DriverNet), so no apples-to-apples precision comparison is possible
- The goal is recall or F1 rather than precision — this workflow computes TP/(TP+FP) only

## Inputs

- cTaG consensus pan-cancer driver-gene predictions (RandomForest output)
- Bailey et al. pan-cancer driver gene list (minus 136 TSGs and 76 OGs used for training; 92-gene evaluation set)
- CGC (Cancer Gene Census) driver gene list
- Tokheim et al. comparison-method predictions (TUSON, 20/20+, DriverNet)
- MutSigCV-calculated mutation rates

## Outputs

- precision_comparison.csv with one row per method (cTaG, TUSON, 20/20+, DriverNet, MutSigCV) per reference list (Bailey et al., CGC) containing precision TP/(TP+FP)
- Ranking of methods by precision for each reference list

## How to apply

Load the cTaG consensus predictions from the RandomForest output (consolidated pan-cancer results, e.g. via consolidateModelPredictions.py), then build the evaluation sets: for Bailey et al., exclude the 136 TSGs and 76 OGs used for training to yield a 92-gene evaluation list; separately load the CGC driver list. For each comparison method apply its published threshold — TUSON q-value < 0.05, 20/20+ q-value < 0.05, DriverNet union across datasets with p < 0.05 — and use MutSigCV-calculated mutation rates for the mutation-rate-based comparison. Compute precision TP/(TP+FP) per method against both reference lists and tabulate into precision_comparison.csv with one row per method per reference list. The rationale is that training-set genes must be excluded to avoid inflating precision for cTaG, and method-specific thresholds must match those reported in Tokheim et al. data for a fair comparison.

## Related tools

- **cTaG** (Source of the consensus pan-cancer driver-gene predictions whose precision is being evaluated) — https://github.com/RamanLab/cTaG
- **MutSigCV** (Calculates mutation rates used as the mutation-rate-based comparison method in the precision comparison)
- **TUSON** (Comparison feature-based method; predictions taken at q-value < 0.05 from Tokheim et al. data)
- **20/20+** (Comparison feature-based method; predictions taken at q-value < 0.05 from Tokheim et al. data)
- **DriverNet** (Comparison network-based method; union across datasets at p < 0.05 from Tokheim et al. data)

## Examples

```
python code/consolidateModelPredictions.py  # consolidate RandomForest pan-cancer predictions, then compute precision TP/(TP+FP) vs Bailey et al. 92-gene and CGC lists into precision_comparison.csv
```

## Evaluation signals

- For the Bailey et al. list, the reproduced ranking must be cTaG best, followed by 20/20+, TUSON, and DriverNet
- For the CGC list, the reproduced ranking must be 20/20+ best, followed by TUSON, cTaG, and DriverNet
- The Bailey et al. evaluation set must contain exactly 92 genes after excluding the 136 TSGs and 76 OGs used for training
- precision_comparison.csv contains one row per method per reference list with precision values in [0, 1]
- Every prediction list uses the correct method-specific threshold (q < 0.05 for TUSON and 20/20+, p < 0.05 union for DriverNet)

## Limitations

- Missense and frameshift entropy/frequency features could not be computed for the Tokheim et al. data, limiting the overlap of features available for comparison
- Precision against CGC is affected by genes like ZNF750 that are established drivers but absent from the CGC driver gene list
- Comparison is restricted to methods with published predictions in Tokheim et al.; Davoli et al. test-set results were not published, preventing test-set performance comparison

## Evidence

- [results] Based on driver genes listed by Bailey et al., cTaG performs best followed by 20/20+, TUSON and DriverNet.: "Based on driver genes listed by Bailey et al., cTaG performs best followed by 20/20+, TUSON and DriverNet."
- [results] For driver genes listed in CGC 20/20+ performs best followed by TUSON, cTaG and DriverNet: "For driver genes listed in CGC 20/20+ performs best followed by TUSON, cTaG and DriverNet"
- [results] Mutation rates were calculated using MutSigCV, a well-known driver gene predictor: "Mutation rates were calculated using MutSigCV, a well-known driver gene predictor, which calculates mutation rates to identify driver genes."
- [discussion] We used data from Tokheim et al. undefined19 to compare with other methods, but missense and frameshift entropy and frequency could not be calculated.: "We used data from Tokheim et al. undefined19 to compare with other methods, but missense and frameshift entropy and frequency could not be calculated."
- [readme] The top-level directories contain code, data and output folders.: "The top-level directories contain code, data and output folders."
