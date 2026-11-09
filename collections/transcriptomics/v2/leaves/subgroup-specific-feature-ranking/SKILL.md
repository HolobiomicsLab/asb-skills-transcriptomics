---
name: subgroup-specific-feature-ranking
description: Use when cells have been clustered or ordered into subgroups (e.g., PDX-pRCC
  vs Pt-mRCC vs PDX-mRCC branches of a Monocle-style MST) and you need the eeSNVs
  and impacted genes significantly enriched in each subgroup.
when_to_use_negative:
- The SNV and gene expression matrices have not been fit with SSrGE (no LASSO-inferred
  eeSNV-to-gene model exists yet)
- The grouping is not cell-based — rank_features_for_a_subgroup requires integer indices
  of cells, not a feature table or bulk samples
- Goal is global eeSNV ranking across all cells rather than subgroup-specific enrichment
  (use ssrge.rank_eeSNVs() / ssrge.rank_genes() instead)
edam_operation: http://edamontology.org/operation_3802
edam_topics:
- http://edamontology.org/topic_3173
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_3518
tools:
- name: SSrGE
  role: Python package providing rank_features_for_a_subgroup to identify eeSNVs and
    impacted genes significantly enriched in a cell subgroup after LASSO-based eeSNV-gene
    model fitting
  repo: https://github.com/lanagarmire/SSrGE.git
- name: scikit-learn
  role: Underlying library (version 0.18) for the LASSO/ElasticNet sparse linear models
    whose non-null coefficients define the eeSNV-gene links being ranked
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1038/s41467-018-07170-5
    title: Using single nucleotide variations in single-cell RNA-seq to identify subpopulations
      and genotype-phenotype linkage
  source_content_hash: sha256:6837837e5c3d1fcd70b3ee53773a053002f84cacd0fff38b9ea8e1c344fa33b0
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/lanagarmire/SSrGE.git
  tools:
  - Numpy
  - Python
  - SSrGE
  - Scipy
  - scikit-learn
  edam_operation: http://edamontology.org/operation_3802
  edam_topics:
  - http://edamontology.org/topic_3173
  - http://edamontology.org/topic_0203
  - http://edamontology.org/topic_3518
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 10
---

# subgroup-specific-feature-ranking

## Summary

Ranks eeSNVs and the genes harboring them for a defined subgroup of single cells using SSrGE's rank_features_for_a_subgroup, flagging features that are significantly enriched among the subgroup's cells. This links genotype markers (eeSNVs) to phenotype-defined subpopulations, e.g., ramification branches in the Kim dataset MST pseudo-time ordering.

## When to use

Use when cells have been clustered or ordered into subgroups (e.g., PDX-pRCC vs Pt-mRCC vs PDX-mRCC branches of a Monocle-style MST) and you need the eeSNVs and impacted genes significantly enriched in each subgroup.

## When NOT to use

- The SNV and gene expression matrices have not been fit with SSrGE (no LASSO-inferred eeSNV-to-gene model exists yet)
- The grouping is not cell-based — rank_features_for_a_subgroup requires integer indices of cells, not a feature table or bulk samples
- Goal is global eeSNV ranking across all cells rather than subgroup-specific enrichment (use ssrge.rank_eeSNVs() / ssrge.rank_genes() instead)

## Inputs

- Fitted SSrGE instance (ssrge.fit(X, Y) already run) on a binary SNV matrix (n_cells x n_SNVs) and gene expression matrix (n_cells x n_genes)
- List of integer cell indices defining the subgroup (e.g., cells in one MST branch of the Kim dataset)
- Optional: snv_id_list (tuples of gene id + SNV id, e.g. genome position) and gene_id_list matching matrix column order

## Outputs

- subgroup.ranked_genes — genes ranked for the subgroup
- subgroup.ranked_eeSNVs — eeSNVs ranked for the subgroup
- subgroup.significant_genes — genes significantly enriched in the subgroup
- subgroup.significant_eeSNVs — eeSNVs significantly more present among subgroup cells

## How to apply

After fitting SSrGE on the SNV matrix (n_cells x n_SNVs) and gene expression matrix (n_cells x n_genes) with ssrge.fit(X, Y), pass the integer indices of the cells belonging to a subgroup to ssrge.rank_features_for_a_subgroup([cell_indices]). A given eeSNV is specific to a subgroup if it is significantly more present among that subgroup's cells; the returned object exposes ranked_genes, ranked_eeSNVs, significant_genes, and significant_eeSNVs. To obtain interpretable gene names, instantiate SSrGE with snv_id_list (tuples of harboring-gene id and user-defined SNV id such as genome position) and gene_id_list whose order matches the gene matrix. In the Kim dataset workflow, annotate each MST ramification (PDX-pRCC, Pt-mRCC, PDX-mRCC) by ranking features for the cells in that branch, then use these subgroup-specific eeSNVs to characterize the impacted genes driving intra-group heterogeneity.

## Related tools

- **SSrGE** (Python package providing rank_features_for_a_subgroup to identify eeSNVs and impacted genes significantly enriched in a cell subgroup after LASSO-based eeSNV-gene model fitting) — https://github.com/lanagarmire/SSrGE.git
- **scikit-learn** (Underlying library (version 0.18) for the LASSO/ElasticNet sparse linear models whose non-null coefficients define the eeSNV-gene links being ranked)

## Examples

```
subgroup = ssrge.rank_features_for_a_subgroup([0, 1, 2, 3, 4, 5]); print(subgroup.significant_eeSNVs); print(subgroup.significant_genes)
```

## Evaluation signals

- subgroup.significant_eeSNVs must be significantly more present among the subgroup's cells than among remaining cells (README definition of subgroup specificity)
- subgroup.ranked_genes and subgroup.ranked_eeSNVs must be non-empty when the subgroup contains cells with non-null LASSO coefficients
- In the Kim dataset MST validation, subgroup-specific eeSNVs reported for a branch must be significantly enriched in that corresponding branch
- ranked genes should include previously validated cancer-relevant genes (the article reports such genes are highly ranked, confirming method significance)
- If snv_id_list/gene_id_list were supplied, gene ids in ranked_genes should match gene_id_list order and SNV ids should resolve to the harboring genes

## Limitations

- LASSO may arbitrarily select one of highly correlated SNV features corresponding to gene expression, so a correlated eeSNV may be absent from the ranking
- SSrGE may miss some driver mutations due to incomplete DNA coverage of scRNA-seq reads
- Specificity is defined statistically by presence enrichment; eeSNV accuracy advantages over gene expression are speculated rather than experimentally verified

## Evidence

- [readme] a given eeSNV is specific to a subgroup if it is signficantly more present amongst the cells from the given subgroup: "a given eeSNV is specific to a subgroup if it is signficantly more present amongst the cells from the given subgroup"
- [methods] rank_features_for_a_subgroup usage example: "subgroup = ssrge.rank_features_for_a_subgroup([0, 1, 2, 3, 4, 5])"
- [discussion] One can also easily rank the eeSNVs and the genes harboring them, for the purpose of identifying robust eeSNVs as genetic markers: "One can also easily rank the eeSNVs and the genes harboring them, for the purpose of identifying robust eeSNVs as genetic markers"
- [readme] a SNV id should be a tuple containing the gene id harboring the given SNV and a user defined SNV id (genome position for example): "a SNV id should be a tuple containing the gene id harboring the given SNV and a user defined SNV id (genome position for example)"
