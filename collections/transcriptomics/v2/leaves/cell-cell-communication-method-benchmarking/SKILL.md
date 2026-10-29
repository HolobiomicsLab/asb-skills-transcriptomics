---
name: cell-cell-communication-method-benchmarking
description: Use when single-cell RNA-Seq data must be interrogated for ligand-receptor
  interactions with multiple CCC methods, or when per-method interaction scores need
  to be combined into a robust consensus ranking via LIANA.
when_to_use_negative:
- The methods of interest additionally infer intracellular processes (e.g. NicheNet,
  Cytotalk, SoptSC), which the article states 'are not directly comparable' to the
  seven ligand-receptor methods.
- The goal is to identify a definitive best method — no gold standard for CCC exists,
  so the benchmark cannot give a definitive answer of what method is best.
- The signalling of interest is long-distance endocrine communication, which scRNA-Seq
  gene expression within a dataset does not capture.
edam_topics:
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_0203
tools:
- name: LIANA
  role: Framework that wraps all seven CCC methods, decouples them from resources,
    and exposes the Robust Rank Aggregate consensus (liana_aggregate/rank_aggregate)
  repo: https://github.com/saezlab/liana
- name: CellPhoneDB
  role: Source of truncated mean scores (average expression of transmitter and receivers)
    that are harmonised, ranked, and fed to the RRA consensus
- name: Connectome
  role: Source of weight_norm scores derived from the product of normalised transmitter
    and receiver expression, ranked for the consensus
- name: NATMI
  role: Source of mean-expression edge weight scores over ConnectomeDB interactions,
    ranked for the consensus
  repo: https://github.com/asrhou/NATMI
- name: SingleCellSignalR
  role: Source of LRscore (regularised score from squared transmitter/receiver expression
    over LRdb), ranked for the consensus
- name: CellChat
  role: Source of probability scores over CellChatDB interactions, ranked for the
    consensus
- name: iTALK
  role: iTALK-inspired logFC mean scores (logged one-versus-all fold change means),
    ranked for the consensus
- name: Cytotalk
  role: Cytotalk-inspired crosstalk scores derived from transmitter and receiver expression,
    ranked for the consensus
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1038/s41467-022-30755-0
    title: Comparison of methods and resources for cell-cell communication inference
      from single-cell RNA-Seq data
  source_content_hash: sha256:5a484cb8ccb7db3c9dd9fbac74c9aa1a425d80b90aa609c6dfced4ec2657101b
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/saezlab/liana
  tools:
  - CellChat
  - CellPhoneDB
  - Connectome
  - Cytotalk
  - LIANA
  - NATMI
  - SingleCellSignalR
  - iTALK
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3308
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.75
n_claims: 12
---

# cell-cell-communication-method-benchmarking

## Summary

Benchmark cell-cell communication inference by running seven ligand-receptor scoring methods (CellPhoneDB truncated mean, Connectome weight_norm, NATMI mean-expression edge weight, SingleCellSignalR LRscore, CellChat probability, iTALK logFC mean, Cytotalk crosstalk score) through LIANA and merging them with Robust Rank Aggregation into one consensus ranking per source-target-ligand-receptor tuple.

## When to use

Use when single-cell RNA-Seq data must be interrogated for ligand-receptor interactions with multiple CCC methods, or when per-method interaction scores need to be combined into a robust consensus ranking via LIANA.

## When NOT to use

- The methods of interest additionally infer intracellular processes (e.g. NicheNet, Cytotalk, SoptSC), which the article states 'are not directly comparable' to the seven ligand-receptor methods.
- The goal is to identify a definitive best method — no gold standard for CCC exists, so the benchmark cannot give a definitive answer of what method is best.
- The signalling of interest is long-distance endocrine communication, which scRNA-Seq gene expression within a dataset does not capture.

## Inputs

- Single-cell RNA-Seq expression matrix with cell-type labels for source and target cell populations
- Per-method interaction score tables from LIANA (CellPhoneDB truncated mean, Connectome weight_norm, NATMI mean-expression edge weight, SingleCellSignalR LRscore, CellChat probability, iTALK logFC mean, Cytotalk crosstalk score)
- Ligand-receptor resource(s): CellChatDB, CellPhoneDB, Ramilowski, ConnectomeDB, LRdb, etc.

## Outputs

- liana_rank_aggregate.csv with one row per source-target-ligand-receptor tuple and an aggregate_rank column with values in [0, 1]
- Per-method harmonised rank tables over the union universe of predicted interactions
- Agreement statistics of method predictions with spatial colocalisation, cytokine activities, and receptor protein abundance

