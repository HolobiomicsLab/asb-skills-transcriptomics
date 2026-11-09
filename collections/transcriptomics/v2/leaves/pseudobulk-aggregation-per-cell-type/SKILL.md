---
name: pseudobulk-aggregation-per-cell-type
description: Use when scRNAseq data (e.g., GSE223109 mouse dermal skin) must be converted
  to pseudobulk per annotated cell type per time point before circadian rhythm detection
  or circadian-time prediction.
when_to_use_negative:
- The goal is single-cell (per-cell) circadian-time prediction — tauFisher cannot
  overcome the high sequencing dropout rate in scRNAseq, so aggregation to pseudobulk
  is mandatory, not optional
- Input is already bulk or pseudobulk expression data (e.g., bulk RNAseq GEO series
  like GSE157077); no per-cell-type aggregation is needed
- Cell-type annotations are unavailable or unreliable — summing across mislabeled
  cells would confound the pseudobulk signal
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: tauFisher
  role: 'Downstream consumer of pseudobulk replicates: predictor genes are subset
    from each aggregated replicate, scaled pairwise gene-expression differences are
    projected onto the trained PCA embedding to predict circadian time'
  repo: https://github.com/micnngo/tauFisher
- name: MetaCycle
  role: Receives CP10K-normalized per-cell-type pseudobulk matrices via meta2d to
    run JTK_Cycle for rhythmic gene detection
- name: dplyr
  role: Used in R to group cells by annotated cell type and sample/ZT time point and
    sum raw UMI counts into pseudobulk matrices; also used to intersect rhythmic gene
    lists
- name: stringr
  role: Used alongside dplyr for handling sample/time-point labels when assembling
    pseudobulk and prediction summary tables
provenance:
  source_task_ids:
  - task_007
  - task_008
  source_papers:
  - doi: 10.1038/s41467-024-48041-6
    title: tauFisher predicts circadian time from a single sample of bulk and single-cell
      pseudobulk transcriptomic data
  source_content_hash: sha256:cb7e50ba0bbedc927cfe063efb28a237ce9031c06cfa806f65f46961a4535833
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/micnngo/tauFisher
  tools:
  - MetaCycle
  - dplyr
  - stats
  - stringr
  - tauFisher
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7
n_claims: 10
---

# pseudobulk-aggregation-per-cell-type

## Summary

Aggregate single-cell UMI counts into per-cell-type, per-sample pseudobulk matrices by summing raw counts across all cells of an annotated type (e.g., dermal fibroblasts vs immune cells) at each circadian time point, enabling tools like JTK_Cycle/MetaCycle and tauFisher that cannot operate on single-cell data.

## When to use

Use when scRNAseq data (e.g., GSE223109 mouse dermal skin) must be converted to pseudobulk per annotated cell type per time point before circadian rhythm detection or circadian-time prediction.

## When NOT to use

- The goal is single-cell (per-cell) circadian-time prediction — tauFisher cannot overcome the high sequencing dropout rate in scRNAseq, so aggregation to pseudobulk is mandatory, not optional
- Input is already bulk or pseudobulk expression data (e.g., bulk RNAseq GEO series like GSE157077); no per-cell-type aggregation is needed
- Cell-type annotations are unavailable or unreliable — summing across mislabeled cells would confound the pseudobulk signal

## Inputs

- Raw single-cell UMI count matrix with cell-level metadata (sample, circadian time point e.g. ZT, annotated cell type such as dermal fibroblasts vs dermal immune cells)
- Cell-type annotation labels from clustering (PCA + Louvain) of the integrated scRNAseq dataset
- List of circadian time points / ZT values per sample (e.g., 6 ZT time points in GSE223109)

## Outputs

- Per-cell-type pseudobulk raw count matrix (cell types x circadian time points), e.g. fibroblast and immune-cell series
- CP10K-normalized pseudobulk matrices formatted for meta2d/JTK_Cycle with NA columns for missing time points
- Bootstrap pseudobulk replicates (500 iterations per ZT) log2-transformed as log2(x+1) for tauFisher prediction
- Aggregated tables such as rhythmic_genes_fibroblast_immune.csv and per_zt_prediction_summary.csv

