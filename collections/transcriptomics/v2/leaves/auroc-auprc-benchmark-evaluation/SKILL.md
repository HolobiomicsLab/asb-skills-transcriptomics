---
name: auroc-auprc-benchmark-evaluation
description: Use when comparing GRN inference tools on synthetic or real single-cell
  datasets where a ground-truth network exists (e.g. BoolODE toggle-switch simulations
  with known topology), so predicted edges can be scored as AUROC/AUPRC/EPR.
when_to_use_negative:
- No ground-truth network is available to score predicted edges against (the article
  relies on BoolODE simulation ground truth or ChIP-Atlas ChIP-seq-derived GRNs).
- The goal is large-scale de novo inference from scratch rather than benchmarking
  — scGATE itself is applicable only to a small (<200) subset of genes with refined
  candidate TFs.
- The candidate regulators exceed ~5 TFs per target — decoding high-order Boolean
  logic beyond k=5 suffers overfitting, so benchmark results in that regime are unreliable.
edam_operation: http://edamontology.org/operation_2945
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: scGATE
  role: Tool under test; infers directed TF-gene edges (scGATE_edge, h_act=7) and
    Boolean logic gates (scGATE_logic, run_mode='simple', top_gates=1) that are scored
    against the ground truth
  repo: github.com/CompBioIPM/scGATE
- name: BoolODE
  role: Generates the synthetic toggle-switch scRNA-seq expression matrices and known
    ground-truth networks used as benchmark inputs
- name: CellOracle
  role: Baseline GRN inference tool run on the same simulated expression matrices
    for AUROC/AUPRC comparison
- name: scGATE 0.1.0
  role: Compiled R package version (scGATE_0.1.0.tar.gz) installed as the source of
    scGATE_logic()/scGATE_edge() functions used in the benchmark
  repo: github.com/CompBioIPM/scGATE
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1093/bib/bbae180
    title: Single-cell multi-omics analysis identifies context-specific gene regulatory
      gates and mechanisms
  source_content_hash: sha256:f3b863f4d6edc81657dd8a0e67c3bb8f783a99355e206a73a08bf4cea44d8aa1
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-4.0
    url: https://github.com/CompBioIPM/scGATE
  tools:
  - BoolODE
  - CellOracle
  - R 4.1.3
  - VGAM
  - arrow
  - scGATE
  - scGATE 0.1.0
  - truncnorm
  edam_operation: http://edamontology.org/operation_2945
  edam_topics:
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8571428571428571
n_claims: 14
---

# auroc-auprc-benchmark-evaluation

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Benchmark evaluation of single-cell GRN inference tools (e.g. scGATE vs. GRNBOOST2, LEAP, CellOracle) by scoring predicted TF-target edges against a known ground-truth network, reporting AUROC and AUPRC/EPR per tool and per condition on BoolODE-simulated single-cell data.

## When to use

Use when comparing GRN inference tools on synthetic or real single-cell datasets where a ground-truth network exists (e.g. BoolODE toggle-switch simulations with known topology), so predicted edges can be scored as AUROC/AUPRC/EPR.

## When NOT to use

- No ground-truth network is available to score predicted edges against (the article relies on BoolODE simulation ground truth or ChIP-Atlas ChIP-seq-derived GRNs).
- The goal is large-scale de novo inference from scratch rather than benchmarking — scGATE itself is applicable only to a small (<200) subset of genes with refined candidate TFs.
- The candidate regulators exceed ~5 TFs per target — decoding high-order Boolean logic beyond k=5 suffers overfitting, so benchmark results in that regime are unreliable.

## Inputs

- BoolODE-simulated scRNA-seq expression matrix (e.g. ClusterI.csv from the scGATE example_data folder)
- Ground-truth TF-target network from the BoolODE simulation
- Predicted edge lists from scGATE and baseline tools (GRNBOOST2, LEAP, CellOracle)
- R 4.1.3 environment with scGATE 0.1.0 and its dependencies (VGAM, truncnorm, arrow)

## Outputs

- Per-tool AUROC and AUPRC/EPR scores per cell type and dropout level
- Assembled benchmark_performance.csv comparing scGATE against baseline GRN inference tools

