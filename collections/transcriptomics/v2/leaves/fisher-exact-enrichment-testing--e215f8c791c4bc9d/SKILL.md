---
name: fisher-exact-enrichment-testing
description: Use when a fitted SSrGE model must be probed for subgroup-specific genotype
  markers — e.g., after separating normal vs. cancer cell indices, to find eeSNVs/genes
  significantly enriched in one subgroup (Fisher's exact test p < 0.05).
when_to_use_negative:
- You only need a global eeSNV/gene ranking across all cells — use ssrge.rank_eeSNVs()
  / ssrge.rank_genes() instead of subgroup-specific testing
- No cell subgroup labels or indices are available (e.g., unsupervised exploration
  without normal/cancer or subtype assignments)
- The features matrix is continuous gene expression rather than a binary/encoded SNV
  presence matrix — subgroup enrichment testing as described operates on eeSNV presence
  patterns
edam_operation: http://edamontology.org/operation_2238
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_2640
tools:
- name: SSrGE
  role: Provides rank_features_for_a_subgroup, which tests eeSNV/genes enrichment
    per subgroup with Fisher's exact test and returns ranked and significant features
  repo: https://github.com/lanagarmire/SSrGE.git
- name: Scipy
  role: Python dependency of SSrGE used for the statistical machinery underlying enrichment
    significance testing
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1038/s41467-018-07170-5
    title: Using single nucleotide variations in single-cell RNA-seq to identify subpopulations
      and genotype-phenotype linkage
  source_content_hash: sha256:ede8b9f61d4735de98cd945afc061f7d3e75eda5caef685aca38cf5175785a99
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
  - tabulate
  edam_operation: http://edamontology.org/operation_2238
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_2640
  provenance_tier: literature
license: CC-BY-4.0
merged_aliases:
- fishers-exact-enrichment-testing
triage_status: promoted
yes_rate: 0.625
n_claims: 8
---

# fisher-exact-enrichment-testing

## Summary

Identifies subgroup-specific eeSNVs and genes in single-cell datasets by testing, with Fisher's exact test, whether a given eeSNV (or gene harboring it) is significantly more present among cells of one subgroup (e.g., HCC cells vs. normal hepatocytes) than in the rest, yielding statistically significant features per subgroup.

## When to use

Use when a fitted SSrGE model must be probed for subgroup-specific genotype markers — e.g., after separating normal vs. cancer cell indices, to find eeSNVs/genes significantly enriched in one subgroup (Fisher's exact test p < 0.05).

## When NOT to use

- You only need a global eeSNV/gene ranking across all cells — use ssrge.rank_eeSNVs() / ssrge.rank_genes() instead of subgroup-specific testing
- No cell subgroup labels or indices are available (e.g., unsupervised exploration without normal/cancer or subtype assignments)
- The features matrix is continuous gene expression rather than a binary/encoded SNV presence matrix — subgroup enrichment testing as described operates on eeSNV presence patterns

## Inputs

- Fitted SSrGE instance (trained on SNV matrix X, gene expression matrix Y, optional CNV matrix C)
- Subgroup definition: list of cell indices (e.g., normal hepatocyte indices or HCC subtype indices)
- Per-cell SNV calls (snv_filtered.vcf per cell folder) and gene expression files (STAR matrix_counts.txt) underlying the matrices

## Outputs

- subgroup.ranked_genes — genes ranked for the subgroup
- subgroup.ranked_eeSNVs — eeSNVs ranked for the subgroup
- subgroup.significant_genes — genes with significant subgroup enrichment (Fisher's exact test p < 0.05, e.g., PRMT2, SULF2)
- subgroup.significant_eeSNVs — eeSNVs with significant subgroup enrichment
- Results table, e.g., hcc_subgroup_results.tsv

## How to apply

After fitting SSrGE on the SNV matrix (n_cells x n_SNVs, binary 0/1, or 0.5 for 0/1 and 1 for 1/1 genotypes) and the gene expression matrix (optionally with a CNV matrix), call ssrge.rank_features_for_a_subgroup() with the list of cell indices defining the subgroup (e.g., normal hepatocyte indices, or each HCC subtype from the Hou/GSE65364 dataset). A given eeSNV is considered specific to a subgroup if it is significantly more present among cells from that subgroup; the method returns subgroup.ranked_genes, subgroup.ranked_eeSNVs, subgroup.significant_genes, and subgroup.significant_eeSNVs, where significance is assessed with Fisher's exact test. Threshold calls at p < 0.05 to report drivers — in the HCC application, PRMT2 and SULF2 emerged as significantly mutated in HCC cells. Write all subgroup rankings and significance results to a table (e.g., hcc_subgroup_results.tsv) for validation against known subtype structure.

## Related tools

- **SSrGE** (Provides rank_features_for_a_subgroup, which tests eeSNV/genes enrichment per subgroup with Fisher's exact test and returns ranked and significant features) — https://github.com/lanagarmire/SSrGE.git
- **Scipy** (Python dependency of SSrGE used for the statistical machinery underlying enrichment significance testing)

## Examples

```
subgroup = ssrge.rank_features_for_a_subgroup(hcc_cell_indices); print subgroup.significant_genes; print subgroup.significant_eeSNVs
```

## Evaluation signals

- subgroup.significant_eeSNVs and subgroup.significant_genes are populated and calls satisfy Fisher's exact test p < 0.05
- Known HCC-associated genes PRMT2 and SULF2 appear as statistically significant mutations in HCC cells (Fisher's exact test, p < 0.05) on the Hou dataset
- Ranked features for the normal hepatocyte subgroup differ from those of each HCC subtype subgroup, consistent with eeSNV-space separation of normal vs. cancer cells
- Output table (e.g., hcc_subgroup_results.tsv) contains rankings and significance results for every queried subgroup without missing indices

## Limitations

- SSrGE may miss some driver mutations due to the incomplete DNA coverage of scRNA-seq reads
- In case of correlated features, LASSO may arbitrarily select one of several highly correlated SNV features corresponding to gene expression
- Fisher's exact test significance is evaluated per subgroup and requires a biologically meaningful subgroup definition; speculative eeSNV claims (e.g., HLA genes as cellular state fingerprints) are not experimentally verified

## Evidence

- [readme] Subgroup-specific eeSNV testing: "Extract specific eeSNVs and impacted genes of a given subgroup. a given eeSNV is specific to a subgroup if it is signficantly more present amongst the cells from the given subgroup"
- [readme] Subgroup ranking API: "subgroup = ssrge.rank_features_for_a_subgroup([0, 1, 2, 3, 4, 5])
print subgroup.significant_genes
print subgroup.significant_eeSNVs"
- [full_text] HCC significant drivers: [evidence span withheld — anchor status: not_found]
- [discussion] LASSO ranking rationale: "One can also easily rank the eeSNVs and the genes harboring them, for the purpose of identifying robust eeSNVs as genetic markers for a variety of cancers."
- [discussion] Driver-mutation coverage limitation: "Nevertheless, SSrGE may miss some driver mutations due to the incomplete DNA coverage of scRNA-seq reads."
