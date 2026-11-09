---
name: cross-method-consensus-scoring
description: Use when multiple per-method ligand-receptor score tables (e.g. truncated
  mean, weight_norm, LRscore, CellChat probability) are available and a single method-agnostic
  consensus ranking of interactions is needed.
when_to_use_negative:
- Methods that additionally infer intracellular processes, such as NicheNet, Cytotalk
  (full), and SoptSC, are not directly comparable and should not be naively pooled
  into the rank aggregate without harmonisation
- When a single method-resource combination is already validated for the question
  and only its raw interaction scores are needed — aggregation adds no information
- When no gold standard or per-method score tables are available; consensus ranking
  cannot substitute for method benchmarking
edam_operation: http://edamontology.org/operation_3802
edam_topics:
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_3170
tools:
- name: LIANA
  role: Executes the seven CCC methods, harmonises and ranks their scores, and applies
    Robust Rank Aggregation via liana_aggregate/rank_aggregate to produce the consensus
    ranking
  repo: https://github.com/saezlab/liana
- name: CellPhoneDB
  role: One of the seven constituent methods contributing truncated-mean scores to
    the consensus
- name: CellChat
  role: One of the seven constituent methods contributing probability scores to the
    consensus
- name: Connectome
  role: One of the seven constituent methods contributing weight_norm scores to the
    consensus
- name: NATMI
  role: One of the seven constituent methods contributing mean-expression edge weights
    to the consensus
  repo: https://github.com/asrhou/NATMI.git
- name: SingleCellSignalR
  role: One of the seven constituent methods contributing LRscore values to the consensus
  repo: https://github.com/SCA-IRCM/SingleCellSignalR_v1
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1038/s41467-022-30755-0
    title: Comparison of methods and resources for cell-cell communication inference
      from single-cell RNA-Seq data
  source_content_hash: sha256:5be70b3539c5aca3122f7ec44a32089b1de5952608bbd9c99f101c25e5c8237e
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/saezlab/liana
  tools:
  - CellChat
  - CellPhoneDB
  - Connectome
  - LIANA
  - NATMI
  - SingleCellSignalR
  edam_operation: http://edamontology.org/operation_3802
  edam_topics:
  - http://edamontology.org/topic_0203
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 6
---

# cross-method-consensus-scoring

## Summary

Produce a single consensus ranking of ligand-receptor interactions from multiple cell-cell communication methods (CellPhoneDB, Connectome, NATMI, SingleCellSignalR, CellChat, iTALK, Cytotalk) using Robust Rank Aggregation as implemented in LIANA, extracting interactions that are preferentially highly ranked across methods.

## When to use

Use when multiple per-method ligand-receptor score tables (e.g. truncated mean, weight_norm, LRscore, CellChat probability) are available and a single method-agnostic consensus ranking of interactions is needed.

## When NOT to use

- Methods that additionally infer intracellular processes, such as NicheNet, Cytotalk (full), and SoptSC, are not directly comparable and should not be naively pooled into the rank aggregate without harmonisation
- When a single method-resource combination is already validated for the question and only its raw interaction scores are needed — aggregation adds no information
- When no gold standard or per-method score tables are available; consensus ranking cannot substitute for method benchmarking

## Inputs

- Per-method ligand-receptor interaction score tables from the seven CCC methods (CellPhoneDB truncated mean, Connectome weight_norm, NATMI mean-expression edge weight, SingleCellSignalR LRscore, CellChat probability, iTALK logFC mean, Cytotalk crosstalk score)
- Union universe of all source-target-ligand-receptor tuples predicted by any method

## Outputs

- Aggregated table with an aggregate_rank column (values in [0, 1]) per source-target-ligand-receptor tuple, written to liana_rank_aggregate.csv

## How to apply

Load the per-method interaction score tables generated via LIANA (CellPhoneDB truncated mean, Connectome weight_norm, NATMI mean-expression edge weight, SingleCellSignalR LRscore, CellChat probability, iTALK logFC mean, Cytotalk crosstalk score) for every source-target-ligand-receptor tuple. Harmonise score orientation so that for every method smaller values denote stronger interactions, then convert each method's scores to ranks computed over the union universe of all interactions predicted by any method. Impute interactions missing from a given method's predictions with the maximum (worst) rank — the most conservative assignment — then apply Robust Rank Aggregation across the ranked columns to compute the aggregate_rank per tuple, exposed as liana_aggregate/rank_aggregate in LIANA. Rationale: the article shows both the choice of method and resource strongly influence predicted interactions, so the consensus extracts preferentially highly-ranked interactions from the distribution of per-method rankings.

## Related tools

- **LIANA** (Executes the seven CCC methods, harmonises and ranks their scores, and applies Robust Rank Aggregation via liana_aggregate/rank_aggregate to produce the consensus ranking) — https://github.com/saezlab/liana
- **CellPhoneDB** (One of the seven constituent methods contributing truncated-mean scores to the consensus)
- **CellChat** (One of the seven constituent methods contributing probability scores to the consensus)
- **Connectome** (One of the seven constituent methods contributing weight_norm scores to the consensus)
- **NATMI** (One of the seven constituent methods contributing mean-expression edge weights to the consensus) — https://github.com/asrhou/NATMI.git
- **SingleCellSignalR** (One of the seven constituent methods contributing LRscore values to the consensus) — https://github.com/SCA-IRCM/SingleCellSignalR_v1

## Examples

```
R: aggregate <- liana_aggregate(liana_res); write.csv(aggregate, 'liana_rank_aggregate.csv')
```

## Evaluation signals

- The output liana_rank_aggregate.csv contains an aggregate_rank column with values in [0, 1] for all tuples
- Every source-target-ligand-receptor tuple in the union universe is present in the aggregate table (no tuples dropped)
- For each method, scores have been reoriented so smaller values denote stronger interactions before ranking
- Interactions missing from a given method's predictions carry the maximum (worst) rank, not NaN or NA
- Aggregate ranks should approximately average the constituent method ranks for tuples consistently ranked across methods, and outperform ranks of tuples highly ranked by only one method

## Limitations

- No gold standard is available and the biological ground truth for CCC is largely unknown, so the consensus cannot identify a definitively best method
- The consensus depends on the choice of constituent methods and resources, which the article shows strongly influence predicted intercellular interactions
- Consensus rankings still reflect gene-expression proxies only — they do not capture endocrine signalling, protein translation/processing/secretion steps, or signalling mediated by non-transcripted molecules such as neurotransmitters

## Evidence

- [abstract] Consensus produced by Robust Rank Aggregation: [evidence span withheld — anchor status: not_found]
- [abstract] Methods and resources strongly influence predictions: "We then examine all possible combinations of methods and resources and show that both strongly influence the predicted intercellular interactions."
- [abstract] LIANA as open-source interface: "we provide LIANA, a LIgand-receptor ANalysis frAmework as an open-source interface to all the resources and methods"
- [readme] Any combination and consensus: "LIANA enables the use of any combination of ligand-receptor methods and resources, and their consensus."
- [intro] Systematic comparison including consensus: "we systematically compare 16 cell-cell communication inference resources and 7 methods, plus the consensus between the methods’ predictions"
