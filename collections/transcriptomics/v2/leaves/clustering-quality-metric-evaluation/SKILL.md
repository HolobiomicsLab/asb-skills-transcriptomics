---
name: clustering-quality-metric-evaluation
description: Use when you have run SCA on a single-cell counts matrix (e.g., setA
  PBMC clusters 1-5) and need to judge which clusters the autoencoder encoding reliably
  reconstructs, especially clusters whose QCM/QCC scores sit near the 0.5 significance
  threshold.
when_to_use_negative:
- Input is not a scRNAseq counts table with pre-defined cluster assignments — QCC/QCM
  only score SCA reconstruction of already-defined clusters, not an initial clustering
  method
- The goal is normalization of the counts themselves (e.g., SCnorm checkCountDepth);
  normalization is only an upstream perturbation whose effect is read out through
  QCM/QCC
- Evaluating clustering of bulk RNAseq or other omics data where the SCA encoded-space
  rationale (TF/miRNA/kinase target hidden layers) does not apply
edam_operation: http://edamontology.org/operation_3432
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: SCA (Sparsely-Connected Autoencoder)
  role: Produces the encoded-space cluster reconstruction that QCM/QCM metrics score
  repo: https://github.com/kendomaniac/rCASC
- name: rCASC
  role: Framework hosting the SCA module (autoencoder/autoencoderAnalysis functions)
    with GUI access; computes QCM and QCC
  repo: https://github.com/kendomaniac/rCASC
- name: Docker
  role: Runs the rCASC SCA containers (docker.io/repbioinfo) to guarantee computational
    reproducibility of the metric computation
  repo: https://github.com/kendomaniac/rCASC
- name: R
  role: Environment in which rCASC and the normalization functions (counts2log etc.)
    are installed and invoked
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1038/s41540-020-00162-6
    title: Sparsely-connected autoencoder (SCA) for single cell RNAseq data mining
  source_content_hash: sha256:82ba502e158a22a902dd61b27b1ea33bbb68d89f03d6b93623c08c7f950df2c8
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/kendomaniac/rCASC
  tools:
  - Docker
  - R
  - SCA (Sparsely-Connected Autoencoder)
  - rCASC
  edam_operation: http://edamontology.org/operation_3432
  edam_topics:
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6
n_claims: 10
---

# clustering-quality-metric-evaluation

## Summary

Evaluates the robustness of SCA-derived single-cell cluster reconstructions using the paper's two purpose-built metrics, QCM (Quality Control of Model) and QCC (Quality Control of Cluster), which quantify whether the Sparsely-Connected Autoencoder reconstructed biologically valuable cell clusters from a scRNAseq counts table.

## When to use

Use when you have run SCA on a single-cell counts matrix (e.g., setA PBMC clusters 1-5) and need to judge which clusters the autoencoder encoding reliably reconstructs, especially clusters whose QCM/QCC scores sit near the 0.5 significance threshold.

## When NOT to use

- Input is not a scRNAseq counts table with pre-defined cluster assignments — QCC/QCM only score SCA reconstruction of already-defined clusters, not an initial clustering method
- The goal is normalization of the counts themselves (e.g., SCnorm checkCountDepth); normalization is only an upstream perturbation whose effect is read out through QCM/QCC
- Evaluating clustering of bulk RNAseq or other omics data where the SCA encoded-space rationale (TF/miRNA/kinase target hidden layers) does not apply

## Inputs

- Single-cell UMI counts matrix (e.g., setA PBMC-derived dataset) with cluster assignments
- Hidden-layer gene-set file built from experimentally validated miRNA targets (miRTarBase)
- Optionally, normalized counts-table variants (log10 + CLR/RLE/FQ/SUM/TMM)

## Outputs

- Per-cluster QCM (Quality Control of Model) scores
- Per-cluster QCC (Quality Control of Cluster) scores
- Comparison table of QCM/QCC profiles across normalization variants (raw vs. six normalized inputs)

## How to apply

