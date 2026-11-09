---
name: cross-method-rare-cell-detection-comparison
description: Use when comparing rare cell-type detection accuracy across methods on
  scRNA-seq data, e.g. testing sensitivity to the number of DE genes distinguishing
  a rare population, or reproducing reported rare clusters in the 68k PBMC dataset.
when_to_use_negative:
- Major cell types only need to be characterized — standard toolkits such as Seurat
  or Scater suffice and rare-cell detectors are unnecessary.
- The dataset is too small for GiniClust-scale comparisons or the rare population
  is not expected to be a minor cluster separable by an inter-cluster gap.
- You need to discriminate doublets rather than genuine minor cell types — the paper
  addresses doublet identification in a separate workflow (293T-Jurkat and 68k PBMC
  subsampled datasets).
edam_operation: http://edamontology.org/operation_2939
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_2229
tools:
- name: GapClust
  role: Primary rare-cell detector; exploits the gap between a minor cluster and its
    neighbouring abundant cluster via a second-order derivative type distance score;
    candidates identified when skewness_k > 2
  repo: github.com/fabotao/GapClust
- name: RaceID
  role: Comparison baseline for rare-cell detection AUC in the replacement-experiment
    benchmark
- name: GiniClust
  role: Comparison baseline; cross-checked to confirm whether rare clusters (e.g.
    3-cell R1) are missed
- name: FiRE
  role: Comparison baseline; cross-checked to confirm whether rare clusters (e.g.
    3-cell R1) are missed
- name: Seurat
  role: scRNA-seq preprocessing/subsetting toolkit for building the benchmark subsets
    and annotating detected rare clusters
provenance:
  source_task_ids:
  - task_003
  - task_006
  source_papers:
  - doi: 10.1038/s41467-021-24489-8
    title: GapClust is a light-weight approach distinguishing rare cells from voluminous
      single cell expression profiles
  source_content_hash: sha256:70584edaa6894c23162544022c0398808ab23c52c0f838a4f6a02422594cd64c
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - FiRE
  - GapClust
  - GiniClust
  - RaceID
  - Seurat
  edam_operation: http://edamontology.org/operation_2939
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_2229
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 11
---

# cross-method-rare-cell-detection-comparison

## Summary

Benchmark GapClust against established rare-cell detectors (RaceID, GiniClust, FiRE) on scRNA-seq data using replacement-experiment sensitivity assays and full-dataset discovery runs, quantified by AUC/F1 of rare-cell recovery.

## When to use

Use when comparing rare cell-type detection accuracy across methods on scRNA-seq data, e.g. testing sensitivity to the number of DE genes distinguishing a rare population, or reproducing reported rare clusters in the 68k PBMC dataset.

## When NOT to use

- Major cell types only need to be characterized — standard toolkits such as Seurat or Scater suffice and rare-cell detectors are unnecessary.
- The dataset is too small for GiniClust-scale comparisons or the rare population is not expected to be a minor cluster separable by an inter-cluster gap.
- You need to discriminate doublets rather than genuine minor cell types — the paper addresses doublet identification in a separate workflow (293T-Jurkat and 68k PBMC subsampled datasets).

## Inputs

- 68k PBMC scRNA-seq expression matrix (raw UMI counts)
- Cell-type annotations or marker-defined subsets (CD19+ B cells, CD14+ monocytes)
- Installed rare-cell detection tools: GapClust, RaceID, GiniClust, FiRE
- DE gene lists (1–144 genes) for replacement-experiment synthetic datasets

## Outputs

- Per-replicate and average rare-cell detection AUC per method as a function of DE gene count (Fig. 3b reproduction)
- Rare cluster assignments, cluster sizes, and marker-gene profiles for the full dataset (e.g. rare_cells_68k_pbmc.csv)
- GapClust-specific vs shared rare clusters across methods (R1: 3 cells; R2: 185 cells)

## How to apply

