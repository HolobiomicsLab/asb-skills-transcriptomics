---
name: benjamini-hochberg-multiple-testing-correction
description: Use when a gene-wise statistical test (e.g., limma moderated t-statistics
  across 46 cases and 23 controls) yields thousands of raw P-values that must be FDR-adjusted
  before ranking seeds or thresholding DEGs.
when_to_use_negative:
- Gene IDs have not yet been normalized/converted (e.g., Entrez to symbols via org.Hs.eg.db)
  — correct the P-values only after gene-level testing is finalized
- The analysis requires family-wise error rate control rather than FDR control (BH
  controls false discovery rate, not per-test or FWER error)
- P-values come from a small pre-specified hypothesis set (e.g., a handful of pathways)
  where no genome-wide multiple-testing burden exists
edam_operation: http://edamontology.org/operation_3661
edam_topics:
- http://edamontology.org/topic_3518
- http://edamontology.org/topic_0203
tools:
- name: limma
  role: Fits gene-wise linear models with empirical Bayes moderation and supplies
    the raw P-values that are then BH-adjusted across all genes
- name: R
  role: Runtime environment in which limma is applied and BH adjustment (method =
    'BH') is performed
- name: SEMgraph
  role: Consumes BH-adjusted gene P-values as seeds for active subnetwork detection
    and applies BH internally (e.g., SEMace with method = 'BH', alpha = 0.05)
  repo: https://github.com/fernandoPalluzzi/SEMgraph
- name: SEMdata
  role: Ships the benchmark COVID-19 RNA-seq dataset (GSE172114, 46 critical vs 23
    non-critical cases) whose DEG P-values are BH-adjusted
  repo: https://github.com/fernandoPalluzzi/SEMdata
- name: org.Hs.eg.db
  role: Gene ID conversion supporting consistent per-gene annotation of the adjusted
    P-value table
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1093/bioinformatics/btad377
    title: 'SEMtree: tree-based structure learning methods with structural equation
      models'
  source_content_hash: sha256:e8a3e4cebd80ae9c7fc1544d69879686aa023e37f4821fe4a192c51c6b292d70
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/fernandoPalluzzi/SEMgraph
  tools:
  - R
  - SEMdata
  - SEMgraph
  - SEMtree()
  - limma
  - org.Hs.eg.db
  edam_operation: http://edamontology.org/operation_3661
  edam_topics:
  - http://edamontology.org/topic_3518
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5454545454545454
n_claims: 11
---

# benjamini-hochberg-multiple-testing-correction

## Summary

Adjust gene-level differential expression P-values for multiple testing with Benjamini-Hochberg (BH) so that per-gene results (e.g., from limma empirical Bayes fits of GSE172114 COVID-19 critical vs non-critical RNA-seq) are controlled for false discovery and can serve as seed P-values for downstream active subnetwork detection and SEMtree analyses.

## When to use

Use when a gene-wise statistical test (e.g., limma moderated t-statistics across 46 cases and 23 controls) yields thousands of raw P-values that must be FDR-adjusted before ranking seeds or thresholding DEGs.

## When NOT to use

- Gene IDs have not yet been normalized/converted (e.g., Entrez to symbols via org.Hs.eg.db) — correct the P-values only after gene-level testing is finalized
- The analysis requires family-wise error rate control rather than FDR control (BH controls false discovery rate, not per-test or FWER error)
- P-values come from a small pre-specified hypothesis set (e.g., a handful of pathways) where no genome-wide multiple-testing burden exists

## Inputs

- Vector/matrix of raw per-gene P-values from limma empirical Bayes differential expression on normalized RNA-seq data (e.g., GSE172114, 46 critical vs 23 non-critical samples)
- Per-gene log2 fold changes and moderated t-statistics from the limma fit

## Outputs

- Per-gene DEG table with gene ID, log2FC, raw P-value, and BH-adjusted P-value (written to covid19_deg_table.csv)
- BH-adjusted P-values usable as seed gene P-values for active subnetwork detection methods

## How to apply

After fitting gene-wise linear models with limma on the normalized RNA-seq expression matrix and extracting moderated t-statistics, log2 fold changes, and raw P-values, apply BH adjustment (method = 'BH') across all genes simultaneously rather than per-pathway. Export the resulting per-gene table (gene ID, log2FC, P-value, adjusted P-value) to covid19_deg_table.csv; BH-adjusted P-values within [0,1] supply the seed gene P-values consumed by the five active subnetwork detection methods and SEMtree-based downstream analyses. The same correction is used within SEMgraph functions such as SEMace, where the BH method with alpha = 0.05 flags significant causal effects.

## Related tools

- **limma** (Fits gene-wise linear models with empirical Bayes moderation and supplies the raw P-values that are then BH-adjusted across all genes)
- **R** (Runtime environment in which limma is applied and BH adjustment (method = 'BH') is performed)
- **SEMgraph** (Consumes BH-adjusted gene P-values as seeds for active subnetwork detection and applies BH internally (e.g., SEMace with method = 'BH', alpha = 0.05)) — https://github.com/fernandoPalluzzi/SEMgraph
- **SEMdata** (Ships the benchmark COVID-19 RNA-seq dataset (GSE172114, 46 critical vs 23 non-critical cases) whose DEG P-values are BH-adjusted) — https://github.com/fernandoPalluzzi/SEMdata
- **org.Hs.eg.db** (Gene ID conversion supporting consistent per-gene annotation of the adjusted P-value table)

## Examples

```
adjP <- p.adjust(fit$p.value, method = 'BH'); deg <- data.frame(gene = rownames(fit), log2FC = fit$coefficients, P = fit$p.value, adjP = adjP); write.csv(deg, 'covid19_deg_table.csv')
```

## Evaluation signals

- The exported DEG table contains exactly one row per gene, with no missing gene IDs
- All BH-adjusted P-values fall within [0,1]
- Adjusted P-values are monotonically non-decreasing with the raw P-value ranking (BH preserves order of raw P-values)
- The adjusted table loads cleanly as the P-value/seed input for the active subnetwork detection step
- Significant calls (e.g., adjusted P < 0.05) form a subset of the raw P < 0.05 calls, never exceeding them

## Limitations

- BH adjustment is applied across all genes genome-wide, so power is reduced relative to testing a pre-filtered gene set; the article provides no discussion-section caveats (discussion text absent from the source document)
- Adjusted P-values depend on the total number of genes tested, so changes to the input gene universe change the adjustment
- BH controls the false discovery rate only; it does not guarantee any individual significant gene is a true positive

## Evidence

- [methods] Differential expression on the benchmark data was assessed with the empirical Bayes technique implemented in the limma R package: [evidence span withheld — anchor status: not_found]
- [other] Workflow step: apply BH across all genes: [evidence span withheld — anchor status: not_found]
- [abstract] Downstream consumption of adjusted P-values as seeds: "from a list of seed (i.e. disease) genes or gene P-values, the perturbed modules with undirected edges are generated with five state-of-the-art active subnetwork detection methods."
- [methods] BH used inside SEMgraph tools with alpha threshold: "ace <- SEMace(graph = alsData$graph, data = data.npn, type = "optimal", effect = "source2sink", method = "BH", alpha = 0.05)"
- [readme] Benchmark dataset description from README: [evidence span withheld — anchor status: external_doc]
