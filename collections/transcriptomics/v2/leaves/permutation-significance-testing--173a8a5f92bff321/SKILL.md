---
name: permutation-significance-testing
description: Use when a NeuronChat run on scRNA-seq or spatial transcriptomics must
  identify which sender–receiver communication links are significant (p < 0.05) before
  aggregating networks, ranking interactions, or comparing conditions.
when_to_use_negative:
- Input is a spatial dataset where the goal is spatial proximity enrichment — that
  uses its own 1000-permutation label-permutation of cell/spot labels (cell_proximity_enrichment_score_single/multiple),
  not the M=100 group-label permutation in run_NeuronChat.
- The analysis goal is method benchmarking (AUROC/AUPRC against retrograde-tracing
  ground truth) without significance filtering context — the shuffled-label ablation
  of task_003 is a different permutation design (shuffling subclass labels to test
  topology dependence, not to assign link significance).
- Expression data has not been preprocessed (normalized to 10,000 scale with pseudocount
  1 and log-transformed) and genes expressed in <25% of cells have not been filtered
  — running the permutation test on raw counts invalidates the Tukey-trimean ensemble
  averages.
edam_operation: http://edamontology.org/operation_2238
edam_topics:
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_3170
tools:
- name: NeuronChat
  role: Implements the permutation test (run_NeuronChat with M=100) that permutes
    cell-group labels, recalculates communication strength, and applies Benjamini–Hochberg
    correction
  repo: https://github.com/Wei-BioMath/NeuronChat
- name: CellChat
  role: Dependency of NeuronChat; its scoring rule is reimplemented within the shared
    NeuronChat workflow and is subject to the same permutation-significance filtering
    in benchmarking
  repo: https://github.com/sqjin/CellChat
provenance:
  source_task_ids:
  - task_001
  - task_004
  - task_005
  - task_007
  - task_008
  - task_003
  source_papers:
  - doi: 10.1038/s41467-023-36800-w
    title: Inferring neuron-neuron communications from single-cell transcriptomics
      through NeuronChat
  source_content_hash: sha256:e39bf4edc8a817367d84a8c4abeed7f40557d8ecbf0ad3a5f9184741e9072464
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/Wei-BioMath/NeuronChat
  tools:
  - CellChat
  - CellPhoneDB
  - NeuronChat
  edam_operation: http://edamontology.org/operation_2238
  edam_topics:
  - http://edamontology.org/topic_0203
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8181818181818182
n_claims: 11
---

# permutation-significance-testing

## Summary

Assess the statistical significance of NeuronChat-inferred intercellular communication strengths by permuting cell-group labels and recomputing ligand-target communication, followed by Benjamini–Hochberg correction so only p < 0.05 links are retained.

## When to use

Use when a NeuronChat run on scRNA-seq or spatial transcriptomics must identify which sender–receiver communication links are significant (p < 0.05) before aggregating networks, ranking interactions, or comparing conditions.

## When NOT to use

- Input is a spatial dataset where the goal is spatial proximity enrichment — that uses its own 1000-permutation label-permutation of cell/spot labels (cell_proximity_enrichment_score_single/multiple), not the M=100 group-label permutation in run_NeuronChat.
- The analysis goal is method benchmarking (AUROC/AUPRC against retrograde-tracing ground truth) without significance filtering context — the shuffled-label ablation of task_003 is a different permutation design (shuffling subclass labels to test topology dependence, not to assign link significance).
- Expression data has not been preprocessed (normalized to 10,000 scale with pseudocount 1 and log-transformed) and genes expressed in <25% of cells have not been filtered — running the permutation test on raw counts invalidates the Tukey-trimean ensemble averages.

## Inputs

- NeuronChat object created with createNeuronChat(normalized_count_mtx, DB='mouse'|'human', group.by=cell_group_vector)
- Normalized count matrix (scaled to 10,000, pseudocount 1, natural-log transformed) with cell-group labels
- Permutation count M (default 100 in the paper's workflows)

## Outputs

- NeuronChat object with per-interaction communication-strength matrices and BH-corrected p-values
- Thresholded weight matrix (# cell groups x # cell groups) with non-significant entries (p >= 0.05) set to zero
- Set of significant interaction pairs (e.g., 109 significant pairs for the VISp dataset)

## How to apply

After computing communication strength LT_i,j for each sender–receiver cell-group pair (product of ligand abundance L_i and target abundance T_j, both from Tukey's-trimean averaged expression), determine significance with run_NeuronChat(x, M=100), which randomly permutes group labels of cells and recalculates communication strength 100 times. Correct the per-interaction p-values for multiple testing with the Benjamini–Hochberg procedure, and set the communication strength of non-significant links (p >= 0.05) to zero before net_aggregation. The number of permutations M is a decision point: M=100 is the paper's default and the setting used in all benchmark tasks (VISp, ALM, ASD vs control), and each run takes roughly 4 minutes, reflecting the O(n) time complexity in the number of permutations — a trade-off between accuracy and computation speed. Rationale: without the permutation test and BH correction, background expression co-occurrence would masquerade as significant neural communication, and downstream benchmark AUROC/AUPRC values (e.g., 0.832/0.915 for VISp) depend on thresholding to significant links only.

## Related tools

- **NeuronChat** (Implements the permutation test (run_NeuronChat with M=100) that permutes cell-group labels, recalculates communication strength, and applies Benjamini–Hochberg correction) — https://github.com/Wei-BioMath/NeuronChat
- **CellChat** (Dependency of NeuronChat; its scoring rule is reimplemented within the shared NeuronChat workflow and is subject to the same permutation-significance filtering in benchmarking) — https://github.com/sqjin/CellChat

## Examples

```
x <- run_NeuronChat(x,M=100); net_aggregated_x <- net_aggregation(x@net,method='weight')
```

## Evaluation signals

- Output weight matrix has dimensions equal to the number of cell groups squared; entries with p >= 0.05 (BH-corrected) are exactly zero, significant entries lie within a valid probability range.
- VISp run on Yao 2021 cortex_data (Glutamatergic cells, subclass grouping) yields 109 significant interaction pairs matching the published Fig. 4 results.
- Permutation p-values are computed over M=100 permutations and corrected with Benjamini–Hochberg; verify the run completed the full permutation set (each run ~4 minutes).
- Shuffled-label control: randomizing subclass labels drops AUROC to a distribution with mean ~0.5, with original-label significance p = 0.010 ± 0.0036 (VISp) and p = 0.017 ± 0.0048 (ALM).

## Limitations

- Trade-off between accuracy and computation speed for permutation-based p-value calculations, which have time complexity O(n) in the number of permutations.
- NeuronChat estimates ligand/target abundance from transcriptomics, which may be inconsistent with protein or metabolite levels.
- M=100 permutations per run (~4 minutes each) makes multi-seed robustness analyses (e.g., the shuffled-label ablation) computationally expensive.

## Evidence

- [discussion] permutation test by randomly permutating group labels of cells: "The statistical significance of each communication is calculated through the permutation test by randomly permutating group labels of cells"
- [discussion] Benjamini–Hochberg correction and p < 0.05 threshold: "The p-value is corrected for multiple tests by using Benjamini–Hochberg procedure"
- [methods] run_NeuronChat call with M=100: "x <- run_NeuronChat(x,M=100)"
- [discussion] p < 0.05 significance and zeroing of non-significant links: "communications with p-value < 0.05 are considered significant; for non-significant communication links, the communication strength values are set to be zeros"
- [discussion] permutation speed trade-off: "There is a trade-off between accuracy and computation speed for permutation-based p-value calculations"
