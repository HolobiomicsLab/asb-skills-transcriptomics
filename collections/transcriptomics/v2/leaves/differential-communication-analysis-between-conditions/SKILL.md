---
name: differential-communication-analysis-between-conditions
description: Use when two or more single-cell/single-nucleus expression matrices with
  cell-type annotations from distinct biological conditions (e.g., ASD vs control
  cortex) must be compared at the level of inferred ligand-receptor communication
  networks.
when_to_use_negative:
- Data is spatially resolved transcriptomics where spatially constrained networks
  and cell-proximity enrichment (400 micron threshold, label permutation) are needed
  rather than condition comparison
- The question concerns short-range autocrine/paracrine signaling in non-neural tissue;
  NeuronChat targets neural-specific communication including long-distance neurotransmitter
  signaling
- Only a single condition/dataset exists and no comparative analysis of multiple datasets
  is intended
edam_operation: http://edamontology.org/operation_3927
edam_topics:
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_0128
tools:
- name: NeuronChat
  role: Infers condition-specific neural communication networks from single-cell expression,
    merges them across conditions, and supports comparative ranking (run_NeuronChat,
    mergeNeuronChat, compareInteractions_Neuron, rankNet_Neuron)
  repo: https://github.com/Wei-BioMath/NeuronChat
- name: CellChat
  role: Dependency of NeuronChat; alternative communication inference whose workflow
    NeuronChat reuses, though it lacks functional gene grouping and can underestimate
    small-molecule neurotransmitter ligand abundance
  repo: https://github.com/sqjin/CellChat
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1038/s41467-023-36800-w
    title: Inferring neuron-neuron communications from single-cell transcriptomics
      through NeuronChat
  source_content_hash: sha256:cd672df27d79b825f64632a50c08969453252551150cdbea61a21e3c74a428c6
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/Wei-BioMath/NeuronChat
  tools:
  - CellChat
  - NeuronChat
  edam_operation: http://edamontology.org/operation_3927
  edam_topics:
  - http://edamontology.org/topic_3308
  - http://edamontology.org/topic_0128
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8333333333333334
n_claims: 6
---

# differential-communication-analysis-between-conditions

## Summary

Infers neural cell-cell communication networks separately per condition with NeuronChat (e.g., ASD vs control cortical snRNA-seq), merges them, and compares link counts and communication weights to identify condition-specific signaling such as reduced neuroligin 3 signaling to oligodendrocyte lineage cells.

## When to use

Use when two or more single-cell/single-nucleus expression matrices with cell-type annotations from distinct biological conditions (e.g., ASD vs control cortex) must be compared at the level of inferred ligand-receptor communication networks.

## When NOT to use

- Data is spatially resolved transcriptomics where spatially constrained networks and cell-proximity enrichment (400 micron threshold, label permutation) are needed rather than condition comparison
- The question concerns short-range autocrine/paracrine signaling in non-neural tissue; NeuronChat targets neural-specific communication including long-distance neurotransmitter signaling
- Only a single condition/dataset exists and no comparative analysis of multiple datasets is intended

## Inputs

- Processed snRNA-seq/scRNA-seq count matrices per condition (e.g., ASD and control cortex) with cell-type annotations
- Cell-type/group label vector for group.by
- NeuronChat interaction database selection ('human' or 'mouse')

## Outputs

- Per-condition aggregated communication networks (net_aggregation output over interaction pairs)
- Merged multi-condition NeuronChat object via mergeNeuronChat
- Per-interaction weight/count comparison table (e.g., asd_vs_control_communication_comparison.csv)
- Comparison plots: circle plots and rankNet barplots (PNG files)

## How to apply

