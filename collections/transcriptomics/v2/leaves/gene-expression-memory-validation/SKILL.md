---
name: gene-expression-memory-validation
description: Use when a normalized scRNA-seq matrix with barcode lineage annotations
  is available and you need to quantify whether lineage-related cells are transcriptomically
  more similar than random pairs before trusting lineage predictions.
when_to_use_negative:
- No independent barcode/lineage ground truth exists for the dataset, so related pairs
  cannot be enumerated
- The population contains only distantly related cells, where GEMLI's expression-memory
  signal is expected to be weak
- The expression matrix is raw counts rather than normalized, quality-controlled values,
  making Pearson correlation distances incomparable across cells
edam_operation: http://edamontology.org/operation_3891
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_0203
tools:
- name: GEMLI
  role: R package providing the GEMLI_items data structure and lineage-inference context
    in which the related-versus-random correlation-distance comparison is performed
  repo: https://github.com/UPSUTER/GEMLI
- name: R
  role: Implementation language for computing Pearson correlation distances, sampling
    unrelated pairs, and running the Mann-Whitney U test
  repo: https://github.com/UPSUTER/GEMLI
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1038/s41467-024-47158-y
    title: Gene-expression memory-based prediction of cell lineages from scRNA-seq
      datasets
  source_content_hash: sha256:6fb8457e498ae2aecb260c0098c3c35d624112af3fe631b5286276db54b4d36a
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/UPSUTER/GEMLI
  tools:
  - GEMLI
  - R
  edam_operation: http://edamontology.org/operation_3891
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.75
n_claims: 12
---

# gene-expression-memory-validation

## Summary

Validates gene expression memory in scRNA-seq lineage data by showing that cells sharing a barcode lineage (e.g., L1210 or primary CD8+ T-lymphocytes from GSE74923) have lower whole-transcriptome Pearson correlation distances than randomly sampled unrelated cell pairs. This is the null-model comparison that underpins GEMLI's lineage inference from scRNA-seq alone.

## When to use

Use when a normalized scRNA-seq matrix with barcode lineage annotations is available and you need to quantify whether lineage-related cells are transcriptomically more similar than random pairs before trusting lineage predictions.

## When NOT to use

- No independent barcode/lineage ground truth exists for the dataset, so related pairs cannot be enumerated
- The population contains only distantly related cells, where GEMLI's expression-memory signal is expected to be weak
- The expression matrix is raw counts rather than normalized, quality-controlled values, making Pearson correlation distances incomparable across cells

## Inputs

- Normalized, quality-controlled gene expression matrix (cells x genes) from scRNA-seq data
- Barcode lineage annotations mapping cell IDs to shared barcode IDs (e.g., GEO GSE74923 L1210 / primary CD8+ T-lymphocyte dataset)
- GEMLI_items list with 'gene_expression' and 'barcodes' slots

## Outputs

- correlation_distance_comparison.csv with mean/median correlation distances per group and Mann-Whitney U p-value
- correlation_distance_comparison.png plotting the related vs. random correlation distance distributions

## How to apply

Load the normalized, quality-controlled expression matrix and barcode lineage annotations (e.g., GEO GSE74923 for L1210/CD8+ T-lymphocytes) in R and assemble them into a GEMLI_items list with 'gene_expression' and 'barcodes' slots. Build a named vector mapping cell IDs (names) to barcode IDs (values) to enumerate all within-lineage (related) cell pairs. Compute Pearson correlation distance (1 - Pearson r) across the whole transcriptome for every related cell pair, then randomly sample an equal number of unrelated cell pairs (different barcodes) as a negative control and compute the same distances. Compare the two distributions with a non-parametric Mann-Whitney U rank-sum test; the rationale is that Phillips et al. showed expression of some genes is stable over multiple cell generations, so related cells should correlate more strongly than unrelated ones.

## Related tools

- **GEMLI** (R package providing the GEMLI_items data structure and lineage-inference context in which the related-versus-random correlation-distance comparison is performed) — https://github.com/UPSUTER/GEMLI
- **R** (Implementation language for computing Pearson correlation distances, sampling unrelated pairs, and running the Mann-Whitney U test) — https://github.com/UPSUTER/GEMLI

## Examples

```
library(GEMLI); dist_rel <- 1 - cor(expr[, cells1], expr[, cells2]); wilcox.test(dist_related, dist_random, alternative="less")
```

## Evaluation signals

- Lineage-related cell pairs show significantly lower correlation distance (higher Pearson r) than random unrelated pairs, with Mann-Whitney U p < 0.05
- The random control contains exactly the same number of pairs as the related set (equal-size sampling)
- Reported mean/median distances in correlation_distance_comparison.csv are finite and consistent with the plotted distributions
- The effect is reproducible across analyzed cell types (e.g., both L1210 and primary CD8+ T-lymphocytes)

## Limitations

- GEMLI performs less robustly in cell populations that contain only distantly related cells
- Large variability in growth rates between lineages may decrease performance (cannot be excluded)
- The gene-expression-memory signal covers only lineages over about 1-5 cell divisions; longer time spans require other lineage-tracing approaches

## Evidence

- [abstract] GEMLI robustly identifies small to medium-sized cell lineages solely from scRNA-seq datasets: "a computational tool allowing to robustly identify small to medium-sized cell lineages solely from scRNA-seq datasets"
- [methods] GEMLI_items structure and predict_lineages entry point: "We can then identify cell lineages through repeated iterative clustering (this may take 2-3min). The `predict_lineages` function takes our GEMLI_items as input"
- [methods] installation of the GEMLI R package: "Simply run `library(devtools)` and then `install_github("UPSUTER/GEMLI", subdir="GEMLI_package_v0")`"
- [discussion] limitations in populations with only distant relatives: "However, GEMLI will generally perform less robustly in cell populations that contain only distantly related cells."
- [readme] basis in stable gene expression over generations: "it was shown that some genes show varying gene expression across cell lineages that is stable over multiple cell generations."
