---
name: cell-cell-communication-benchmarking
description: Use when comparing ligand-receptor scoring rules for cell-cell communication
  inference on scRNA-seq data and you need an apples-to-apples benchmark on the same
  interaction database, same input matrix, and same permutation/significance pipeline.
when_to_use_negative:
- Do not use when the goal is to apply NeuronChat for inference on a new neural dataset
  rather than to comparatively evaluate scoring rules — use the standard NeuronChat
  pipeline instead
- Do not use when methods would be evaluated on their own native interaction databases;
  the benchmark requires all methods to be scored on the identical NeuronChatDB ligand-target
  pairs and identical input expression data
- Do not use for protein-ligand paracrine signaling contexts where small-molecule
  neurotransmitter signaling (glutamate, GABA) is not relevant and existing protein-based
  databases suffice
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_0203
tools:
- name: NeuronChat
  role: Provides the shared computational workflow (createNeuronChat, run_NeuronChat,
    net_aggregation) into which all three scoring rules are reimplemented for the
    benchmark; also the method expected to attain the highest AUROC/AUPRC
  repo: https://github.com/Wei-BioMath/NeuronChat
- name: CellChat
  role: Its scoring rule is reimplemented within the NeuronChat workflow, computing
    ligand abundance without functional gene grouping, to serve as a benchmark comparator
    on NeuronChatDB entries
- name: CellPhoneDB
  role: Its scoring rule is reimplemented within the same workflow (AND logic via
    geometric mean or minimum over all contributing genes) as a second benchmark comparator
- name: NeuronChatAnalysis2022
  role: Repository of scripts used to produce the results of the NeuronChat study,
    including benchmark figure reproduction
  repo: https://github.com/Wei-BioMath/NeuronChatAnalysis2022
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1038/s41467-023-36800-w
    title: Inferring neuron-neuron communications from single-cell transcriptomics
      through NeuronChat
  source_content_hash: sha256:23a016d7fdcbf589611793ec5222d267a6a0f63e5f2b339b3535db2cc589efee
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/Wei-BioMath/NeuronChat
  tools:
  - CellChat
  - CellPhoneDB
  - NeuronChat
  - NeuronChatAnalysis2022
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9090909090909091
n_claims: 11
---

# cell-cell-communication-benchmarking

## Summary

Fairly benchmark cell-cell communication inference scoring rules (NeuronChat vs reimplemented CellChat and CellPhoneDB) by running all three within one shared computational workflow on the identical NeuronChatDB ligand-target database and input expression data, then comparing AUROC and AUPRC on aggregated and per-interaction communication networks.

## When to use

Use when comparing ligand-receptor scoring rules for cell-cell communication inference on scRNA-seq data and you need an apples-to-apples benchmark on the same interaction database, same input matrix, and same permutation/significance pipeline.

## When NOT to use

- Do not use when the goal is to apply NeuronChat for inference on a new neural dataset rather than to comparatively evaluate scoring rules — use the standard NeuronChat pipeline instead
- Do not use when methods would be evaluated on their own native interaction databases; the benchmark requires all methods to be scored on the identical NeuronChatDB ligand-target pairs and identical input expression data
- Do not use for protein-ligand paracrine signaling contexts where small-molecule neurotransmitter signaling (glutamate, GABA) is not relevant and existing protein-based databases suffice

## Inputs

- Processed scRNA-seq count matrix (cortex_data, Yao 2021), Glutamatergic cells from VISp and ALM
- NeuronChatDB mouse interaction database (183 mouse entries of intercellular molecular interactions)
- Cell group label vector (subclass)
- Reimplemented CellChat and CellPhoneDB scoring rules defined on NeuronChatDB entries

## Outputs

- benchmark_auroc_auprc.csv with AUROC and AUPRC for NeuronChat, CellChat, and CellPhoneDB per region (VISp, ALM) on aggregated and individual interaction-pair networks
- Aggregated communication weight matrices per method (thresholded at the 80% quantile)
- Per-method communication networks with non-significant links (BH-corrected p>=0.05) set to zero

## How to apply

