---
name: silhouette-score-distance-matrix-validation
description: Use when a patient/specimen-level distance matrix (Wasserstein/EMD between
  cell-cluster distributions) has been computed and you need an unsupervised quality
  metric of that matrix, or when benchmarking distance methods with ARI/Silhouette/AUCPR/Spearman
  metrics.
when_to_use_negative:
- Input is raw per-cell expression or structure-level features rather than an already-computed
  sample-by-sample distance matrix — cluster distributions must be built first (e.g.,
  via Seurat for pathomics structures or Monocle 2 for PhEMD input)
- 'You need to correct for technical artifacts or batch effects: the paper notes PILOT
  ''does not explicitly correct for such potential technical artefacts'', and Silhouette
  on W will reflect any technology/isolation-method confounding rather than fix it'
- Comparing datasets with non-uniform tissue collection or mixed technologies (single-nucleus
  vs. single-cell, droplet vs. well-based), which the authors state is still an open
  challenge
edam_operation: http://edamontology.org/operation_3432
edam_topics:
- http://edamontology.org/topic_0091
tools:
- name: PILOT
  role: Produces the Wasserstein sample distance matrix W whose Silhouette score is
    evaluated; provides select_best_sil validation plotting
  repo: https://github.com/CostaLab/PILOT
- name: POT library
  role: emd2 function computes the optimal-transport Wasserstein distances underlying
    W
- name: PhEMD
  role: Competing EMD-based specimen distance method whose matrices are benchmarked
    with the same Silhouette metric
- name: SCANCell
  role: Competing method (PhEMD non-linear embeddings + association networks) benchmarked
    with Silhouette
- name: Multi-scale PHATE
  role: Competing multi-resolution embedding/clustering method benchmarked with Silhouette
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1038/s44320-023-00003-8
    title: Detection of PatIent-Level distances from single cell genomics and pathomics
      data with Optimal Transport (PILOT)
  source_content_hash: sha256:834613f556d187794faa0cc33b2f075fbf8c6a1f64d384ec85aaf66b3f3add9f
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/CostaLab/PILOT
  tools:
  - Multi-scale PHATE
  - PILOT
  - POT library
  - PhEMD
  - SCANCell
  - Seurat
  edam_operation: http://edamontology.org/operation_3432
  edam_topics:
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 9
---

# silhouette-score-distance-matrix-validation

## Summary

