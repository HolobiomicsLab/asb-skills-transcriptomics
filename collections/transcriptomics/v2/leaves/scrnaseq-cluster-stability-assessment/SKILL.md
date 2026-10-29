---
name: scrnaseq-cluster-stability-assessment
description: Use when scRNAseq clusters must be validated for reconstruction robustness
  — e.g. after clustering with an SCA-encoded space (TF/miRNA/kinase targets) on datasets
  like setA, or when comparing a variant latent space (e.g. vSCA) against a baseline
  SCA run via QCM/QCC trajectory superimposability.
when_to_use_negative:
- Input is raw FASTQ or an unfiltered counts matrix — apply upstream rCASC preprocessing/filtering
  (filterZeros, lorenzFilter, SCnorm checkCountDepth) before stability assessment
- The question is de novo cluster discovery rather than validation of existing clusters
  — use clustering analysis itself, not QCM/QCC assessment
- The clustering method is not SCA-based and provides no encoded latent space, since
  QCM evaluates the autoencoder model and QCC the SCA-specific cluster reconstruction
edam_operation: http://edamontology.org/operation_3891
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_2269
tools:
- name: SCA
  role: Sparsely-connected autoencoder whose encoded space (TF targets etc.) is used
    for clustering; its QCM/QCC metrics are the stability outputs being assessed
  repo: https://github.com/kendomaniac/SCAtutorial
- name: rCASC
  role: Framework wrapping SCA (autoencoderAnalysis4Clustering, permutationClustering)
    and executed through Docker containers for reproducible computation
  repo: https://github.com/kendomaniac/rCASC
- name: Docker
  role: Daemon hosting the repbioinfo containers in which the rCASC SCA stability
    workflow runs
  repo: https://github.com/kendomaniac/rCASC
- name: devtools
  role: Used to install rCASC into the R environment (install_github("kendomaniac/rCASC"))
  repo: https://github.com/kendomaniac/rCASC
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1038/s41540-020-00162-6
    title: Sparsely-connected autoencoder (SCA) for single cell RNAseq data mining
  source_content_hash: sha256:d12a58382863ba607ca45f9703ba5cb04f3d40f4531835b83c0aa909bb93122d
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/kendomaniac/SCAtutorial
  tools:
  - Docker
  - R
  - SCA
  - devtools
  - rCASC
  edam_operation: http://edamontology.org/operation_3891
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_2269
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6923076923076923
n_claims: 13
---

# scrnaseq-cluster-stability-assessment

## Summary

Assess the robustness and functional quality of single-cell RNAseq clusters using the SCA/rCASC quality metrics QCM (Quality Control of Model) and QCC (Quality Control of Cluster), which quantify whether a sparsely-connected autoencoder latent space reliably reconstructs valuable cell clusters.

## When to use

Use when scRNAseq clusters must be validated for reconstruction robustness — e.g. after clustering with an SCA-encoded space (TF/miRNA/kinase targets) on datasets like setA, or when comparing a variant latent space (e.g. vSCA) against a baseline SCA run via QCM/QCC trajectory superimposability.

## When NOT to use

- Input is raw FASTQ or an unfiltered counts matrix — apply upstream rCASC preprocessing/filtering (filterZeros, lorenzFilter, SCnorm checkCountDepth) before stability assessment
- The question is de novo cluster discovery rather than validation of existing clusters — use clustering analysis itself, not QCM/QCC assessment
- The clustering method is not SCA-based and provides no encoded latent space, since QCM evaluates the autoencoder model and QCC the SCA-specific cluster reconstruction

## Inputs

- Filtered single-cell UMI counts table (e.g. setA PBMC-derived dataset)
- Cluster annotations for the subpopulations of interest (e.g. cluster 1/2 of setA)
- Prior knowledge encoding matrix (TF targets, miRNA targets, kinase targets, or immune signatures) defining the SCA latent space
- Baseline SCA QCM/QCC trajectories for comparison runs (e.g. plain TF-based SCA vs vSCA)

## Outputs

