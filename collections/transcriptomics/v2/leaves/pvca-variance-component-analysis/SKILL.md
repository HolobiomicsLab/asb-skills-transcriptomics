---
name: pvca-variance-component-analysis
description: Use when evaluating a batch-correction method's output embeddings and
  you must quantify residual batch-driven versus biological (cell-type) variance,
  e.g., after JIVE/Seurat/Harmony integration.
when_to_use_negative:
- The input lacks batch and cell-type annotations, since PVCA cannot decompose variance
  across those labels without them
- The goal is batch correction itself rather than post-hoc evaluation of corrected
  representations — use jive()/scJIVE, Seurat v5 integration, Harmony, LIGER, or ComBat-seq
  instead
- The data are bulk multi-omics (e.g., TCGA BRCA gene expression/methylation/miRNA
  matrices) where the original JIVE vignette workflow (BRCA_data, showVarExplained,
  showHeatmaps, showPCA) applies rather than the single-cell benchmark metrics
edam_topics:
- http://edamontology.org/topic_0091
tools:
- name: JIVE (scJIVE)
  role: Produces the joint/individual decomposition whose outputs are evaluated with
    PVCA; joint rank fixed at 10, individual ranks at 20 in the simulation benchmark
  repo: https://github.com/oconnell-statistics-lab/scJIVE
- name: r.jive
  role: R package providing the jive() decomposition function applied per batch to
    the preprocessed Bacher T-cell matrix
- name: Seurat v5
  role: Benchmark comparator whose integrated embeddings are scored with PVCA, kBET,
    ASW, and LISI
- name: Harmony
  role: Benchmark comparator (sigma=2, lambda=0.1) whose aligned embeddings are evaluated
    with PVCA
- name: LIGER
  role: Benchmark comparator on simulated balanced/unbalanced-batch scRNA-seq data
- name: ComBat-seq
  role: Benchmark comparator whose corrected counts are evaluated with PVCA and companion
    metrics
- name: gplots
  role: Plotting dependency used alongside the JIVE R implementation for visualization
    of results
provenance:
  source_task_ids:
  - task_002
  - task_003
  source_papers:
  - doi: 10.1093/bioadv/vbae134
    title: Batch-effect correction in single-cell RNA sequencing data using JIVE
  source_content_hash: sha256:6067671275c9476c2f3c814b1274e0fc6509c2f769ac293f7b79c5f0b790c6d9
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/oconnell-statistics-lab/scJIVE
  tools:
  - ComBat-seq
  - Harmony
  - JIVE (scJIVE)
  - LIGER
  - R
  - Seurat v5
  - gplots
  - r.jive
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.45454545454545453
n_claims: 11
---

# pvca-variance-component-analysis

## Summary

Applies PVCA (principal variance component analysis) to quantify the fraction of variance attributable to batch versus cell type in corrected scRNA-seq representations, as used in the JIVE batch-correction benchmark to compare scJIVE against Seurat v5, Harmony, LIGER, and ComBat-seq.

## When to use

Use when evaluating a batch-correction method's output embeddings and you must quantify residual batch-driven versus biological (cell-type) variance, e.g., after JIVE/Seurat/Harmony integration.

## When NOT to use

- The input lacks batch and cell-type annotations, since PVCA cannot decompose variance across those labels without them
- The goal is batch correction itself rather than post-hoc evaluation of corrected representations — use jive()/scJIVE, Seurat v5 integration, Harmony, LIGER, or ComBat-seq instead
- The data are bulk multi-omics (e.g., TCGA BRCA gene expression/methylation/miRNA matrices) where the original JIVE vignette workflow (BRCA_data, showVarExplained, showHeatmaps, showPCA) applies rather than the single-cell benchmark metrics

## Inputs

- Batch-corrected low-rank embeddings from scJIVE (jive() output), Seurat v5, Harmony, LIGER, or ComBat-seq
- Batch labels (e.g., sequencing batch for the Bacher CD4+ T-cell data)
- Cell-type / cluster labels (e.g., the 'clusts' vector of jointly present sample clusters)
- Preprocessed log-normalized count matrices with top 2000 variable genes selected

## Outputs

- PVCA variance shares per method: pvca_batch_share and pvca_cluster_share columns in metrics_table.csv
- Per-method benchmark table combining PVCA shares with scaled kBET acceptance rate, ASW (batch and cell type), and LISI, organized by method and batch-balance condition
- t-SNE embeddings colored by batch and cell cluster for each corrected method

