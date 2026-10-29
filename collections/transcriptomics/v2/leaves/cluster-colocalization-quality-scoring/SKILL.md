---
name: cluster-colocalization-quality-scoring
description: Use when scoring how reproducibly a reference single-cell cluster is
  reconstructed across multiple SCA clustering runs, or when validating SCA model
  stability with QCC/QCM against the 0.5 threshold.
when_to_use_negative:
- Input is a raw UMI count matrix (e.g., an inDrop or Cell Ranger counts table) rather
  than cluster-assignment labels from SCA runs — filtering/counting steps like filterZeros
  or lorenzFilter apply first
- Only a single deterministic clustering result is available, since QCC/QCM require
  multiple SCA runs to measure co-localization and consistency
- The goal is functional annotation of clusters (TFs, miRNAs, kinases driving clusterization)
  rather than quantifying reconstruction robustness
edam_operation: http://edamontology.org/operation_3432
edam_topics:
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_3577
tools:
- name: rCASC
  role: Framework in which SCA analysis and its QCC/QCM scoring metrics are implemented
    as modules
  repo: github.com/kendomaniac/rCASC
- name: SCA
  role: Sparsely-Connected Autoencoder whose repeated clustering runs produce the
    labelings scored by QCC/QCM
  repo: github.com/kendomaniac/SCAtutorial
- name: devtools
  role: Used to install rCASC from GitHub via install_github before running the metrics
  repo: github.com/kendomaniac/rCASC
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1038/s41540-020-00162-6
    title: Sparsely-connected autoencoder (SCA) for single cell RNAseq data mining
  source_content_hash: sha256:8101b9b77cc265f084ddbe6fc0ade5be9401ba7484ab303e34181d468a40d101
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - R
  - SCA
  - devtools
  - rCASC
  edam_operation: http://edamontology.org/operation_3432
  edam_topics:
  - http://edamontology.org/topic_3308
  - http://edamontology.org/topic_3577
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8571428571428571
n_claims: 14
---

# cluster-colocalization-quality-scoring

## Summary

Implements the QCC (Quality Control of Cluster) and QCM (Quality Control of Model) metrics from the SCA/rCASC paper to quantify how well the Sparsely-Connected Autoencoder reconstructs valuable cell clusters and how consistent its neural network results are across runs, with QCC and QCM >= 0.5 defining a cluster explainable by SCA analysis.

## When to use

Use when scoring how reproducibly a reference single-cell cluster is reconstructed across multiple SCA clustering runs, or when validating SCA model stability with QCC/QCM against the 0.5 threshold.

## When NOT to use

- Input is a raw UMI count matrix (e.g., an inDrop or Cell Ranger counts table) rather than cluster-assignment labels from SCA runs — filtering/counting steps like filterZeros or lorenzFilter apply first
- Only a single deterministic clustering result is available, since QCC/QCM require multiple SCA runs to measure co-localization and consistency
- The goal is functional annotation of clusters (TFs, miRNAs, kinases driving clusterization) rather than quantifying reconstruction robustness

## Inputs

- Cell-to-cluster assignment matrices (cluster labels per cell) from multiple SCA clustering runs
- Reference cluster labels defining the cell set whose reconstruction frequency is scored
- Synthetic paired run labelings with controlled overlap for unit testing

## Outputs

- QCC score per reference cluster, normalized to [0,1]
- QCM clustering-consistency score across random SCA run pairs, in [0,1]
- Binary explainability flag (QCC and QCM >= 0.5)
- Pass/fail results and computed metric values written to qcc_qcm_unit_test_report.txt

## How to apply

Read the article's Methods definitions of QCC and QCM, which extend the CSS scoring framework. Implement QCC in R as the co-localization frequency of cells belonging to a reference cluster across multiple SCA clustering runs, returning a score normalized to the 0-1 range, and implement QCM as a measure of clustering consistency computed across random pairs of SCA runs, likewise constrained to 0-1. Generate synthetic cluster-assignment matrices — e.g., paired run labelings with controlled overlap — that exercise both the high-agreement and low-agreement regimes. Run unit tests asserting both metrics return values within [0,1] and that the 0.5 threshold cleanly separates high-quality from low-quality synthetic cases. Write pass/fail results and computed metric values to qcc_qcm_unit_test_report.txt. A cluster is declared explainable by SCA analysis only when both QCC and QCM are >= 0.5.

## Related tools

- **rCASC** (Framework in which SCA analysis and its QCC/QCM scoring metrics are implemented as modules) — github.com/kendomaniac/rCASC
- **SCA** (Sparsely-Connected Autoencoder whose repeated clustering runs produce the labelings scored by QCC/QCM) — github.com/kendomaniac/SCAtutorial
- **devtools** (Used to install rCASC from GitHub via install_github before running the metrics) — github.com/kendomaniac/rCASC

## Examples

```
install.packages("devtools"); library(devtools); install_github("kendomaniac/rCASC")
```

## Evaluation signals

- Both QCC and QCM return values within [0,1] for all tested inputs
- The 0.5 threshold cleanly separates synthetic high-agreement from low-agreement cluster-assignment pairs
- Identical labelings across runs yield QCC = QCM = 1; disjoint labelings yield values near 0
- QCC computed per reference cluster matches the expected co-localization frequency on synthetic matrices with controlled overlap
- Pass/fail results and computed metric values are present in qcc_qcm_unit_test_report.txt

## Limitations

- The metrics quantify reconstruction consistency, not biological correctness of a cluster; a stable but wrong clustering still scores high
- The QCC/QCM >= 0.5 explainability threshold is the paper's suggested convention and is heuristic
- Scores depend on running SCA multiple times; scRNAseq is very noisy and dropout affects poorly expressed genes, which can destabilize cluster assignments feeding the metrics

## Evidence

- [full_text] QCC/QCM definition with 0.5 threshold: [evidence span withheld — anchor status: not_found]
- [abstract] Abstract claim of the two metrics: "We implemented two new metrics, QCC (Quality Control of Cluster) and QCM (Quality Control of Model), which allow quantifying the ability of SCA to reconstruct valuable cell clusters"
- [discussion] Robustness metrics as key element: "A very important element of our SCA implementation is the availability of metrics estimating the robustness of the SCA encoding."
- [readme] SCA available as rCASC module: [evidence span withheld — anchor status: not_found]
