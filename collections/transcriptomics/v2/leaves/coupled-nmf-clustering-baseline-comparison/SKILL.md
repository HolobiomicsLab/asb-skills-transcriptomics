---
name: coupled-nmf-clustering-baseline-comparison
description: Use when evaluating whether adding bulk HiChIP loop data (DC3) improves
  subpopulation clustering of scRNA-seq/scATAC-seq cells, by comparing against coupled
  NMF (no loop data) and separate per-datatype NMF under the same conditions.
when_to_use_negative:
- When the goal is deconvolution of bulk HiChIP loop counts or inference of subpopulation-specific
  enhancer-promoter contacts — coupled NMF consumes no HiChIP data, so it cannot address
  these
- When scRNA-seq and scATAC-seq come from jointly profiled single cells (e.g. sci-CAR
  multimodal data) rather than separate samples from the same population — the coupled
  NMF design assumes separate single-cell samples
edam_operation: http://edamontology.org/operation_3432
edam_topics:
- http://edamontology.org/topic_0203
tools:
- name: DC3
  role: The full method whose no-loop-data counterpart is coupled NMF; DC3's coupled-clustering
    core reduces to the coupled NMF baseline when the bulk HiChIP input is omitted,
    and is run in parallel (50 runs per dropout rate) to compute relative error reduction
  repo: https://github.com/SUwonglab/DC3
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1038/s41467-019-12547-1
    title: DC3 is a method for deconvolution and coupled clustering from bulk and
      single-cell genomics data
  source_content_hash: sha256:55baf835200faad3ce3e118630748a19b27726733e7f0ea121400a37528f1c8b
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/SUwonglab/DC3
  tools:
  - DC3
  edam_operation: http://edamontology.org/operation_3432
  edam_topics:
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8888888888888888
n_claims: 9
---

# coupled-nmf-clustering-baseline-comparison

## Summary

Runs the coupled NMF baseline from Duren et al. — coupled clustering of scRNA-seq and scATAC-seq without loop data — alongside per-datatype NMF to benchmark how much bulk HiChIP information (DC3) actually improves single-cell clustering, e.g. across simulated dropout rates.

## When to use

Use when evaluating whether adding bulk HiChIP loop data (DC3) improves subpopulation clustering of scRNA-seq/scATAC-seq cells, by comparing against coupled NMF (no loop data) and separate per-datatype NMF under the same conditions.

## When NOT to use

- When the goal is deconvolution of bulk HiChIP loop counts or inference of subpopulation-specific enhancer-promoter contacts — coupled NMF consumes no HiChIP data, so it cannot address these
- When scRNA-seq and scATAC-seq come from jointly profiled single cells (e.g. sci-CAR multimodal data) rather than separate samples from the same population — the coupled NMF design assumes separate single-cell samples

## Inputs

- scRNA-seq expression matrix E (genes x cells) from the heterogeneous population
- scATAC-seq openness matrix O (enhancers x cells) from the same population
- True subpopulation labels for cluster-assignment error scoring (e.g. GM12878 vs K562)
- Chosen dropout rates for simulated dropout (e.g. [0, 0.5, 0.8, 0.9])

## Outputs

- Per-run cluster assignments for scRNA-seq and scATAC-seq cells from coupled NMF and from per-datatype NMF
- Mean classification error rates (with standard deviation) per method per dropout rate, e.g. written to dropout_clustering_error_rates.csv

## How to apply

Build the comparison inputs exactly as DC3: the scRNA-seq expression matrix E (genes x cells), the scATAC-seq openness matrix O (enhancers x cells), and true subpopulation labels for scoring. Where the protocol simulates dropout, draw Bernoulli masks per gene or enhancer site and zero out entries in E and O at each chosen rate (e.g. 0, 0.5, 0.8, 0.9). Run the coupled NMF baseline (coupled clustering of E and O without loop data) and plain per-datatype NMF independently at each condition, using the same number of independent runs with different random initializations as the DC3 runs (50 runs per condition in the in silico GM12878/K562 mixture). For each run, compute the classification error between true labels (e.g. GM12878 vs K562) and predicted cluster assignments, average over scRNA-seq and scATAC-seq cells and over runs, and tabulate mean error with standard deviation per method per dropout rate. Rationale: the baseline isolates the contribution of the bulk HiChIP coupled term; the paper validates DC3 by showing it reduces classification error more than four folds versus this no-loop-data clustering at 80% dropout or higher.

## Related tools

- **DC3** (The full method whose no-loop-data counterpart is coupled NMF; DC3's coupled-clustering core reduces to the coupled NMF baseline when the bulk HiChIP input is omitted, and is run in parallel (50 runs per dropout rate) to compute relative error reduction) — https://github.com/SUwonglab/DC3

## Evaluation signals

- At 80% dropout or higher, incorporating loop data (DC3 vs coupled NMF) should reduce mean classification error by more than four folds, matching the paper's reported result
- With no simulated dropout (rate 0), coupled NMF and DC3 error rates should be similar since the loop term adds little when scRNA/scATAC signal is complete
- Coupled NMF should generally outperform plain per-datatype NMF, since joint analysis of E and O should inform the shared subpopulation structure
- Error-rate table should contain mean and standard deviation aggregated over scRNA-seq and scATAC-seq cells and over 50 independent runs per condition

## Limitations

- Coupled NMF assumes the scRNA-seq and scATAC-seq samples come from the same population but are separately profiled; it does not ingest bulk HiChIP, so it cannot produce subpopulation-specific loop counts or peak-gene links
- Clustering quality degrades as gene/enhancer dropout increases; without the loop-data coupled term, classification error grows sharply at 80% dropout or higher
- Results depend on random initialization, so a single run is insufficient — the paper uses 50 independent runs with different initializations per condition

## Evidence

- [readme] Coupled NMF baseline as prior work from Duren et al.: "Recently Duren et al. proposed a coupled NMF (coupled non-negative matrix factorization) method to cluster cells in each sample and to infer both the expression profile and accessibility profile of"
- [intro] DC3 is introduced under various input-combination settings, of which the coupled NMF baseline is the no-HiChIP setting: "here we introduce DC3 as a method for the joint analysis of bulk and single cell data under various settings of input data combinations"
- [discussion] DC3's benefit over the baseline is improved single-cell clustering: "We showed that DC3 can decompose bulk profiles into subpopulation-specific profiles and at the same time enhance clustering performance of the single-cell data."