Run the SCA module of rCASC (autoencoder/autoencoderAnalysis functions executed in Docker containers from docker.io/repbioinfo) on the counts matrix with a hidden layer built from experimentally validated miRNA-target (miRTarBase) gene sets, then compute QCM and QCC per cluster. QCM assesses neural-network reconstruction quality while QCC quantifies the ability of SCA to reconstruct valuable cell clusters; clusters scoring near the 0.5 threshold (e.g., clusters 2 = Monocytes and 3 = HSC in the setA dataset) are the ones whose membership is sensitive to upstream perturbations. If testing robustness, repeat the SCA run on normalized variants of the input counts (log10 plus CLR, RLE, FQ, SUM, TMM — with TMM and RLE suited to UMI data) and compare the QCM/QCC profiles across normalizations. Validation is done by confirming that near-threshold clusters shift across normalizations while low-quality clusters (1, 4, 5) remain minimally affected, and that top-ranked cluster-specific miRNAs (e.g., miR-191 and miR-132-3p under TMM; miR-187-3p under FQ and SUM) are consistent.

## Related tools

- **SCA (Sparsely-Connected Autoencoder)** (Produces the encoded-space cluster reconstruction that QCM/QCM metrics score) — https://github.com/kendomaniac/rCASC
- **rCASC** (Framework hosting the SCA module (autoencoder/autoencoderAnalysis functions) with GUI access; computes QCM and QCC) — https://github.com/kendomaniac/rCASC
- **Docker** (Runs the rCASC SCA containers (docker.io/repbioinfo) to guarantee computational reproducibility of the metric computation) — https://github.com/kendomaniac/rCASC
- **R** (Environment in which rCASC and the normalization functions (counts2log etc.) are installed and invoked)

## Examples

```
install.packages("devtools"); library(devtools); install_github("kendomaniac/rCASC")
```

## Evaluation signals

- QCM and QCC scores are reported for every cluster (e.g., clusters 1-5 of setA), not only for high-scoring clusters
- Clusters whose QCM/QCC lie near the 0.5 threshold (clusters 2 and 3 in setA) show score variation across the six normalizations, while low-quality clusters (1, 4, 5) remain minimally affected
- The QCC/QCM profile correctly identifies the specific clusters for which the SCA encoding space is a key element of cell aggregation
- Top-ranked cluster-specific miRNAs from the SCA encoded space match the published marker set (miR-191, miR-132-3p under TMM; miR-187-3p under FQ and SUM)
- SCA runs are executed in pinned Docker containers (docker.io/repbioinfo) so metric computation is reproducible

## Limitations

- scRNAseq is very noisy and dropout affects poorly expressed genes; transcription factors and kinases are lowly expressed and can be notably affected by dropout
- It is currently not possible to quantify miRNAs together with mRNAs at single-cell level, so TF-miRNA circuits cannot be directly measured from the encoded space
- Normalization choice matters only for clusters near the 0.5 threshold; conclusions about near-threshold clusters are not transferable to clearly accepted/rejected clusters
- SCA usage could be particularly challenging for life science and medical personnel lacking strong computation skills (mitigated by the rCASC GUI)

## Evidence

- [abstract] We implemented two new metrics, QCC (Quality Control of Cluster) and QCM (Quality Control of Model), which allow quantifying the ability of SCA to reconstruct valuable cell clusters: "We implemented two new metrics, QCC (Quality Control of Cluster) and QCM (Quality Control of Model), which allow quantifying the ability of SCA to reconstruct valuable cell clusters"
- [discussion] A very important element of our SCA implementation is the availability of metrics estimating the robustness of the SCA encoding.: "A very important element of our SCA implementation is the availability of metrics estimating the robustness of the SCA encoding."
- [abstract] SCA efficacy comes from its ability to reconstruct only specific clusters, thus indicating only those clusters where the SCA encoding space is a key element for cells aggregation: "SCA efficacy comes from its ability to reconstruct only specific clusters, thus indicating only those clusters where the SCA encoding space is a key element for cells aggregation"
- [readme] SCA analysis was added as extention of rCASC.: [evidence span withheld — anchor status: not_found]
