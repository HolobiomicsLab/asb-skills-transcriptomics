---
name: subgroup-eesnv-gene-ranking
description: Use when cell subgroups (e.g. normal vs cancer cells or known cancer
  subtypes) are predefined and you need eeSNV and gene rankings plus significance
  tests per subgroup from an already-fitted SSrGE model.
when_to_use_negative:
- Subgroup membership is not known a priori and must be discovered — use the global
  ssrge.rank_eeSNVs() ranking and clustering in eeSNV space first instead
- The SSrGE model has not yet been fitted on the SNV and gene expression matrices
  — subgroup ranking requires a fitted instance
- Data are bulk rather than single-cell SNV/expression matrices from the same cells,
  since subgroup ranking tests per-cell feature presence
edam_operation: http://edamontology.org/operation_3227
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_0199
tools:
- name: SSrGE
  role: Provides the rank_features_for_a_subgroup() API that ranks eeSNVs and genes
    for a given subgroup and returns significant_genes and significant_eeSNVs
  repo: https://github.com/lanagarmire/SSrGE.git
- name: Python 2.7
  role: Runtime required to instantiate and run the SSrGE package (pip2 install -r
    requirements.txt)
  repo: https://github.com/lanagarmire/SSrGE.git
- name: scikit-learn 0.18
  role: Backend for the sparse LASSO/ElasticNet models whose coefficients feed the
    accumulative eeSNV and gene rankings
- name: opoirion/ssrge
  role: Docker container producing the per-cell snv_filtered.vcf and STAR matrix_counts.txt
    files that build the subgroup-ranking input matrices
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1038/s41467-018-07170-5
    title: Using single nucleotide variations in single-cell RNA-seq to identify subpopulations
      and genotype-phenotype linkage
  source_content_hash: sha256:300b9678dfe01cfe5efe0f048868a4c4ef9977b0b1917e8365c4cdd599e44a5f
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
  - opoirion/ssrge
  - scikit-learn
  - scikit-learn 0.18
  - tabulate
  edam_operation: http://edamontology.org/operation_3227
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_0199
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5454545454545454
n_claims: 11
---

# subgroup-eesnv-gene-ranking

## Summary

SSrGE's subgroup analysis module ranks eeSNVs and genes for a defined cell subgroup (e.g. normal hepatocytes vs HCC subtypes), flagging features significantly enriched among subgroup cells, thereby linking genotype markers to phenotype subpopulations.

## When to use

Use when cell subgroups (e.g. normal vs cancer cells or known cancer subtypes) are predefined and you need eeSNV and gene rankings plus significance tests per subgroup from an already-fitted SSrGE model.

## When NOT to use

- Subgroup membership is not known a priori and must be discovered — use the global ssrge.rank_eeSNVs() ranking and clustering in eeSNV space first instead
- The SSrGE model has not yet been fitted on the SNV and gene expression matrices — subgroup ranking requires a fitted instance
- Data are bulk rather than single-cell SNV/expression matrices from the same cells, since subgroup ranking tests per-cell feature presence

## Inputs

- A fitted SSrGE instance (trained on binary SNV matrix X, gene expression matrix Y, optional CNV matrix C)
- List of cell indices defining each subgroup (e.g. normal hepatocytes, each HCC subtype)
- Optional SNV id list as (gene_id, SNV position) tuples and gene id list for named rankings

## Outputs

