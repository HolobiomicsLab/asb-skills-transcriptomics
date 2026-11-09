---
name: cnv-covariate-sensitivity-check
description: Use when SSrGE eeSNV/gene rankings from scRNA-seq (e.g., Patel, Ting,
  Kim, Miyamoto cancer datasets) need a sensitivity check against copy-number variation
  as an additional predictor.
when_to_use_negative:
- No CNV data are available for the same cells (the fit call requires a n_cells x
  n_genes CNV matrix matching the expression matrix)
- The goal is subgroup-specific feature ranking (rank_features_for_a_subgroup) rather
  than global ranking stability
edam_operation: http://edamontology.org/operation_2935
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_0625
tools:
- name: SSrGE
  role: Fits LASSO models on SNV and gene expression matrices, accepts an optional
    CNV matrix in fit(X, Y, C), and provides rank_eeSNVs()/rank_genes() for ranking
    comparisons
  repo: https://github.com/lanagarmire/SSrGE.git
- name: scikit-learn
  role: Underlying library (version 0.18 in the original study) providing the LASSO
    implementation used by SSrGE
  repo: http://scikit-learn.org/
- name: Python 2.7
  role: Runtime environment required by the SSrGE package
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1038/s41467-018-07170-5
    title: Using single nucleotide variations in single-cell RNA-seq to identify subpopulations
      and genotype-phenotype linkage
  source_content_hash: sha256:4110dd5486215841b85cab86081cd8b063a4f8e2260ab18c0442de7a629dbdc7
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
  edam_operation: http://edamontology.org/operation_2935
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_0625
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5714285714285714
n_claims: 7
---

# cnv-covariate-sensitivity-check

## Summary

Refit SSrGE LASSO models with a copy-number-variation (CNV) matrix passed as a covariate to the fit call, then compare eeSNV/gene rankings with the SNV-only models (Kendall-Tau close to 1) to test whether CNVs alter the genotype-phenotype linkage rankings.

## When to use

Use when SSrGE eeSNV/gene rankings from scRNA-seq (e.g., Patel, Ting, Kim, Miyamoto cancer datasets) need a sensitivity check against copy-number variation as an additional predictor.

## When NOT to use

- No CNV data are available for the same cells (the fit call requires a n_cells x n_genes CNV matrix matching the expression matrix)
- The goal is subgroup-specific feature ranking (rank_features_for_a_subgroup) rather than global ranking stability

## Inputs

- SNV matrix (n_cells x n_SNVs, binary, from scRNA-seq SNV calling or .vcf files)
- Gene expression matrix (n_cells x n_genes, float counts/TPM)
- CNV matrix (n_cells x n_genes, per-gene CNV level)
- snv_id_list (tuples of harboring gene id and user-defined SNV id, e.g., genomic position)
- gene_id_list matching the gene expression matrix columns

## Outputs

- CNV-adjusted eeSNV rankings (ssrge.rank_eeSNVs() output)
- CNV-adjusted gene rankings (ssrge.rank_genes() output)
- Kendall-Tau correlation between with-CNV and without-CNV gene rankings
- Comparison table (e.g., table2_eesnv_gene_rankings.csv) with any flagged rank discrepancies

## How to apply

For each cancer dataset, first fit the SSrGE LASSO models on the SNV matrix (n_cells x n_SNVs) and gene expression matrix (n_cells x n_genes) and record rank_eeSNVs()/rank_genes() output. Then refit with the optional CNV matrix (n_cells x n_genes, CNV level per gene) passed as a third argument to ssrge.fit(X, Y, C), keeping snv_id_list (tuples of harboring gene and genomic position) and gene_id_list fixed so ranks are comparable. Recompute rank_eeSNVs() and rank_genes() on the CNV-adjusted models. Compute the Kendall-Tau correlation between the with-CNV and without-CNV gene rankings; in the paper this was close to 1 with p = 0, indicating top gene ranks (e.g., HLA-A, HLA-B, HLA-C, HLA-DRA, B2M, KRAS, AR, KLK3, EGFR) are invariant to CNV adjustment.

## Related tools

- **SSrGE** (Fits LASSO models on SNV and gene expression matrices, accepts an optional CNV matrix in fit(X, Y, C), and provides rank_eeSNVs()/rank_genes() for ranking comparisons) — https://github.com/lanagarmire/SSrGE.git
- **scikit-learn** (Underlying library (version 0.18 in the original study) providing the LASSO implementation used by SSrGE) — http://scikit-learn.org/
- **Python 2.7** (Runtime environment required by the SSrGE package)

## Examples

```
ssrge = SSrGE(snv_id_list=snv_id_list, gene_id_list=gene_id_list); ssrge.fit(X, Y, C); ranks_cnv = ssrge.rank_genes()
```

## Evaluation signals

- Kendall-Tau correlation between gene rankings with and without the CNV matrix is close to 1 with p = 0
- Top-ranked genes remain HLA-A, HLA-B, HLA-C, HLA-DRA across the four human cancer datasets after CNV adjustment
- Flagged cancer-gene positions are preserved: B2M 7th (Ting) / 45th (Patel), KRAS 13th (Kim), AR 6th and KLK3 19th (Miyamoto), EGFR 88th of 4225 genes (Patel)
- Discrepancies versus published Table 2 positions are flagged in table2_eesnv_gene_rankings.csv rather than silently accepted

## Limitations

- SSrGE may miss some driver mutations due to incomplete DNA coverage of scRNA-seq reads
- LASSO may arbitrarily select one of highly correlated SNV features corresponding to gene expression, which can affect rank stability
- CNV adjustment 'does not significantly alter the top gene ranks' in the published datasets, so sensitivity signals may be dataset-dependent

## Evidence

- [readme] The fit method can take an additional CNV matrix of shape (n_cells x n_genes), and describing the CNV level for each gene.: "The fit method can take an additional CNV matrix of shape (n_cells x n_genes), and describing the CNV level for each gene."
- [other] adding CNVs does not significantly alter the top gene ranks (Kendall-Tau close to 1, p = 0): [evidence span withheld — anchor status: not_found]
- [methods] ranked_feature = ssrge.rank_eeSNVs(): "ranked_feature = ssrge.rank_eeSNVs()"
- [discussion] LASSO may select one of those highly correlated SNV features that correspond to GE: "but in case of correlated features, LASSO may select one of those highly correlated SNV features that correspond to GE"
