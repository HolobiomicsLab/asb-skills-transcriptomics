---
name: connectivity-auroc-auprc-benchmark-evaluation
description: Use when a ligand-receptor-inferred cell-cell communication network must
  be validated against a ground-truth anatomical connectivity map, reporting AUROC
  and AUPRC against the 0.5-chance AUROC baseline.
when_to_use_negative:
- The inference task is short-range autocrine/paracrine signaling rather than long-distance
  neuron-neuron communication, since NeuronChat and this benchmark target neural-specific
  neurotransmitter signaling
- No experimentally identified ground-truth connectivity (e.g., retrograde labeling)
  is available for the region being inferred, so no AUROC/AUPRC can be computed
- Input data lacks cell_subclass group labels needed to define cell groups for createNeuronChat
edam_operation: null
edam_topics:
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_2229
tools:
- name: NeuronChat
  role: Infers the neuron-neuron communication network from scRNA-seq (createNeuronChat,
    run_NeuronChat, net_aggregation) whose links are ranked for benchmarking
  repo: https://github.com/Wei-BioMath/NeuronChat
- name: CellChat
  role: Pre-existing cell-cell communication inference method referenced as context
    for ligand-receptor-based network inference workflows
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1038/s41467-023-36800-w
    title: Inferring neuron-neuron communications from single-cell transcriptomics
      through NeuronChat
  source_content_hash: sha256:54cbe1bb353f01e42e987823f1320a2db88c3791403bd4a4e517d354a224990a
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/Wei-BioMath/NeuronChat
  tools:
  - CellChat
  - NeuronChat
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3308
  - http://edamontology.org/topic_2229
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 9
---

# connectivity-auroc-auprc-benchmark-evaluation

## Summary

Benchmark an inferred intercellular communication network against experimentally identified neural connectivity (e.g., ALM monosynaptic retrograde labeling) by ranking predicted links and computing AUROC and AUPRC, following NeuronChat's thresholded-weight aggregation default.

## When to use

Use when a ligand-receptor-inferred cell-cell communication network must be validated against a ground-truth anatomical connectivity map, reporting AUROC and AUPRC against the 0.5-chance AUROC baseline.

## When NOT to use

- The inference task is short-range autocrine/paracrine signaling rather than long-distance neuron-neuron communication, since NeuronChat and this benchmark target neural-specific neurotransmitter signaling
- No experimentally identified ground-truth connectivity (e.g., retrograde labeling) is available for the region being inferred, so no AUROC/AUPRC can be computed
- Input data lacks cell_subclass group labels needed to define cell groups for createNeuronChat

## Inputs

- Processed scRNA-seq count data and metadata for the mouse cortex (cortex_data from Yao 2021: target_df and meta), subset to ALM Glutamatergic CTX subclasses
- Ground-truth connectivity from monosynaptic retrograde labeling of the ALM projection network
- Mouse NeuronChatDB interaction database ('mouse' option in createNeuronChat)

## Outputs

- Aggregated communication network (net_aggregation output over all interaction pairs)
- AUROC score for predicted links vs. ground-truth connectivity (expected 0.764 for ALM)
- AUPRC score for predicted links vs. ground-truth connectivity (expected 0.768 for ALM)

## How to apply

Load the cortex_data example dataset (Yao 2021 target_df and meta), subset the ALM region's Glutamatergic CTX subclasses, create a NeuronChat object grouped by cell_subclass with the 'mouse' NeuronChatDB, and run inference with run_NeuronChat(x, M=100); the inference filters genes expressed in less than 25% of cells, computes communication strength from Tukey-trimean-averaged ligand/target abundance (geometric mean across functional gene groups, arithmetic mean within redundant groups), and zeroes links with BH-corrected p >= 0.05. Aggregate the per-interaction networks over all interaction pairs with net_aggregation(x@net, method='weight'), which applies the benchmark-default thresholded-weight setting that thresholds ak at the 80% quantile. Rank the predicted communication links against the ALM retrograde-labeling ground-truth connectivity and compute AUROC and AUPRC; a random classifier yields AUROC 0.5 and values closer to 1 indicate better prediction. Validate by checking that the recovered scores match the reported benchmark values (ALM: AUROC 0.764, AUPRC 0.768; VISp AUROC 0.832) within numerical tolerance.

## Related tools

- **NeuronChat** (Infers the neuron-neuron communication network from scRNA-seq (createNeuronChat, run_NeuronChat, net_aggregation) whose links are ranked for benchmarking) — https://github.com/Wei-BioMath/NeuronChat
- **CellChat** (Pre-existing cell-cell communication inference method referenced as context for ligand-receptor-based network inference workflows)

## Examples

```
x <- createNeuronChat(normalized_count_mtx,DB='mouse',group.by = cell_group_vector); x <- run_NeuronChat(x,M=100); net_aggregated_x <- net_aggregation(x@net,method='weight')
```

## Evaluation signals

- Recovered ALM benchmark scores match AUROC 0.764 and AUPRC 0.768 within numerical tolerance
- VISp control region recovers AUROC 0.832 when benchmarked identically
- Inferred AUROC substantially exceeds the 0.5 random-classifier baseline
- Non-significant communication links (BH-corrected p >= 0.05) are set to zero before aggregation
- Aggregation uses the benchmark-default thresholded-weight setting with threshold ak at the 80% quantile

## Limitations

- Ligand/target abundance estimated from transcriptomics may be inconsistent with protein or metabolite levels
- NeuronChatDB may contain missing curated interaction entries, leading to bias in inference
- Permutation-based p-value calculations face a trade-off between accuracy and computation speed (time complexity O(n) in number of permutations), so the choice of M=100 permutations affects runtime
- CellChat and CellPhoneDB workflows without functional gene grouping can dramatically underestimate ligand abundance for small-molecule neurotransmitters such as glutamate and GABA

## Evidence

- [full_text] When benchmarked against the ALM projection network identified by retrograde labeling, NeuronChat's aggregated inferred network achieves an AUROC of 0.764: [evidence span withheld — anchor status: not_found]
- [discussion] This method is the default setting for benchmarking NeuronChat, with threshold ak defined as the 80% quantile: "This method is the default setting for benchmarking NeuronChat, with threshold ak defined as the 80% quantile"
- [methods] data(list='cortex_data'): "# load example data: cortex data from Yao 2021, processed
data(list='cortex_data')"
- [readme] creat NeuronChat object; choose the database 'mouse' for mouse data: [evidence span withheld — anchor status: external_doc]