## How to apply

Run PVCA on each corrected representation produced by the batch-correction pipeline (e.g., scJIVE joint scores with joint rank 10 and individual ranks 20, Seurat v5 integration, Harmony with sigma=2 and lambda=0.1, LIGER, ComBat-seq) alongside the scaled kBET acceptance rate, average silhouette width (ASW) on batch and cell type, and LISI. In the JIVE workflow, each batch is treated as a separate data source so the joint structure captures biology while the individual structures capture technical batch variation; PVCA then verifies this decomposition by reporting the batch and cell-cluster variance shares. For the Bacher CD4+ T-cell data (2000 genes x 1867 cells, two of fifteen batches), validate that the PVCA batch share falls near the reported ~4.9% (accept ~4-6%) and the cell-cluster share near ~9.3%. Assemble PVCA shares with kBET, ASW, and LISI into a metrics CSV with one row per method and condition, mirroring the paper's Tables 4-6 of Section 3.2. Prior to decomposition, matrices can be scaled by total variation, with row-orthogonality enforced between joint and individual estimates.

## Related tools

- **JIVE (scJIVE)** (Produces the joint/individual decomposition whose outputs are evaluated with PVCA; joint rank fixed at 10, individual ranks at 20 in the simulation benchmark) — https://github.com/oconnell-statistics-lab/scJIVE
- **r.jive** (R package providing the jive() decomposition function applied per batch to the preprocessed Bacher T-cell matrix)
- **Seurat v5** (Benchmark comparator whose integrated embeddings are scored with PVCA, kBET, ASW, and LISI)
- **Harmony** (Benchmark comparator (sigma=2, lambda=0.1) whose aligned embeddings are evaluated with PVCA)
- **LIGER** (Benchmark comparator on simulated balanced/unbalanced-batch scRNA-seq data)
- **ComBat-seq** (Benchmark comparator whose corrected counts are evaluated with PVCA and companion metrics)
- **gplots** (Plotting dependency used alongside the JIVE R implementation for visualization of results)

## Examples

```
devtools::install_github("oconnell-statistics-lab/scJIVE"); library(r.jive); res <- jive(list(batch1 = counts1, batch2 = counts2), method = "bic")
```

## Evaluation signals

- On the Bacher CD4+ T-cell data, the PVCA batch share falls near the reported ~4.9% (accept ~4-6%) and the cell-cluster share near ~9.3%
- metrics_table.csv exists with named columns kBET, ASW, LISI, pvca_batch_share, pvca_cluster_share for every method row
- Every method-by-condition row in the CSV carries a kBET acceptance rate, batch ASW, cell-type ASW, and LISI value
- Qualitative ordering matches the reported benchmark: JIVE achieves the best cell-type ASW under balanced batches, and the best kBET acceptance rate on the Bacher T-cell data with Seurat v5 close behind
- Row-orthogonality holds between joint and individual JIVE estimates feeding the PVCA decomposition

## Limitations

- JIVE's original implementation was designed for bulk sequencing data and is computationally infeasible for large-scale scRNA-seq datasets, so the enhanced scJIVE implementation must be used before applying PVCA in the single-cell setting
- The Bacher evaluation retained only two of fifteen batches, so PVCA shares reflect that reduced 2000 genes x 1867 cells configuration rather than the full dataset
- JIVE performed best on balanced batch scenarios and cell-type preservation; under unbalanced batch conditions the method ranking can differ, so PVCA-based conclusions are condition-dependent

## Evidence

- [abstract] JIVE performed best in terms of preserving cell-type effects and in scenarios in which the batch sizes are balanced.: "JIVE performed best in terms of preserving cell-type effects and in scenarios in which the batch sizes are balanced."
- [abstract] We benchmarked the results against four popular tools, Seurat v5, Harmony, LIGER, and Combat-seq, which were developed for this purpose.: "We benchmarked the results against four popular tools, Seurat v5, Harmony, LIGER, and Combat-seq, which were developed for this purpose."
- [methods] The estimated joint structure corresponds to three previously identified BRCA sample clusters: "We see that the estimated joint corresponds well to the three previously identified clusters"
- [intro] scJIVE can be used to extract shared biological patterns from multi-source sequencing data while adjusting for individual non-biological variations (i.e., batch effect).: "scJIVE can be used to extract shared biological patterns from multi-source sequencing data while adjusting for individual non-biological variations (i.e., batch effect)."
