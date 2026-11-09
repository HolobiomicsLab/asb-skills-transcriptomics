---
name: boolode-toggle-switch-simulation-data-handling
description: Use when benchmarking GRN inference tools (scGATE vs GRNBOOST2, LEAP,
  CellOracle) on synthetic single-cell data where a known ground-truth network and
  simulated expression matrices are available.
when_to_use_negative:
- Input is real scRNA-seq data needing library-size normalization — set library_size_normalization="True"
  rather than the simulation-specific "False" quantile-rescaling path
- 'Large-scale inference from scratch: scGATE is limited to a small (<200) subset
  of genes with a refined candidate TF list, so whole-transcriptome GRN inference
  is out of scope'
- Data with unknown ground truth where AUROC/EPR against a reference network cannot
  be computed
edam_topics:
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_3308
tools:
- name: scGATE
  role: R package providing scRNA_seq_preprocessing(), scGATE_logic(), and scGATE_edge()
    used to preprocess simulated data and infer edges/gates for benchmarking
  repo: https://github.com/CompBioIPM/scGATE
- name: BoolODE
  role: Simulator generating the synthetic toggle-switch scRNA-seq data with ground-truth
    networks
- name: CellOracle
  role: Baseline GRN inference tool run on the same simulated expression matrices
    for AUROC/EPR comparison
- name: R 4.1.3
  role: Runtime environment in which the scGATE code and benchmark are executed
- name: arrow
  role: R dependency used to read parquet inputs (e.g. base GRN dataframes) via read_parquet()
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1093/bib/bbae180
    title: Single-cell multi-omics analysis identifies context-specific gene regulatory
      gates and mechanisms
  source_content_hash: sha256:2ec63c626b4ab068db357949cf423e05089d02f9763c0caee87878503510a56f
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-4.0
    url: https://github.com/CompBioIPM/scGATE
  repo_url: https://github.com/CompBioIPM/scGATE
  tools:
  - BoolODE
  - CellOracle
  - R 4.1.3
  - VGAM
  - arrow
  - scGATE
  - truncnorm
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0203
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 1.0
n_claims: 13
---

# boolode-toggle-switch-simulation-data-handling

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Ingest, preprocess, and score the BoolODE-simulated synthetic toggle-switch scRNA-seq data (e.g. ClusterI.csv from the scGATE example_data folder) so that inferred TF-target edges can be benchmarked against the known ground-truth network using AUROC and EPR.

## When to use

Use when benchmarking GRN inference tools (scGATE vs GRNBOOST2, LEAP, CellOracle) on synthetic single-cell data where a known ground-truth network and simulated expression matrices are available.

## When NOT to use

- Input is real scRNA-seq data needing library-size normalization — set library_size_normalization="True" rather than the simulation-specific "False" quantile-rescaling path
- Large-scale inference from scratch: scGATE is limited to a small (<200) subset of genes with a refined candidate TF list, so whole-transcriptome GRN inference is out of scope
- Data with unknown ground truth where AUROC/EPR against a reference network cannot be computed

## Inputs

- BoolODE-simulated toggle-switch scRNA-seq expression matrix (CSV, e.g. example_data/ClusterI.csv)
- Known BoolODE ground-truth TF-target network for the toggle switch
- Network-complexity and decoy-TF variants of the toggle-switch simulation
- scGATE R package (scGATE_0.1.0.tar.gz) with VGAM, truncnorm, arrow dependencies

## Outputs

- Preprocessed expression matrix with values in (0,1) and low-variability genes (sd <= 0.20) removed
- Inferred logic gates with log10 Bayes Factors from scGATE_logic()
- BF-ranked TF-target edge lists from scGATE_edge()
- Per-tool, per-condition AUROC / AUPRC / EPR scores assembled as benchmark_performance.csv

## How to apply

Load the BoolODE-simulated toggle-switch expression matrix (e.g. ClusterI.csv from the scGATE example_data folder) and preprocess it with scGATE's scRNA_seq_preprocessing() setting library_size_normalization="False", which quantile-rescales expression into the (0,1) interval instead of library-size normalizing, then drop genes with standard deviation <= 0.20. For cell-level structure, perform Louvain clustering on the BoolODE simulated data as described in the accompanying Jupyter notebook. Run inference with scGATE_logic() (run_mode="simple", top_gates=1) and scGATE_edge() (h_act=7) on the same simulated matrices across the network-complexity and decoy-TF toggle-switch variants, alongside baseline tools such as CellOracle. Score each tool's predicted edges against the known BoolODE ground-truth network, computing AUROC and AUPRC/EPR per tool and per condition, and assemble the results into a benchmark_performance.csv.

## Related tools

- **scGATE** (R package providing scRNA_seq_preprocessing(), scGATE_logic(), and scGATE_edge() used to preprocess simulated data and infer edges/gates for benchmarking) — https://github.com/CompBioIPM/scGATE
- **BoolODE** (Simulator generating the synthetic toggle-switch scRNA-seq data with ground-truth networks)
- **CellOracle** (Baseline GRN inference tool run on the same simulated expression matrices for AUROC/EPR comparison)
- **R 4.1.3** (Runtime environment in which the scGATE code and benchmark are executed)
- **arrow** (R dependency used to read parquet inputs (e.g. base GRN dataframes) via read_parquet())

## Examples

```
normalized_counts <- scRNA_seq_preprocessing(data = data_scRNA_seq, library_size_normalization = "False", tf_list = NA)
```

## Evaluation signals

- benchmark_performance.csv must show scGATE's AUROC/AUPRC exceeding baselines (GRNBOOST2, LEAP, CellOracle), consistent with the article's reported AUROC > 0.98 across cell types and dropout levels
- Preprocessed expression values must lie within the (0,1) interval after library_size_normalization="False" quantile rescaling
- All genes with standard deviation <= 0.20 must be removed before inference
- Edge lists from scGATE_edge() must be ranked by Bayes Factor and gates report log10 BF values
- Scores must be computed per cell type and per dropout level against the known BoolODE ground-truth network

## Limitations

- BoolODE package version is not stated in the article, complicating exact reproduction
- scGATE results depend on toggle-switch variants (network complexity, decoy TFs); synthetic ground truth may miss interactions such as large protein complexes
- scGATE is applicable only to a small (<200) subset of genes with refined candidate TFs, so the benchmark does not test large-scale inference from scratch

## Evidence
