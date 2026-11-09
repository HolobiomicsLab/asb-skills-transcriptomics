---
name: wilcoxon-rank-sum-testing
description: Use when comparing two independent score or expression distributions
  without distributional assumptions — e.g., testing whether 800-gene-set AUROCs exceed
  100-gene-set AUROCs, or identifying marker genes by comparing subtype cells against
  all other interneurons within one study.
when_to_use_negative:
- The comparison involves more than two independent groups (e.g., testing AUROC across
  >2 gene-set size bins), where a Kruskal-Wallis test or analogous multi-group method
  is required instead.
- Subtype assignments are not yet validated across datasets (no reciprocal top hits
  with AUROC >= 0.9 from MetaNeighborUS), so the subtype-vs-rest contrast has no replicated
  cell identity to test against.
- Count-level modeling with library-size and dispersion control is required (e.g.,
  raw UMI counts requiring a negative-binomial framework) rather than rank-based testing
  on RPKM values.
tools:
- name: MetaNeighbor
  role: Generates the per-gene-set AUROC score matrices that are compared with the
    Wilcoxon rank-sum test; its MetaNeighborUS reciprocal-top-hit outputs define the
    replicated subtype labels used in per-study differential expression tests
  repo: https://github.com/gillislab/MetaNeighbor
- name: R
  role: Runtime environment in which wilcox.test rank-sum testing, Fisher's method
    combination, and BH FDR correction are executed
provenance:
  source_task_ids:
  - task_003
  - task_007
  source_papers:
  - doi: 10.1038/s41467-018-03282-0
    title: Characterizing the replicability of cell types defined by single cell RNA-sequencing
      data using MetaNeighbor
  source_content_hash: sha256:1d66c975165f26299064f15c790bee7be67752fa5e603382dd28cd6d5bfe3541
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/gillislab/MetaNeighbor
  tools:
  - Matrix
  - MetaNeighbor
  - R
  - SummarizedExperiment
  - gplots
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 10
---

# wilcoxon-rank-sum-testing

## Summary

Non-parametric Wilcoxon rank-sum (Mann-Whitney) testing used in the MetaNeighbor replicability framework both to compare AUROC distributions (e.g., 100-gene vs 800-gene sets) and to run one-sided per-study differential expression tests that are combined across studies with Fisher's method to recover interneuron subtype marker genes.

## When to use

Use when comparing two independent score or expression distributions without distributional assumptions — e.g., testing whether 800-gene-set AUROCs exceed 100-gene-set AUROCs, or identifying marker genes by comparing subtype cells against all other interneurons within one study.

## When NOT to use

- The comparison involves more than two independent groups (e.g., testing AUROC across >2 gene-set size bins), where a Kruskal-Wallis test or analogous multi-group method is required instead.
- Subtype assignments are not yet validated across datasets (no reciprocal top hits with AUROC >= 0.9 from MetaNeighborUS), so the subtype-vs-rest contrast has no replicated cell identity to test against.
- Count-level modeling with library-size and dispersion control is required (e.g., raw UMI counts requiring a negative-binomial framework) rather than rank-based testing on RPKM values.

## Inputs

- Matrix of AUROC scores per gene set (e.g., 100-gene vs 800-gene random sets from supervised MetaNeighbor runs)
- RPKM expression matrix (gene-by-cell) for GSE60361 and GSE71585 cortical interneurons
- Cell type / subtype label assignments per cell (e.g., replicated subtype assignments from MetaNeighborUS reciprocal-top-hit analysis)

## Outputs

- Wilcoxon rank-sum P value and test statistic comparing AUROC distributions (expected P < 2.2E-16 for 100 vs 800 gene sets)
- Per-gene, per-study P values and log2 fold changes from one-sided subtype-vs-rest tests
- Fisher-method meta-analysis P values per gene per subtype with BH-adjusted FDR
- Ranked marker gene lists (e.g., marker_genes_fdr001.csv) and AUROC summary statistics (gene_set_size_auroc_results.csv)

## How to apply

