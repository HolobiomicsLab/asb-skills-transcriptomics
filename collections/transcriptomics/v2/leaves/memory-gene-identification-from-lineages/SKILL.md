---
name: memory-gene-identification-from-lineages
description: Use when you have a quality-controlled scRNA-seq expression matrix with
  cell-to-lineage annotations (cellular barcodes or GEMLI-predicted lineages) and
  want to enumerate and categorize heritably expressed genes.
when_to_use_negative:
- The dataset lacks lineage annotations and gene expression memory is too weak for
  GEMLI to predict lineages, e.g. populations containing only distantly related cells
- The biological question is long-term lineage tracing over weeks or months, which
  is beyond the ~1–5 cell divisions scope of gene-expression-memory-based inference
- The input is single-cell multi-omics or spatial data without a gene-by-cell expression
  matrix suitable for per-lineage CV2 aggregation
edam_operation: http://edamontology.org/operation_0314
edam_topics:
- http://edamontology.org/topic_0203
tools:
- name: GEMLI
  role: R package providing memory-gene calling and lineage prediction used to obtain
    or supply the lineage groupings over which CV2 is computed
  repo: https://github.com/UPSUTER/GEMLI
- name: R
  role: Environment in which GEMLI is installed and the per-lineage aggregation, CV2,
    permutation testing, skewness classification, and topGO enrichment are executed
  repo: https://github.com/UPSUTER/GEMLI
- name: devtools
  role: Used to install the GEMLI package from GitHub
  repo: https://github.com/UPSUTER/GEMLI
- name: topGO
  role: Bioconductor package for GO biological process over-representation analysis
    on the quantitative/qualitative memory gene sets
provenance:
  source_task_ids:
  - task_006
  - task_003
  source_papers:
  - doi: 10.1038/s41467-024-47158-y
    title: Gene-expression memory-based prediction of cell lineages from scRNA-seq
      datasets
  source_content_hash: sha256:a2bd8d1a22ef39df990a5283e18038be31a6aabee4ce354df10a9b88448d3257
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/UPSUTER/GEMLI
  tools:
  - GEMLI
  - R
  - devtools
  - topGO
  edam_operation: http://edamontology.org/operation_0314
  edam_topics:
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.875
n_claims: 8
---

# memory-gene-identification-from-lineages

## Summary

Identify genes with gene expression memory — genes whose expression is stably inherited across cell divisions — by comparing expression variability within barcode-annotated (or GEMLI-predicted) lineages against lineage-size-matched random groupings, then classify memory genes into quantitative and qualitative categories.

## When to use

Use when you have a quality-controlled scRNA-seq expression matrix with cell-to-lineage annotations (cellular barcodes or GEMLI-predicted lineages) and want to enumerate and categorize heritably expressed genes.

## When NOT to use

- The dataset lacks lineage annotations and gene expression memory is too weak for GEMLI to predict lineages, e.g. populations containing only distantly related cells
- The biological question is long-term lineage tracing over weeks or months, which is beyond the ~1–5 cell divisions scope of gene-expression-memory-based inference
- The input is single-cell multi-omics or spatial data without a gene-by-cell expression matrix suitable for per-lineage CV2 aggregation

## Inputs

- Normalized, quality-controlled scRNA-seq gene expression matrix (genes x cells)
- Barcode-based lineage annotations mapping cell IDs to barcode IDs (or a GEMLI cell-by-cell lineage confidence matrix converted to lineage assignments)
- Optional: GO term definitions for downstream enrichment of memory gene categories (e.g. via Bioconductor topGO)

## Outputs

- Gene-level table with CV2, permutation p-value, skewness, and memory class columns (e.g. memory_genes_MEF.csv)
- Counts of total, quantitative, and qualitative memory genes
- Optionally: GO biological process over-representation scores per memory gene set and Spearman correlation of enrichment scores between lineage sources (e.g. correlation_results.csv)

## How to apply

Load the normalized gene expression matrix (rows = genes, columns = cells) together with barcode lineage annotations mapping cell IDs to barcode IDs, e.g. the MEF dataset from GEO accession GSE99915. Aggregate expression per barcoded lineage and compute, for every gene, the coefficient of variation squared (CV2) of lineage-mean expression across lineages. Build a per-gene null CV2 distribution using random lineage-size-matched cell groupings and call memory genes whose lineage-mean CV2 is significantly higher than random (p ≤ 0.05); the rationale is that truly heritable genes vary far more between lineages than expected by chance grouping. Classify significant memory genes by expression-distribution skewness: skewness < 3 assigns the quantitative class (widely/highly expressed, conserved within lineages) and skewness ≥ 3 assigns the qualitative class (highly variable, expressed in few lineages). Memory gene sets derived from GEMLI-predicted lineages can substitute barcode ground truth, since GO-term enrichment profiles of the two sets are highly correlated (Spearman correlation > 0.7, Fig. 3a).

## Related tools

- **GEMLI** (R package providing memory-gene calling and lineage prediction used to obtain or supply the lineage groupings over which CV2 is computed) — https://github.com/UPSUTER/GEMLI
- **R** (Environment in which GEMLI is installed and the per-lineage aggregation, CV2, permutation testing, skewness classification, and topGO enrichment are executed) — https://github.com/UPSUTER/GEMLI
- **devtools** (Used to install the GEMLI package from GitHub) — https://github.com/UPSUTER/GEMLI
- **topGO** (Bioconductor package for GO biological process over-representation analysis on the quantitative/qualitative memory gene sets)

## Examples

```
library(devtools); install_github("UPSUTER/GEMLI", subdir="GEMLI_package_v0"); library(GEMLI)
```

## Evaluation signals

- Memory gene counts fall within the reported ranges: total 3668 ± 1131, quantitative 2286 ± 890, qualitative 1382 ± 413
- Every memory gene passes the permutation test with p ≤ 0.05 against lineage-size-matched random groupings
- Each called memory gene is assigned exactly one class via the skewness < 3 / ≥ 3 threshold, with no unclassified genes
- GO biological process enrichment scores from barcode-derived and GEMLI-predicted-lineage memory gene sets show Spearman correlation > 0.7
- Output table contains the required columns (CV2, p-value, skewness, class) with one row per expressed gene

## Limitations

- GEMLI performs less robustly in cell populations that contain only distantly related cells, so predicted-lineage-based memory gene calls degrade in such datasets
- Large variability in growth rates between lineages may decrease GEMLI's performance
- Gene expression memory-based lineage inference is limited to lineages over about 1–5 cell divisions; longer time spans require other lineage-tracing approaches

## Evidence

- [readme] The approach is based on findings of Phillips et al. 2019 (doi.org/10.1038/s41467-019-09189-8) where it was shown that some genes show varying gene expression across cell lineages that is stable over multiple cell generations.: "some genes show varying gene expression across cell lineages that is stable over multiple cell generations"
- [methods] Simply run `library(devtools)` and then `install_github("UPSUTER/GEMLI", subdir="GEMLI_package_v0")`: "library(devtools)` and then `install_github("UPSUTER/GEMLI", subdir="GEMLI_package_v0")"
- [intro] Here, we quantitatively dissect the maintenance of gene expression in cell lineages for a broad range of cell types.: "Here, we quantitatively dissect the maintenance of gene expression in cell lineages for a broad range of cell types."
- [discussion] However, GEMLI will generally perform less robustly in cell populations that contain only distantly related cells.: "GEMLI will generally perform less robustly in cell populations that contain only distantly related cells."
- [discussion] we cannot exclude that large variability in growth rates between lineages may decrease its performance.: "we cannot exclude that large variability in growth rates between lineages may decrease its performance."
