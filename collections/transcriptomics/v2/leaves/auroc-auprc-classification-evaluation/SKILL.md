---
name: auroc-auprc-classification-evaluation
description: Use when two or more communication-inference methods must be compared
  fairly on identical ligand-target pairs and expression data, with AUROC/AUPRC ranking
  of aggregated and per-interaction-pair networks.
when_to_use_negative:
- The methods being compared use different ligand-receptor databases or different
  preprocessing — AUROC/AUPRC differences would then confound database choice with
  scoring rule
- The goal is biological interpretation of a single dataset's communication network
  rather than method benchmarking — use the standard NeuronChat analysis/visualization
  workflow instead
- Network weights are spatially constrained (MERFISH/seqFISH+ spatially constrained
  networks) rather than the benchmark's thresholded weight aggregation setting
tools:
- name: NeuronChat
  role: 'Hosts the shared computational workflow: createNeuronChat, run_NeuronChat
    (M=100 permutation test with BH correction), net_aggregation, compareInteractions_Neuron,
    and rankNet_Neuron; provides the reference scoring being benchmarked'
  repo: https://github.com/Wei-BioMath/NeuronChat
- name: CellChat
  role: Its scoring rule is re-implemented within the NeuronChat workflow (ligand
    abundance without functional gene grouping) as a benchmark comparator
  repo: https://github.com/sqjin/CellChat
- name: CellPhoneDB
  role: Its scoring rule is re-implemented within the NeuronChat workflow (AND logic
    via geometric mean or minimum over all contributing genes) as a benchmark comparator
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1038/s41467-023-36800-w
    title: Inferring neuron-neuron communications from single-cell transcriptomics
      through NeuronChat
  source_content_hash: sha256:ad198dee7323f01cd311e68d208c88ec3e68e4a47ea92c8fa0487710a5540bf6
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/Wei-BioMath/NeuronChat
  tools:
  - CellChat
  - CellPhoneDB
  - NeuronChat
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.4
n_claims: 5
---

# auroc-auprc-classification-evaluation

## Summary

Benchmarks cell-cell communication inference methods (NeuronChat vs. reimplemented CellChat and CellPhoneDB) by scoring their inferred networks with AUROC and AUPRC under a single shared computational workflow and ligand-target database, so performance differences reflect scoring rules rather than workflow or database differences.

## When to use

Use when two or more communication-inference methods must be compared fairly on identical ligand-target pairs and expression data, with AUROC/AUPRC ranking of aggregated and per-interaction-pair networks.

## When NOT to use

- The methods being compared use different ligand-receptor databases or different preprocessing — AUROC/AUPRC differences would then confound database choice with scoring rule
- The goal is biological interpretation of a single dataset's communication network rather than method benchmarking — use the standard NeuronChat analysis/visualization workflow instead
- Network weights are spatially constrained (MERFISH/seqFISH+ spatially constrained networks) rather than the benchmark's thresholded weight aggregation setting

## Inputs

- processed cortex_data example dataset (Yao 2021) scRNA-seq counts, subset to Glutamatergic cells for VISp and ALM regions
- NeuronChatDB mouse ligand-target interaction database (183 mouse entries)
- cell group labels (subclass) used for group.by and for permutation testing

## Outputs

- AUROC and AUPRC metrics per method (NeuronChat, CellChat, CellPhoneDB) for VISp and ALM aggregated networks
- AUROC and AUPRC per individual interaction-pair network
- benchmark_auroc_auprc.csv metrics table
- compareInteractions_Neuron and rankNet_Neuron outputs confirming the method ranking

## How to apply

Re-implement the competing scoring rules (CellChat, CellPhoneDB) inside the NeuronChat workflow, computing ligand abundance WITHOUT functional gene grouping (AND logic via geometric mean or minimum over all contributing genes) on the identical NeuronChatDB mouse interaction entries. Build the NeuronChat object from the processed cortex_data (Yao 2021) with createNeuronChat(DB='mouse', group.by=subclass), run run_NeuronChat with M=100 permutations per region (VISp, ALM), apply Benjamini-Hochberg correction, and set non-significant links (p>=0.05) to zero. Aggregate each method's inferred network over all interaction pairs with net_aggregation(method='weight'), then apply the benchmark default threshold ak defined as the 80% quantile. Compute AUROC and AUPRC for all three methods on both the aggregated networks and the individual interaction-pair networks, compare with compareInteractions_Neuron and rankNet_Neuron to reproduce the expected ranking, and write all metrics to benchmark_auroc_auprc.csv. The rationale is that isolating the scoring rule while holding workflow, database, and input expression fixed is the only way the AUROC/AUPRC difference is attributable to the method itself.

## Related tools

- **NeuronChat** (Hosts the shared computational workflow: createNeuronChat, run_NeuronChat (M=100 permutation test with BH correction), net_aggregation, compareInteractions_Neuron, and rankNet_Neuron; provides the reference scoring being benchmarked) — https://github.com/Wei-BioMath/NeuronChat
- **CellChat** (Its scoring rule is re-implemented within the NeuronChat workflow (ligand abundance without functional gene grouping) as a benchmark comparator) — https://github.com/sqjin/CellChat
- **CellPhoneDB** (Its scoring rule is re-implemented within the NeuronChat workflow (AND logic via geometric mean or minimum over all contributing genes) as a benchmark comparator)

## Examples

```
x <- createNeuronChat(cortex_data, DB='mouse', group.by='subclass'); x <- run_NeuronChat(x, M=100); net_aggregated <- net_aggregation(x@net, method='weight')
```

## Evaluation signals

- The output table must reproduce the paper's finding that NeuronChat attains the highest AUROC and AUPRC on both VISp and ALM aggregated networks and individual interaction-pair networks
- All three methods are scored on the identical NeuronChatDB ligand-target pairs and identical input expression data — any schema check should confirm shared interaction entries across methods
- Non-significant links (Benjamini-Hochberg corrected p>=0.05) must be set to zero in every method's network before aggregation
- The aggregated network for each method results from net_aggregation(method='weight') followed by threshold ak at the 80% quantile, applied uniformly across methods
- The permutation test used M=100 label permutations per region for all methods

## Limitations

- CellChat and CellPhoneDB implemented without categorizing contributing genes into functional groups use AND logic (geometric mean or minimum) for all contributing genes, which can dramatically underestimate ligand abundance for small-molecule neurotransmitters such as glutamate and GABA
- NeuronChatDB may contain missing curated interaction entries, leading to bias in the inference; subunit stoichiometries of heteromeric receptors are not contained in NeuronChatDB
- Permutation-based p-value calculation has a trade-off between accuracy and computation speed (time complexity O(n) in the number of permutations); M=100 is a runtime/accuracy compromise
- Ligand/target abundance estimated from transcriptomics may be inconsistent with actual protein or metabolite levels

## Evidence

- [discussion] Same-workflow reimplementation is the paper's stated benchmarking design: "We use the same computational workflow of NeuronChat for the implementation of CellChat and CellPhoneDB"
- [discussion] Benchmark threshold definition: "This method is the default setting for benchmarking NeuronChat, with threshold ak defined as the 80% quantile"
- [discussion] Why CellChat/CellPhoneDB scoring underestimates neurotransmitter ligand abundance: "both of CellChat and CellPhoneDB are implemented without categorizing the contributing genes into different functional groups and thus use AND logic"
- [methods] Network aggregation step in the shared workflow: "net_aggregated_x <- net_aggregation(x@net,method = 'weight')"
- [discussion] Permutation testing and correction parameters: "The p-value is corrected for multiple tests by using Benjamini–Hochberg procedure"
