---
name: hierarchical-clustering-heatmap-visualization
description: Use when you have per-tissue significant sQTL tables (sqtls-*fdr.nominal.tsv)
  indexed by geneId-snpId variant-gene pairs and need to test whether sQTL effect
  sizes are shared across tissues and whether tissue groups (e.g. brain vs non-brain)
  cluster apart.
when_to_use_negative:
- The input is a single-tissue sQTL result set with no cross-tissue replicate tables
  — no tissue-pair correlation can be computed
- The goal is fine-mapping or causal co-localization of paired sQTL/eQTL effects —
  the article notes available methods are not applicable within their multivariate
  non-parametric framework and LD confounding cannot be resolved
- The question concerns individual splicing events rather than global changes in transcript
  isoform relative abundances, which is what MD effect sizes capture
edam_operation: http://edamontology.org/operation_0362
edam_topics:
- http://edamontology.org/topic_3053
- http://edamontology.org/topic_3308
tools:
- name: sQTLseekeR2
  role: R package underlying the sqtlseeker2-nf pipeline that produces the per-tissue
    significant sQTL tables and MD effect sizes correlated across tissues
  repo: https://github.com/guigolab/sQTLseekeR2
- name: sqtlseeker2-nf
  role: Nextflow pipeline (run with nextflow/Docker) that generates the per-tissue
    sQTL catalog (sqtls-*fdr.nominal.tsv) with --min_md minimum effect size filtering
  repo: https://github.com/guigolab/sqtlseeker2-nf
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1038/s41467-020-20578-2
    title: Identification and analysis of splicing quantitative trait loci across
      multiple tissues in the human genome
  source_content_hash: sha256:1ad72fa6351b407872ddede0c6b1b094194f02143d00e1afa42cc16bca183260
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/guigolab/sQTLseekeR2
  tools:
  - sQTLseekeR2
  - sqtlseeker2-nf
  edam_operation: http://edamontology.org/operation_0362
  edam_topics:
  - http://edamontology.org/topic_3053
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6
n_claims: 5
---

# hierarchical-clustering-heatmap-visualization

## Summary

Quantifies cross-tissue sharing of sQTLs by computing pairwise Pearson correlations of MD effect sizes over shared significant variant-gene pairs, then applies hierarchical clustering to the tissue-by-tissue correlation matrix and renders the clustering heatmap (Fig. 2d) that separates brain subregions from non-brain tissues.

## When to use

Use when you have per-tissue significant sQTL tables (sqtls-*fdr.nominal.tsv) indexed by geneId-snpId variant-gene pairs and need to test whether sQTL effect sizes are shared across tissues and whether tissue groups (e.g. brain vs non-brain) cluster apart.

## When NOT to use

- The input is a single-tissue sQTL result set with no cross-tissue replicate tables — no tissue-pair correlation can be computed
- The goal is fine-mapping or causal co-localization of paired sQTL/eQTL effects — the article notes available methods are not applicable within their multivariate non-parametric framework and LD confounding cannot be resolved
- The question concerns individual splicing events rather than global changes in transcript isoform relative abundances, which is what MD effect sizes capture

## Inputs

- Per-tissue significant sQTL result tables (sqtls-*fdr.nominal.tsv) from the sqtlseeker2-nf pipeline, indexed by geneId-snpId variant-gene pairs
- MD effect sizes (maximum difference in relative expression between genotype groups) for each significant sQTL
- GTEx v8 multi-tissue sQTL catalog

## Outputs

- Tissue-by-tissue Pearson correlation matrix of shared sQTL MD effect sizes
- Hierarchical clustering dendrogram / heatmap reproducing Fig. 2d (brain subregions vs all other tissues as two major clusters)
- Per-sQTL tissue-specificity values (st) contrasting brain vs non-brain specificity

## How to apply

Load the per-tissue significant sQTL result tables (sqtls-*fdr.nominal.tsv) from the chained sqtlseeker2-nf catalog and index them by geneId-snpId variant-gene pairs. For each tissue pair, retain variant-gene pairs that pass the FDR significance level (default 0.05) in both tissues and extract their MD effect sizes, the maximum difference in relative expression between genotype groups (the pipeline reports only effects above the --min_md default of 0.05). Compute pairwise Pearson correlation coefficients of MD effect sizes across all tissue pairs in R, then apply hierarchical clustering to the resulting tissue-by-tissue correlation matrix to obtain the two-cluster tissue structure. Render the clustering heatmap reproducing Fig. 2d and compute tissue-specificity values (st) for each sQTL to contrast brain vs non-brain specificity. Validate by expert review of the heatmap against the published Fig. 2d two-cluster (brain vs non-brain) sharing structure.

## Related tools

- **sQTLseekeR2** (R package underlying the sqtlseeker2-nf pipeline that produces the per-tissue significant sQTL tables and MD effect sizes correlated across tissues) — https://github.com/guigolab/sQTLseekeR2
- **sqtlseeker2-nf** (Nextflow pipeline (run with nextflow/Docker) that generates the per-tissue sQTL catalog (sqtls-*fdr.nominal.tsv) with --min_md minimum effect size filtering) — https://github.com/guigolab/sqtlseeker2-nf

## Examples

```
Rscript -e 'm <- read.delim("md_matrix.tsv"); c <- cor(m); hc <- hclust(as.dist(1-c)); heatmap(c, Rowv=as.dendrogram(hc), Colv=as.dendrogram(hc))'
```

## Evaluation signals

- Hierarchical clustering of the tissue-by-tissue correlation matrix recovers the two-cluster structure: brain subregions grouped together, separate from all other tissues
- Within-group mean correlation of MD effect sizes is high (reported ~0.80 within brain subregions and ~0.78 within the non-brain cluster) versus ~0.64 between groups
- Every tissue-pair correlation is computed only over variant-gene pairs passing FDR <= 0.05 in both tissues; shared-pair counts are non-empty for all pairs in the heatmap
- Computed tissue-specificity values (st) agree with the reported brain vs non-brain tissue-specificity pattern
- Rendered heatmap is validated by expert review against the published Fig. 2d

## Limitations

- LD confounding: the analysis cannot distinguish whether shared sQTL/eQTL effects are caused by the same variant or two different variants in LD
- A specific statistical test for tissue-specific heteropleiotropic effects is lacking; the authors note 'we lack a specific statistical test to assess whether a given variant has a regulatory effect on the expression of a gene only in one tissue'
- Reported effect sizes depend on the --min_md filter (default 0.05); the pipeline default performs no LD-based variant clustering (--ld 0), so correlated variants may inflate sharing estimates

## Evidence

- [full_text] Hierarchical clustering of sQTL sharing across GTEx tissues groups brain subregions together, separate from all other tissues, which form a second major cluster with high within-group sharing.: [evidence span withheld — anchor status: not_found]
- [methods] Pairwise correlation method on FDR-filtered shared pairs.: [evidence span withheld — anchor status: not_found]
- [methods] Effect size filter parameter of the pipeline.: "--min_md MIN_MD             minimum effect size reported (default: 0.05)"
- [abstract] Core cross-tissue sQTL sharing finding motivating the clustering analysis.: "We report that a core set of sQTLs is shared across multiple tissues."
- [full_text] Clustering and correlation computed in R on MD effect sizes.: [evidence span withheld — anchor status: not_found]