## How to apply

Sum raw UMI counts over all cells of each annotated cell type (fibroblasts and immune cells) within each sample/circadian time point to build per-cell-type pseudobulk count matrices, using dplyr grouping in R. For JTK_Cycle analysis, normalize each pseudobulk sample by its total read count scaled to CP10K and format the matrix for meta2d with columns ordered by circadian time, inserting NA columns for missing time points as required. For tauFisher phase prediction, build bootstrap pseudobulk replicates by sampling n cells with replacement (n = 20% of the immune-cell count) for 500 iterations per ZT, then log2-transform as log2(x+1) because tauFisher cannot run on single cells due to dropout. Cell annotation itself follows CellRanger (MM10) alignment, SCTransform normalization per sample with 3250 integration features, and PCA/Louvain clustering; cells with 850-7800 features and <13% mitochondrial genes are kept upstream.

## Related tools

- **tauFisher** (Downstream consumer of pseudobulk replicates: predictor genes are subset from each aggregated replicate, scaled pairwise gene-expression differences are projected onto the trained PCA embedding to predict circadian time) — https://github.com/micnngo/tauFisher
- **MetaCycle** (Receives CP10K-normalized per-cell-type pseudobulk matrices via meta2d to run JTK_Cycle for rhythmic gene detection)
- **dplyr** (Used in R to group cells by annotated cell type and sample/ZT time point and sum raw UMI counts into pseudobulk matrices; also used to intersect rhythmic gene lists)
- **stringr** (Used alongside dplyr for handling sample/time-point labels when assembling pseudobulk and prediction summary tables)

## Examples

```
pseudobulk <- counts %>% as.data.frame() %>% group_by(sample, cell_type) %>% summarise(across(everything(), sum), .groups='drop')
```

## Evaluation signals

- Reproducing the JTK_Cycle analysis on normalized pseudobulk data should recover 1946 rhythmic genes in dermal fibroblasts, 432 in dermal immune cells, and exactly 79 genes rhythmic in both cell types
- Shared rhythmic genes should be predominantly related to the core clock network and metabolism
- For phase-heterogeneity analysis, the standard deviation of 500 bootstrapped circadian-phase predictions should be greater for immune cells than fibroblasts at five of six ZT time points (Rao's test for homogeneity, Wallraff test of angular distances)
- Verify every sample x cell-type combination has a pseudobulk column, with NA columns inserted for missing circadian time points as required by JTK_Cycle
- Check that pseudobulk values for bootstrap replicates are log2(x+1) transformed before test_tauFisher is applied

## Limitations

- Aggregation is required because tauFisher cannot predict circadian time at single-cell resolution due to high scRNAseq dropout
- The paper's claim of greater circadian phase heterogeneity in immune cells relies only on computational analysis of bootstrapped pseudobulk replicates, with no experimental validation
- The number of bootstrap iterations (500) and the sample size (20% of immune-cell count) are specific choices; results depend on these settings

## Evidence

- [other] Sum raw UMI counts over all cells of each annotated cell type (fibroblasts and immune cells) within each sample/circadian time point to build per-cell-type pseudobulk count matrices.: [evidence span withheld — anchor status: not_found]
- [discussion] tauFisher cannot predict circadian time at single-cell level due to scRNAseq dropout: "it cannot overcome the high sequencing dropout rate in scRNAseq and thus cannot predict circadian time at a single cell level"
- [abstract] Application of tauFisher at a cell-type level in a single-cell RNAseq dataset collected from mouse dermal skin: "Application of tauFisher at a cell-type level in a single-cell RNAseq dataset collected from mouse dermal skin"
- [other] Bootstrap pseudobulk replicates built by sampling n cells with replacement and log2-transforming before tauFisher prediction: [evidence span withheld — anchor status: not_found]
