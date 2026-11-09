---
name: retrograde-tracing-benchmark-evaluation
description: Use when validating an inferred neuron-neuron communication network for
  a cortical region against gold-standard anatomical connectivity from monosynaptic
  retrograde-tracing labels.
when_to_use_negative:
- No experimentally derived projection/connectivity ground truth (e.g., retrograde-tracing
  labels) exists for the region, so AUROC/AUPRC cannot be computed
- The goal is general cell-cell communication inference with CellChat/CellPhoneDB-style
  paracrine ligand-receptor modeling rather than benchmarking neurotransmitter-based
  neural communication against anatomy
- Input is spatial transcriptomics requiring spatially constrained networks rather
  than the region-level scRNA-seq benchmark
edam_operation: http://edamontology.org/operation_3927
edam_topics:
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_3474
tools:
- name: NeuronChat
  role: Infers the neural-specific communication network from the region-subset scRNA-seq
    data whose predicted connectivity is scored against retrograde-tracing labels
  repo: https://github.com/Wei-BioMath/NeuronChat
- name: CellChat
  role: Baseline dependency of NeuronChat and comparator method implemented with the
    same workflow for benchmarking context
  repo: https://github.com/sqjin/CellChat
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1038/s41467-023-36800-w
    title: Inferring neuron-neuron communications from single-cell transcriptomics
      through NeuronChat
  source_content_hash: sha256:92f26840bcbb1cda496da4ad922dfe533516d8c278b039e1bed7188e71c2b778
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
  - http://edamontology.org/topic_3474
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9230769230769231
n_claims: 13
---

# retrograde-tracing-benchmark-evaluation

## Summary

Benchmarks NeuronChat-inferred communication networks against an experimentally identified projection network from monosynaptic retrograde labeling by computing AUROC and AUPRC (e.g., VISp: AUROC 0.832, AUPRC 0.915).

## When to use

Use when validating an inferred neuron-neuron communication network for a cortical region against gold-standard anatomical connectivity from monosynaptic retrograde-tracing labels.

## When NOT to use

- No experimentally derived projection/connectivity ground truth (e.g., retrograde-tracing labels) exists for the region, so AUROC/AUPRC cannot be computed
- The goal is general cell-cell communication inference with CellChat/CellPhoneDB-style paracrine ligand-receptor modeling rather than benchmarking neurotransmitter-based neural communication against anatomy
- Input is spatial transcriptomics requiring spatially constrained networks rather than the region-level scRNA-seq benchmark

## Inputs

- Processed Yao 2021 mouse cortex scRNA-seq dataset (cortex_data) with region and cell_subclass annotations
- Monosynaptic retrograde-labeling projection-network labels (binary ground-truth connectivity) for the target region
- NeuronChat 'mouse' interaction database (NeuronChatDB)

## Outputs

- Aggregated region-specific communication network (net_aggregation weight matrix over cell subclasses)
- AUROC and AUPRC benchmark metrics (e.g., visp_benchmark_metrics.csv: AUROC 0.832, AUPRC 0.915 for VISp)

## How to apply

Subset the processed Yao 2021 cortex dataset (cortex_data) to the target region, e.g. region_name <- 'VISp', keeping Glutamatergic cells, then build a NeuronChat object with the 'mouse' interaction database grouping cells by cell_subclass. Run run_NeuronChat with M=100 permutations; genes expressed in less than 25% of cells are filtered out, p-values are Benjamini–Hochberg corrected, and communications with p < 0.05 are kept significant (non-significant links set to zero). Aggregate per-interaction networks with net_aggregation(x@net, method='weight'), using thresholded weight aggregation whose threshold ak is defined as the 80% quantile — the default benchmarking setting. Compare predicted connectivity scores against retrograde-tracing labels, compute AUROC and AUPRC, and write the metrics (e.g., visp_benchmark_metrics.csv).

## Related tools

- **NeuronChat** (Infers the neural-specific communication network from the region-subset scRNA-seq data whose predicted connectivity is scored against retrograde-tracing labels) — https://github.com/Wei-BioMath/NeuronChat
- **CellChat** (Baseline dependency of NeuronChat and comparator method implemented with the same workflow for benchmarking context) — https://github.com/sqjin/CellChat

## Examples

```
x <- createNeuronChat(visp_counts, DB='mouse', group.by = target_df_single$cell_subclass); x <- run_NeuronChat(x, M=100); net <- net_aggregation(x@net, method='weight')
```

## Evaluation signals

- Computed VISp AUROC and AUPRC match the values reported in Fig. 2c,d of the article (AUROC 0.832, AUPRC 0.915)
- Both AUROC and AUPRC are significantly better than random classifiers
- run_NeuronChat uses M=100 permutations and Benjamini–Hochberg correction with only p < 0.05 links retained as significant
- net_aggregation applies thresholded weight aggregation with threshold ak at the 80% quantile (default benchmark setting)
- Genes expressed in less than 25% of cells are excluded before inference

## Limitations

- Transcriptomics-based ligand/target abundance estimates may be inconsistent with actual protein or metabolite levels
- NeuronChatDB may contain missing curated interaction entries, leading to bias in inference; heteromeric receptor subunit stoichiometries are largely unknown
- There is a trade-off between accuracy and computation speed for permutation-based p-value calculations (time complexity O(n) in number of permutations)

## Evidence

- [results] Benchmark finding: [evidence span withheld — anchor status: not_found]
- [methods] Benchmark workflow: "x <- run_NeuronChat(x,M=100)"
- [discussion] Threshold default: "This method is the default setting for benchmarking NeuronChat, with threshold ak defined as the 80% quantile"
- [discussion] Gene filter: "Because the genes expressed in less than 25% of cells are filtered out"
- [readme] README usage: "net_aggregated_x <- net_aggregation(x@net,method = 'weight')"
