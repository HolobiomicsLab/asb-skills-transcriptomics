---
name: random-cell-pair-sampling
description: Use when validating lineage-inference or gene-expression-memory claims
  by comparing transcriptome similarity of barcode-defined related cell pairs against
  an equal-sized set of randomly sampled unrelated pairs from the same scRNA-seq dataset.
when_to_use_negative:
- The population contains only distantly related cells, where GEMLI and memory-based
  similarity signals are known to perform less robustly
- The goal is de novo lineage prediction rather than validating a lineage-similarity
  signal; use the GEMLI predict_lineages / test_lineage_prediction workflow instead
- Expression values are raw counts rather than normalized, quality-controlled data,
  making whole-transcriptome Pearson correlations incomparable across cells
edam_operation: http://edamontology.org/operation_3463
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_0203
tools:
- name: GEMLI
  role: R package providing the GEMLI_items data structure ('gene_expression' and
    'barcodes' slots) used to organize expression matrix and barcode annotations for
    pair-wise correlation analysis
  repo: https://github.com/UPSUTER/GEMLI
- name: R
  role: Language in which the correlation distance computation, random pair sampling,
    and Mann-Whitney U test are implemented
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1038/s41467-024-47158-y
    title: Gene-expression memory-based prediction of cell lineages from scRNA-seq
      datasets
  source_content_hash: sha256:5c3b357e1118709a26fce7e9f3b16aabfd224af6353f73af996d88fb30a9b114
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/UPSUTER/GEMLI
  tools:
  - GEMLI
  - R
  edam_operation: http://edamontology.org/operation_3463
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7
n_claims: 10
---

# random-cell-pair-sampling

## Summary

Samples unrelated cell pairs (cells with different barcode IDs) from a normalized scRNA-seq expression matrix and computes their Pearson correlation distances as a negative-control distribution for testing whether lineage-related cells show gene expression memory.

## When to use

Use when validating lineage-inference or gene-expression-memory claims by comparing transcriptome similarity of barcode-defined related cell pairs against an equal-sized set of randomly sampled unrelated pairs from the same scRNA-seq dataset.

## When NOT to use

- The population contains only distantly related cells, where GEMLI and memory-based similarity signals are known to perform less robustly
- The goal is de novo lineage prediction rather than validating a lineage-similarity signal; use the GEMLI predict_lineages / test_lineage_prediction workflow instead
- Expression values are raw counts rather than normalized, quality-controlled data, making whole-transcriptome Pearson correlations incomparable across cells

## Inputs

- Normalized, quality-controlled gene expression matrix (cells x genes) from scRNA-seq data (e.g., GEO GSE74923 L1210 / CD8+ T-lymphocyte datasets)
- Barcode lineage annotations mapping cell IDs to barcode IDs (named vector with cell IDs as names and barcode IDs as values)
- GEMLI_items list with 'gene_expression' and 'barcodes' slots
- Count of related (within-lineage) cell pairs to match when sampling unrelated pairs

## Outputs

- Vector of Pearson correlation distances (1 - Pearson r) for randomly sampled unrelated cell pairs
- Mann-Whitney U rank-sum p-value comparing related vs. random pair distance distributions
- correlation_distance_comparison.csv with mean/median distances and p-value
- correlation_distance_comparison.png distribution plot

## How to apply

After enumerating all within-lineage (shared barcode ID) cell pairs from the lineage annotations, randomly sample an equal number of unrelated cell pairs (cells carrying different barcodes) from the same normalized, quality-controlled expression matrix. Compute Pearson correlation distances (1 - Pearson r) across the whole transcriptome for these random pairs, mirroring exactly the metric applied to the related pairs so the two distributions are directly comparable. Package the expression matrix and barcode annotations into a GEMLI_items list with 'gene_expression' and 'barcodes' slots (named vector mapping cell IDs to barcode IDs) when working in the GEMLI R framework. Compare the random-pair distance distribution against the related-pair distribution with a non-parametric rank-sum (Mann-Whitney U) test; the expectation from gene expression memory is that related cells show significantly LOWER correlation distance (higher transcriptome correlation) than random pairs. This was demonstrated on the L1210 and primary CD8+ T-lymphocyte datasets (GEO GSE74923), where related cells consistently showed higher whole-transcriptome correlation than randomly sampled cells.

## Related tools

- **GEMLI** (R package providing the GEMLI_items data structure ('gene_expression' and 'barcodes' slots) used to organize expression matrix and barcode annotations for pair-wise correlation analysis) — https://github.com/UPSUTER/GEMLI
- **R** (Language in which the correlation distance computation, random pair sampling, and Mann-Whitney U test are implemented)

## Examples

```
GEMLI_items <- list(gene_expression = expr_matrix, barcodes = barcode_map); rand.pairs <- sample.unrelated.pairs(barcodes = GEMLI_items$barcodes, n = length(related_pairs)); d <- 1 - cor(expr_matrix[, rand.pairs$cell1], expr_matrix[, rand.pairs$cell2], method = "pearson")
```

## Evaluation signals

- Random pair set contains only cells with different barcode IDs and its size exactly equals the number of related (within-lineage) pairs sampled
- Mean/median Pearson correlation distance of lineage-related pairs is lower than that of the random pairs (lower distance = higher transcriptome correlation)
- Mann-Whitney U rank-sum test yields p < 0.05, the pre-specified acceptance criterion for the correlation_distance_comparison
- The same distance metric (1 - Pearson r over the whole transcriptome) is applied identically to both related and random pair sets
- Outputs correlation_distance_comparison.csv and correlation_distance_comparison.png are produced with matching summary statistics

## Limitations

- The negative control only establishes a relative enrichment; if the dataset contains only distantly related cells, the signal between related and random pairs may be too weak to detect
- Large variability in growth rates between lineages may decrease performance of memory-based lineage analyses (cannot be excluded)
- Gene expression memory underpinning the expected difference persists over only about 1-5 cell divisions; random-pair comparisons over more distant lineages will not show the effect

## Evidence

- [intro] Here, we quantitatively dissect the maintenance of gene expression in cell lineages for a broad range of cell types.: "Here, we quantitatively dissect the maintenance of gene expression in cell lineages for a broad range of cell types."
- [readme] The approach is based on findings of Phillips et al. 2019 ... where it was shown that some genes show varying gene expression across cell lineages that is stable over multiple cell generations.: "it was shown that some genes show varying gene expression across cell lineages that is stable over multiple cell generations."
- [other] Randomly sample an equal number of unrelated cell pairs (different barcodes) and compute the same Pearson correlation distances as the negative control.: [evidence span withheld — anchor status: not_found]
- [discussion] GEMLI will generally perform less robustly in cell populations that contain only distantly related cells.: "GEMLI will generally perform less robustly in cell populations that contain only distantly related cells."
