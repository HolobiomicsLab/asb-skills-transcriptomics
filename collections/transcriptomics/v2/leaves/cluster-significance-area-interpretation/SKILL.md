---
name: cluster-significance-area-interpretation
description: Use when SCA/SSCA runs on scRNAseq clusters (e.g. setA clusters 1-5)
  have produced QCM/QCC values and you must decide which clusters are significantly
  reconstructed and compare latent-space combinations.
when_to_use_negative:
- Input is a raw counts table with no SCA/SSCA QCM/QCC output yet — first run the
  autoencoder analyses before interpreting the significance area
- The goal is de novo cluster discovery/clustering itself rather than evaluating reconstruction
  quality of pre-defined clusters
- The dataset lacks experimentally validated latent-space inputs (TF targets, miRNA
  targets, kinase targets, immune signatures), so SCA encoding spaces cannot be constructed
edam_operation: http://edamontology.org/operation_2945
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_0203
tools:
- name: SCA (Sparsely-Connected Autoencoder)
  role: Produces the QCM/QCC metrics and cluster reconstructions whose significance
    area is interpreted
  repo: https://github.com/kendomaniac/rCASC
- name: rCASC
  role: Framework implementing the SCA/SSCA modules (with GUI) used to generate per-cluster
    quality metrics
  repo: https://github.com/kendomaniac/rCASC
- name: Docker
  role: Encapsulates the reproducible computation environment in which rCASC SCA modules
    run
- name: R
  role: Environment in which rCASC is installed and QCM/QCC results are assembled
    into comparison tables
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1038/s41540-020-00162-6
    title: Sparsely-connected autoencoder (SCA) for single cell RNAseq data mining
  source_content_hash: sha256:91b7747b3037c89fcc8f49ac8cdfa832bffdda095d1586c81cf0b547a7cb8ffd
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/kendomaniac/rCASC
  tools:
  - Docker
  - R
  - SCA (Sparsely-Connected Autoencoder)
  - rCASC
  edam_operation: http://edamontology.org/operation_2945
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 12
---

# cluster-significance-area-interpretation

## Summary

Interprets SCA/SSCA cluster reconstruction outcomes by reading QCM (Quality Control of Model) and QCC (Quality Control of Cluster) values to identify which cell clusters fall in the 'significant area' (mean QCM and QCC ≥0.5), revealing where the encoded latent space (TF, miRNA, IS, kinase knowledge) is key to cell aggregation.

## When to use

Use when SCA/SSCA runs on scRNAseq clusters (e.g. setA clusters 1-5) have produced QCM/QCC values and you must decide which clusters are significantly reconstructed and compare latent-space combinations.

## When NOT to use

- Input is a raw counts table with no SCA/SSCA QCM/QCC output yet — first run the autoencoder analyses before interpreting the significance area
- The goal is de novo cluster discovery/clustering itself rather than evaluating reconstruction quality of pre-defined clusters
- The dataset lacks experimentally validated latent-space inputs (TF targets, miRNA targets, kinase targets, immune signatures), so SCA encoding spaces cannot be constructed

## Inputs

- Per-cluster QCM (Quality Control of Model) and QCC (Quality Control of Cluster) values from SCA/SSCA runs
- Cell-cluster assignments (clusters 1-5) from the setA single-cell counts table
- Latent-space configuration identifiers (TF-only, miRNA-only, IS-only, kinase-only, TF+miRNA, TF+miRNA+IS, TF+miRNA+IS+Kinase)

## Outputs

- Cluster-significance calls (clusters entering the significant area, mean QCM and QCC ≥0.5) per latent-space combination
- Comparison table of QCM/QCC across knowledge-group combinations (e.g. table1_ssca_qcm_qcc.csv)
- Per-cluster marker evidence annotations (e.g. miR-129-2-3P for HSC, POU2F2 negation for naïve T cell)

## How to apply

