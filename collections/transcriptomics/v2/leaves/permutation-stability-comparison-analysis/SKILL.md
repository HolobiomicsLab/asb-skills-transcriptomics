---
name: permutation-stability-comparison-analysis
description: Use when running NeuronChat's run_NeuronChat permutation test and you
  must verify that significance calls at a modest M (e.g., 100) are stable and consistent
  with a large-M reference run (e.g., 1000).
when_to_use_negative:
- You only need the aggregated communication network (net_aggregation over interaction
  pairs) and not validated significance calls — a single default run suffices without
  the M=1000 reference.
- You are analyzing spatial proximity enrichment, which uses its own label-permutation
  scheme (cell_proximity_enrichment_score with 1000 permutations by default) rather
  than link-significance permutations.
edam_operation: http://edamontology.org/operation_2238
edam_topics:
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_3170
tools:
- name: NeuronChat
  role: Runs the permutation test for communication-link significance (run_NeuronChat
    with configurable M) whose stability across M values is being assessed
  repo: https://github.com/Wei-BioMath/NeuronChat
- name: CellChat
  role: R dependency of NeuronChat whose framework underlies the workflow; alternative
    implementation sharing the same computational workflow for significance testing
  repo: https://github.com/sqjin/CellChat
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1038/s41467-023-36800-w
    title: Inferring neuron-neuron communications from single-cell transcriptomics
      through NeuronChat
  source_content_hash: sha256:c8cd4fdecceeba7dd1caef48e2c529e9a7e0fd9fd2265855ae195178d72afc70
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/Wei-BioMath/NeuronChat
  tools:
  - CellChat
  - NeuronChat
  edam_operation: http://edamontology.org/operation_2238
  edam_topics:
  - http://edamontology.org/topic_0203
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8888888888888888
n_claims: 9
---

# permutation-stability-comparison-analysis

## Summary

Assesses how many permutations (M) are needed for reliable NeuronChat link-significance calls by comparing significant-link sets from a low-permutation run (e.g., M=100) against a high-permutation run (e.g., M=1000) after Benjamini-Hochberg correction.

## When to use

Use when running NeuronChat's run_NeuronChat permutation test and you must verify that significance calls at a modest M (e.g., 100) are stable and consistent with a large-M reference run (e.g., 1000).

## When NOT to use

- You only need the aggregated communication network (net_aggregation over interaction pairs) and not validated significance calls — a single default run suffices without the M=1000 reference.
- You are analyzing spatial proximity enrichment, which uses its own label-permutation scheme (cell_proximity_enrichment_score with 1000 permutations by default) rather than link-significance permutations.

## Inputs

- NeuronChat object built with createNeuronChat from a normalized count matrix (e.g., cortex_data, Yao 2021) with DB='mouse' and group.by cell-group vector (e.g., subclass)
- Permutation count M for run_NeuronChat (e.g., 100 and 1000)

## Outputs

- Per-run sets of significant interaction links (BH-corrected p < 0.05) with non-significant communication strengths set to zero
- CSV comparison table of significant-link overlap counts and per-link p-value agreement between M=100 and M=1000 runs

## How to apply

Build two identical NeuronChat objects (e.g., from the packaged cortex_data of Yao 2021 via createNeuronChat with DB='mouse', grouping cells by subclass), then run run_NeuronChat with M=100 on one and M=1000 on the other. For each run, correct interaction p-values with the Benjamini-Hochberg procedure, call links significant at p<0.05, and set non-significant communication strengths to zero. Compare the resulting significant-link sets via overlap counts and per-link p-value agreement, saving the comparison as a CSV table. The rationale is that permutation-based p-values trade computation speed for accuracy (O(n) time in the number of permutations), so M=100 is the practical default but should be validated against larger M before drawing biological conclusions.

## Related tools

- **NeuronChat** (Runs the permutation test for communication-link significance (run_NeuronChat with configurable M) whose stability across M values is being assessed) — https://github.com/Wei-BioMath/NeuronChat
- **CellChat** (R dependency of NeuronChat whose framework underlies the workflow; alternative implementation sharing the same computational workflow for significance testing) — https://github.com/sqjin/CellChat

## Examples

```
x <- createNeuronChat(normalized_count_mtx,DB='mouse',group.by=cell_group_vector); x <- run_NeuronChat(x,M=100); x1000 <- run_NeuronChat(x,M=1000)
```

## Evaluation signals

- Significant-link sets at M=100 and M=1000 should show high overlap, matching the stability reported in Supplementary Fig. 3 of the paper.
- Per-link p-values between the M=100 and M=1000 runs should agree closely for links retained in both.
- All compared p-values must have been Benjamini-Hochberg corrected before calling significance at p<0.05.
- In every run, non-significant communication strengths must be exactly zero after thresholding.
- Both runs must use identical settings (same object, same database, same cell grouping) so differences reflect only the permutation count M.

## Limitations

- There is a trade-off between accuracy and computation speed for permutation-based p-value calculations (time complexity O(n) in the number of permutations), so low M may yield noisier p-values.
- P-values are corrected for multiple testing with Benjamini-Hochberg; skipping correction changes the significant-link set.
- Genes expressed in less than 25% of cells are filtered out upstream, which bounds which links can ever be called significant.

## Evidence

- [discussion] permutation test by randomly permutating group labels: "The statistical significance of each communication is calculated through the permutation test by randomly permutating group labels of cells"
- [discussion] BH correction and significance threshold: "The p-value is corrected for multiple tests by using Benjamini–Hochberg procedure"
- [discussion] non-significant links set to zero: "communications with p-value < 0.05 are considered significant; for non-significant communication links, the communication strength values are set to be zeros"
- [methods] M=100 default run: "x <- run_NeuronChat(x,M=100)"
- [discussion] accuracy-speed trade-off: "There is a trade-off between accuracy and computation speed for permutation-based p-value calculations"
- [readme] basic invocation from README: "# calculation of communication networks  
x <- run_NeuronChat(x,M=100)"
