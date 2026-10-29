---
name: scrna-seq-spectral-clustering-accuracy-evaluation
description: Use when a raw scRNA-seq count matrix with known cell-subpopulation labels
  (e.g., embryonic stages) is available and you must quantify whether imputation improves
  cell-population clustering accuracy against no-imputation and competitor imputers.
when_to_use_negative:
- Cell-subpopulation labels are unknown, so supervised accuracy measures (ARI, Jaccard,
  NMI, purity) cannot be computed against a ground truth
- The data are droplet-based with severely high dropout rates, where the paper notes
  imputation efficiency degrades
- The goal is differential expression or trajectory analysis rather than clustering
  accuracy benchmarking
edam_operation: http://edamontology.org/operation_3432
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_2269
tools:
- name: scImpute
  role: Primary imputation method under test; scimpute() run on the raw count matrix
    with drop_thre = 0.5, Kcluster = number of embryonic stages, parallel via ncores
  repo: https://github.com/Vivianstats/scImpute
- name: MAGIC
  role: Competitor imputation method benchmarked for comparison; also used to quantify
    over-smoothing via two-cell Pearson correlation
- name: SAVER
  role: Competitor imputation method (Bayesian, borrows information across genes)
    benchmarked against scImpute
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1038/s41467-018-03405-7
    title: An accurate and robust imputation method scImpute for single-cell RNA-seq
      data
  source_content_hash: sha256:2ae18a838e14fecd0880c74ea3b53c4cb630ffe81c3cbc75dbe4c5b34640f138
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/Vivianstats/scImpute
  tools:
  - MAGIC
  - SAVER
  - scImpute
  edam_operation: http://edamontology.org/operation_3432
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_2269
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9090909090909091
n_claims: 11
---

# scrna-seq-spectral-clustering-accuracy-evaluation

## Summary

Benchmarks whether dropout imputation (scImpute vs MAGIC vs SAVER vs none) improves clustering of cell subpopulations in scRNA-seq count matrices, using spectral clustering on the first two PCs and four accuracy measures (ARI, Jaccard, NMI, purity).

## When to use

Use when a raw scRNA-seq count matrix with known cell-subpopulation labels (e.g., embryonic stages) is available and you must quantify whether imputation improves cell-population clustering accuracy against no-imputation and competitor imputers.

## When NOT to use

- Cell-subpopulation labels are unknown, so supervised accuracy measures (ARI, Jaccard, NMI, purity) cannot be computed against a ground truth
- The data are droplet-based with severely high dropout rates, where the paper notes imputation efficiency degrades
- The goal is differential expression or trajectory analysis rather than clustering accuracy benchmarking

## Inputs

- Raw scRNA-seq count matrix (genes as rows, cells as columns), e.g., GSE45719 mouse preimplantation embryo data (268 cells)
- Known cell-subpopulation labels (e.g., embryonic stage assignments) for computing ARI, Jaccard, NMI, purity

## Outputs

- imputation_clustering_benchmark.csv with ARI, Jaccard index, NMI, and purity per imputation method and per k (6, 8, 10, 12, 14)
- Imputed count matrices from scImpute (scimpute_count.csv), MAGIC, and SAVER
- Two-cell Pearson correlation values before and after imputation per method

## How to apply

Start from a raw count matrix with genes as rows and cells as columns (e.g., GSE45719 mouse preimplantation embryo, 268 cells). Impute dropouts with scimpute() using drop_thre = 0.5, Kcluster set to the known number of cell subpopulations (embryonic stages), and parallel cores via ncores; run MAGIC and SAVER on the same matrix for comparison. For each imputed (and the raw) matrix, project cells onto the first two principal components and run spectral clustering at k = 6, 8, 10, 12, 14. At each k compute adjusted Rand index, Jaccard index, NMI, and purity against known labels, and compute two-cell Pearson correlations before and after imputation to check over-smoothing. Assemble per-k, per-method results into a benchmark CSV; correct application shows scImpute attaining the best score under all four measures, with two-cell Pearson correlation rising about 0.72 to 0.82 for scImpute versus about 0.95 for MAGIC (evidence of MAGIC's excessive smoothing).

## Related tools

- **scImpute** (Primary imputation method under test; scimpute() run on the raw count matrix with drop_thre = 0.5, Kcluster = number of embryonic stages, parallel via ncores) — https://github.com/Vivianstats/scImpute
- **MAGIC** (Competitor imputation method benchmarked for comparison; also used to quantify over-smoothing via two-cell Pearson correlation)
- **SAVER** (Competitor imputation method (Bayesian, borrows information across genes) benchmarked against scImpute)

## Examples

```
scimpute(count_path = "raw_count.csv", infile = "csv", outfile = "csv", out_dir = "./", labeled = FALSE, drop_thre = 0.5, Kcluster = 6, ncores = 10)
```

## Evaluation signals

- scImpute achieves the best score under all four measures (ARI, Jaccard, NMI, purity) at every tested k (6, 8, 10, 12, 14) versus no imputation, MAGIC, and SAVER
- Two-cell Pearson correlation rises from ~0.72 (raw) to ~0.82 after scImpute, while MAGIC inflates it to ~0.95, flagging over-smoothing
- Imputed output matrix has the same dimensions (genes x cells) as the raw input count matrix
- Benchmark CSV contains a row for each method x k combination with all four accuracy metrics populated

## Limitations

- MAGIC and SAVER alter all gene expression levels including those unaffected by dropouts, potentially introducing new biases and eliminating meaningful biological variation
- Imputation efficiency degrades when dropout rates in the raw data are severely high (droplet-based technologies)
- Choice of the Kcluster parameter (number of cell populations) is left to the user; this benchmark assumes the number of stages is known
- No scImpute software version is specified in the article

## Evidence

- [other] Spectral clustering evaluation protocol: [evidence span withheld — anchor status: not_found]
- [intro] MAGIC/SAVER over-smoothing caveat: "Both MAGIC and SAVER would alter all gene expression levels including those unaffected by dropouts"
- [methods] Dropout probability threshold parameter: "Only the values that have **dropout probability** larger than `drop_thre` are imputed by `scimpute`."
- [readme] Clustering as supported downstream use of scImpute: "clustering of cell populations"
- [readme] Recommendation to run on whole-genome count matrix: [evidence span withheld — anchor status: external_doc]