- subgroup.ranked_genes (genes ranked by their eeSNV association within the subgroup)
- subgroup.ranked_eeSNVs (eeSNVs ranked for subgroup specificity)
- subgroup.significant_genes (genes with significant subgroup association, Fisher's exact test)
- subgroup.significant_eeSNVs (eeSNVs significantly enriched in the subgroup, e.g. p < 0.05)
- Summary table of per-subgroup rankings and significance results (e.g. hcc_subgroup_results.tsv)

## How to apply

After fitting SSrGE on the binary SNV matrix (n_cells x n_SNVs, X_c,n=1 if present, optionally 0.5 for 0/1 and 1 for 1/1 genotypes) and the gene expression matrix with ssrge.fit(X, Y, C) (optionally with a CNV matrix), call ssrge.rank_features_for_a_subgroup() passing the list of cell indices belonging to the subgroup of interest (e.g. normal hepatocyte indices or one HCC subtype). A feature is deemed subgroup-specific if it is significantly more present among the subgroup's cells, assessed via Fisher's exact test (e.g. PRMT2 and SULF2 associations in HCC cells reaching p < 0.05). Retrieve subgroup.ranked_genes, subgroup.ranked_eeSNVs, subgroup.significant_genes, and subgroup.significant_eeSNVs, and repeat for each subgroup (normal, HCC subtype 1, HCC subtype 2). Write the per-subgroup rankings and significance results to an output table such as hcc_subgroup_results.tsv, then validate that eeSNVs separate the subgroups as expected.

## Related tools

- **SSrGE** (Provides the rank_features_for_a_subgroup() API that ranks eeSNVs and genes for a given subgroup and returns significant_genes and significant_eeSNVs) — https://github.com/lanagarmire/SSrGE.git
- **Python 2.7** (Runtime required to instantiate and run the SSrGE package (pip2 install -r requirements.txt)) — https://github.com/lanagarmire/SSrGE.git
- **scikit-learn 0.18** (Backend for the sparse LASSO/ElasticNet models whose coefficients feed the accumulative eeSNV and gene rankings)
- **opoirion/ssrge** (Docker container producing the per-cell snv_filtered.vcf and STAR matrix_counts.txt files that build the subgroup-ranking input matrices)

## Examples

```
from garmire_SSrGE.ssrge import SSrGE; ssrge.fit(X, Y, C); subgroup = ssrge.rank_features_for_a_subgroup([0, 1, 2, 3, 4, 5]); print subgroup.significant_eeSNVs
```

## Evaluation signals

- subgroup.significant_eeSNVs and subgroup.significant_genes are non-empty and associations reach Fisher's exact test p < 0.05 (e.g. PRMT2 and SULF2 in HCC cells)
- Ranked eeSNVs from the normal-hepatocyte subgroup differ from those of each HCC subtype subgroup, consistent with the separation seen in Fig. 7
- The top-ranked eeSNVs, when used alone (e.g. via nb_ranked_features), reproduce the ~92% eeSNV overlap of the simplified model
- Rankings are reproducible across repeated fits, and SNV ids resolve to the expected harboring genes when snv_id_list/gene_id_list are supplied
- Output table (e.g. hcc_subgroup_results.tsv) contains ranked_genes, ranked_eeSNVs, significant_genes, and significant_eeSNVs for every specified subgroup

## Limitations

- SSrGE may miss some driver mutations due to the incomplete DNA coverage of scRNA-seq reads
- In case of correlated features, LASSO may arbitrarily select only one of several highly correlated SNV features corresponding to a gene's expression
- A subgroup eeSNV is defined only by significant over-presence among subgroup cells; eeSNV-accuracy advantages and HLA-gene fingerprint claims remain speculative and not experimentally verified

## Evidence

- [methods] Subgroup analysis via rank_features_for_a_subgroup: "subgroup = ssrge.rank_features_for_a_subgroup([0, 1, 2, 3, 4, 5])"
- [readme] Definition of subgroup-specific eeSNV: "a given eeSNV is specific to a subgroup if it is signficantly more present amongst the cells from the given subgroup"
- [readme] Subgroup outputs available: "print subgroup.ranked_genes
print subgroup.ranked_eeSNVs

print subgroup.significant_genes
print subgroup.significant_eeSNVs"
- [discussion] Subgroup ranking purpose: "One can also easily rank the eeSNVs and the genes harboring them, for the purpose of identifying robust eeSNVs as genetic markers"
- [discussion] LASSO correlation caveat: "but in case of correlated features, LASSO may select one of those highly correlated SNV features that correspond to GE"
