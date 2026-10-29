---
name: cancer-gene-census-reference-list-validation
description: Use when a driver-gene prediction method must be benchmarked for precision
  against curated reference driver sets (Bailey et al. pan-cancer list or CGC), especially
  to rank methods like cTaG, TUSON, 20/20+ and DriverNet on the same evaluation genes.
when_to_use_negative:
- The genes under evaluation are part of the 136 TSGs / 76 OGs used to train the cTaG
  model — these must be excluded from the Bailey et al. evaluation set, so precision
  on them would be inflated.
- The goal is recall or mutation-rate comparison (e.g. cTaG vs MutSigCV mutation-frequency
  analysis) rather than precision against curated driver lists.
- Missense and frameshift entropy/frequency features are required for the comparison
  method but cannot be computed from the Tokheim et al. data, limiting comparable
  overlap.
edam_operation: http://edamontology.org/operation_3501
edam_topics:
- http://edamontology.org/topic_2640
- http://edamontology.org/topic_0622
tools:
- name: cTaG
  role: Supplies the consensus RandomForest driver-gene predictions whose precision
    is evaluated
  repo: https://github.com/RamanLab/cTaG
- name: MutSigCV
  role: Provides mutation rates used for the mutation-rate-based comparison in the
    benchmark
- name: TUSON
  role: Comparison method; predictions taken from Tokheim et al. data at q-value <
    0.05
- name: 20/20+
  role: Comparison method; predictions taken from Tokheim et al. data at q-value <
    0.05
- name: DriverNet
  role: Comparison method; union across datasets with p < 0.05 taken from Tokheim
    et al. data
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1038/s41598-021-04015-y
    title: Novel ratio-metric features enable the identification of new driver genes
      across cancer types
  source_content_hash: sha256:3ce1fcee050e43bf700776592ff698a3b8f0e72d802a6f172b7a22cb71670273
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
yes_rate: 1.0
n_claims: 14
---

# cancer-gene-census-reference-list-validation

## Summary

Validates and compares predicted pan-cancer driver genes by computing precision TP/(TP+FP) against curated reference lists — the Bailey et al. pan-cancer driver list (excluding the 136 TSGs and 76 OGs used for training, leaving a 92-gene evaluation set) and the Cancer Gene Census (CGC) driver gene list — for cTaG, TUSON, 20/20+ and DriverNet.

## When to use

Use when a driver-gene prediction method must be benchmarked for precision against curated reference driver sets (Bailey et al. pan-cancer list or CGC), especially to rank methods like cTaG, TUSON, 20/20+ and DriverNet on the same evaluation genes.

## When NOT to use

- The genes under evaluation are part of the 136 TSGs / 76 OGs used to train the cTaG model — these must be excluded from the Bailey et al. evaluation set, so precision on them would be inflated.
- The goal is recall or mutation-rate comparison (e.g. cTaG vs MutSigCV mutation-frequency analysis) rather than precision against curated driver lists.
- Missense and frameshift entropy/frequency features are required for the comparison method but cannot be computed from the Tokheim et al. data, limiting comparable overlap.

## Inputs

- cTaG consensus pan-cancer driver-gene predictions (RandomForest output, consolidated results)
- Bailey et al. pan-cancer driver gene list
- Cancer Gene Census (CGC) driver gene list (from github.com/RamanLab/IdentifyTSGOG/tree/master/data/cgc_genes)
- Tokheim et al. comparison-method predictions (TUSON, 20/20+, DriverNet)
- MutSigCV-calculated mutation rates

## Outputs

- precision_comparison.csv with one row per method per reference list (cTaG, TUSON, 20/20+, DriverNet x Bailey 92-gene set, CGC list)
- Per-method precision TP/(TP+FP) values

## How to apply

Load the consensus pan-cancer RandomForest driver-gene predictions from the cTaG package's RandomForest output. Build the Bailey et al. evaluation set by excluding the 136 TSGs and 76 OGs used for training, yielding a 92-gene evaluation set; separately load the CGC driver gene list. For cTaG compute precision TP/(TP+FP) against each reference list. Retrieve comparison-method predictions from the Tokheim et al. data using the specified thresholds — TUSON with q-value < 0.05, 20/20+ with q-value < 0.05, and DriverNet as the union across datasets with p < 0.05 — and compute their precision against the same two lists; use MutSigCV-calculated mutation rates for the mutation-rate-based comparison. Tabulate per-method, per-list precision values into precision_comparison.csv and check the ranking reproduces the published result.

## Related tools

- **cTaG** (Supplies the consensus RandomForest driver-gene predictions whose precision is evaluated) — https://github.com/RamanLab/cTaG
- **MutSigCV** (Provides mutation rates used for the mutation-rate-based comparison in the benchmark)
- **TUSON** (Comparison method; predictions taken from Tokheim et al. data at q-value < 0.05)
- **20/20+** (Comparison method; predictions taken from Tokheim et al. data at q-value < 0.05)
- **DriverNet** (Comparison method; union across datasets with p < 0.05 taken from Tokheim et al. data)

## Examples

```
python code/consolidateModelPredictions.py  # then compute precision TP/(TP+FP) of cTaG predictions vs Bailey 92-gene and CGC lists into precision_comparison.csv
```

## Evaluation signals

- Against the Bailey et al. 92-gene list, the precision ranking must be cTaG > 20/20+ > TUSON > DriverNet.
- Against the CGC driver list, the precision ranking must be 20/20+ > TUSON > cTaG > DriverNet.
- The Bailey et al. evaluation set must contain 92 genes after excluding the 136 TSGs and 76 OGs used for training.
- precision_comparison.csv has exactly one row per method per reference list, and every precision value is TP/(TP+FP) in [0,1].

## Limitations

- Missense and frameshift entropy and frequency features could not be calculated for the Tokheim et al. data used for method comparison.
- Davoli et al. test set results were not published, so test-set performance comparison was not possible.
- cTaG's training genes (136 TSGs, 76 OGs) must be excluded from evaluation lists to avoid circularity.

## Evidence

- [results] Bailey ranking: "Based on driver genes listed by Bailey et al., cTaG performs best followed by 20/20+, TUSON and DriverNet."
- [results] CGC ranking: "For driver genes listed in CGC 20/20+ performs best followed by TUSON, cTaG and DriverNet"
- [discussion] Tokheim comparison data: "We used data from Tokheim et al. undefined19 to compare with other methods"
- [discussion] Tokheim feature limitation: "We used data from Tokheim et al. undefined19 to compare with other methods, but missense and frameshift entropy and frequency could not be calculated."
- [readme] training gene lists: "The genes were labelled as TSG or OG based on labels given by Cancer Gene Census (CGC), and the rest were marked Unlabelled."
