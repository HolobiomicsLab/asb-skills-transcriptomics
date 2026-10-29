---
name: mutsigcv-mutation-rate-comparison
description: Use when reproducing or extending the precision benchmarking of cTaG
  against MutSigCV, TUSON, 20/20+ and DriverNet on the Bailey et al. 92-gene pan-cancer
  evaluation set and the CGC driver gene list.
when_to_use_negative:
- Evaluating tissue-specific cTaG predictions rather than the pan-cancer consensus
  model — this skill covers the consolidated pan-cancer comparison only
- No gold-standard driver list (Bailey et al. or CGC) is available, so precision TP/(TP+FP)
  cannot be computed against a reference set
- Tokheim et al. comparison-method outputs are unavailable, since missense/frameshift
  entropy and frequency features could not be calculated for that data in the original
  study
edam_topics:
- http://edamontology.org/topic_2648
- http://edamontology.org/topic_0622
tools:
- name: cTaG
  role: Provides the consensus RandomForest driver-gene predictions being evaluated
    for precision
  repo: https://github.com/RamanLab/cTaG
- name: MutSigCV
  role: Calculates mutation rates used for the mutation-rate-based comparison of predicted
    genes
- name: TUSON
  role: Feature-based comparison method applied with q-value < 0.05 threshold from
    Tokheim et al. data
- name: 20/20+
  role: Feature-based comparison method applied with q-value < 0.05 threshold from
    Tokheim et al. data
- name: DriverNet
  role: Network-based comparison method, taking the union across datasets with p <
    0.05
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1038/s41598-021-04015-y
    title: Novel ratio-metric features enable the identification of new driver genes
      across cancer types
  source_content_hash: sha256:895d678100f46960d4d9a688e7d122bf8c00d2b60ebc2e840c7fd9f27c27d7d0
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
  - http://edamontology.org/topic_2648
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 11
---

# mutsigcv-mutation-rate-comparison

## Summary

Reproduces the cTaG paper's benchmarking of driver-gene prediction precision: cTaG consensus RandomForest predictions are compared against TUSON, 20/20+ and DriverNet (with MutSigCV-calculated mutation rates) using the Bailey et al. pan-cancer driver list and the CGC driver gene list as gold standards.

## When to use

Use when reproducing or extending the precision benchmarking of cTaG against MutSigCV, TUSON, 20/20+ and DriverNet on the Bailey et al. 92-gene pan-cancer evaluation set and the CGC driver gene list.

## When NOT to use

- Evaluating tissue-specific cTaG predictions rather than the pan-cancer consensus model — this skill covers the consolidated pan-cancer comparison only
- No gold-standard driver list (Bailey et al. or CGC) is available, so precision TP/(TP+FP) cannot be computed against a reference set
- Tokheim et al. comparison-method outputs are unavailable, since missense/frameshift entropy and frequency features could not be calculated for that data in the original study

## Inputs

- cTaG consensus pan-cancer driver-gene predictions (RandomForest consolidated output)
- Bailey et al. pan-cancer driver gene list (minus 136 training TSGs and 76 training OGs, giving the 92-gene evaluation set)
- Cancer Gene Census (CGC) driver gene list
- Tokheim et al. comparison-method prediction data (TUSON, 20/20+, DriverNet)
- MutSigCV-calculated mutation rates

## Outputs

- precision_comparison.csv with one row per method per reference list (precision TP/(TP+FP))
- Method ranking against Bailey et al. list: cTaG > 20/20+ > TUSON > DriverNet
- Method ranking against CGC list: 20/20+ > TUSON > cTaG > DriverNet

## How to apply

Load the cTaG consensus driver-gene predictions from the package's RandomForest output (consolidated pan-cancer results via consolidateModelPredictions.py). Build the Bailey et al. evaluation set by excluding the 136 TSGs and 76 OGs used for training, yielding the 92-gene list, and separately load the CGC driver gene list from the repository's cgc_genes data. Apply the comparison-method thresholds from Tokheim et al.: TUSON with q-value < 0.05, 20/20+ with q-value < 0.05, and DriverNet as the union across datasets with p < 0.05, using MutSigCV-calculated mutation rates for the mutation-rate-based comparison. Compute precision TP/(TP+FP) per method against both reference lists and tabulate one row per method per reference list (precision_comparison.csv). The rationale is that the paper deliberately benchmarks against both a curated pan-cancer list (Bailey et al.) and CGC, and against both feature-based (TUSON, 20/20+) and network-based (DriverNet) methods.

## Related tools

- **cTaG** (Provides the consensus RandomForest driver-gene predictions being evaluated for precision) — https://github.com/RamanLab/cTaG
- **MutSigCV** (Calculates mutation rates used for the mutation-rate-based comparison of predicted genes)
- **TUSON** (Feature-based comparison method applied with q-value < 0.05 threshold from Tokheim et al. data)
- **20/20+** (Feature-based comparison method applied with q-value < 0.05 threshold from Tokheim et al. data)
- **DriverNet** (Network-based comparison method, taking the union across datasets with p < 0.05)

## Examples

```
python code/plotMutRate.py  # plots mutation rate for genes predicted by cTaG and MutSigCV (output/MutSigCVComparisson)
```

## Evaluation signals

- Reproduced ranking against Bailey et al. matches the paper: cTaG best, followed by 20/20+, TUSON, DriverNet
- Reproduced ranking against CGC matches the paper: 20/20+ best, followed by TUSON, cTaG, DriverNet
- Bailey et al. evaluation set contains exactly 92 genes after excluding the 136 TSGs and 76 OGs used for training
- Comparison thresholds applied correctly: TUSON q < 0.05, 20/20+ q < 0.05, DriverNet union across datasets with p < 0.05
- precision_comparison.csv has one row per method per reference list covering all four methods and both gold-standard lists

## Limitations

- Missense and frameshift entropy and frequency features could not be calculated for the Tokheim et al. comparison data, limiting the comparison overlap
- Davoli et al. test set results were not published, preventing a test-set performance comparison with that method
- cTaG uses binary classification (TSG vs OG), which the authors note as a limitation of the model being benchmarked

## Evidence

- [results] Based on driver genes listed by Bailey et al., cTaG performs best followed by 20/20+, TUSON and DriverNet.: "Based on driver genes listed by Bailey et al., cTaG performs best followed by 20/20+, TUSON and DriverNet."
- [results] For driver genes listed in CGC 20/20+ performs best followed by TUSON, cTaG and DriverNet: "For driver genes listed in CGC 20/20+ performs best followed by TUSON, cTaG and DriverNet"
- [results] Mutation rates were calculated using MutSigCV, a well-known driver gene predictor, which calculates mutation rates to identify driver genes.: "Mutation rates were calculated using MutSigCV, a well-known driver gene predictor, which calculates mutation rates to identify driver genes."
- [discussion] We used data from Tokheim et al. to compare with other methods, but missense and frameshift entropy and frequency could not be calculated.: "We used data from Tokheim et al. undefined19 to compare with other methods, but missense and frameshift entropy and frequency could not be calculated."
- [readme] MutSigCVComparisson     # Results of MutSigCV and comparisson of mutation rates: "MutSigCVComparisson     # Results of MutSigCV and comparisson of mutation rates"