For each condition, build a NeuronChat object with createNeuronChat(..., DB='human', group.by=cell type) from the normalized expression matrix, then infer networks with run_NeuronChat(x, M=100), which permutes cell-group labels 100 times and applies Benjamini-Hochberg correction; non-significant links (p >= 0.05) are set to zero. Aggregate each condition's network over all interaction pairs with net_aggregation(x@net, method='weight'), combine with mergeNeuronChat(add.names=c('ASD','Control')), and quantify overall differences with compareInteractions_Neuron(measure=c('count','weight'), comparison=1:2). Rank interaction pairs by differential weight and count using rankNet_Neuron(mode='comparison', comparison=1:2, stacked=FALSE) and extract the signaling rows of interest (e.g., NLGN3-mediated NRXN-NLGN links to oligodendrocytes and OPCs); optionally add computeNetSimilarityPairwise_Neuron, netEmbedding, and netClustering to separate shared from condition-specific patterns. Write the per-interaction weight/count comparison table (e.g., asd_vs_control_communication_comparison.csv) and circle-plot/rankNet PNGs. This comparison mode is justified because NeuronChat's neural-specific database (NeuronChatDB, 190 human / 183 mouse entries) includes small-molecule neurotransmitter signaling that protein-only ligand-receptor tools like CellChat and CellPhoneDB miss, and its functional gene-group AND/OR logic avoids underestimating ligand abundance.

## Related tools

- **NeuronChat** (Infers condition-specific neural communication networks from single-cell expression, merges them across conditions, and supports comparative ranking (run_NeuronChat, mergeNeuronChat, compareInteractions_Neuron, rankNet_Neuron)) — https://github.com/Wei-BioMath/NeuronChat
- **CellChat** (Dependency of NeuronChat; alternative communication inference whose workflow NeuronChat reuses, though it lacks functional gene grouping and can underestimate small-molecule neurotransmitter ligand abundance) — https://github.com/sqjin/CellChat

## Examples

```
x <- createNeuronChat(normalized_count_mtx, DB='human', group.by=cell_group_vector); x <- run_NeuronChat(x, M=100); net <- net_aggregation(x@net, method='weight')
```

## Evaluation signals

- All reported differential links satisfy the significance criterion: permutation p < 0.05 after Benjamini-Hochberg correction with M=100 permutations; non-significant links set to zero
- run_NeuronChat executed with M=100 permutations per condition; aggregated networks obtained via net_aggregation(x@net, method='weight')
- Directionality check: NLGN3-mediated signaling to OPC/oligodendrocytes (NRXN1-NLGN3, NRXN2-NLGN3, NRXN3-NLGN3) shows lower weight in ASD than control
- Merged object carries condition names (add.names=c('ASD','Control')) and compareInteractions_Neuron returns both 'count' and 'weight' measures
- Comparison table contains a weight and count column per interaction pair, and plots render without missing-condition errors

## Limitations

- Permutation-based p-values trade accuracy against computation speed (O(n) in number of permutations); M=100 affects precision
- Ligand/target abundance estimated from transcriptomics may be inconsistent with actual protein or metabolite levels
- NeuronChatDB may contain missing curated interaction entries, leading to bias in inference; subunit stoichiometries of heteromeric receptors are not contained in NeuronChatDB
- Genes expressed in less than 25% of cells are filtered out, which can remove low-abundance but relevant interactions

## Evidence

- [discussion] Permutation significance and BH correction: "The statistical significance of each communication is calculated through the permutation test by randomly permutating group labels of cells"
- [discussion] Non-significant links zeroed: "communications with p-value < 0.05 are considered significant; for non-significant communication links, the communication strength values are set to be zeros"
- [readme] Core comparative workflow from README: "x <- run_NeuronChat(x,M=100)
# aggregating networks over interaction pairs
net_aggregated_x <- net_aggregation(x@net,method = 'weight')"
- [abstract] Disease-application relevance: "predicting communication pattern changes in diseased brains with autism spectrum disorder"
- [discussion] Why not CellChat/CellPhoneDB for neurotransmitters: "both of CellChat and CellPhoneDB are implemented without categorizing the contributing genes into different functional groups and thus use AND logic (geometric mean or minimum) for all contributing"
