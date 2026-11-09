---
name: single-cell-clustering-robustness-quantification
description: Use when you need to quantify the robustness or reproducibility of clusters
  produced by repeated stochastic single-cell clustering runs (e.g., SCA runs in rCASC),
  or when implementing cluster-stability validation metrics with a 0.5 explainability
  threshold.
when_to_use_negative:
- You have only a single clustering run — QCC and QCM are defined across multiple
  SCA runs and random run pairs, so a one-off clustering cannot be scored
- Your goal is to perform the clustering itself or to annotate cluster function (e.g.,
  SSCA signatures of TFs, miRNAs, kinases) rather than to evaluate clustering robustness
- No reference/valuable cluster definition exists — QCC measures reconstruction of
  a specific reference cluster, so it is undefined without one
edam_operation: http://edamontology.org/operation_3432
edam_topics:
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_3170
tools:
- name: SCA (Sparsely-Connected Autoencoder)
  role: Source of the repeated stochastic clusterings that QCC and QCM evaluate; its
    robustness metrics are the subject of this skill
  repo: github.com/kendomaniac/rCASC
- name: rCASC
  role: Framework in which SCA analysis is implemented as a module (with GUI), providing
    the workflow for generating multiple clustering runs to score
  repo: github.com/kendomaniac/rCASC
- name: devtools
  role: R package used to install rCASC (install_github('kendomaniac/rCASC')) from
    which the SCA/QCC/QCM modules are obtained
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1038/s41540-020-00162-6
    title: Sparsely-connected autoencoder (SCA) for single cell RNAseq data mining
  source_content_hash: sha256:d1792c22c73cb097e0bdb37b2744e394b435978d026f9a73329188e658c785d4
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - MAGIC
  - NMF
  - R
  - SC3
  - SCA (Sparsely-Connected Autoencoder)
  - Seurat
  - devtools
  - rCASC
  edam_operation: http://edamontology.org/operation_3432
  edam_topics:
  - http://edamontology.org/topic_3308
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
merged_aliases:
- single-cell-clustering-robustness-evaluation
triage_status: promoted
yes_rate: 0.8888888888888888
n_claims: 9
---

# single-cell-clustering-robustness-quantification

## Summary

Implements the QCC (Quality Control of Cluster) and QCM (Quality Control of Model) metrics from the SCA/rCASC paper, which quantify how faithfully the Sparsely-Connected Autoencoder reconstructs valuable cell clusters across repeated runs. QCC scores co-localization of reference-cluster cells across runs and QCM scores run-to-run cluster consistency, with QCC and QCM >= 0.5 defining a cluster explainable by SCA analysis.

## When to use

Use when you need to quantify the robustness or reproducibility of clusters produced by repeated stochastic single-cell clustering runs (e.g., SCA runs in rCASC), or when implementing cluster-stability validation metrics with a 0.5 explainability threshold.

## When NOT to use

- You have only a single clustering run — QCC and QCM are defined across multiple SCA runs and random run pairs, so a one-off clustering cannot be scored
- Your goal is to perform the clustering itself or to annotate cluster function (e.g., SSCA signatures of TFs, miRNAs, kinases) rather than to evaluate clustering robustness
- No reference/valuable cluster definition exists — QCC measures reconstruction of a specific reference cluster, so it is undefined without one

## Inputs

- Multiple cluster-assignment matrices (cell-to-cluster label vectors) from repeated SCA runs over the same scRNAseq counts table
- Reference cluster labels identifying the 'valuable' cell subpopulations to test for reconstruction
- Synthetic paired run labelings with controlled overlap (for unit-testing the metric implementations)

## Outputs

- QCC score per reference cluster, in [0,1], measuring cell co-localization frequency across SCA runs
- QCM score per cluster, in [0,1], measuring cluster consistency between random pairs of SCA runs
- Binary explainable/not-explainable call per cluster using the QCC and QCM >= 0.5 threshold
- Unit-test report (qcc_qcm_unit_test_report.txt) with pass/fail results and computed metric values

## How to apply

First read the article's Methods definitions of QCC and QCM, which extend the CSS scoring framework by quantifying how well SCA reconstructs valuable cell clusters. Run the SCA clustering multiple times (in rCASC, with controlled random seeds) to obtain multiple cluster-assignment matrices over the same cells. Compute QCC as the co-localization frequency of cells belonging to a reference cluster across these multiple SCA runs, normalized to 0-1; compute QCM as the consistency between random pairs of SCA runs, also constrained to 0-1. Apply the paper's operational cutoff: a cluster with QCC and QCM >= 0.5 is considered explainable by SCA analysis, indicating those clusters where the SCA encoding space is a key element for cell aggregation. Validate the implementation with synthetic cluster-assignment matrices exercising both high-agreement and low-agreement regimes, asserting both metrics stay within [0,1] and that the 0.5 threshold cleanly separates the regimes, writing pass/fail results and computed values to qcc_qcm_unit_test_report.txt.

## Related tools

- **SCA (Sparsely-Connected Autoencoder)** (Source of the repeated stochastic clusterings that QCC and QCM evaluate; its robustness metrics are the subject of this skill) — github.com/kendomaniac/rCASC
- **rCASC** (Framework in which SCA analysis is implemented as a module (with GUI), providing the workflow for generating multiple clustering runs to score) — github.com/kendomaniac/rCASC
- **devtools** (R package used to install rCASC (install_github('kendomaniac/rCASC')) from which the SCA/QCC/QCM modules are obtained)

## Examples

```
install.packages("devtools"); library(devtools); install_github("kendomaniac/rCASC")
```

## Evaluation signals

- Both QCC and QCM return values within the [0,1] range for all inputs, including adversarial synthetic cluster-assignment matrices
- High-agreement synthetic paired labelings score above 0.5 and low-agreement labelings score below 0.5, so the paper's threshold cleanly separates the regimes
- QCC approaches its maximum when cells of a reference cluster co-localize in the same cluster across all SCA runs, and drops as co-localization frequency decreases
- QCM is computed across random pairs of SCA runs and is invariant to the order in which run pairs are drawn
- qcc_qcm_unit_test_report.txt records pass/fail status and the computed metric values for every test case

## Limitations

- QCC/QCM require running SCA multiple times, which is compute-intensive; a Docker Swarm-based workflow for computer farms was still under implementation in the paper
- The 0.5 cutoff for QCC and QCM is an operational threshold defining 'explainable by SCA analysis', not a statistical test of significance
- Metrics inherit scRNAseq noise and dropout bias, which is greater for poorly expressed genes such as transcription factors and kinases

## Evidence

- [abstract] Two new metrics QCC/QCM quantify SCA cluster reconstruction: "We implemented two new metrics, QCC (Quality Control of Cluster) and QCM (Quality Control of Model), which allow quantifying the ability of SCA to reconstruct valuable cell clusters"
- [discussion] SCA implementation provides robustness metrics: "A very important element of our SCA implementation is the availability of metrics estimating the robustness of the SCA encoding."
- [abstract] SCA reconstruction defines which clusters are explainable: "SCA efficacy comes from its ability to reconstruct only specific clusters, thus indicating only those clusters where the SCA encoding space is a key element for cells aggregation"
- [readme] SCA usage via rCASC tutorial/README: [evidence span withheld — anchor status: not_found]
