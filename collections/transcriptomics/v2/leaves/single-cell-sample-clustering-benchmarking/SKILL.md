---
name: single-cell-sample-clustering-benchmarking
description: Use when multiple sample-level single-cell/pathomics methods must be
  compared on patient clustering and trajectory accuracy across many datasets with
  uniform metrics and statistical ranking.
when_to_use_negative:
- The prior competing approaches did not perform any benchmarking, so no comparable
  published metric tables exist to reproduce for other methods outside these four
- Input data mix distinct technologies (single nucleus vs. single cell) or isolation
  methods (droplet vs. well based) — the article flags cross-technology analysis as
  an open challenge, not a supported benchmark setting
- You only have raw single-cell data and no per-sample disease labels — ARI/AUCPR/Spearman
  evaluation cannot be computed without ground-truth labels
edam_operation: http://edamontology.org/operation_2930
edam_topics:
- http://edamontology.org/topic_2229
- http://edamontology.org/topic_0091
tools:
- name: PILOT
  role: Method under test; computes Wasserstein distances between cluster distributions
    via POT's emd2
  repo: https://github.com/CostaLab/PILOT
- name: PhEMD
  role: Competing method using EMD-based specimen distances with Monocle 2 pre-processing
- name: SCANCell
  role: Competing method using PhEMD non-linear embeddings with association networks
- name: Multi-scale PHATE
  role: Competing method using multi-resolution clustering of non-linear embeddings
- name: POT library
  role: Provides the emd2 optimal-transport solver used to compute sample distances
- name: Seurat
  role: Uniform pre-processing pipeline for pathomics structure normalization and
    clustering
- name: Monocle 2
  role: Normalization, dimension reduction, and clustering of cells for the PhEMD
    input pipeline
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1038/s44320-023-00003-8
    title: Detection of PatIent-Level distances from single cell genomics and pathomics
      data with Optimal Transport (PILOT)
  source_content_hash: sha256:b0204f3303186a8623a3fc24ff535c88bb9fe9bdaf68413c9cb0659e188ab695
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/CostaLab/PILOT
  tools:
  - Monocle 2
  - Multi-scale PHATE
  - PILOT
  - POT library
  - PhEMD
  - SCANCell
  - Seurat
  edam_operation: http://edamontology.org/operation_2930
  edam_topics:
  - http://edamontology.org/topic_2229
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.625
n_claims: 8
---

# single-cell-sample-clustering-benchmarking

## Summary

Benchmark patient-level sample-clustering and trajectory methods (PILOT, PhEMD, SCANCell, Multi-scale PHATE) across 12 public single-cell and pathomics datasets using ARI, Silhouette, AUCPR, and Spearman metrics with Friedman–Nemenyi ranking.

## When to use

Use when multiple sample-level single-cell/pathomics methods must be compared on patient clustering and trajectory accuracy across many datasets with uniform metrics and statistical ranking.

## When NOT to use

- The prior competing approaches did not perform any benchmarking, so no comparable published metric tables exist to reproduce for other methods outside these four
- Input data mix distinct technologies (single nucleus vs. single cell) or isolation methods (droplet vs. well based) — the article flags cross-technology analysis as an open challenge, not a supported benchmark setting
- You only have raw single-cell data and no per-sample disease labels — ARI/AUCPR/Spearman evaluation cannot be computed without ground-truth labels

## Inputs

- Twelve public single-cell and pathomics datasets (GEO, GSA, EGA, Zenodo, .h5ad files such as Kidney_IgAN_T.h5ad)
- Pre-processed AnnData objects with per-sample cluster distributions and disease labels
- Pathomics structure datasets from github.com/CostaLab/PILOT/Tutorial/Datasets
- Disease labels / severity scores per sample for ARI, AUCPR, and Spearman computation

## Outputs

- benchmark_metrics.csv with ARI, Silhouette, AUCPR, and Spearman for all four methods on all twelve datasets
- Friedman–Nemenyi method ranking table (reproducing Appendix Tables S1–S2)
- Four benchmark panels corresponding to Fig. 2A–D

## How to apply