First, reproduce the replacement-experiment benchmark: subset 500 CD19+ B cells from the 68k PBMC matrix with 2, 5, or 10 CD14+ monocytes, then generate 100 replicates per monocyte count in which 1–144 differentially expressed genes are replaced per the paper's replacement design; run GapClust, RaceID, GiniClust, and FiRE on each replicate and average rare-cell detection AUCs across replicates. Second, run the full-dataset discovery comparison: apply GapClust (github.com/fabotao/GapClust) to the whole 68k PBMC expression matrix with the default candidate-selection criterion skewness_k > 2, then cross-check the top-ranked minor clusters against GiniClust and FiRE outputs. Decision point: a cluster counted as GapClust-specific (e.g. the 3-cell CD14+ monocyte cluster R1) only when competitors miss it. Validation uses reported anchors: GapClust AUC ≈ 0.89 at 50 DE genes with 2 monocytes vs RaceID ≈ 0.63 and GiniClust/FiRE ≈ 0.5, plus cluster sizes (3 and 185 cells) and marker signatures (CD14+, CD34+/megakaryocyte).

## Related tools

- **GapClust** (Primary rare-cell detector; exploits the gap between a minor cluster and its neighbouring abundant cluster via a second-order derivative type distance score; candidates identified when skewness_k > 2) — github.com/fabotao/GapClust
- **RaceID** (Comparison baseline for rare-cell detection AUC in the replacement-experiment benchmark)
- **GiniClust** (Comparison baseline; cross-checked to confirm whether rare clusters (e.g. 3-cell R1) are missed)
- **FiRE** (Comparison baseline; cross-checked to confirm whether rare clusters (e.g. 3-cell R1) are missed)
- **Seurat** (scRNA-seq preprocessing/subsetting toolkit for building the benchmark subsets and annotating detected rare clusters)

## Examples

```
devtools::install_github("fabotao/GapClust"); results <- GapClust(expr_matrix, K_candidates = which(skewness_k > 2))
```

## Evaluation signals

- In the 2-monocyte setting at 50 DE genes, GapClust average AUC ≈ 0.89 while RaceID ≈ 0.63 and GiniClust/FiRE ≈ 0.5 (Fig. 3b anchor).
- Full 68k PBMC run recovers R1 with exactly 3 CD14+ cells (missed by GiniClust and FiRE) and R2 with 185 CD34+ megakaryocyte cells.
- R2 cells show the expected erythrocyte-marker expression pattern relative to remaining CD34+ cells (Fig. 6a-d).
- AUC curves are monotone summaries over 100 replicates per monocyte count; per-method curves should be smooth averages, not single-run values.
- GapClust candidate ranking respects the skewness_k > 2 default criterion; output files (e.g. rare_cells_68k_pbmc.csv) contain cluster assignments, sizes, and marker profiles.

## Limitations

- Competitor tools have scale limits: RaceID has unrealistic running time on datasets with thousands of cells, and GiniClust was reported to fail on very large datasets, restricting fair benchmarking ranges.
- FiRE does not discriminate outliers from minor cell types, so it behaves differently on outlier-vs-cluster tasks (AUC ≈ 0.5 in the 2-monocyte benchmark).
- No version numbers for GiniClust, RaceID, CellSIUS, FiRE, EDGE are given in the article, complicating exact reproduction.
- GapClust's design assumes a detectable gap between the minor cluster and its nearest abundant neighbour; populations lacking such a gap may be missed.
- README prerequisites constrain the environment: R >= 3.4.0, Seurat >= 3.1.0, rflann >= 1.8.4, irlba >= 2.3.3, e1071 >= 1.7-0.1.

## Evidence

- [results] GapClust AUC at 50 DE genes (2-monocyte setting): [evidence span withheld — anchor status: not_found]
- [discussion] skewness criterion: "Candidate k can be identified if the skewness<k> > 2."
- [intro] GapClust design rationale: "the design of GapClust is inspired by the observation that for a particular data point in a minor cluster Cn of size n, then the distance to its nearest neighbours outside Cn is far greater than to"
- [abstract] GapClust scope statement: "Here, we develop GapClust, a light-weight algorithm to detect rare cell types from ultra-large scRNA-seq datasets"
- [readme] README prerequisites: "Seurat >=  3.1.0
rflann >=1.8.4 (GitHub: YeeJeremy/rflann)
irlba >=2.3.3
e1071 >= 1.7-0.1"
