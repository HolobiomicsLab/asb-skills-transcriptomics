---
name: single-cell-rnaseq-library-size-filtering
description: Use when a raw scRNA-seq count matrix (e.g. Drop-seq, thousands of cells)
  contains extreme-depth cells or very sparse genes that would distort downstream
  imputation or Gini-distribution comparisons.
when_to_use_negative:
- The input is already normalized and QC-filtered (e.g. a Seurat object post-QC) —
  re-filtering by the same cutoffs would double-correct library sizes
- The dataset is not droplet-based scRNA-seq; bulk RNA-seq has far fewer zeros and
  does not need this depth-based cell filtering
- Cell-type identification or clustering is the goal and deliberately retaining very
  small or large library-size cells is part of the analysis plan
edam_operation: http://edamontology.org/operation_3695
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: ALRA
  role: Downstream zero-preserving imputation applied to the filtered, GAPDH-normalized
    matrix (rank k = 14, rsvd randomized SVD)
  repo: https://github.com/KlugerLab/ALRA
- name: rsvd
  role: Randomized SVD backend required by ALRA for computing the rank-k approximation
    after filtering
  repo: https://github.com/KlugerLab/ALRA
- name: Seurat
  role: Alternative R framework in which ALRA is integrated (RunALRA) on QC-filtered
    scRNA-seq data
  repo: https://github.com/satijalab/seurat
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1038/s41467-021-27729-z
    title: Zero-preserving imputation of single-cell RNA-seq data
  source_content_hash: sha256:25876d30f7f42e072d2656fa5838d48b6eddc612a173652e323463152af5dc2e
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/KlugerLab/ALRA
  tools:
  - ALRA
  - SAVER
  - Seurat
  - rsvd
  edam_operation: http://edamontology.org/operation_3695
  edam_topics:
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7272727272727273
n_claims: 11
---

# single-cell-rnaseq-library-size-filtering

## Summary

Quality-control procedure for droplet-based scRNA-seq matrices that removes poor-quality cells by library-size cutoffs (keep cells with total counts between 500 and 20,000) and low-abundance genes (mean expression >= 0.1), plus GAPDH-percentile outlier removal, before imputation or normalization.

## When to use

Use when a raw scRNA-seq count matrix (e.g. Drop-seq, thousands of cells) contains extreme-depth cells or very sparse genes that would distort downstream imputation or Gini-distribution comparisons.

## When NOT to use

- The input is already normalized and QC-filtered (e.g. a Seurat object post-QC) — re-filtering by the same cutoffs would double-correct library sizes
- The dataset is not droplet-based scRNA-seq; bulk RNA-seq has far fewer zeros and does not need this depth-based cell filtering
- Cell-type identification or clustering is the goal and deliberately retaining very small or large library-size cells is part of the analysis plan

## Inputs

- Raw scRNA-seq count matrix (cells x genes), e.g. Drop-seq melanoma data (8640 cells, 32,287 genes) from Torre et al. (2018)
- Optional comparator gene list (e.g. 26-gene RNA FISH panel) for restricting to shared genes
- Reference housekeeping gene (GAPDH) for percentile-based outlier removal and normalization

## Outputs

- Filtered and GAPDH-normalized expression matrix (e.g. 8498 cells, 12,241 genes, 16 genes shared with FISH)
- gini_fish_correlation.csv containing FISH-vs-Drop-seq Gini coefficient Pearson correlations for the 15 shared genes (excluding GAPDH)

## How to apply

Compute each cell's library size (total counts) on the raw Drop-seq matrix (e.g. 8640 cells x 32,287 genes from Torre et al. 2018) and discard cells with library size greater than 20,000 or less than 500. Separately, remove genes whose mean expression is less than 0.1, and intersect gene lists with any comparator dataset (e.g. 16 genes shared with RNA FISH, leaving 16 shared genes / 12,241 genes total). Then remove cells in the bottom and top tenth percentiles of GAPDH expression, since extreme GAPDH indicates technical outliers, and normalize all genes by GAPDH expression prior to running ALRA (rank k = 14 via rsvd randomized SVD). Applying these cutoffs here reduced 8640 cells to 8498 cells, giving a stable normalized matrix whose zero structure is dominated by dropout rather than sequencing depth artifacts.

## Related tools

- **ALRA** (Downstream zero-preserving imputation applied to the filtered, GAPDH-normalized matrix (rank k = 14, rsvd randomized SVD)) — https://github.com/KlugerLab/ALRA
- **rsvd** (Randomized SVD backend required by ALRA for computing the rank-k approximation after filtering) — https://github.com/KlugerLab/ALRA
- **Seurat** (Alternative R framework in which ALRA is integrated (RunALRA) on QC-filtered scRNA-seq data) — https://github.com/satijalab/seurat

## Examples

```
lib <- colSums(raw_counts); keep <- lib >= 500 & lib <= 20000; filtered <- raw_counts[keep, , drop=FALSE]; result.completed <- alra(filtered)[[3]]
```

## Evaluation signals

- Cell count after filtering should be ~8498 from the original 8640 Drop-seq cells and gene count ~12,241 from 32,287
- Number of genes shared with the 26-gene RNA FISH panel should be 16 (15 used downstream after excluding GAPDH)
- No retained cell should have library size > 20,000 or < 500, and no retained gene should have mean expression < 0.1
- All retained cells must fall strictly within the inner 99.8% of the GAPDH expression distribution (excluding bottom/top tenth percentiles)
- Pearson correlation of Gini coefficients (FISH vs ALRA-imputed Drop-seq) should be approximately 0.8, vs ~0.3 raw and ~0.6 for SAVER

## Limitations

- Cutoffs (500/20,000 counts, mean >= 0.1, GAPDH tenth percentiles) were tuned for this Drop-seq melanoma dataset; other datasets may need recalibrated thresholds
- Filtering by GAPDH percentiles assumes GAPDH is reliably expressed and its distribution reflects technical quality, which may not hold in all tissues
- ALRA imputation quality improves with the full dataset — results on subsets (e.g. the bundled b_nk_example) can be visibly worse than paper-scale runs

## Evidence

- [methods] Filter cells with library size greater than 20,000 or less than 500 and genes with mean expression less than 0.1, yielding 8498 cells, 12,241 genes: [evidence span withheld — anchor status: not_found]
- [methods] Remove cells in the bottom and top tenth percentiles of GAPDH expression and normalize all genes by GAPDH expression.: [evidence span withheld — anchor status: not_found]
- [intro] the fraction of zeros in a typical scRNA-seq expression matrix is much higher, even exceeding 90% with recently developed droplet-based techniques: "the fraction of zeros in a typical scRNA-seq expression matrix is much higher, even exceeding 90% with recently developed droplet-based techniques"
- [readme] The only prerequisite for ALRA is installation of the randomized SVD package RSVD which can be installed as install.packages('rsvd').: "The only prerequisite for ALRA is installation of the randomized SVD package RSVD which can be installed as `install.packages('rsvd')`."
