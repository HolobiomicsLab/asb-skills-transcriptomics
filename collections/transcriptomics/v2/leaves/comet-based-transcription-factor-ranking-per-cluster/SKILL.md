---
name: comet-based-transcription-factor-ranking-per-cluster
description: Use when a cell cluster has already been reconstructed by TF-target-informed
  SCA (QCC and QCM > 0.5 for the majority of cells) and the goal is to identify transcription
  factors specific for that cluster from the latent space.
when_to_use_negative:
- The target cluster failed reconstruction — SCA reconstructs only specific clusters,
  so if QCC/QCM do not exceed 0.5 for the majority of cells, COMET ranking on that
  cluster's encoded space is not meaningful.
- The input is the raw counts table itself — COMET must be run on the SCA-encoded
  latent space, not directly on the raw scRNAseq/spatial counts matrix.
- No cluster structure has been defined yet — this skill assumes clusters were already
  defined by a clustering analysis (e.g., SIMLR) of the scRNAseq or spatial transcriptomics
  data.
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: SCA (Sparsely-Connected Autoencoder)
  role: Produces the TF-target-informed encoded latent space on which COMET is run
    to rank transcription factors
  repo: https://github.com/kendomaniac/rCASC
- name: rCASC
  role: Framework hosting the SCA module (with QCC/QCM metrics) whose latent space
    is the input for COMET TF ranking
  repo: https://github.com/kendomaniac/rCASC
- name: COMET (cometsc)
  role: Runs the specificity test on the SCA latent-space frequency table to rank
    TFs by COMETsc/TP/TN
- name: SIMLR
  role: Provides the pre-existing cluster assignment (e.g., 9 clusters on the Visium
    breast cancer section) defining the cluster-vs-rest contingency used by COMET
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1038/s41540-020-00162-6
    title: Sparsely-connected autoencoder (SCA) for single cell RNAseq data mining
  source_content_hash: sha256:22025fd51e407400b99a74ec84491efcd5ada448f54c0a84fa00d20cceaeb886
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/kendomaniac/rCASC
  tools:
  - COMET (cometsc)
  - Docker
  - R
  - SCA (Sparsely-Connected Autoencoder)
  - SIMLR
  - rCASC
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5454545454545454
n_claims: 11
---

# COMET-based transcription factor ranking per cluster

## Summary

Rank transcription factors for a specific cell cluster by running COMET (cometsc) on the SCA-encoded latent-space frequency table, yielding per-TF specificity statistics (COMETsc, TP, TN). This step assigns functional annotation (e.g., SOX5 for cluster 7 of a Visium breast cancer section) to clusters that the sparsely-connected autoencoder successfully reconstructed.

## When to use

Use when a cell cluster has already been reconstructed by TF-target-informed SCA (QCC and QCM > 0.5 for the majority of cells) and the goal is to identify transcription factors specific for that cluster from the latent space.

## When NOT to use

- The target cluster failed reconstruction — SCA reconstructs only specific clusters, so if QCC/QCM do not exceed 0.5 for the majority of cells, COMET ranking on that cluster's encoded space is not meaningful.
- The input is the raw counts table itself — COMET must be run on the SCA-encoded latent space, not directly on the raw scRNAseq/spatial counts matrix.
- No cluster structure has been defined yet — this skill assumes clusters were already defined by a clustering analysis (e.g., SIMLR) of the scRNAseq or spatial transcriptomics data.

## Inputs

- SCA latent space / encoded frequency table for the reconstructed cluster (from the rCASC SCA autoencoder with TF-target-informed sparsely-connected hidden layers)
- Cluster assignment of cells/spots (e.g., SIMLR annotation assigning Visium spots to 9 clusters)
- QCC and QCM quality-control metrics per cluster (threshold > 0.5 for the majority of cells)

## Outputs

- Ranked transcription factor table for the cluster, listing TF name, COMETsc statistic, and TP/TN fractions (e.g., SOX5, 2.88E-184, TP 0.623, TN 0.968)
- Cluster-level functional annotation identifying the top TF specific to that cluster