After running SCA or SSCA in the rCASC/Docker environment on a counts table with pre-computed cluster assignments (e.g. setA clusters 1-5: B cells, monocytes, HSC, naïve T cell), collect per-cluster QCM and QCC values. A cluster is called significant when its mean QCM and QCC reach ≥0.5, i.e. it enters the significant reconstruction area, indicating the SCA encoded space is a key element for that cluster's cell aggregation. Build baselines from single-knowledge-group latent spaces (TF-only, miRNA-only, IS-only, kinase-only), then evaluate combined latent spaces (TF+miRNA, TF+miRNA+IS, TF+miRNA+IS+Kinase) and compare: combined spaces should show higher QCM/QCC than each individual group. Interpret repositioning as evidence of knowledge integration: cluster 3 (HSC, marker miR-129-2-3P) becomes significant with TF+miRNA, clusters 4/5 with TF+miRNA+IS, and no cluster is significant with kinase alone. Assemble values and significance calls into a table (e.g. table1_ssca_qcm_qcc.csv) mirroring Table 1/Fig. 5 of the paper, tying significant clusters to marker evidence (e.g. miR-129-2-3P for HSC, POU2F2 negation for naïve T cells).

## Related tools

- **SCA (Sparsely-Connected Autoencoder)** (Produces the QCM/QCC metrics and cluster reconstructions whose significance area is interpreted) — https://github.com/kendomaniac/rCASC
- **rCASC** (Framework implementing the SCA/SSCA modules (with GUI) used to generate per-cluster quality metrics) — https://github.com/kendomaniac/rCASC
- **Docker** (Encapsulates the reproducible computation environment in which rCASC SCA modules run)
- **R** (Environment in which rCASC is installed and QCM/QCC results are assembled into comparison tables)

## Examples

```
library(devtools); install_github("kendomaniac/rCASC")  # then run SCA/SSCA modules in rCASC on setA and collect per-cluster QCM/QCC
```

## Evaluation signals

- Each cluster classified as significant has mean QCM and QCC ≥0.5
- Combined latent spaces (TF+miRNA, TF+miRNA+IS, TF+miRNA+IS+Kinase) show higher QCM/QCC than each single-knowledge-group baseline
- Cluster 3 (HSC) is significant under TF+miRNA but not under miRNA alone; clusters 4/5 are significant under TF+miRNA+IS
- No cluster reaches the significant area with kinase-only latent space
- Output table reproduces the cluster-significance pattern of Table 1 / Fig. 5 with marker evidence (e.g. miR-129-2-3P for cluster 3)

## Limitations

- SCA reconstructs only specific clusters — a non-significant cluster may simply mean the chosen encoded space is not key to its aggregation, not that the cluster is biologically invalid
- Interpretation depends on pre-computed cluster assignments (clusters 1-5); it does not perform clustering itself
- Transcription factors and kinases are lowly expressed and affected by dropout, which can degrade reconstruction metrics for their latent spaces

## Evidence

- [abstract] QCM and QCC quantify SCA's ability to reconstruct valuable cell clusters: "We implemented two new metrics, QCC (Quality Control of Cluster) and QCM (Quality Control of Model), which allow quantifying the ability of SCA to reconstruct valuable cell clusters"
- [abstract] SCA reconstructs only specific clusters, indicating where encoding space matters: "SCA efficacy comes from its ability to reconstruct only specific clusters, thus indicating only those clusters where the SCA encoding space is a key element for cells aggregation"
- [discussion] SSCA combines biological knowledge in a unique latent space: "combines different biological knowledge, i.e., TFs, miRNAs, IS, Kinases, in a unique latent space"
- [discussion] Cluster 5 signature refined by POU2F2 negation: "the SSCA signature for cluster 5 (naïve T cell) refined the definition of the cluster, since POU2F2 negation fits well with naïve T cell."
- [discussion] Metrics estimate robustness of the SCA encoding: "A very important element of our SCA implementation is the availability of metrics estimating the robustness of the SCA encoding."
- [discussion] Top ranked TFs from setA clusters 1 and 2: "top ranked transcription factors, derived from SCA encoded space for cluster 1 and 2 of the blood-cell based dataset (setA), which were shown to be key elements for B cells and monocytes."
