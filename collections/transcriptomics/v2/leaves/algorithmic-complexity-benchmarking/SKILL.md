---
name: algorithmic-complexity-benchmarking
description: Use when a paper asserts a time complexity (e.g., Augusta's O(mn²D²)
  for MI-based GRN inference from RNA-Seq count tables) and you must verify it empirically
  by timing scaled inputs and fitting runtime to the parameterized model.
when_to_use_negative:
- Benchmarking the refinement or BN-conversion stages — the article states the refinement
  of an initial GRN may be computationally very demanding, so the O(mn²D²) model applies
  only to the MI-based initial GRN step.
- Input lacks adjacent time points (e.g., steady-state or single-sample expression
  matrices) — edge orientation depends on the most significant difference (MSD) among
  adjacent pairs of time points, so the timed workload cannot be reproduced.
- The aim is to compare network quality rather than runtime scaling — the sensitivity/specificity/accuracy
  figures here are sanity anchors, not the benchmark target.
edam_topics:
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_0091
tools:
- name: Augusta
  role: MI-based GRN inference package whose O(mn²D²) initial-GRN step (RNASeq_to_GRN)
    is the workload being timed and fitted
  repo: https://github.com/JanaMus/Augusta
- name: Python
  role: 'Runtime environment and benchmark harness: pip install Augusta, drive RNASeq_to_GRN
    on scaled count tables, log seconds, and fit the O(mn²D²) model'
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1016/j.csbj.2024.01.013
    title: 'Augusta: From RNA‐Seq to gene regulatory networks and Boolean models'
  source_content_hash: sha256:ce195e90360d0203fd4cbe6ba151f84a23eb216343a81ad5c94200580fe897d2
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-ND-4.0
    url: https://github.com/WanluLiuLab/CRISPRTE
  repo_url: https://github.com/JanaMus/Augusta
  tools:
  - Augusta
  - Python
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3308
  - http://edamontology.org/topic_0203
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7777777777777778
n_claims: 9
---

# algorithmic-complexity-benchmarking

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Empirically verifies a claimed asymptotic runtime — here Augusta's O(mn²D²) for the mutual-information-based initial GRN inference — by timing the algorithm on progressively scaled time-series count tables, fitting measured seconds to the parameterized model, and anchoring absolute runtime against the paper's 83.07 s DREAM 4 MI-only benchmark.

## When to use