## How to apply

Run the seven scoring methods through LIANA, decoupling each method from its default resource so any combination of the 16 resources (e.g. CellChatDB, CellPhoneDB, Ramilowski, ConnectomeDB, LRdb) can be tested. Harmonise score orientation so that for every method smaller values denote stronger interactions (magnitude ranks), then convert each method's scores to ranks computed over the union universe of all interactions predicted by any method. Impute interactions missing from a given method's predictions with the maximum (worst) rank — the most conservative assignment — before applying Robust Rank Aggregation across the ranked columns to compute aggregate_rank per source-target-ligand-receptor tuple, as exposed by liana_aggregate/rank_aggregate in LIANA. Write the result to liana_rank_aggregate.csv and verify aggregate_rank lies in [0, 1] for all tuples. Because both the choice of method and resource strongly influence predicted interactions, evaluate agreement of predictions with independent modalities such as spatial colocalisation, cytokine activities, and receptor protein abundance rather than claiming one method is best.

## Related tools

- **LIANA** (Framework that wraps all seven CCC methods, decouples them from resources, and exposes the Robust Rank Aggregate consensus (liana_aggregate/rank_aggregate)) — https://github.com/saezlab/liana
- **CellPhoneDB** (Source of truncated mean scores (average expression of transmitter and receivers) that are harmonised, ranked, and fed to the RRA consensus)
- **Connectome** (Source of weight_norm scores derived from the product of normalised transmitter and receiver expression, ranked for the consensus)
- **NATMI** (Source of mean-expression edge weight scores over ConnectomeDB interactions, ranked for the consensus) — https://github.com/asrhou/NATMI
- **SingleCellSignalR** (Source of LRscore (regularised score from squared transmitter/receiver expression over LRdb), ranked for the consensus)
- **CellChat** (Source of probability scores over CellChatDB interactions, ranked for the consensus)
- **iTALK** (iTALK-inspired logFC mean scores (logged one-versus-all fold change means), ranked for the consensus)
- **Cytotalk** (Cytotalk-inspired crosstalk scores derived from transmitter and receiver expression, ranked for the consensus)

## Examples

```
liana_agg <- rank_aggregate(liana_wrap(sce)); write.csv(liana_agg, "liana_rank_aggregate.csv")
```

## Evaluation signals

- liana_rank_aggregate.csv exists and the aggregate_rank column is present with values in [0, 1] for all source-target-ligand-receptor tuples.
- Every method's score column is re-oriented so that smaller values denote stronger interactions (magnitude ranks) before ranking.
- Interactions missing from a given method's predictions carry the maximum (worst) rank, and ranking is computed over the union universe of all interactions predicted by any method.
- Predictions are checked for coherence with spatial colocalisation, cytokine activities, and receptor protein abundance.
- Re-running with a different method-resource combination changes the interaction predictions, confirming both method and resource were varied as intended.

## Limitations

- No gold standard is available and the biological ground truth for CCC is largely unknown, so no definitive best method can be identified.
- scRNA-Seq gene expression is limited to cells within the dataset and does not capture long-distance endocrine signalling events.
- Using the number of interactions between two cell types as a proxy for communication intensity is likely biased by the choice of CCC inference method; methods inferring intracellular processes (NicheNet, Cytotalk, SoptSC) are not directly comparable.

## Evidence

- [abstract] we systematically compare 16 cell-cell communication inference resources and 7 methods, plus the consensus between the methods’ predictions: "we systematically compare 16 cell-cell communication inference resources and 7 methods, plus the consensus between the methods’ predictions"
- [intro] All results were generated using LIANA: "All results were generated using LIANA—a LIgand-receptor ANalysis frAmework (Fig. 1; available at https://github.com/saezlab/liana)"
- [abstract] we assess the agreement of cell-cell communication methods with spatial colocalisation, cytokine activities, and receptor protein abundance and find that predictions are generally coherent with those: "we assess the agreement of cell-cell communication methods with spatial colocalisation, cytokine activities, and receptor protein abundance and find that predictions are generally coherent with those"
- [discussion] No gold standard; analyses cannot identify a best method: "Since a gold standard is currently not available and the biological ground truth is largely unknown2,25, our analyses cannot give a definitive answer of what method is best."
- [readme] LIANA enables any combination of methods and resources plus their consensus: "LIANA enables the use of any combination of ligand-receptor methods and resources, and their consensus."
