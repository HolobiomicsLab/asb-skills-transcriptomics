---
name: cross-tissue-sqtl-effect-size-correlation-analysis
description: Use when you have per-tissue significant sQTL results (e.g. GTEx sqtls-*fdr.nominal.tsv
  tables) and need to quantify, cluster, or visualize cross-tissue sQTL sharing, especially
  to test whether brain tissues share sQTLs differently from non-brain tissues.
when_to_use_negative:
- You only have eQTL (expression) results rather than splicing QTL results with MD
  effect sizes — this skill's correlations are defined over splicing effect sizes
- You need causal colocalization between paired sQTL/eQTL effects — the paper notes
  same-variant vs LD-different-variant cases cannot be distinguished here
- Your sQTL tables have not been filtered to a common significance level in both tissues
  of each pair (default FDR 0.05), since correlations over non-shared pairs are not
  comparable
edam_operation: http://edamontology.org/operation_3432
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: sQTLseekeR2
  role: Produces the significant sQTL calls and the MD effect sizes (maximum difference
    in relative expression between genotype groups) that are correlated across tissue
    pairs
  repo: https://github.com/guigolab/sQTLseekeR2
- name: sqtlseeker2-nf
  role: Nextflow pipeline that generates the per-tissue significant sQTL tables used
    as inputs to the cross-tissue correlation and clustering analysis
  repo: https://github.com/guigolab/sqtlseeker2-nf
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1038/s41467-020-20578-2
    title: Identification and analysis of splicing quantitative trait loci across
      multiple tissues in the human genome
  source_content_hash: sha256:203b0836e6406c69233b8f6f3e710820871d1b74a245cbf78110578133a80d1f
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/guigolab/sQTLseekeR2
  tools:
  - sQTLseekeR2
  - sqtlseeker2-nf
  edam_operation: http://edamontology.org/operation_3432
  edam_topics:
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 8
---

# cross-tissue-sqtl-effect-size-correlation-analysis

## Summary

Quantifies splicing QTL sharing across GTEx tissues by correlating MD (maximum difference) effect sizes of variant-gene pairs significant in both tissues of each pair, then applies hierarchical clustering to the tissue-by-tissue correlation matrix. It reproduces the paper's finding that brain subregions form a distinct sQTL sharing cluster (Fig. 2d).

## When to use

Use when you have per-tissue significant sQTL results (e.g. GTEx sqtls-*fdr.nominal.tsv tables) and need to quantify, cluster, or visualize cross-tissue sQTL sharing, especially to test whether brain tissues share sQTLs differently from non-brain tissues.

## When NOT to use

- You only have eQTL (expression) results rather than splicing QTL results with MD effect sizes — this skill's correlations are defined over splicing effect sizes
- You need causal colocalization between paired sQTL/eQTL effects — the paper notes same-variant vs LD-different-variant cases cannot be distinguished here
- Your sQTL tables have not been filtered to a common significance level in both tissues of each pair (default FDR 0.05), since correlations over non-shared pairs are not comparable

## Inputs

- Per-tissue significant sQTL result tables (sqtls-*fdr.nominal.tsv) indexed by geneId-snpId variant-gene pairs
- MD (maximum difference) effect sizes per significant sQTL in each tissue
- GTEx multi-tissue sQTL catalog from the sqtlseeker2-nf pipeline

## Outputs

- Tissue-by-tissue matrix of Pearson correlation coefficients of MD effect sizes
- Hierarchical clustering dendrogram / heatmap reproducing Fig. 2d (brain subregions vs all other tissues)
- Per-sQTL tissue-specificity values (st) contrasting brain vs non-brain

## How to apply

Load the per-tissue significant sQTL tables produced by sqtlseeker2-nf and index them by geneId-snpId variant-gene pairs. For each tissue pair, retain variant-gene pairs significant at FDR 0.05 in both tissues and extract their MD effect sizes (maximum difference in relative expression between genotype groups), the multivariate effect size reported by sQTLseekeR2. Compute pairwise Pearson correlation coefficients of MD effect sizes across all tissue pairs in R, then apply hierarchical clustering to the resulting tissue-by-tissue correlation matrix to recover the two-cluster structure. Compute tissue-specificity values (st) for each sQTL to contrast brain vs non-brain specificity. Validation is by expert review of the clustering heatmap against published Fig. 2d and agreement of st values with the reported specificity pattern.

## Related tools

- **sQTLseekeR2** (Produces the significant sQTL calls and the MD effect sizes (maximum difference in relative expression between genotype groups) that are correlated across tissue pairs) — https://github.com/guigolab/sQTLseekeR2
- **sqtlseeker2-nf** (Nextflow pipeline that generates the per-tissue significant sQTL tables used as inputs to the cross-tissue correlation and clustering analysis) — https://github.com/guigolab/sqtlseeker2-nf

## Examples

```
./nextflow run guigolab/sqtlseeker2-nf -with-docker
```

## Evaluation signals

- Hierarchical clustering of the correlation matrix reproduces the published two-cluster structure: brain subregions group together, separate from a second cluster containing all non-brain tissues
- Mean within-group Pearson correlations match the reported values (approximately 0.80 within brain subregions, 0.78 within non-brain tissues) with a lower between-group mean (~0.64)
- Every variant-gene pair included in a tissue-pair correlation passes the FDR significance level (0.05) in both tissues
- Computed tissue-specificity (st) values agree with the reported brain vs non-brain specificity pattern
- Rendered heatmap is reviewed by an expert against Fig. 2d of the source paper

## Limitations

- The analysis is limited by the multivariate, non-parametric framework: standard colocalization and fine-mapping methods are not directly applicable, so sQTL sharing between tissues cannot be traced to the same causal variant
- Differences in sGene counts across tissues could be confounded by different preprocessing steps and statistical methodologies
- Tissue-specificity results should be treated with caution; the authors note analogous heteropleiotropy analyses are preliminary and further work is required to establish biological relevance

## Evidence

- [abstract] We report that a core set of sQTLs is shared across multiple tissues.: "We report that a core set of sQTLs is shared across multiple tissues."
- [discussion] Our implementation uses Nextflow and Docker: "our implementation of an enhanced pipeline for sQTL mapping based on sQTLseekeR2, Nextflow and Docker"
- [methods] sqtlseeker2-nf is the pipeline for splicing QTL mapping: "sqtlseeker2-nf ~ A pipeline for splicing QTL mapping"
- [readme] The pipeline tests associations between splicing ratios and variants in cis: "Test for association between splicing ratios and genetic variants in *cis* (nominal pass)"
- [methods] Min multivariate effect size (MD) threshold: "--min_md MIN_MD             minimum effect size reported (default: 0.05)"
