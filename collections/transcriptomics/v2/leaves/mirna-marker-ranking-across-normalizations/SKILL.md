---
name: mirna-marker-ranking-across-normalizations
description: Use when scRNAseq clusters must be functionally annotated with miRNA
  markers via a miRNA-target autoencoder latent space, and you need to know whether
  normalization of the input counts table changes which markers/scores emerge.
when_to_use_negative:
- You only need a single SCA run on one already-normalized matrix and do not intend
  to compare normalization effects or rank markers across runs
- The goal is counting/alignment from FASTQ (use cellrangerCount/indropCounts) rather
  than normalization sensitivity analysis of an existing counts table
- miRNA-target annotation is not the goal — TF or kinase-target hidden layers serve
  a different functional query in the SCA framework
edam_operation: http://edamontology.org/operation_3435
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_3518
tools:
- name: SCA (Sparsely-Connected Autoencoder)
  role: Autoencoder whose miRTarBase-derived hidden layer produces the encoded space
    used to rank cluster-specific miRNA markers
  repo: https://github.com/kendomaniac/rCASC
- name: rCASC
  role: Framework hosting the SCA module (autoencoder/autoencoderAnalysis functions)
    and counts2log for log10 conversion of the counts table
  repo: https://github.com/kendomaniac/rCASC
- name: Docker
  role: Runs the rCASC SCA containers from docker.io/repbioinfo to guarantee computational
    reproducibility
- name: R
  role: Environment in which rCASC is installed via devtools and the normalization/SCA
    workflow is executed
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1038/s41540-020-00162-6
    title: Sparsely-connected autoencoder (SCA) for single cell RNAseq data mining
  source_content_hash: sha256:2dcaf122fb9d73c9870d9630f8f07fa4f720b6b8876fb1422128cd9b2f8c65f5
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/kendomaniac/rCASC
  tools:
  - Docker
  - R
  - SCA (Sparsely-Connected Autoencoder)
  - rCASC
  edam_operation: http://edamontology.org/operation_3435
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_3518
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 12
---

# miRNA marker ranking across normalizations

## Summary

Ranks cluster-specific miRNA markers from the SCA encoded space (hidden layer built from miRTarBase experimentally validated miRNA-target gene sets) and tests how input counts-table normalization (log10 + CLR/RLE/FQ/SUM/TMM) shifts QCM/QCC scores, mainly for clusters near the 0.5 significance threshold.

## When to use

Use when scRNAseq clusters must be functionally annotated with miRNA markers via a miRNA-target autoencoder latent space, and you need to know whether normalization of the input counts table changes which markers/scores emerge.

## When NOT to use

- You only need a single SCA run on one already-normalized matrix and do not intend to compare normalization effects or rank markers across runs
- The goal is counting/alignment from FASTQ (use cellrangerCount/indropCounts) rather than normalization sensitivity analysis of an existing counts table
- miRNA-target annotation is not the goal — TF or kinase-target hidden layers serve a different functional query in the SCA framework

## Inputs

- Single-cell UMI counts matrix with cluster assignments (e.g., setA PBMC-derived dataset, clusters 1-5)
- miRTarBase experimentally validated miRNA-target gene sets for the SCA hidden layer
- rCASC installation with SCA module and Docker daemon (docker.io/repbioinfo containers)

## Outputs

- Per-cluster QCM and QCC scores for each of the six normalized counts versions (log10, CLR, RLE, FQ, SUM, TMM)
- Ranked cluster-specific miRNA markers from the SCA encoded space (e.g., miR-191 and miR-132-3p under TMM; miR-187-3p under FQ and SUM)
- Identification of normalization-sensitive clusters (those near the 0.5 QCM/QCC threshold, e.g., clusters 2 and 3)

## How to apply