For gene-set size effects, sample 100 random gene sets of 100 genes and 100 of 800 genes from expressed genes in mn_data (GOmouse/mn_data SummarizedExperiment), run supervised MetaNeighbor with leave-one-dataset-out cross-validation for each set, then test AUROC distributions with a two-sided or one-sided Wilcoxon rank-sum test (expected P < 2.2E-16, with mean AUROC ~0.75 ± 0.06 for 100-gene sets vs ~0.87 ± 0.02 for 800-gene sets). For marker discovery, run a one-sided Wilcoxon rank-sum test per replicated subtype and per source study (GSE60361, GSE71585), comparing subtype cells against all other interneurons on RPKM expression, to obtain per-study P values and log2 fold changes. Combine per-study P values with Fisher's method, apply Benjamini-Hochberg FDR correction, and retain genes passing FDR < 0.001 and log2FC > 2. Rationale: the rank-based test is robust to the skewed, bounded AUROC scale and to RPKM expression distributions, and its one-sided form directly encodes the enrichment hypothesis for a marker.

## Related tools

- **MetaNeighbor** (Generates the per-gene-set AUROC score matrices that are compared with the Wilcoxon rank-sum test; its MetaNeighborUS reciprocal-top-hit outputs define the replicated subtype labels used in per-study differential expression tests) — https://github.com/gillislab/MetaNeighbor
- **R** (Runtime environment in which wilcox.test rank-sum testing, Fisher's method combination, and BH FDR correction are executed)

## Examples

```
wilcox.test(auroc_800, auroc_100, alternative = "greater")
```

## Evaluation signals

- Wilcoxon comparison of 800-gene vs 100-gene AUROC distributions must yield P < 2.2E-16, with means ~0.87 ± 0.02 and ~0.75 ± 0.06 respectively.
- One-sided Wilcoxon DE tests must recover canonical Cre-driver markers Sst, Pvalb, Vip, Cck, and Htr3a in their expected subtype marker lists after Fisher combination and BH FDR < 0.001 with log2FC > 2.
- Ptn should appear as a top-ranked novel candidate enriched in the Sst Chodl / Som-Nos1 subtypes, confirming the marker pipeline's sensitivity to novel candidates.
- An uninformative control gene set of top-expression-centile genes should score near chance (AUROC ~0.47, below the 0.5 random-guessing baseline), confirming the test discriminates real signal from null sets.
- Spearman correlation between cross-validation performance bins and AUROC should be approximately 0.67, validating that ranked gene-set quality tracks rank-sum-tested performance.

## Limitations

- The rank-based test is not ideal for scaling in the R environment because of the ranking step, as sample counts grow (MetaNeighbor run times reach hours for 10,000 samples).
- An AUROC of 0.5 reflects chance-level performance, and moderate scores below the reciprocal-top-hit / AUROC > 0.95 threshold remain ambiguous, so Wilcoxon-supported marker lists for subtly different types require appropriate controls.
- Without field-wide data standards and cell-type labels in shared data, the subtype groups feeding per-study Wilcoxon contrasts may be inconsistently defined across studies.

## Evidence

- [abstract] Across tasks we find that large sets of variably expressed genes can identify replicable cell types with high accuracy: "Across tasks we find that large sets of variably expressed genes can identify replicable cell types with high accuracy"
- [intro] Meta-analysis of differential expression across replicable subtypes identifies markers: "Meta-analysis of differential expression across these highly replicable interneuron subtypes correctly identified canonical marker genes, as well as new candidates that may be used for improved"
- [readme] An AUROC score of 0.5 means that we have performed as well as if we had randomly guessed the cell's identity.: "An AUROC score of 0.5 means that we have performed as well as if we had randomly guessed the cell's identity."
- [intro] Cross-dataset validation hides labels for one dataset at a time: [evidence span withheld — anchor status: not_found]
- [discussion] While reciprocal top hits and AUROCs >0.95 reflect extreme confidence in a highly concordant cell type, more moderate scores are still meaningful: "While reciprocal top hits and AUROCs >0.95 reflect extreme confidence in a highly concordant cell type, more moderate scores are still meaningful"