Use when a paper asserts a time complexity (e.g., Augusta's O(mn²D²) for MI-based GRN inference from RNA-Seq count tables) and you must verify it empirically by timing scaled inputs and fitting runtime to the parameterized model.

## When NOT to use

- Benchmarking the refinement or BN-conversion stages — the article states the refinement of an initial GRN may be computationally very demanding, so the O(mn²D²) model applies only to the MI-based initial GRN step.
- Input lacks adjacent time points (e.g., steady-state or single-sample expression matrices) — edge orientation depends on the most significant difference (MSD) among adjacent pairs of time points, so the timed workload cannot be reproduced.
- The aim is to compare network quality rather than runtime scaling — the sensitivity/specificity/accuracy figures here are sanity anchors, not the benchmark target.

## Inputs

- Time-series RNA-Seq count table (e.g., DREAM 4 In Silico Network Challenge first experiment: 100 genes, 21 samples)
- Claimed complexity expression O(mn²D²) with its parameter mapping (m = time points, n = genes, D = discretization bins)
- Augusta package installed via pip in a Python 3.7/3.8 virtual environment
- Paper-reported baseline: 83.07 s MI-only DREAM 4 runtime with sensitivity 0.38, specificity 0.54, accuracy 0.54

## Outputs

- timing_report.csv with measured seconds for progressively scaled gene counts (with m, n, D per run)
- Fitted scaling verdict confirming or refuting the O(mn²D²) complexity claim
- initial_grn_adjacency_matrix.csv — the MI-based network each timed run produces (acceptance anchor: nonzero edge recovery)

## How to apply

Run the MI-based initial GRN step via Augusta.RNASeq_to_GRN on a time-series count table — the paper's baseline workload is the DREAM 4 In Silico Network Challenge first experiment (100 genes, 21 samples) after CPM, RPKM, or TPM normalization. Discretize expression values by binning with D = min(n/5, 10), compute gene-wise entropy H(X), joint entropy H(X,Y), and pairwise mutual information in nats for every gene pair, and orient TF-TG edges by the most significant difference (MSD) among adjacent pairs of time points; this is the workload whose cost is claimed O(mn²D²), where m is the number of time points (expected in the order of units to tens at maximum) and n the number of genes. Progressively scale the input by increasing gene counts n while keeping the pipeline fixed, re-deriving D at each scale, and record wall-clock seconds per run in timing_report.csv together with m, n, and D. Fit measured seconds to the term m·n²·D² (e.g., log-log regression), noting that D saturates at 10 once n ≥ 50 so the n² term dominates large-n scaling. Accept the benchmark when the fitted growth matches the claimed complexity within an order of magnitude and the unscaled DREAM 4 run lands near the paper's 83.07 s MI-only runtime with nonzero edge recovery (sensitivity 0.38, specificity 0.54, accuracy 0.54).

## Related tools

- **Augusta** (MI-based GRN inference package whose O(mn²D²) initial-GRN step (RNASeq_to_GRN) is the workload being timed and fitted) — https://github.com/JanaMus/Augusta
- **Python** (Runtime environment and benchmark harness: pip install Augusta, drive RNASeq_to_GRN on scaled count tables, log seconds, and fit the O(mn²D²) model)

## Examples

```
Augusta.RNASeq_to_GRN(count_table_input = 'MyCT_file.csv', promoter_length = My_number, genbank_file_input = 'MyGB_file.gb', normalization_type = 'My_string', motifs_max_time = My_seconds)
```

## Evaluation signals

- Log-log fit of measured seconds against m·n²·D² across scaled gene counts yields slope ≈ 1, i.e., runtime grows roughly 4× per doubling of n once the D = min(n/5, 10) dependence is accounted for.
- The unscaled 100-gene, 21-sample DREAM 4 first-experiment run completes in the same order of magnitude as the paper's 83.07 s MI-only benchmark.
- Each timed run also emits initial_grn_adjacency_matrix.csv with nonzero edges, and quality metrics fall in the reported MI-only band (sensitivity 0.38, specificity 0.54, accuracy 0.54).
- timing_report.csv contains one row per scaled input with explicit m, n, D, and measured seconds columns.
- Varying m within the units-to-tens range at fixed n changes runtime approximately linearly, confirming the m factor of the model.

## Limitations

- D = min(n/5, 10) ties the bin count to n, so scaling n changes two factors of the cost; for n ≥ 50 the D² term saturates at 100 and the n² term dominates.
- m is expected 'in the order of units to tens at maximum', so the linear-in-m component can only be probed over a narrow range with real count tables.
- Refinement (de novo TFBM discovery, database incorporation) is outside the O(mn²D²) model and is computationally very demanding; parallelization is not yet implemented, so full-pipeline timings will not fit the model.
- Runtime conformance says nothing about accuracy: the MI-only network reaches sensitivity 0.38, and the paper cautions that inferred networks are approximations, not perfectly accurate representations of biological regulation.

## Evidence

- [other] computational complexity O(mn2D2), where m is the number of time points in the input count table (expected in the order of units to tens at maximum) and n represents the number of genes.: [evidence span withheld — anchor status: not_found]
- [other] setting the number of bins to D = min(n/5, 10), where n is the number of genes: [evidence span withheld — anchor status: not_found]
- [results] On the DREAM 4 dataset, Augusta without refinement (MI only) achieved sensitivity 0.38, specificity 0.54, accuracy 0.54 in 83.07 s: [evidence span withheld — anchor status: not_found]
- [methods] an initial Gene Regulatory Network (initial GRN) is inferred using a mutual information (MI) calculation: "an initial Gene Regulatory Network (initial GRN) is inferred using a mutual information (MI) calculation"
- [discussion] the refinement of an initial GRN may be computationally very demanding: "the refinement of an initial GRN may be computationally very demanding"
- [readme] Augusta.RNASeq_to_GRN(count_table_input = 'MyCT_file.csv', promoter_length = My_number, genbank_file_input = 'MyGB_file.gb', normalization_type = 'My_string', motifs_max_time = My_seconds): [evidence span withheld — anchor status: external_doc]
