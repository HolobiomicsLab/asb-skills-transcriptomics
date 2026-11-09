---
name: rank-normalisation-and-missing-value-imputation
description: Use when combining interaction scores from multiple CCC methods with
  incomparable magnitudes and differing interaction coverage into a single consensus
  ranking, e.g. as the preprocessing stage before Robust Rank Aggregation in LIANA.
when_to_use_negative:
- Scores are already on a comparable scale and every method reports the same interaction
  universe — direct averaging may be appropriate instead of ranking
- The goal is to compare the individual methods rather than merge them; LIANA was
  designed to expose each method separately before any consensus
- Methods infer intracellular processes (e.g. NicheNet, Cytotalk native workflow,
  SoptSC) that are not directly comparable as interaction scores
edam_operation: http://edamontology.org/operation_2928
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: LIANA
  role: Provides rank_aggregate/liana_aggregate, which harmonises score orientation,
    ranks over the union universe, imputes worst ranks, and applies Robust Rank Aggregation
  repo: https://github.com/saezlab/liana
- name: CellPhoneDB
  role: Source of truncated-mean interaction scores to be rank-normalised
- name: NATMI
  role: Source of mean-expression edge weight scores to be rank-normalised
  repo: https://github.com/asrhou/NATMI
- name: CellChat
  role: Source of probability scores to be rank-normalised
- name: SingleCellSignalR
  role: Source of LRscore interaction scores to be rank-normalised
  repo: https://github.com/SCA-IRCM/SingleCellSignalR_v1
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1038/s41467-022-30755-0
    title: Comparison of methods and resources for cell-cell communication inference
      from single-cell RNA-Seq data
  source_content_hash: sha256:c28efb00c51aedd55e5872c378f2898dfb713494e39898703f23fb11f9a4de38
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/saezlab/liana
  tools:
  - CellChat
  - CellPhoneDB
  - LIANA
  - NATMI
  - SingleCellSignalR
  edam_operation: http://edamontology.org/operation_2928
  edam_topics:
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.875
n_claims: 8
---

# Rank normalisation and missing-value imputation

## Summary

Prepares heterogeneous ligand-receptor interaction scores (e.g. CellPhoneDB truncated mean, NATMI mean-expression edge weight, SingleCellSignalR LRscore, CellChat probability, Cytotalk crosstalk score) for consensus aggregation in LIANA by orienting scores so smaller values denote stronger interactions, converting them to ranks over the union universe of interactions, and imputing interactions missing from a method with the maximum (worst) rank.

## When to use

Use when combining interaction scores from multiple CCC methods with incomparable magnitudes and differing interaction coverage into a single consensus ranking, e.g. as the preprocessing stage before Robust Rank Aggregation in LIANA.

## When NOT to use

- Scores are already on a comparable scale and every method reports the same interaction universe — direct averaging may be appropriate instead of ranking
- The goal is to compare the individual methods rather than merge them; LIANA was designed to expose each method separately before any consensus
- Methods infer intracellular processes (e.g. NicheNet, Cytotalk native workflow, SoptSC) that are not directly comparable as interaction scores

## Inputs

- Per-method ligand-receptor interaction score tables from LIANA (one column per method: truncated mean, weight_norm, mean-expression edge weight, LRscore, probability, logFC mean, crosstalk score)
- Union universe of all ligand-receptor interactions predicted by any method, keyed by source-target-ligand-receptor tuple

## Outputs

- Rank-normalised score matrix with one ranked column per method over the union interaction universe
- Table with imputed worst-rank entries for interactions absent from a method's predictions
- liana_rank_aggregate.csv containing the aggregate_rank column in [0, 1] per tuple

## How to apply

Load the per-method interaction score tables generated via LIANA (CellPhoneDB truncated mean, Connectome weight_norm, NATMI mean-expression edge weight, SingleCellSignalR LRscore, CellChat probability, iTALK logFC mean, Cytotalk crosstalk score). Harmonise score orientation so that for every method smaller values denote stronger interactions. Convert each method's scores to ranks computed over the union universe of all interactions predicted by any method, so ranks are comparable across methods despite different scales. Impute interactions missing from a given method's predictions with the maximum (worst) rank — the most conservative assignment — because methods disagree on both scores and coverage. The resulting ranked columns feed Robust Rank Aggregation to produce aggregate_rank per source-target-ligand-receptor tuple.

## Related tools

- **LIANA** (Provides rank_aggregate/liana_aggregate, which harmonises score orientation, ranks over the union universe, imputes worst ranks, and applies Robust Rank Aggregation) — https://github.com/saezlab/liana
- **CellPhoneDB** (Source of truncated-mean interaction scores to be rank-normalised)
- **NATMI** (Source of mean-expression edge weight scores to be rank-normalised) — https://github.com/asrhou/NATMI
- **CellChat** (Source of probability scores to be rank-normalised)
- **SingleCellSignalR** (Source of LRscore interaction scores to be rank-normalised) — https://github.com/SCA-IRCM/SingleCellSignalR_v1

## Examples

```
liana_agg <- rank_aggregate(liana_res); write.csv(liana_agg, "liana_rank_aggregate.csv", row.names = FALSE)
```

## Evaluation signals

- All scores are reoriented such that smaller values denote stronger interactions for every method column
- Missing interactions for a method carry the maximum (worst) rank rather than NA or zero
- All methods are ranked over the identical union universe of interactions, so rank ranges are aligned
- The aggregate_rank column is present with values in [0, 1] for all source-target-ligand-receptor tuples in liana_rank_aggregate.csv

## Limitations

- Missing-value imputation with the worst rank is conservative: an interaction predicted by only one method can never outrank an interaction agreed upon by many methods on coverage alone
- Both the choice of method and resource strongly influence predicted intercellular interactions, so the consensus inherits bias from the method/resource panel included
- No gold standard is available for CCC, so the correctness of the consensus cannot be validated against biological ground truth

## Evidence

- [abstract] LIANA exposes each method and the consensus: "we systematically compare 16 cell-cell communication inference resources and 7 methods, plus the consensus between the methods’ predictions"
- [abstract] Method and resource choice affects predictions: "We then examine all possible combinations of methods and resources and show that both strongly influence the predicted intercellular interactions."
- [readme] LIANA combines methods and their consensus: "LIANA enables the use of any combination of ligand-receptor methods and resources, and their consensus."
