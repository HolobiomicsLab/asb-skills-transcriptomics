---
name: cross-run-clustering-consistency-measurement
description: Use when scRNAseq cells have been clustered by Sparsely-Connected Autoencoder
  (SCA) across repeated runs and you must quantify cluster reconstruction reproducibility
  (QCC/QCM >= 0.5) or validate the autoencoder's clustering stability.
when_to_use_negative:
- Only a single clustering result exists (no repeated SCA runs) — QCC and QCM are
  cross-run metrics and cannot be computed without multiple run labelings
- The goal is functional annotation of clusters (TF/miRNA/kinase mining from the SCA
  encoded space) rather than evaluating the robustness/reproducibility of the clustering
  itself
- Clusters were produced by a non-SCA method — the 0.5 explainability threshold was
  calibrated for SCA's selective cluster reconstruction and does not transfer automatically
edam_operation: http://edamontology.org/operation_3432
edam_topics:
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_3474
tools:
- name: SCA
  role: Sparsely-Connected Autoencoder module whose repeated clustering runs supply
    the cross-run label assignments scored by QCC/QCM
  repo: https://github.com/kendomaniac/SCAtutorial
- name: rCASC
  role: R framework hosting the SCA module; the QCC/QCM scoring metrics are designed
    and implemented within this environment
  repo: https://github.com/kendomaniac/rCASC
- name: R
  role: Language used to implement QCC/QCM from their formal definitions and to run
    the unit tests
  repo: https://github.com/kendomaniac/rCASC
- name: devtools
  role: Used to install rCASC (install_github) so the SCA module and its scoring metrics
    can be exercised
  repo: https://github.com/kendomaniac/rCASC
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1038/s41540-020-00162-6
    title: Sparsely-connected autoencoder (SCA) for single cell RNAseq data mining
  source_content_hash: sha256:4d577fb20442fc2654a3a586552f36101b248e834f333994aa3317c3769f551b
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/kendomaniac/SCAtutorial
  tools:
  - R
  - SCA
  - devtools
  - rCASC
  edam_operation: http://edamontology.org/operation_3432
  edam_topics:
  - http://edamontology.org/topic_3308
  - http://edamontology.org/topic_3474
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.625
n_claims: 8
---

# cross-run-clustering-consistency-measurement

## Summary

Implements the SCA paper's QCC (Quality Control of Cluster) and QCM (Quality Control of Model) metrics, which quantify how consistently cell clusters are reconstructed across multiple SCA clustering runs, using QCC/QCM >= 0.5 to declare a cluster 'explainable' by SCA analysis.

## When to use

Use when scRNAseq cells have been clustered by Sparsely-Connected Autoencoder (SCA) across repeated runs and you must quantify cluster reconstruction reproducibility (QCC/QCM >= 0.5) or validate the autoencoder's clustering stability.

## When NOT to use

- Only a single clustering result exists (no repeated SCA runs) — QCC and QCM are cross-run metrics and cannot be computed without multiple run labelings
- The goal is functional annotation of clusters (TF/miRNA/kinase mining from the SCA encoded space) rather than evaluating the robustness/reproducibility of the clustering itself
- Clusters were produced by a non-SCA method — the 0.5 explainability threshold was calibrated for SCA's selective cluster reconstruction and does not transfer automatically

## Inputs

- Multiple cluster-assignment matrices (cell x run label vectors) produced by repeated SCA runs on the same scRNAseq dataset
- Reference cluster definitions (cells belonging to a cluster of interest, e.g., blood-cell clusters 1 and 2 of setA)
- Synthetic paired run labelings with controlled overlap for unit-testing the metrics

## Outputs

- Per-reference-cluster QCC score normalized to [0,1]
- Per-run-pair QCM consistency score normalized to [0,1]
- Pass/fail unit-test report with computed metric values (qcc_qcm_unit_test_report.txt)

## How to apply