- QCM (Quality Control of Model) trajectory per permutation/run
- QCC (Quality Control of Cluster) trajectory per cluster
- Overlay plots of QCM/QCC curves across SCA variants
- COMET-ranked transcription factor markers per cluster (e.g. CHD4, CEBPA) for functional cross-validation

## How to apply

Run the SCA (or variant) encoding within the rCASC framework via Docker containers (docker.io/repbioinfo) and compute QCM and QCC trajectories with the rCASC wrapper autoencoderAnalysis4Clustering; the SCA implementation embeds permutation-based stability assessment (autoencoder4clustering/permutationClustering) so clustering robustness is estimated over permutations rather than a single run. QCM evaluates the neural network (autoencoder model) quality while QCC quantifies the ability to reconstruct valuable cell clusters. To benchmark a modified encoding (e.g. the variational vSCA with TF-target latent space), overlay its QCM/QCC curves on the baseline SCA run: near-superimposable trajectories indicate no specific improvement. Reinforce the metric-based verdict with functional validation — rank cluster markers with COMET (cometsc/cometsc2) and check that top transcription factors match known signatures (e.g. CHD4 for B-cell cluster 1 and CEBPA for monocyte cluster 2 of setA, as in Table 1).

## Related tools

- **SCA** (Sparsely-connected autoencoder whose encoded space (TF targets etc.) is used for clustering; its QCM/QCC metrics are the stability outputs being assessed) — https://github.com/kendomaniac/SCAtutorial
- **rCASC** (Framework wrapping SCA (autoencoderAnalysis4Clustering, permutationClustering) and executed through Docker containers for reproducible computation) — https://github.com/kendomaniac/rCASC
- **Docker** (Daemon hosting the repbioinfo containers in which the rCASC SCA stability workflow runs) — https://github.com/kendomaniac/rCASC
- **devtools** (Used to install rCASC into the R environment (install_github("kendomaniac/rCASC"))) — https://github.com/kendomaniac/rCASC

## Examples

```
install.packages("devtools"); library(devtools); install_github("kendomaniac/rCASC"); library(rCASC)
```

## Evaluation signals

- QCM and QCC trajectories are produced for the permutation-based runs, not just a single clustering
- When benchmarking a variant (e.g. vSCA vs plain TF-based SCA), curves are judged 'nearly superimposable' to declare no improvement
- Top COMET markers for assessed clusters match expected functional TFs (setA: CHD4 for cluster 1, CEBPA for cluster 2, consistent with Table 1)
- The 4-gene SCA signatures include the top-ranked COMET transcription factors, linking metric outcome to signature content
- The run reproduces within the Dockerized rCASC environment satisfying computation reproducibility (Sandve rules)

## Limitations

- A variant encoding can be metric-equivalent yet more complex: vSCA's added variational complexity gave no specific improvement over plain TF-based SCA on setA
- SCA reconstructs only specific clusters — QCC is informative only for clusters where the SCA encoding space is a key element of cell aggregation
- Transcription factors and kinases are lowly expressed and notably affected by dropout, which can perturb TF-driven latent spaces
- scRNAseq is very noisy and produces more zeros than expected, with greater bias for poorly expressed genes

## Evidence

- [abstract] QCC/QCM quantify reconstruction quality: "We implemented two new metrics, QCC (Quality Control of Cluster) and QCM (Quality Control of Model), which allow quantifying the ability of SCA to reconstruct valuable cell clusters"
- [abstract] SCA reconstructs only specific clusters: "SCA efficacy comes from its ability to reconstruct only specific clusters, thus indicating only those clusters where the SCA encoding space is a key element for cells aggregation"
- [discussion] SCA implementation provides robustness metrics: "A very important element of our SCA implementation is the availability of metrics estimating the robustness of the SCA encoding."
- [methods] Docker requirement for the workflow: "Docker daemon installed on the machine, for more info see this document"
- [discussion] setA top-ranked TFs are functionally key: "top ranked transcription factors, derived from SCA encoded space for cluster 1 and 2 of the blood-cell based dataset (setA), which were shown to be key elements for B cells and monocytes."
- [readme] rCASC installation via devtools from GitHub: [evidence span withheld — anchor status: external_doc]
