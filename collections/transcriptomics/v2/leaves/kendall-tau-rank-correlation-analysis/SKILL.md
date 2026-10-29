---
name: kendall-tau-rank-correlation-analysis
description: Use when two ranked feature lists (e.g., SSrGE gene rankings fitted with
  vs. without a CNV matrix) must be compared for ordering stability across cancer
  scRNA-seq datasets.
when_to_use_negative:
- When comparing unranked feature sets or continuous coefficient vectors — a correlation
  of raw values, not orderings, is needed
- When the two rankings contain different gene sets (Kendall-Tau requires concordant
  items in both ordered lists)
edam_operation: http://edamontology.org/operation_0564
edam_topics:
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_3523
tools:
- name: SSrGE
  role: Produces the with/without-CNV gene rankings via rank_genes() that are correlated
    with Kendall-Tau
  repo: https://github.com/lanagarmire/SSrGE.git
- name: Scipy
  role: Python library (SSrGE requirement) for computing Kendall-Tau rank correlation
- name: Python 2.7
  role: Runtime environment in which SSrGE ranking and correlation computations are
    executed
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1038/s41467-018-07170-5
    title: Using single nucleotide variations in single-cell RNA-seq to identify subpopulations
      and genotype-phenotype linkage
  source_content_hash: sha256:f48e4a88006a5224caccdf1973733bf2e94832f1ff0c7bfac108ee67e01c87a7
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/lanagarmire/SSrGE.git
  tools:
  - Numpy
  - Python
  - Python 2.7
  - SSrGE
  - Scipy
  - scikit-learn
  - tabulate
  edam_operation: http://edamontology.org/operation_0564
  edam_topics:
  - http://edamontology.org/topic_0203
  - http://edamontology.org/topic_3523
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5454545454545454
n_claims: 11
---

# kendall-tau-rank-correlation-analysis

## Summary

Compute Kendall-Tau rank correlation between two ordered gene (or eeSNV) rankings to quantify whether an added covariate — here an optional CNV matrix in SSrGE — significantly alters the accumulative-coefficient ranks.

## When to use

Use when two ranked feature lists (e.g., SSrGE gene rankings fitted with vs. without a CNV matrix) must be compared for ordering stability across cancer scRNA-seq datasets.

## When NOT to use

- When comparing unranked feature sets or continuous coefficient vectors — a correlation of raw values, not orderings, is needed
- When the two rankings contain different gene sets (Kendall-Tau requires concordant items in both ordered lists)

## Inputs

- Ranked gene list from SSrGE fit without CNVs (ssrge.rank_genes() output)
- Ranked gene list from SSrGE fit with CNV matrix passed to ssrge.fit(X, Y, C)
- Published Table 2 gene rank positions for flagged cancer genes (HLA-A, HLA-B, HLA-C, HLA-DRA, B2M, KRAS, AR, KLK3, EGFR)

## Outputs

- Kendall-Tau correlation coefficient between with-CNV and without-CNV gene rankings per dataset
- Associated p-value for the rank correlation
- Discrepancy flags recorded in table2_eesnv_gene_rankings.csv when reproduced ranks deviate from Table 2

## How to apply

After fitting SSrGE LASSO models on the SNV matrix (n_cells x n_SNVs) against the gene expression matrix for each dataset (Patel, Ting, Kim, Miyamoto), call ssrge.rank_genes() twice: once from the base fit (ssrge.fit(X, Y)) and once from the CNV-adjusted fit (ssrge.fit(X, Y, C) with a CNV matrix of shape n_cells x n_genes). Convert each ranking into ordinal positions of the same gene set (e.g., 4225 genes in Patel) and compute the Kendall-Tau correlation between the two ordered lists. A tau close to 1 with p = 0 indicates that adding CNVs as predictors does not significantly alter the top gene ranks, validating the ranking invariance. Report tau per dataset and flag any gene whose published Table 2 position (e.g., HLA-A 3rd in Kim, B2M 7th in Ting, EGFR 88th of 4225 in Patel) diverges in the reproduced ranking.

## Related tools

- **SSrGE** (Produces the with/without-CNV gene rankings via rank_genes() that are correlated with Kendall-Tau) — https://github.com/lanagarmire/SSrGE.git
- **Scipy** (Python library (SSrGE requirement) for computing Kendall-Tau rank correlation)
- **Python 2.7** (Runtime environment in which SSrGE ranking and correlation computations are executed)

## Examples

```
from scipy.stats import kendalltau; tau, p = kendalltau(rank_genes_without_cnv, rank_genes_with_cnv)
```

## Evaluation signals

- Kendall-Tau between gene rankings with and without CNVs is close to 1 with p = 0, confirming ranking stability
- Reproduced gene positions match published Table 2: HLA-A 3rd (Kim) and 28th (Patel), B2M 7th (Ting) and 45th (Patel), KRAS 13th (Kim), AR 6th and KLK3 19th (Miyamoto), EGFR 88th of 4225 (Patel)
- HLA genes (HLA-A, HLA-B, HLA-C, HLA-DRA) appear among top-ranked genes in all four human datasets
- Discrepancies are flagged in table2_eesnv_gene_rankings.csv rather than silently accepted

## Limitations

- LASSO may arbitrarily select one of highly correlated SNV features corresponding to GE, which can perturb ranks
- SSrGE may miss some driver mutations due to the incomplete DNA coverage of scRNA-seq reads
- Rank stability (tau ≈ 1) does not establish biological validity of individual gene positions

## Evidence

- [results] Adding CNVs does not significantly alter the top gene ranks (Kendall-Tau close to 1, p = 0).: [evidence span withheld — anchor status: not_found]
- [discussion] Ranking eeSNVs and genes via accumulative coefficients: "One can also easily rank the eeSNVs and the genes harboring them, for the purpose of identifying robust eeSNVs as genetic markers for a variety of cancers."
- [readme] SSrGE rank_genes usage from README: "ssrge.fit(X, Y)

print ssrge.rank_genes()"
- [readme] CNV matrix accepted by fit: "The fit method can take an additional CNV matrix of shape (n_cells x n_genes), and describing the CNV level for each gene."