Run SCA (the rCASC module) clustering multiple times on the same scRNAseq counts matrix so each run yields a cell-to-cluster label assignment. For each reference cluster, compute QCC as the co-localization frequency: how often cells belonging to that reference cluster land together in the same cluster across the SCA runs, normalized to the 0-1 range. Compute QCM as clustering consistency measured across random pairs of SCA runs, likewise constrained to 0-1. Apply the paper's decision threshold: clusters with both QCC and QCM >= 0.5 are considered explainable by SCA analysis, since SCA is expected to reconstruct only specific clusters where its encoded space drives aggregation. Validate the implementation with synthetic cluster-assignment matrices (paired run labelings with controlled high- and low-overlap regimes), asserting values stay in [0,1] and the 0.5 cutoff separates high- from low-quality cases, writing results to qcc_qcm_unit_test_report.txt.

## Related tools

- **SCA** (Sparsely-Connected Autoencoder module whose repeated clustering runs supply the cross-run label assignments scored by QCC/QCM) — https://github.com/kendomaniac/SCAtutorial
- **rCASC** (R framework hosting the SCA module; the QCC/QCM scoring metrics are designed and implemented within this environment) — https://github.com/kendomaniac/rCASC
- **R** (Language used to implement QCC/QCM from their formal definitions and to run the unit tests) — https://github.com/kendomaniac/rCASC
- **devtools** (Used to install rCASC (install_github) so the SCA module and its scoring metrics can be exercised) — https://github.com/kendomaniac/rCASC

## Examples

```
Rscript -e 'devtools::install_github("kendomaniac/rCASC"); # run SCA clustering n times, then compute QCC (co-localization frequency per reference cluster) and QCM (consistency over random run pairs); stopifnot(qcc>=0 & qcc<=1, qcm>=0.5) ; write qcc_qcm_unit_test_report.txt'
```

## Evaluation signals

- Both QCC and QCM return values strictly within the [0,1] range for all synthetic and real inputs
- Synthetic high-agreement run pairs score QCC and QCM >= 0.5 while low-agreement pairs fall below 0.5, so the 0.5 threshold cleanly separates the regimes
- Identical labelings across runs yield a consistency score of 1; fully non-co-localizing reference cells yield QCC near 0
- Report file qcc_qcm_unit_test_report.txt records pass/fail per assertion together with the computed metric values
- Real SCA runs reproduce the paper's behavior that only specific clusters (where the SCA encoded space drives aggregation) pass the 0.5 explainability cutoff

## Limitations

- The QCC/QCM >= 0.5 explainability cutoff is a convention from this paper; it separates suggested high/low quality but is not a universal significance threshold
- QCM depends on which random pairs of SCA runs are sampled, so scores can vary with run sampling; SCA clustering is stochastic
- SCA analyses are noisy because scRNAseq is noisy and dropout affects lowly expressed genes (e.g., transcription factors and kinases), which destabilizes clustering and hence consistency scores
- The rCASC/SCA tooling requires a 64-bit Linux machine with the Docker daemon installed; Swarm-based farm execution was still under implementation

## Evidence

- [abstract] QCC/QCM metrics exist to quantify SCA cluster reconstruction: "We implemented two new metrics, QCC (Quality Control of Cluster) and QCM (Quality Control of Model), which allow quantifying the ability of SCA to reconstruct valuable cell clusters"
- [abstract] SCA is expected to reconstruct only specific clusters: "SCA efficacy comes from its ability to reconstruct only specific clusters, thus indicating only those clusters where the SCA encoding space is a key element for cells aggregation"
- [discussion] Scoring metrics were designed within the SCA implementation: "we developed a user-friendly implementation of SCA and designed specific scoring metrics to evaluate the efficacy of SCA analysis"
- [discussion] Metrics estimate robustness of the SCA encoding: "A very important element of our SCA implementation is the availability of metrics estimating the robustness of the SCA encoding."
- [readme] SCA is used within rCASC on scRNAseq data: [evidence span withheld — anchor status: not_found]
