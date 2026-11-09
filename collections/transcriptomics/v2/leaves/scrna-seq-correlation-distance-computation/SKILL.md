---
name: scrna-seq-correlation-distance-computation
description: Use when you have normalized scRNA-seq expression data with barcode lineage
  annotations (e.g., GEO GSE74923) and need to test whether related cells are transcriptionally
  more similar than random pairs.
when_to_use_negative:
- The cell population contains only distantly related cells, since GEMLI performs
  less robustly in such populations and correlation-distance contrasts may lack closely
  related pairs
- The dataset lacks barcode lineage annotations, in which case lineages must first
  be predicted (e.g., with GEMLI predict_lineages) rather than assumed
- Lineages span long time spans (weeks or months); memory-based inference covers only
  about 1-5 cell divisions
edam_operation: http://edamontology.org/operation_3463
edam_topics:
- http://edamontology.org/topic_0203
tools:
- name: GEMLI
  role: R package providing the GEMLI_items data structure used to organize the expression
    matrix and barcode annotations for lineage-related pair analysis
  repo: https://github.com/UPSUTER/GEMLI
- name: R
  role: Language used to compute Pearson correlation distances, sample unrelated pairs,
    and run the Mann-Whitney U rank-sum test
  repo: https://github.com/UPSUTER/GEMLI
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1038/s41467-024-47158-y
    title: Gene-expression memory-based prediction of cell lineages from scRNA-seq
      datasets
  source_content_hash: sha256:9863dc9b7815b715df93e8182e1eee8b506530357490a36b3d3a95826edba1ac
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/UPSUTER/GEMLI
  tools:
  - GEMLI
  - R
  edam_operation: http://edamontology.org/operation_3463
  edam_topics:
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.75
n_claims: 8
---

# scrna-seq-correlation-distance-computation

## Summary

Quantifies gene-expression memory by computing whole-transcriptome Pearson correlation distances (1 - Pearson r) between lineage-related cell pairs and comparing them to equal-sized samples of random unrelated pairs from normalized scRNA-seq data.

## When to use

Use when you have normalized scRNA-seq expression data with barcode lineage annotations (e.g., GEO GSE74923) and need to test whether related cells are transcriptionally more similar than random pairs.

## When NOT to use

- The cell population contains only distantly related cells, since GEMLI performs less robustly in such populations and correlation-distance contrasts may lack closely related pairs
- The dataset lacks barcode lineage annotations, in which case lineages must first be predicted (e.g., with GEMLI predict_lineages) rather than assumed
- Lineages span long time spans (weeks or months); memory-based inference covers only about 1-5 cell divisions

## Inputs

- Normalized, quality-controlled scRNA-seq gene expression matrix (cells x genes)
- Barcode lineage annotations (cell IDs mapped to barcode/lineage IDs, e.g., from GEO GSE74923 L1210 or CD8+ T-lymphocyte datasets)
- GEMLI_items list with 'gene_expression' and 'barcodes' slots

## Outputs

- correlation_distance_comparison.csv with mean/median correlation distances and Mann-Whitney U p-value
- correlation_distance_comparison.png showing the related vs. random pair distance distributions

## How to apply

Assemble the normalized, quality-controlled expression matrix and barcode lineage annotations into a GEMLI_items list with 'gene_expression' and 'barcodes' slots. Build a named vector mapping cell IDs to barcode IDs so that all within-lineage (related) cell pairs sharing a barcode can be enumerated. For every related pair, compute the Pearson correlation distance (1 - Pearson r) across the whole transcriptome, then randomly sample an equal number of unrelated pairs (different barcodes) and compute the same metric as the negative control. Compare the two distance distributions with a non-parametric rank-sum (Mann-Whitney U) test, reporting mean/median distances and the p-value. The rationale is that genes with stable expression across multiple cell generations (gene expression memory, as established by Phillips et al. 2019) make lineage-related cells more transcriptome-correlated than random pairs, forming the basis of GEMLI.

## Related tools

- **GEMLI** (R package providing the GEMLI_items data structure used to organize the expression matrix and barcode annotations for lineage-related pair analysis) — https://github.com/UPSUTER/GEMLI
- **R** (Language used to compute Pearson correlation distances, sample unrelated pairs, and run the Mann-Whitney U rank-sum test) — https://github.com/UPSUTER/GEMLI

## Examples

```
related_dist <- sapply(pairs, function(p) 1 - cor(expr[, p["cell1"]], expr[, p["cell2"]], method="pearson")); wilcox.test(related_dist, random_dist)
```

## Evaluation signals

- Lineage-related cells exhibit significantly lower correlation distance than random pairs (rank-sum test p < 0.05)
- The number of randomly sampled unrelated pairs equals the number of enumerated related pairs
- Mean and median correlation distances for the related group are smaller than for the random-pair control
- Output files correlation_distance_comparison.csv and correlation_distance_comparison.png are produced with the recorded statistics

## Limitations

- GEMLI performs less robustly in cell populations that contain only distantly related cells
- Large variability in growth rates between lineages may decrease performance
- Memory-based lineage scope is limited to lineages over about 1-5 cell divisions; longer-term lineage relationships require other approaches

## Evidence

- [readme] some genes show varying gene expression across cell lineages that is stable over multiple cell generations: "it was shown that some genes show varying gene expression across cell lineages that is stable over multiple cell generations"
- [abstract] GEMLI robustly identifies lineages from scRNA-seq: "a computational tool allowing to robustly identify small to medium-sized cell lineages solely from scRNA-seq datasets"
- [intro] quantitative dissection of gene expression maintenance in lineages: "Here, we quantitatively dissect the maintenance of gene expression in cell lineages for a broad range of cell types."
- [readme] GEMLI installation in R: "GEMLI is an R package to predict cell lineages (cells with a common ancestor) from single cell RNA sequencing datasets"
- [discussion] GEMLI less robust for distant relatives: "However, GEMLI will generally perform less robustly in cell populations that contain only distantly related cells."
