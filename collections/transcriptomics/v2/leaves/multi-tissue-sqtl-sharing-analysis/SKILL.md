---
name: multi-tissue-sqtl-sharing-analysis
description: Use when you have per-tissue significant sQTL result tables (e.g. sqtls-*fdr.nominal.tsv)
  from GTEx or another multi-tissue cohort and want to measure cross-tissue sQTL sharing
  and cluster tissues by splicing-effect similarity.
when_to_use_negative:
- You need to determine whether paired sQTL/eQTL effects come from the same variant
  or two variants in LD — the article explicitly states co-localization and fine-mapping
  methods are not applicable within this multivariate framework
- You lack per-tissue significant sQTL call sets with MD effect sizes and only have
  raw splicing phenotype matrices (map sQTLs first with sqtlseeker2-nf)
- You want a specific statistical test of tissue-specific heteropleiotropic effects
  — the authors note no such test exists
edam_operation: http://edamontology.org/operation_3432
edam_topics:
- http://edamontology.org/topic_3517
- http://edamontology.org/topic_0622
tools:
- name: sQTLseekeR2
  role: R package producing the per-tissue significant sQTL tables and MD effect sizes
    that feed the sharing correlation and clustering
  repo: https://github.com/guigolab/sQTLseekeR2
- name: Nextflow
  role: Workflow engine running sqtlseeker2-nf to generate the sQTL catalog used as
    input to this skill
  repo: https://github.com/guigolab/sqtlseeker2-nf
- name: R
  role: Used to compute pairwise Pearson correlation coefficients of MD effect sizes
    and hierarchical clustering of the tissue correlation matrix
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1038/s41467-020-20578-2
    title: Identification and analysis of splicing quantitative trait loci across
      multiple tissues in the human genome
  source_content_hash: sha256:3c493f4baa672a7e488ca7c90ec85e7a773ded42c23a5afdd8012f963a24613d
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/guigolab/sQTLseekeR2
  tools:
  - Nextflow
  - R
  - sQTLseekeR2
  edam_operation: http://edamontology.org/operation_3432
  edam_topics:
  - http://edamontology.org/topic_3517
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.875
n_claims: 8
---

# multi-tissue-sqtl-sharing-analysis

## Summary

Quantifies the extent to which sQTLs discovered by sQTLseekeR2 across GTEx tissues are shared, by correlating MD effect sizes of significant variant-gene pairs between tissue pairs and applying hierarchical clustering to the tissue-by-tissue correlation matrix. It reveals whether brain tissues form a distinct sQTL-sharing cluster from non-brain tissues.

## When to use

Use when you have per-tissue significant sQTL result tables (e.g. sqtls-*fdr.nominal.tsv) from GTEx or another multi-tissue cohort and want to measure cross-tissue sQTL sharing and cluster tissues by splicing-effect similarity.

## When NOT to use

- You need to determine whether paired sQTL/eQTL effects come from the same variant or two variants in LD — the article explicitly states co-localization and fine-mapping methods are not applicable within this multivariate framework
- You lack per-tissue significant sQTL call sets with MD effect sizes and only have raw splicing phenotype matrices (map sQTLs first with sqtlseeker2-nf)
- You want a specific statistical test of tissue-specific heteropleiotropic effects — the authors note no such test exists

## Inputs

- Per-tissue significant sQTL tables (sqtls-*fdr.nominal.tsv) from the sQTLseekeR2/sqtlseeker2-nf catalog
- MD effect sizes (maximum difference in relative expression between genotype groups) for significant variant-gene pairs
- GTEx v8 tissue labels distinguishing brain subregions from non-brain tissues

## Outputs

- Tissue-by-tissue Pearson correlation matrix of shared sQTL MD effect sizes
- Hierarchical clustering heatmap of tissue sQTL-sharing structure (reproducing Fig. 2d)
- Tissue-specificity values (st) per sQTL contrasting brain vs non-brain sharing
- Cluster assignments grouping brain subregions separately from all other tissues

## How to apply

Index the per-tissue significant sQTL tables (sqtls-*fdr.nominal.tsv) produced by sqtlseeker2-nf by geneId-snpId variant-gene pairs. For each tissue pair, retain variant-gene pairs significant at FDR 0.05 in both tissues and extract their MD effect sizes (maximum difference in relative expression between genotype groups, reported with --min_md default 0.05). Compute pairwise Pearson correlation coefficients of these MD effect sizes across all tissue pairs in R, then apply hierarchical clustering to the tissue-by-tissue correlation matrix to recover the two-cluster brain vs non-brain structure. Render the clustering heatmap (reproducing Fig. 2d) and compute tissue-specificity values (st) per sQTL to contrast brain vs non-brain specificity.

## Related tools

- **sQTLseekeR2** (R package producing the per-tissue significant sQTL tables and MD effect sizes that feed the sharing correlation and clustering) — https://github.com/guigolab/sQTLseekeR2
- **Nextflow** (Workflow engine running sqtlseeker2-nf to generate the sQTL catalog used as input to this skill) — https://github.com/guigolab/sqtlseeker2-nf
- **R** (Used to compute pairwise Pearson correlation coefficients of MD effect sizes and hierarchical clustering of the tissue correlation matrix)

## Examples

```
./nextflow run guigolab/sqtlseeker2-nf -with-docker
```

## Evaluation signals

- Hierarchical clustering of the correlation matrix reproduces the published two-cluster structure: brain subregions group together, all other tissues form a second major cluster (Fig. 2d comparison)
- Within-group mean Pearson r of MD effect sizes reaches ~0.80/0.78 (brain/other) while between-group sharing drops to ~0.64
- Only variant-gene pairs significant at FDR 0.05 in both tissues of a pair are included in each pairwise correlation
- Computed tissue-specificity values (st) agree with the reported brain vs non-brain specificity pattern on expert review

## Limitations

- Cannot distinguish whether paired sQTL effects are caused by the same variant or by two different variants in LD
- No specific statistical test exists to assess whether a variant has a regulatory effect in only one tissue (tissue-specific heteropleiotropy)
- Co-localization and fine-mapping methods are not directly applicable within this multivariate, non-parametric framework

## Evidence

- [results] Hierarchical clustering of sQTL sharing across GTEx tissues groups brain subregions together, separate from all other tissues, which form a second major cluster: [evidence span withheld — anchor status: not_found]
- [methods] For each tissue pair, retain variant-gene pairs that pass the FDR significance level (default 0.05) in both tissues and extract their MD effect sizes: [evidence span withheld — anchor status: not_found]
- [methods] Compute pairwise Pearson correlation coefficients of MD effect sizes across all tissue pairs in R: [evidence span withheld — anchor status: not_found]
- [abstract] We report that a core set of sQTLs is shared across multiple tissues.: "We report that a core set of sQTLs is shared across multiple tissues."
- [readme] A pipeline for splicing quantitative trait loci (sQTL) mapping.: "A pipeline for splicing quantitative trait loci (sQTL) mapping."