Compute the Silhouette score of a sample-level distance matrix (e.g., the Wasserstein distance matrix W produced by PILOT via POT's emd2) to validate that the patient-level distances reflect coherent sample groupings before downstream clustering and trajectory analysis, and to compare methods in the 12-dataset PILOT benchmark.

## When to use

Use when a patient/specimen-level distance matrix (Wasserstein/EMD between cell-cluster distributions) has been computed and you need an unsupervised quality metric of that matrix, or when benchmarking distance methods with ARI/Silhouette/AUCPR/Spearman metrics.

## When NOT to use

- Input is raw per-cell expression or structure-level features rather than an already-computed sample-by-sample distance matrix — cluster distributions must be built first (e.g., via Seurat for pathomics structures or Monocle 2 for PhEMD input)
- You need to correct for technical artifacts or batch effects: the paper notes PILOT 'does not explicitly correct for such potential technical artefacts', and Silhouette on W will reflect any technology/isolation-method confounding rather than fix it
- Comparing datasets with non-uniform tissue collection or mixed technologies (single-nucleus vs. single-cell, droplet vs. well-based), which the authors state is still an open challenge

## Inputs

- Sample-level distance matrix W (Wasserstein distances between cluster distributions, e.g., from POT emd2 on .h5ad AnnData inputs like Kidney_IgAN_T.h5ad)
- Per-sample disease/severity labels for interpretation and benchmark aggregation
- AnnData object passed to PILOT plotting/validation functions (e.g., select_best_sil)

## Outputs

- Silhouette score per method per dataset (numeric, higher is better)
- Rows for benchmark_metrics.csv feeding the Friedman–Nemenyi ranking (Appendix Tables S1–S2)
- PILOT select_best_sil diagnostic plot for choosing/validating cluster-resolution quality

## How to apply

After estimating the sample distance matrix W — in PILOT, Wasserstein distances between cluster distributions computed with POT's emd2, where the cost matrix is the cosine distance between cluster medians in PCA space — compute the Silhouette score directly on W (no Euclidean embedding) to quantify how well samples group by known disease labels without re-clustering. PILOT exposes pl.pl.select_best_sil to explore this quality score (with a starting parameter such as start = 0.2). In the benchmark workflow, compute Silhouette for each method (PILOT, PhEMD, SCANCell, Multi-scale PHATE) on each of the 12 datasets, pair it with the Adjusted Rand Index from Leiden clustering of the same matrix, then aggregate across datasets and apply the Friedman test with Nemenyi post-hoc correction to produce method rankings (p-value < 0.05 reported in Appendix Table S1). A good distance matrix yields a high Silhouette consistent with the disease labels and with the ARI ordering.

## Related tools

- **PILOT** (Produces the Wasserstein sample distance matrix W whose Silhouette score is evaluated; provides select_best_sil validation plotting) — https://github.com/CostaLab/PILOT
- **POT library** (emd2 function computes the optimal-transport Wasserstein distances underlying W)
- **PhEMD** (Competing EMD-based specimen distance method whose matrices are benchmarked with the same Silhouette metric)
- **SCANCell** (Competing method (PhEMD non-linear embeddings + association networks) benchmarked with Silhouette)
- **Multi-scale PHATE** (Competing multi-resolution embedding/clustering method benchmarked with Silhouette)

## Examples

```
pl.pl.select_best_sil(adata, start = 0.2)
```

## Evaluation signals

- Silhouette is computed directly on the sample distance matrix W for all four methods on all twelve datasets — any missing method/dataset cell invalidates the benchmark table
- Silhouette ordering should be consistent with the ARI obtained by Leiden clustering of the same W; the Friedman–Nemenyi test must reproduce the reported method ranking with p-value < 0.05 (Appendix Table S1)
- PILOT should attain the highest average ARI and a significantly superior Nemenyi rank versus PhEMD, pseudo-bulk, proportions+PHATE, and cell-proportion baselines (Fig. 2A–D)
- select_best_sil diagnostics should be run with a defined start (e.g., start = 0.2) and the chosen configuration should precede, not follow, trajectory estimation
- Pre-processing filters must be applied before scoring: e.g., low-quality samples excluded (<1000 genes on average per cell), trajectory outliers (Embryos E12–E15) removed, and kidney biopsies restricted with 10/2 main PCA components for glomeruli/tubules

## Limitations

- The EIPLGraph trajectory root sample was 'visually selected' rather than chosen by an objective rule, so Silhouette validates the distance matrix but not the trajectory orientation
- PILOT (and hence Silhouette on its W) assumes tissue samples were collected uniformly (similar tissue areas, same technology); analysis across distinct technologies or isolation methods is an open challenge
- Technical artifacts (e.g., in the lung and COVID-19 atlases) are not corrected, only statistically tested for influence, so Silhouette may be confounded by batch structure
- Software/package versions for pilotpy, scanpy, pandas, seaborn, matplotlib are not specified, limiting exact reproducibility of the scores

## Evidence

- [discussion] Clustering analysis can be performed by providing W as input to a Leiden clustering algorithm (Traag et al, 2019).: "Clustering analysis can be performed by providing W as input to a Leiden clustering algorithm (Traag et al, 2019)."
- [methods] select_best_sil: "pl.pl.select_best_sil(adata, start = 0.2)"
- [readme] PILOT Wasserstein distance definition: "PILOT uses optimal transport to compute the Wasserstein distance between two individual single-cell samples."
- [intro] Benchmark scope: "we perform a benchmarking study to compare PILOT and competing approaches on their performance in 12 public single-cell and pathomics data sets"
- [results] Aggregate metric and ranking table: [evidence span withheld — anchor status: not_found]