Download the twelve public datasets (GSE174188, GSE158055, Zenodo 7227571, GSE211799, EGAS00001006052, Zenodo 7435911, GSE169285, CRA001160, and kidney pathomics sets) and apply a uniform pre-processing pipeline: Seurat for pathomics structure normalization/clustering and Monocle 2 for the PhEMD input pipeline, with dataset-specific filters (e.g. kidney biopsies only, 10 and 2 main PCA components for glomeruli and tubules, excluding four pancreas E12–E15 outlier samples). Run all four methods per dataset — PILOT via POT's emd2 Wasserstein distances between cluster distributions, PhEMD EMD specimen distances, SCANCell, and Multi-scale PHATE. For clustering, cluster samples from each distance matrix with Leiden and compute the Adjusted Rand Index against disease labels plus the Silhouette score of the distance matrix; for trajectories, estimate diffusion-map/EIPLGraph backbones and compute AUCPR of disease-status prediction and Spearman correlation with severity. Aggregate all metrics and apply the Friedman test with Nemenyi post-hoc correction to produce method rankings, writing the full table to benchmark_metrics.csv.

## Related tools

- **PILOT** (Method under test; computes Wasserstein distances between cluster distributions via POT's emd2) — https://github.com/CostaLab/PILOT
- **PhEMD** (Competing method using EMD-based specimen distances with Monocle 2 pre-processing)
- **SCANCell** (Competing method using PhEMD non-linear embeddings with association networks)
- **Multi-scale PHATE** (Competing method using multi-resolution clustering of non-linear embeddings)
- **POT library** (Provides the emd2 optimal-transport solver used to compute sample distances)
- **Seurat** (Uniform pre-processing pipeline for pathomics structure normalization and clustering)
- **Monocle 2** (Normalization, dimension reduction, and clustering of cells for the PhEMD input pipeline)

## Examples

```
pl.pl.select_best_sil(adata, start = 0.2)
```

## Evaluation signals

- Friedman–Nemenyi test must yield PILOT the highest average ARI with rankings significantly superior to all competitors (p-value < 0.05, Appendix Table S1)
- Each metric (ARI, Silhouette, AUCPR, Spearman) must be computed for all four methods on all twelve datasets — no missing cells in benchmark_metrics.csv
- Resulting panels must reproduce the ordering reported in Fig. 2A–D and Appendix Tables S1–S2
- Dataset-specific filters must be respected (e.g. only kidney-biopsy samples; 10 and 2 PCA components for glomeruli and tubules)
- Trajectory AUCPR and Spearman values must derive from diffusion maps followed by EIPLGraph backbone inference per method

## Limitations

- Technical artifacts are not explicitly corrected; PILOT only provides statistical tests to evaluate whether technical factors influence predictions
- The EIPLGraph trajectory root sample was visually selected rather than chosen by an objective rule — a reproducibility gap
- Four pancreas samples (Embryos E12–E15) were excluded as trajectory outliers; benchmark results depend on such exclusions

## Evidence

- [results] PILOT obtained the highest average ARI score and its ranking was significantly superior to all competing methods: [evidence span withheld — anchor status: not_found]
- [intro] we perform a benchmarking study to compare PILOT and competing approaches on their performance in 12 public single-cell and pathomics data sets: "we perform a benchmarking study to compare PILOT and competing approaches on their performance in 12 public single-cell and pathomics data sets"
- [discussion] PILOT is based on the emd2 function from POT library (Flamary et al, 2021): "PILOT is based on the emd2 function from POT library (Flamary et al, 2021), which implements the best solver described in (Bonneel et al, 2011)."
- [discussion] We employed a uniform pre-processing pipeline utilizing Seurat (Hao et al, 2021) for the normalization and clustering of structures.: "We employed a uniform pre-processing pipeline utilizing Seurat (Hao et al, 2021) for the normalization and clustering of structures."
- [discussion] Clustering analysis can be performed by providing W as input to a Leiden clustering algorithm (Traag et al, 2019).: "Clustering analysis can be performed by providing W as input to a Leiden clustering algorithm (Traag et al, 2019)."
- [readme] There are five tutorials, one for Myocardial Infarction (single cell data): [evidence span withheld — anchor status: external_doc]
