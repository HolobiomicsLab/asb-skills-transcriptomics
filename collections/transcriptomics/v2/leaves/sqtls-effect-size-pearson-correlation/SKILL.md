---
name: sqtls-effect-size-pearson-correlation
description: Use when you have per-tissue significant sQTL catalogs (e.g. sqtls-*fdr.nominal.tsv
  from sqtlseeker2-nf) and need to measure to what extent sQTLs are shared across
  GTEx tissues, or whether a tissue group such as brain has a distinct sharing pattern.
when_to_use_negative:
- You only have eQTL (expression) effect sizes rather than sQTL splicing MD effect
  sizes — this measures splicing-QTL sharing, not eQTL sharing
- Variant-gene pairs are significant in only one of the two tissues being compared
  — correlations must be computed on pairs significant in both tissues
- You need formal co-localization or fine-mapping of causal variants; the article
  notes such methods are not directly applicable within this multivariate, non-parametric
  framework and paired effects could be two variants in LD
tools:
- name: sQTLseekeR2
  role: R package whose MD effect size statistic for significant sQTLs is the quantity
    correlated across tissue pairs
  repo: https://github.com/guigolab/sQTLseekeR2
- name: Nextflow
  role: Workflow engine running sqtlseeker2-nf, which produces the per-tissue significant
    sQTL tables used as input
  repo: https://github.com/guigolab/sqtlseeker2-nf
- name: R
  role: Used to compute pairwise Pearson correlations of MD effect sizes and hierarchical
    clustering of the tissue correlation matrix
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1038/s41467-020-20578-2
    title: Identification and analysis of splicing quantitative trait loci across
      multiple tissues in the human genome
  source_content_hash: sha256:4ee0bfb0f480ffeaa950d61afb7b9397495af50e9d42b9abc3ea9146e4ff255c
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/guigolab/sQTLseekeR2
  tools:
  - Nextflow
  - R
  - sQTLseekeR2
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7142857142857143
n_claims: 7
---

# sqtls-effect-size-pearson-correlation

## Summary

Quantify cross-tissue sharing of sQTLs by computing pairwise Pearson correlations of MD (maximum difference in relative expression) effect sizes over variant-gene pairs significant in both tissues of each pair, then cluster tissues by the resulting correlation matrix. This reproduces the Garrido-Martín et al. (2021) finding that brain subregions form a distinct sharing cluster (within-group mean r ~0.80/0.78 vs ~0.64 between groups).

## When to use

Use when you have per-tissue significant sQTL catalogs (e.g. sqtls-*fdr.nominal.tsv from sqtlseeker2-nf) and need to measure to what extent sQTLs are shared across GTEx tissues, or whether a tissue group such as brain has a distinct sharing pattern.

## When NOT to use

- You only have eQTL (expression) effect sizes rather than sQTL splicing MD effect sizes — this measures splicing-QTL sharing, not eQTL sharing
- Variant-gene pairs are significant in only one of the two tissues being compared — correlations must be computed on pairs significant in both tissues
- You need formal co-localization or fine-mapping of causal variants; the article notes such methods are not directly applicable within this multivariate, non-parametric framework and paired effects could be two variants in LD

## Inputs

- Per-tissue significant sQTL tables (sqtls-*fdr.nominal.tsv) indexed by geneId-snpId variant-gene pairs, from the GTEx v8 sqtlseeker2-nf catalog
- MD effect sizes (maximum difference in relative expression between genotype groups) per significant sQTL

## Outputs

- Tissue-by-tissue Pearson correlation matrix of MD effect sizes
- Hierarchical clustering dendrogram / heatmap of tissue sQTL sharing (reproducing Fig. 2d)
- Per-sQTL tissue-specificity values (st) contrasting brain vs non-brain sharing

## How to apply

Load the per-tissue significant sQTL result tables produced by sqtlseeker2-nf (sqtls-*fdr.nominal.tsv) and index them by geneId-snpId variant-gene pairs. For each tissue pair, retain variant-gene pairs significant in both tissues (FDR < 0.05) and extract their MD effect sizes — the maximum difference in relative expression of transcript isoforms between genotype groups, reported by sQTLseekeR2 with a default minimum of 0.05. Compute pairwise Pearson correlation coefficients of MD effect sizes across all tissue pairs in R. Apply hierarchical clustering to the resulting tissue-by-tissue correlation matrix to obtain the two-cluster structure (brain subregions vs non-brain tissues), render the heatmap corresponding to Fig. 2d, and compute tissue-specificity values (st) per sQTL to contrast brain vs non-brain specificity.

## Related tools

- **sQTLseekeR2** (R package whose MD effect size statistic for significant sQTLs is the quantity correlated across tissue pairs) — https://github.com/guigolab/sQTLseekeR2
- **Nextflow** (Workflow engine running sqtlseeker2-nf, which produces the per-tissue significant sQTL tables used as input) — https://github.com/guigolab/sqtlseeker2-nf
- **R** (Used to compute pairwise Pearson correlations of MD effect sizes and hierarchical clustering of the tissue correlation matrix)

## Examples

```
Rscript -e 'md <- merge(t1[,c("snpId","geneId","md")], t2[,c("snpId","geneId","md")], by=c("snpId","geneId")); cor.test(md$md.x, md$md.y, method="pearson")'
```

## Evaluation signals

- Hierarchical clustering of the correlation matrix recovers the published two-cluster structure: brain subregions grouped together, separate from a second cluster containing all non-brain tissues
- Within-group mean Pearson r values match the reported pattern: ~0.80 within brain, ~0.78 within the non-brain cluster, ~0.64 between clusters
- Only variant-gene pairs passing the FDR significance level (0.05) in both tissues are included in each pairwise comparison
- Computed tissue-specificity (st) values agree with the reported brain vs non-brain specificity pattern on expert review against Fig. 2d

## Limitations

- Correlation of MD effect sizes measures sharing of effect direction/magnitude, not identity of causal variants; same-variant vs LD-variant contributions cannot be distinguished
- sQTL/eQTL comparison conclusions can be confounded by different preprocessing and statistical methodologies across analyses
- Results depend on the sqtlseeker2-nf nominal pass output and its --min_md effect-size threshold (default 0.05)

## Evidence

- [abstract] Core shared sQTLs: "We report that a core set of sQTLs is shared across multiple tissues."
- [methods] MD effect size threshold: "--min_md MIN_MD             minimum effect size reported (default: 0.05)"
- [readme] Pipeline provenance: "Test for association between splicing ratios and genetic variants in *cis* (nominal pass)"