Load the single-cell counts matrix with cluster assignments (e.g., setA PBMC clusters 1-5, with cluster 2 = Monocytes and cluster 3 = HSC near the 0.5 threshold). Generate six versions of the input counts: a log10 conversion (rCASC counts2log) plus CLR, RLE, FQ, SUM and TMM global-scaling normalizations, noting TMM and RLE are the variants suited to UMI data. For each normalized matrix run the Sparsely-Connected Autoencoder with a hidden layer built from experimentally validated miRNA-target (miRTarBase) gene sets, via the SCA module of rCASC executed in Docker containers. Compute QCM (Quality Control of Model) and QCC (Quality Control of Cluster) per cluster per normalization, then rank cluster-specific miRNA markers from the SCA encoded space. The rationale: QCM/QCC quantify the ability of SCA to reconstruct valuable cell clusters, and normalization is expected to act mainly on clusters whose metrics lie near the 0.5 threshold, while low-quality clusters are minimally affected.

## Related tools

- **SCA (Sparsely-Connected Autoencoder)** (Autoencoder whose miRTarBase-derived hidden layer produces the encoded space used to rank cluster-specific miRNA markers) — https://github.com/kendomaniac/rCASC
- **rCASC** (Framework hosting the SCA module (autoencoder/autoencoderAnalysis functions) and counts2log for log10 conversion of the counts table) — https://github.com/kendomaniac/rCASC
- **Docker** (Runs the rCASC SCA containers from docker.io/repbioinfo to guarantee computational reproducibility)
- **R** (Environment in which rCASC is installed via devtools and the normalization/SCA workflow is executed)

## Examples

```
install_github("kendomaniac/rCASC"); log10_counts <- counts2log(input_counts); autoencoderAnalysis(cluster="2", scratch=paste0(tmpdir(), "/"), normalization="TMM", seed=1224)
```

## Evaluation signals

- QCM/QCC values for clusters 2 and 3 change across the six normalizations and lie near the 0.5 threshold
- QCM/QCC for low-quality clusters (1, 4, 5) remain minimally affected across normalizations
- Top-ranked miRNA markers under TMM are miR-191 and miR-132-3p, matching Fig. 4 and Table 1
- Top-ranked miRNA markers under FQ and SUM are miR-187-3p, matching Fig. 4 and Table 1
- Each SCA run reports its QCM/QCC metrics, confirming the encoding-robustness metrics were produced per cluster

## Limitations

- Normalization sensitivity is concentrated on near-threshold clusters; low-quality clusters show minimal QCM/QCC change, so the comparison is uninformative for those
- SCA usage can be challenging for life-science personnel lacking strong computational skills, hence the Docker/GUI wrapper
- scRNAseq is very noisy and produces more zeros than expected (dropouts), a bias greater for poorly expressed genes such as transcription factors
- It is currently not possible to quantify miRNAs together with mRNAs at single-cell level, so TF-miRNA circuits cannot be directly measured

## Evidence

- [abstract] QCM/QCC metrics definition: "We implemented two new metrics, QCC (Quality Control of Cluster) and QCM (Quality Control of Model), which allow quantifying the ability of SCA to reconstruct valuable cell clusters"
- [abstract] SCA encoded space from validated data: "SCA encoded space, derived by different experimentally validated data (TF targets, miRNA targets, Kinase targets, and cancer-related immune signatures), can be used to grasp single cell"
- [methods] rCASC installation in R: "The following commands allow the rCASC installation in [*R*](https://cran.r-project.org/) environment"
- [methods] counts2log filter step: "Converting a count table in log10: **counts2log**"
- [methods] Docker requirement for reproducibility: "Docker daemon installed on the machine, for more info see this document"
- [discussion] setA dataset clusters: "top ranked transcription factors, derived from SCA encoded space for cluster 1 and 2 of the blood-cell based dataset (setA), which were shown to be key elements for B cells and monocytes."
- [readme] SCAtutorial reproducibility setup: [evidence span withheld — anchor status: not_found]
