---
name: neuron-projection-prediction-evaluation
description: Use when validating inferred cell-cell communication networks against
  anatomical projection ground truth, or when checking whether benchmark performance
  depends on cell-type labels versus network topology via label-shuffling permutation.
when_to_use_negative:
- Input is spatial transcriptomics data or non-cortex scRNA-seq data for which no
  retrograde-labeling projection ground truth exists for scoring AUROC/AUPRC
- The goal is to test significance of individual communication links via label permutation
  inside run_NeuronChat, rather than to assess whether overall benchmark performance
  depends on cell-type labels
edam_operation: http://edamontology.org/operation_2239
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: NeuronChat
  role: Creates NeuronChat objects (DB='mouse', group.by = cell subclass), runs the
    communication inference via run_NeuronChat(M=100), and aggregates networks via
    net_aggregation for AUROC/AUPRC evaluation against projection ground truth
  repo: https://github.com/Wei-BioMath/NeuronChat
- name: NeuronChatAnalysis2022
  role: Analysis scripts/repository for reproducing the benchmarking figures and shuffled-label
    robustness results of the NeuronChat study
  repo: https://github.com/Wei-BioMath/NeuronChatAnalysis2022
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1038/s41467-023-36800-w
    title: Inferring neuron-neuron communications from single-cell transcriptomics
      through NeuronChat
  source_content_hash: sha256:e1dd84c84f420942b11505b087bc71b019bddf9a64ccb167c9777e0437870f1c
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/Wei-BioMath/NeuronChat
  tools:
  - NeuronChat
  - NeuronChatAnalysis2022
  edam_operation: http://edamontology.org/operation_2239
  edam_topics:
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 7
---

# neuron-projection-prediction-evaluation

## Summary

Benchmark NeuronChat's inferred communication networks against retrograde-labeling projection ground truth (VISp and ALM), and run a shuffled-label robustness control to test whether performance reflects cell-type expression signal rather than graph topology alone.

## When to use

Use when validating inferred cell-cell communication networks against anatomical projection ground truth, or when checking whether benchmark performance depends on cell-type labels versus network topology via label-shuffling permutation.

## When NOT to use

- Input is spatial transcriptomics data or non-cortex scRNA-seq data for which no retrograde-labeling projection ground truth exists for scoring AUROC/AUPRC
- The goal is to test significance of individual communication links via label permutation inside run_NeuronChat, rather than to assess whether overall benchmark performance depends on cell-type labels

## Inputs

- Processed mouse cortex scRNA-seq data (cortex_data, Yao 2021) with Glutamatergic cells from VISp and ALM
- Cell subclass labels per cell (group.by vector)
- Retrograde-labeling projection ground-truth networks for VISp and ALM

## Outputs

- AUROC/AUPRC distributions for projection prediction under original and shuffled subclass labels
- Per-region shuffled-label p-values (VISp p = 0.010 ± 0.0036; ALM p = 0.017 ± 0.0048)
- Aggregated communication-weight networks (net_aggregation output)
- shuffled_label_pvalues.csv summary file

## How to apply

Load the processed mouse cortex scRNA-seq data (cortex_data from Yao 2021), subset the Glutamatergic cells for VISp and ALM, and create NeuronChat objects with DB='mouse' and group.by set to cell subclass. Run the baseline inference with run_NeuronChat(M=100) and aggregate the inferred networks with net_aggregation(method='weight') to obtain communication strengths used to predict ground-truth projections, scoring with AUROC/AUPRC under the thresholded weight aggregation whose threshold ak is the 80% quantile (the default for benchmarking). As the ablation control, randomly shuffle the subclass labels across cells while keeping the expression matrix unchanged, rerun run_NeuronChat(M=100) on the permuted labels (each run takes roughly 4 minutes), recompute AUROC/AUPRC, and repeat over multiple random seeds to estimate the mean and standard deviation of shuffled-label p-values per region. Rationale: if AUROC for original labels is significantly higher than for shuffled labels, performance reflects cell-type-specific expression rather than the specific graph topology of the retrograde-labeling ground truth.

## Related tools

- **NeuronChat** (Creates NeuronChat objects (DB='mouse', group.by = cell subclass), runs the communication inference via run_NeuronChat(M=100), and aggregates networks via net_aggregation for AUROC/AUPRC evaluation against projection ground truth) — https://github.com/Wei-BioMath/NeuronChat
- **NeuronChatAnalysis2022** (Analysis scripts/repository for reproducing the benchmarking figures and shuffled-label robustness results of the NeuronChat study) — https://github.com/Wei-BioMath/NeuronChatAnalysis2022

## Examples

```
x <- createNeuronChat(normalized_count_mtx, DB='mouse', group.by = cell_group_vector); x <- run_NeuronChat(x, M=100); net_aggregated_x <- net_aggregation(x@net, method = 'weight')
```

## Evaluation signals

- Reproduced shuffled-label p-values fall within the reported uncertainties: 0.010 ± 0.0036 for VISp and 0.017 ± 0.0048 for ALM
- Shuffled-label AUROC drops to a distribution with mean ~0.5 (chance level), while AUROC for original labels is significantly higher
- Aggregated network is produced only after run_NeuronChat(M=100), matching the workflow order createNeuronChat → run_NeuronChat → net_aggregation
- Shuffling permutes subclass labels only; the expression matrix must remain unchanged across permutations
- Per-region AUROC/AUPRC summaries and shuffled-label p-values are written to shuffled_label_pvalues.csv

## Limitations

- Each run_NeuronChat(M=100) run takes roughly 4 minutes, and permutation-based p-value calculation has a trade-off between accuracy and computation speed (time complexity O(n) in number of permutations)
- NeuronChatDB may contain missing curated interaction entries, leading to bias in the inference
- Ligand/target abundance estimated from transcriptomics may be inconsistent with protein or metabolite levels
- Genes expressed in less than 25% of cells are filtered out, which constrains the interaction space evaluated

## Evidence

- [methods] run_NeuronChat call: "x <- run_NeuronChat(x,M=100)"
- [methods] net_aggregation call: "net_aggregated_x <- net_aggregation(x@net,method = 'weight')"
- [discussion] thresholded weight aggregation default for benchmarking: "This method is the default setting for benchmarking NeuronChat, with threshold ak defined as the 80% quantile"
- [discussion] gene filtering cutoff: "Because the genes expressed in less than 25% of cells are filtered out"
- [readme] NeuronChat purpose: "The goal of NeuronChat is to infer, visualize and analyze neural-specific cell-cell communication from single cell transcriptomics or spatially resolved transcriptomics data."
- [discussion] permutation significance: "The statistical significance of each communication is calculated through the permutation test by randomly permutating group labels of cells"
