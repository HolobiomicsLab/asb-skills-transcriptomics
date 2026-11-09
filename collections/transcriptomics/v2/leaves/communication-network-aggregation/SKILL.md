---
name: communication-network-aggregation
description: Use when you have run run_NeuronChat(x, M=100) on scRNA-seq data and
  need a single aggregated cell-group-to-cell-group network (x@net) to rank predicted
  links against experimental connectivity such as the ALM/VISp retrograde-labeling
  ground truth.
when_to_use_negative:
- You need per-interaction (per ligand-receptor pair) communication strengths rather
  than a single group-level network — use x@net directly instead
- Working with spatially constrained networks where links must be filtered by spatial
  proximity scores rather than aggregated weights
- You want to keep only the top links by total weight for visualization (e.g., top
  10 links) rather than full benchmark aggregation
edam_operation: http://edamontology.org/operation_3927
edam_topics:
- http://edamontology.org/topic_0203
tools:
- name: NeuronChat
  role: Provides the net_aggregation() function that combines the per-interaction
    networks in x@net into a single aggregated communication network over all interaction
    pairs
  repo: https://github.com/Wei-BioMath/NeuronChat
provenance:
  source_task_ids:
  - task_002
  - task_003
  source_papers:
  - doi: 10.1038/s41467-023-36800-w
    title: Inferring neuron-neuron communications from single-cell transcriptomics
      through NeuronChat
  source_content_hash: sha256:eb5c266f93781d354a5f7fa4fcc612d7d603a5da810eb43057e193ef020587c6
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/Wei-BioMath/NeuronChat
  tools:
  - CellChat
  - NeuronChat
  edam_operation: http://edamontology.org/operation_3927
  edam_topics:
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.875
n_claims: 8
---

# communication-network-aggregation

## Summary

Aggregate NeuronChat's per-interaction communication networks over all ligand-receptor interaction pairs into a single weighted cell-group-level network, using the benchmark-default thresholded-weight setting (threshold ak at the 80% quantile). This step is required to rank predicted communication links against ground-truth connectivity in projection-network benchmarks.

## When to use

Use when you have run run_NeuronChat(x, M=100) on scRNA-seq data and need a single aggregated cell-group-to-cell-group network (x@net) to rank predicted links against experimental connectivity such as the ALM/VISp retrograde-labeling ground truth.

## When NOT to use

- You need per-interaction (per ligand-receptor pair) communication strengths rather than a single group-level network — use x@net directly instead
- Working with spatially constrained networks where links must be filtered by spatial proximity scores rather than aggregated weights
- You want to keep only the top links by total weight for visualization (e.g., top 10 links) rather than full benchmark aggregation

## Inputs

- NeuronChat object network slot x@net (per-interaction communication strength matrices from run_NeuronChat(x, M=100))
- Aggregation method string ('weight' for thresholded-weight benchmarking)

## Outputs

- net_aggregated_x: a single weighted directed matrix of aggregated communication strength between cell groups
- Ranked communication links used for AUROC/AUPRC evaluation against ground-truth connectivity

## How to apply

After run_NeuronChat has inferred per-interaction communication strengths (Tukey-trimean-averaged ligand/target abundance, geometric mean across functional gene groups, arithmetic mean within redundant groups, with non-significant links at p >= 0.05 zeroed), pass the network object x@net to net_aggregation to sum/combine strengths across all interaction pairs. For benchmarking against monosynaptic retrograde-labeling connectivity (e.g., ALM and VISp in the cortex_data dataset from Yao 2021), use method='weight' with the thresholded-weight aggregation that thresholds ak at the 80% quantile — this is the paper's default benchmark setting. The resulting aggregated network is a weighted directed matrix between cell subclasses, which is then ranked and scored against ground truth via AUROC and AUPRC (the paper recovers AUROC 0.764 and AUPRC 0.768 for ALM). Rerun the aggregation under shuffled cell-subclass labels (keeping the expression matrix fixed) to confirm that performance is not driven by graph topology alone — the paper reports shuffled-label p-values of 0.010 ± 0.0036 (VISp) and 0.017 ± 0.0048 (ALM).

## Related tools

- **NeuronChat** (Provides the net_aggregation() function that combines the per-interaction networks in x@net into a single aggregated communication network over all interaction pairs) — https://github.com/Wei-BioMath/NeuronChat

## Examples

```
net_aggregated_x <- net_aggregation(x@net, method = 'weight')
```

## Evaluation signals

- The aggregated network must reproduce the paper's ALM benchmark metrics when ranked against the retrograde-labeling ground truth: AUROC 0.764 and AUPRC 0.768 (within numerical tolerance)
- The thresholded-weight aggregation must threshold ak at the 80% quantile, matching the benchmark default
- Non-significant links (p >= 0.05 after Benjamini-Hochberg correction) must already be zeroed in x@net before aggregation
- Shuffled-label control: rerunning run_NeuronChat with permuted subclass labels should drop AUROC to a distribution with mean ~0.5, with p-values near 0.010 (VISp) and 0.017 (ALM)

## Limitations

- The thresholded-weight method with the 80% quantile threshold is the paper's benchmark default; other aggregation settings may not reproduce the reported AUROC/AUPRC values
- Aggregation collapses per-interaction information, so the resulting group-level network cannot be used to attribute communication strength to specific ligand-receptor pairs
- Permutation-based significance testing inside run_NeuronChat is slow (~4 minutes per run, time complexity O(n) in number of permutations), which compounds when rerunning for shuffle controls over multiple seeds

## Evidence

- [methods] Aggregate the communication networks over all interaction pairs: [evidence span withheld — anchor status: not_found]
- [discussion] Thresholded weight aggregation uses 80% quantile for benchmarking: "This method is the default setting for benchmarking NeuronChat, with threshold ak defined as the 80% quantile"
- [readme] README usage shows aggregation step: "# aggregating networks over interaction pairs
net_aggregated_x <- net_aggregation(x@net,method = 'weight')"
- [discussion] Non-significant links zeroed before aggregation: "communications with p-value < 0.05 are considered significant; for non-significant communication links, the communication strength values are set to be zeros"
