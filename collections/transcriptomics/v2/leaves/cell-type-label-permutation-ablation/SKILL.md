---
name: cell-type-label-permutation-ablation
description: Use when benchmarking NeuronChat's projection prediction on retrograde-labeling
  ground truth and you must rule out that benchmark performance is driven by node
  cell-type labels rather than the specific ground-truth graph topology.
when_to_use_negative:
- When testing significance of individual ligand-receptor communications within a
  single NeuronChat object — that is handled by run_NeuronChat's internal permutation
  of group labels, not by this benchmark-level label ablation.
- When the ground-truth network is not a projection/anatomical connectivity graph,
  since an AUROC drop under label shuffling only answers questions about labeled graph-node
  performance.
- When the goal is inference quality itself rather than validating what drives benchmark
  performance — this control does not improve or change the inferred network.
edam_operation: http://edamontology.org/operation_3659
edam_topics:
- http://edamontology.org/topic_2269
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_3307
tools:
- name: NeuronChat
  role: Runs the communication inference (createNeuronChat, run_NeuronChat with M=100)
    on both original and label-permuted data, and aggregates networks via net_aggregation
    for AUROC/AUPRC scoring
  repo: https://github.com/Wei-BioMath/NeuronChat
- name: NeuronChatAnalysis2022
  role: Repository of scripts for reproducing the paper's benchmark results, including
    the shuffled-label control on the Yao 2021 cortex dataset
  repo: https://github.com/Wei-BioMath/NeuronChatAnalysis2022
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1038/s41467-023-36800-w
    title: Inferring neuron-neuron communications from single-cell transcriptomics
      through NeuronChat
  source_content_hash: sha256:43f7c6a28e35e4bef9cb2530462785704a3fd1e70d1c3adb3aa449e5a5443752
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/Wei-BioMath/NeuronChat
  tools:
  - NeuronChat
  - NeuronChatAnalysis2022
  edam_operation: http://edamontology.org/operation_3659
  edam_topics:
  - http://edamontology.org/topic_2269
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_3307
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 11
---

# cell-type-label-permutation-ablation

## Summary

A permutation-based ablation control for NeuronChat benchmarking: shuffle cell-subclass labels across cells while keeping the expression matrix and graph topology fixed, rerun inference, and compare AUROC/AUPRC against the unshuffled labels to test whether predictive performance on retrograde-labeling ground truth (VISp/ALM) reflects cell-type identity rather than graph topology.

## When to use

Use when benchmarking NeuronChat's projection prediction on retrograde-labeling ground truth and you must rule out that benchmark performance is driven by node cell-type labels rather than the specific ground-truth graph topology.

## When NOT to use

- When testing significance of individual ligand-receptor communications within a single NeuronChat object — that is handled by run_NeuronChat's internal permutation of group labels, not by this benchmark-level label ablation.
- When the ground-truth network is not a projection/anatomical connectivity graph, since an AUROC drop under label shuffling only answers questions about labeled graph-node performance.
- When the goal is inference quality itself rather than validating what drives benchmark performance — this control does not improve or change the inferred network.

## Inputs

- Processed mouse cortex scRNA-seq data (cortex_data, Yao 2021), subset to Glutamatergic cells in VISp and ALM
- Per-cell subclass labels (group.by vector) used for permutation
- Retrograde-labeling projection ground-truth networks for VISp and ALM
- NeuronChatDB mouse interaction database (DB='mouse')

## Outputs

- Per-region (VISp, ALM) AUROC/AUPRC distributions computed under shuffled cell-subclass labels
- Per-region p-values comparing original vs shuffled labels across random seeds, written to shuffled_label_pvalues.csv

## How to apply

Load the processed cortex_data from Yao 2021, subset Glutamatergic cells for VISp and ALM, and create NeuronChat objects with DB='mouse' and group.by set to cell subclass. Run the baseline with run_NeuronChat(M=100) (~4 minutes per run) and aggregate with net_aggregation(method='weight'); for benchmark scoring use the thresholded-weight aggregation where the threshold ak is the 80% quantile, the default benchmarking setting. Ablate by randomly shuffling the subclass labels across cells while leaving the expression matrix unchanged, then rerun run_NeuronChat(M=100) on the permuted labels; the underlying rationale is NeuronChat's own permutation logic, which assesses significance by randomly permutating group labels of cells. Recompute AUROC/AUPRC distributions for projection prediction under shuffled labels, repeat the procedure over multiple random seeds, and report mean ± SD of the per-region p-values. Accept the run if shuffled-label AUROC collapses to a distribution with mean ~0.5 and reproduced p-values fall within the reported uncertainties: 0.010 ± 0.0036 for VISp and 0.017 ± 0.0048 for ALM; write results to shuffled_label_pvalues.csv.

## Related tools

- **NeuronChat** (Runs the communication inference (createNeuronChat, run_NeuronChat with M=100) on both original and label-permuted data, and aggregates networks via net_aggregation for AUROC/AUPRC scoring) — https://github.com/Wei-BioMath/NeuronChat
- **NeuronChatAnalysis2022** (Repository of scripts for reproducing the paper's benchmark results, including the shuffled-label control on the Yao 2021 cortex dataset) — https://github.com/Wei-BioMath/NeuronChatAnalysis2022

## Examples

```
x <- createNeuronChat(normalized_count_mtx,DB='mouse',group.by = cell_group_vector); x <- run_NeuronChat(x,M=100); net_aggregated_x <- net_aggregation(x@net,method='weight')
```

## Evaluation signals

- Shuffled-label AUROC distribution has mean ~0.5, while AUROC with original labels is significantly higher than with shuffled labels.
- Reproduced shuffled-label p-values fall within reported uncertainties: 0.010 ± 0.0036 for VISp and 0.017 ± 0.0048 for ALM.
- Graph topology is unchanged between baseline and permuted runs (only subclass labels are shuffled; expression matrix untouched).
- Multiple random seeds are run and p-values are reported as mean ± standard deviation per region.
- Per-region AUROC/AUPRC summaries and shuffled-label p-values are written to shuffled_label_pvalues.csv.

## Limitations

- Permutation-based p-value calculations trade accuracy against computation speed (~4 minutes per run_NeuronChat call with M=100), so many seeds increase runtime.
- Ligand/target abundance is estimated from transcriptomics and may be inconsistent with protein or metabolite levels, which bounds what label-shuffling can reveal about the inference signal.
- Permutation p-values need multiple-testing correction (Benjamini–Hochberg) and become less accurate at low permutation counts.

## Evidence

- [discussion] Permutation is the core significance mechanism NeuronChat applies to group labels, motivating the label-shuffle ablation: "The statistical significance of each communication is calculated through the permutation test by randomly permutating group labels of cells"
- [discussion] Thresholded weight aggregation with the 80% quantile is the default for benchmarking, defining how inferred networks are scored against ground truth: "This method is the default setting for benchmarking NeuronChat, with threshold ak defined as the 80% quantile"
- [methods] The cortex_data dataset from Yao 2021 is the starting input for the VISp/ALM benchmark: "# load example data: cortex data from Yao 2021, processed"
- [readme] run_NeuronChat with M=100 is the concrete inference call rerun on permuted labels: "x <- run_NeuronChat(x,M=100)"
- [readme] Networks are aggregated over interaction pairs with the weight method to obtain communication strengths for AUROC prediction: "net_aggregated_x <- net_aggregation(x@net,method = 'weight')"
