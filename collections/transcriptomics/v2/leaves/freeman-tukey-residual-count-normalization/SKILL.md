---
name: freeman-tukey-residual-count-normalization
description: Use when raw scRNA-seq count matrices (e.g., SingleCellExperiment objects)
  are overdispersed and you need a chi-squared-residual-based dimension reduction
  or multi-batch integration embedding that outperforms log-transform PCA.
when_to_use_negative:
- The input is not raw count data — CA residual decomposition expects a counts matrix;
  pre-log-transformed 'logcounts' input defeats the stated advantage of 'avoiding
  distortive log-transformation'.
- 'The benchmark ground truth requires independently validated cell-type labels: the
  article notes the analyzed datasets ''did not have independently validated cell
  type annotations, so performance is limited by the original cell type assignments''.'
- The analysis depends on knowing optimal settings a priori from data characteristics
  — the paper cautions that 'parameters like number of "true" clusters are unknown
  a priori and depend on the specific research question and context'.
edam_operation: http://edamontology.org/operation_2939
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: corral
  role: Single-table CA dimension reduction of scRNA-seq counts using Freeman–Tukey
    residuals; interfaces directly with SingleCellExperiment Bioconductor classes
  repo: github.com/laurenhsu1/corral
- name: corralm
  role: Integrative multi-table dimension reduction / batch integration of multiple
    count matrices into shared FT-residual CA embeddings
  repo: github.com/laurenhsu1/corral
- name: R
  role: Implementation language for the corral package and surrounding pipeline
- name: Bioconductor
  role: Package ecosystem providing SingleCellExperiment classes that corral interfaces
    with directly
- name: Harmony
  role: Optional downstream batch correction applied to corralm FT embeddings (e.g.,
    pancreas ASW benchmark)
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1038/s41598-022-26434-1
    title: Correspondence analysis for dimension reduction, batch integration, and
      visualization of single-cell RNA-seq data
  source_content_hash: sha256:1638d6cf73840de5aa8da5a15bd54f0b717cbf4a4da10db4c207bb234ffae5cf
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - Bioconductor
  - Harmony
  - MNN
  - R
  - corral
  - corralm
  edam_operation: http://edamontology.org/operation_2939
  edam_topics:
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9090909090909091
n_claims: 11
---

# freeman-tukey-residual-count-normalization

## Summary

Compute cell embeddings from single-cell RNA-seq count matrices via correspondence analysis using Freeman–Tukey (FT) residuals instead of the standard log(x+1) + PCA transform, implemented in the corral/corralm R/Bioconductor package. FT-residual CA avoids the distortive log-transformation and was the most performant CA variant across diverse datasets in the Hsu & Culhane 2023 benchmark.

## When to use

Use when raw scRNA-seq count matrices (e.g., SingleCellExperiment objects) are overdispersed and you need a chi-squared-residual-based dimension reduction or multi-batch integration embedding that outperforms log-transform PCA.

## When NOT to use

- The input is not raw count data — CA residual decomposition expects a counts matrix; pre-log-transformed 'logcounts' input defeats the stated advantage of 'avoiding distortive log-transformation'.
- The benchmark ground truth requires independently validated cell-type labels: the article notes the analyzed datasets 'did not have independently validated cell type annotations, so performance is limited by the original cell type assignments'.
- The analysis depends on knowing optimal settings a priori from data characteristics — the paper cautions that 'parameters like number of "true" clusters are unknown a priori and depend on the specific research question and context'.

## Inputs

- Raw scRNA-seq count matrix (e.g., SingleCellExperiment object with counts assay)
- Cell and batch/sample metadata annotations for multi-table runs
- Single count matrix (corral) or concatenated multi-batch count matrices (corralm)

## Outputs

- Low-dimensional cell embeddings computed from the chi-squared (Freeman–Tukey) residual matrix
- Reduced-dimension representation stored in the SingleCellExperiment object for downstream clustering, visualization, and biplot interpretation
- Comparative metrics tables (e.g., ASW_celltype and 1-ASWbatch per method on 8 PCs)

## How to apply

Load counts into SingleCellExperiment objects (retaining cell type and batch annotations) and call corral on a single table or corralm on concatenated multi-batch data, selecting Freeman–Tukey residuals as the chi-squared residual variant. CA proceeds by SVD of the residual matrix computed from the counts directly, avoiding the typical log(x + 1) transformation followed by PCA. For batch integration, supply a batch/sample split variable to corralm to align multiple tables into a shared low-dimensional embedding; the FT variant is the default recommendation because it was most performant overall in cluster recovery tasks. The resulting cell embeddings can be plugged into downstream clustering, visualization, or batch-correction pipelines (e.g., Harmony or MNN applied to the corralm embeddings) by simple pipeline substitution of CA for PCA. Verify success by checking that embeddings recover known cell-type structure (e.g., ARI/silhouette comparable to or better than glmPCA and standard CA) while preserving batch separation behavior.

## Related tools

- **corral** (Single-table CA dimension reduction of scRNA-seq counts using Freeman–Tukey residuals; interfaces directly with SingleCellExperiment Bioconductor classes) — github.com/laurenhsu1/corral
- **corralm** (Integrative multi-table dimension reduction / batch integration of multiple count matrices into shared FT-residual CA embeddings) — github.com/laurenhsu1/corral
- **R** (Implementation language for the corral package and surrounding pipeline)
- **Bioconductor** (Package ecosystem providing SingleCellExperiment classes that corral interfaces with directly)
- **Harmony** (Optional downstream batch correction applied to corralm FT embeddings (e.g., pancreas ASW benchmark))

## Examples

```
devtools::install_github('laurenhsu1/corral')
```

## Evaluation signals

- Clustering accuracy from FT-residual CA embeddings should match or beat standard CA and glmPCA — the benchmark reports better or comparable performance in 8 out of 9 datasets.
- On multi-dataset integration tasks (e.g., Baron/Lawlor/Muraro pancreas), FT-based corralm+Harmony should reach ASW_celltype vs 1-ASWbatch on 8 PCs comparable to established routines like Seurat.
- Embeddings must be computed directly from the counts residual matrix — confirm no log(x + 1) transformation was applied upstream.
- Output reduced-dim slot in the SingleCellExperiment object should contain cell coordinates in a low-dimensional space usable for clustering, trajectory analysis, and cell type classification.
- CA biplot should allow features and cells to be interpreted in the same space, supporting exploratory checks of cluster structure.

## Limitations

- No tool or package versions for corral or its dependencies are reported, so exact reproducibility of versions is not documented.
- Benchmarking ground-truth labels came from original dataset publications and were not independently validated; ARI can penalize methods that better distinguish sub-populations or rare cell types relative to those reference labels.
- High-quality ground-truth benchmarking datasets are largely simple 'pseudo'-cell mixtures or pools of distinct cell types and do not reflect real-world complex-tissue conditions.
- There is not yet a systematic benchmarking framework to identify the optimal dimension-reduction approach based on dataset characteristics.

## Evidence

- [abstract] CA with Freeman-Tukey residuals performs especially well across diverse datasets: "CA with Freeman-Tukey residuals performs especially well across diverse datasets"
- [intro] FT residuals as especially performant variant: "In particular, we find that CA with Freeman–Tukey residuals, an alternative chi-squared statistic, is especially performant across a variety of test cases."
- [abstract] CA avoids log-transformation and is fast/scalable: "CA is based on decomposition of a chi-squared residual matrix, avoiding distortive log-transformation"
- [readme] corral package purpose: "corral is a package to perform correspondence analysis on count data, particularly for single-cell (e.g., RNA-seq)."