## How to apply

First confirm the target cluster is one of the clusters reconstructed by the SCA autoencoder, i.e., its QCC (Quality Control of Cluster) and QCM (Quality Control of Model) metrics exceed 0.5 for the majority of cells — SCA only meaningfully reconstructs specific clusters where the encoding space drives cell aggregation. Then run COMET (cometsc) on the SCA-encoded space, which combines experimentally validated TF-target, miRNA-target, kinase-target, and immune-signature knowledge in a unique latent space, building a frequency/contingency table of latent features for the cluster versus all other cells. COMET ranks TFs by specificity using the COMETsc statistic together with TP and TN fractions; e.g., for cluster 7 of the Visium breast cancer Block A Section 1 (9 SIMLR clusters), the top-ranked TF was SOX5 with COMETsc = 2.88E-184, TP = 0.623, TN = 0.968. Accept the annotation only if the TF appears as top-ranked in the COMET table with reported COMETsc/TP/TN values and the cluster alone passed the QCC/QCM > 0.5 threshold.

## Related tools

- **SCA (Sparsely-Connected Autoencoder)** (Produces the TF-target-informed encoded latent space on which COMET is run to rank transcription factors) — https://github.com/kendomaniac/rCASC
- **rCASC** (Framework hosting the SCA module (with QCC/QCM metrics) whose latent space is the input for COMET TF ranking) — https://github.com/kendomaniac/rCASC
- **COMET (cometsc)** (Runs the specificity test on the SCA latent-space frequency table to rank TFs by COMETsc/TP/TN)
- **SIMLR** (Provides the pre-existing cluster assignment (e.g., 9 clusters on the Visium breast cancer section) defining the cluster-vs-rest contingency used by COMET)

## Examples

```
library(devtools); install_github("kendomaniac/rCASC")
```

## Evaluation signals

- QCC and QCM exceed 0.5 for the majority of cells in the cluster being annotated — the result is accepted only for clusters reconstructed by SCA.
- The COMET table lists the expected TF as top-ranked; for the Visium breast cancer cluster 7, SOX5 must appear first with COMETsc = 2.88E-184, TP = 0.623, TN = 0.968.
- Only the reconstructed cluster(s) carry a TF annotation; clusters not passing QCC/QCM > 0.5 should have no valid COMET-based TF ranking.
- The COMET output contains per-TF TP and TN fractions in plausible ranges (0–1) alongside the COMETsc statistic, consistent with a cluster-vs-rest frequency table.

## Limitations

- SCA reconstructs only specific clusters, so only those clusters where the SCA encoding space is a key element for cell aggregation can receive a TF ranking.
- Transcription factors are lowly expressed genes and can be notably affected by dropout, weakening the signal in scRNAseq-derived latent spaces.
- miRNAs cannot be quantified together with mRNAs at single-cell level, so TF-miRNA circuits cannot be directly measured even though miRNA-target knowledge is embedded in the latent space.

## Evidence

- [results] COMET ranking is run on the SCA latent space for a reconstructed cluster (task card workflow): [evidence span withheld — anchor status: not_found]
- [abstract] QCC/QCM gating of clusters eligible for TF ranking: "We implemented two new metrics, QCC (Quality Control of Cluster) and QCM (Quality Control of Model), which allow quantifying the ability of SCA to reconstruct valuable cell clusters"
- [abstract] Latent space integrates validated TF/miRNA/kinase/immune-signature knowledge: "SCA encoded space, derived by different experimentally validated data (TF targets, miRNA targets, Kinase targets, and cancer-related immune signatures)"
- [discussion] SCA encoded space yields top-ranked TFs specific to clusters: "top ranked transcription factors, derived from SCA encoded space for cluster 1 and 2 of the blood-cell based dataset (setA)"
- [discussion] Dropout affects lowly expressed TFs: "Transcription factors and kinases are encoded by genes characterized by a relatively low expression in cells, thus they can be notably affected by dropout."