Load the processed cortex_data example dataset (Yao 2021), subset Glutamatergic cells for the VISp and ALM regions, and build NeuronChat objects with createNeuronChat(DB='mouse', group.by=subclass). Run run_NeuronChat with M=100 permutations, applying the Benjamini-Hochberg correction and setting non-significant links (p>=0.05) to zero. Reimplement the CellChat and CellPhoneDB scoring rules inside the same NeuronChat workflow, computing ligand abundance WITHOUT functional gene grouping (AND logic via geometric mean or minimum over all contributing genes) on the identical NeuronChatDB mouse interaction entries — this is the key fairness constraint. For each method, aggregate networks over all interaction pairs with net_aggregation(method='weight') and apply the benchmark default threshold ak defined as the 80% quantile. Compute AUROC and AUPRC on both aggregated networks and individual interaction-pair networks for VISp and ALM, and write all metrics to benchmark_auroc_auprc.csv.

## Related tools

- **NeuronChat** (Provides the shared computational workflow (createNeuronChat, run_NeuronChat, net_aggregation) into which all three scoring rules are reimplemented for the benchmark; also the method expected to attain the highest AUROC/AUPRC) — https://github.com/Wei-BioMath/NeuronChat
- **CellChat** (Its scoring rule is reimplemented within the NeuronChat workflow, computing ligand abundance without functional gene grouping, to serve as a benchmark comparator on NeuronChatDB entries)
- **CellPhoneDB** (Its scoring rule is reimplemented within the same workflow (AND logic via geometric mean or minimum over all contributing genes) as a second benchmark comparator)
- **NeuronChatAnalysis2022** (Repository of scripts used to produce the results of the NeuronChat study, including benchmark figure reproduction) — https://github.com/Wei-BioMath/NeuronChatAnalysis2022

## Examples

```
x <- createNeuronChat(normalized_count_mtx,DB='mouse',group.by = cell_group_vector); x <- run_NeuronChat(x,M=100); net_aggregated_x <- net_aggregation(x@net,method = 'weight')
```

## Evaluation signals

- All three methods must be evaluated on identical NeuronChatDB mouse interaction entries and identical input expression data — any divergence in the interaction universe invalidates the comparison
- The output table benchmark_auroc_auprc.csv must show NeuronChat attaining the highest AUROC and AUPRC on both VISp and ALM aggregated networks (and individual interaction-pair networks), reproducing the paper's finding
- Non-significant communication links (Benjamini-Hochberg corrected p>=0.05 after M=100 permutations) must be set to zero in every method's network
- Aggregated networks must use weight-based net_aggregation with the benchmark default threshold ak = 80% quantile
- Cross-check method ranking with compareInteractions_Neuron and rankNet_Neuron to confirm the reported ordering

## Limitations

- Reimplementing CellChat and CellPhoneDB without functional gene grouping can dramatically underestimate ligand abundance for small-molecule neurotransmitters (glutamate, GABA), which is a source of their lower benchmark scores
- NeuronChatDB may contain missing curated interaction entries, leading to bias in the inference
- Permutation-based p-value calculations (M=100 here) trade accuracy against computation speed (time complexity O(n) in the number of permutations)
- Ligand/target abundance estimated from transcriptomics may be inconsistent with protein or metabolite levels

## Evidence

- [discussion] We use the same computational workflow of NeuronChat for the implementation of CellChat and CellPhoneDB: "We use the same computational workflow of NeuronChat for the implementation of CellChat and CellPhoneDB"
- [discussion] This method is the default setting for benchmarking NeuronChat, with threshold ak defined as the 80% quantile: "This method is the default setting for benchmarking NeuronChat, with threshold ak defined as the 80% quantile"
- [discussion] both of CellChat and CellPhoneDB are implemented without categorizing the contributing genes into different functional groups and thus use AND logic (geometric mean or minimum): "both of CellChat and CellPhoneDB are implemented without categorizing the contributing genes into different functional groups and thus use AND logic (geometric mean or minimum)"
- [discussion] communications with p-value < 0.05 are considered significant; for non-significant communication links, the communication strength values are set to be zeros: "communications with p-value < 0.05 are considered significant; for non-significant communication links, the communication strength values are set to be zeros"
- [readme] The goal of NeuronChat is to infer, visualize and analyze neural-specific cell-cell communication from single cell transcriptomics: "The goal of NeuronChat is to infer, visualize and analyze neural-specific cell-cell communication from single cell transcriptomics"