## How to apply

Load the BoolODE-simulated toggle-switch Cluster I expression matrix (ClusterI.csv) and preprocess it with scRNA_seq_preprocessing() using library_size_normalization="False" so counts are quantile-rescaled into the (0,1) interval, dropping genes with standard deviation <= 0.20. Run scGATE_logic() with run_mode="simple" and top_gates=1 and scGATE_edge() with h_act=7 to obtain Bayes-Factor-ranked TF-target edge lists, and run each baseline tool (CellOracle, GRNBOOST2, LEAP, GRNBOOST2 variants) on the identical expression matrices across the network-complexity and decoy-TF variants of the toggle switch. Score every tool's predicted edges against the known BoolODE ground-truth network, computing AUROC and AUPRC/EPR per tool and per condition (cell type, dropout level), then assemble the results into a benchmark_performance.csv. Correctness is judged by scGATE's AUROC/AUPRC exceeding all baselines, consistent with the article's reported AUROC > 0.98 across all cell types and dropout levels.

## Related tools

- **scGATE** (Tool under test; infers directed TF-gene edges (scGATE_edge, h_act=7) and Boolean logic gates (scGATE_logic, run_mode='simple', top_gates=1) that are scored against the ground truth) — github.com/CompBioIPM/scGATE
- **BoolODE** (Generates the synthetic toggle-switch scRNA-seq expression matrices and known ground-truth networks used as benchmark inputs)
- **CellOracle** (Baseline GRN inference tool run on the same simulated expression matrices for AUROC/AUPRC comparison)
- **scGATE 0.1.0** (Compiled R package version (scGATE_0.1.0.tar.gz) installed as the source of scGATE_logic()/scGATE_edge() functions used in the benchmark) — github.com/CompBioIPM/scGATE

## Examples

```
normalized_counts <- scRNA_seq_preprocessing(data = data_scRNA_seq, library_size_normalization = "False", tf_list = NA)
```

## Evaluation signals

- benchmark_performance.csv assembles AUROC and AUPRC/EPR for every tool and condition without missing entries.
- scGATE's AUROC exceeds the baselines (GRNBOOST2, LEAP, CellOracle) on every cell type and dropout level, consistent with the reported AUROC > 0.98.
- All tools are scored against the same known BoolODE ground-truth network on identical expression matrices across the network-complexity and decoy-TF variants of the toggle switch.
- Input expression matrix passed to scGATE contains normalized counts within the (0,1) interval after scRNA_seq_preprocessing() with library_size_normalization="False".
- Genes with standard deviation <= 0.20 have been removed before inference (low-variability gene filtering).

## Limitations

- scGATE is currently applicable to a rather small (<200) subset of genes with a refined list of candidate regulatory TFs based on external hints, so benchmarking large networks from scratch is out of scope.
- Decoding high-order Boolean logic involving more than 5 transcriptional regulators remains challenging due to overfitting; scGATE recommends fitting logic gates among subsets with up to k=5 candidate TFs.
- Large protein complex interactions are missed both in the current model and in the ground-truth networks, which can bias AUROC/EPR comparisons.
- Directed TF-gene networks fail to convey combinatorial relationships among regulating factors, so edge-level metrics alone do not capture logic-gate accuracy.

## Evidence

- [results] BoolODE simulation ground truth underlies the benchmark: "BoolODE package [21] was utilized for simulating scRNA-seq data"
- [abstract] Article's benchmarking claim against existing tools: "evaluated using synthetic and real single-cell multi-omics data from mouse tissues and human blood, demonstrating its superiority over existing tools"
- [readme] Preprocessing requirement for the expression matrix fed to inference and scoring: "A gene expression matrix with normalized counts within the (0,1) interval,"
- [results] Ground-truth GRN generation from ChIP-Atlas for real-data benchmarks: "ground-truth GRNs for these tissues were generated from the mouse TF chromatin immunoprecipitation"
- [methods] Recommendation limiting logic-gate subset size in benchmark design: [evidence span withheld — anchor status: not_found]
