---
name: weighted-network-aggregation-thresholding
description: Use when a NeuronChat run has produced per-interaction communication
  networks (x@net) and you need one aggregated cell-group connectivity matrix, e.g.
  to benchmark predicted connectivity against retrograde-tracing labels.
when_to_use_negative:
- You are still inferring communication networks and have not yet run run_NeuronChat
  — aggregation requires the per-interaction networks in x@net
- You need per-interaction (ligand–receptor-specific) communication strengths rather
  than an aggregate sender–receiver network
- Your data are spatial transcriptomics where spatially constrained network filtering
  (spatial proximity score thresholds), not weight thresholding, is the intended aggregation
  mode
edam_operation: http://edamontology.org/operation_3927
edam_topics:
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_0605
tools:
- name: NeuronChat
  role: Provides run_NeuronChat to infer per-interaction communication networks and
    net_aggregation(method='weight') to aggregate them into a single weighted network
    with 80%-quantile thresholding for benchmarking
  repo: https://github.com/Wei-BioMath/NeuronChat
- name: CellChat
  role: Dependency of NeuronChat (R dependency CellChat >= 1.1.3) and comparator method
    whose workflow implementation aggregates without functional gene grouping
  repo: https://github.com/sqjin/CellChat
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1038/s41467-023-36800-w
    title: Inferring neuron-neuron communications from single-cell transcriptomics
      through NeuronChat
  source_content_hash: sha256:4f611e79fd89a41bc8656b4be523ee371cecbf7827fe9c629a781f5a5eb3aed7
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
  - http://edamontology.org/topic_0605
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6
n_claims: 5
---

# weighted-network-aggregation-thresholding

## Summary

Aggregates NeuronChat's per-interaction communication strength matrices into a single cell-group-level weighted network using thresholded weight aggregation, where the threshold ak is defined as the 80% quantile — the default setting for benchmarking NeuronChat.

## When to use

Use when a NeuronChat run has produced per-interaction communication networks (x@net) and you need one aggregated cell-group connectivity matrix, e.g. to benchmark predicted connectivity against retrograde-tracing labels.

## When NOT to use

- You are still inferring communication networks and have not yet run run_NeuronChat — aggregation requires the per-interaction networks in x@net
- You need per-interaction (ligand–receptor-specific) communication strengths rather than an aggregate sender–receiver network
- Your data are spatial transcriptomics where spatially constrained network filtering (spatial proximity score thresholds), not weight thresholding, is the intended aggregation mode

## Inputs

- NeuronChat object x with slot x@net containing per-interaction communication strength matrices after run_NeuronChat (M=100 permutations)
- Cell-group labels (e.g. cell_subclass from the Yao 2021 cortex_data, VISp subset, Glutamatergic cells)
- Binary ground-truth connectivity labels (e.g. monosynaptic retrograde-labeling projection network) for benchmarking

## Outputs

- net_aggregated_x: an aggregated cell-group x cell-group weighted communication network (matrix)
- Benchmark metrics such as AUROC and AUPRC written to a file (e.g. visp_benchmark_metrics.csv)

## How to apply

After run_NeuronChat(x, M=100) has produced per-interaction communication matrices (stored in x@net), pass them to net_aggregation(x@net, method='weight'). With the thresholded weight option, a threshold ak is set at the 80% quantile of the weights — this is the article's default for benchmarking — and links below the threshold are removed so the aggregated network is sparse enough to score against binary ground truth. The resulting matrix is then compared element-wise against experimental labels (e.g. the monosynaptic retrograde-labeling projection network for mouse VISp) to compute AUROC and AUPRC. The rationale is that individual interaction-level networks are noisy and sparse; thresholded weight aggregation concentrates signal into a single connectivity score per sender–receiver cell-group pair suitable for classification metrics.

## Related tools

- **NeuronChat** (Provides run_NeuronChat to infer per-interaction communication networks and net_aggregation(method='weight') to aggregate them into a single weighted network with 80%-quantile thresholding for benchmarking) — https://github.com/Wei-BioMath/NeuronChat
- **CellChat** (Dependency of NeuronChat (R dependency CellChat >= 1.1.3) and comparator method whose workflow implementation aggregates without functional gene grouping) — https://github.com/sqjin/CellChat

## Examples

```
x <- run_NeuronChat(x, M=100); net_aggregated_x <- net_aggregation(x@net, method = 'weight')
```

## Evaluation signals

- The VISp aggregated network should reproduce the article benchmark: AUROC of 0.832 and AUPRC of 0.915 against the retrograde-labeling projection network, significantly better than random classifiers
- The aggregation call must use method='weight' with the 80% quantile threshold ak (the documented default for benchmarking)
- Non-significant communications (Benjamini–Hochberg-corrected p >= 0.05 from M=100 permutations) must have been set to zero in x@net before aggregation
- Output should be a square weighted matrix over the same cell groups (subclasses) used to create the NeuronChat object via createNeuronChat with DB='mouse'
- Genes expressed in less than 25% of cells should have been filtered out upstream of inference

## Limitations

- There is a trade-off between accuracy and computation speed for permutation-based p-value calculations (O(n) in the number of permutations), which must be redone for all new applications of NeuronChat before aggregation
- Thresholded weight aggregation at the 80% quantile is tuned for benchmarking against binary labels; other analysis goals may need different aggregation choices
- NeuronChatDB may contain missing curated interaction entries and lacks heteromeric receptor subunit stoichiometries, biasing the inferred weights that feed into the aggregated network

## Evidence

- [methods] net_aggregated_x <- net_aggregation(x@net,method = 'weight'): "net_aggregated_x <- net_aggregation(x@net,method = 'weight')"
- [discussion] Thresholded weight aggregation default: "This method is the default setting for benchmarking NeuronChat, with threshold ak defined as the 80% quantile"
- [discussion] Significance filtering before aggregation: "communications with p-value < 0.05 are considered significant; for non-significant communication links, the communication strength values are set to be zeros"
- [readme] Basic usage sequence from README: [evidence span withheld — anchor status: not_found]
