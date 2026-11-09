---
name: t-cell-subtype-concordance-assessment
description: Use when clustering scRNA-seq data from CITE-seq experiments where antibody-derived
  tags (ADTs) provide independent protein phenotypes such as CD4/CD8 that can be compared
  against mRNA-derived cluster labels.
when_to_use_negative:
- Data lack matched protein-level (CITE-seq ADT) measurements, so no independent CD4/CD8
  phenotype exists to check clusters against — rely on NMI/ARI against known labels
  instead
- Cell types of interest are not T cells (e.g. the human liver dataset in the paper)
  — the CD4/CD8 rules do not apply
- Protein measurement normalization has not been performed; raw ADT values cannot
  yield reliable subtype calls
edam_operation: http://edamontology.org/operation_3432
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: scDCC
  role: Source of the cluster labels being validated; constrained variant uses CD4/CD8
    subtype rules as pairwise constraints whose effect is checked
  repo: https://github.com/ttgump/scDCC
- name: Scanpy
  role: Preprocessing of the mRNA counts matrix and handling of the 10X CITE-seq PBMC
    data before clustering and concordance checks
- name: PyTorch
  role: Backend for training scDCC with must-link/cannot-link loss terms (ml_weight,
    cl_weight) whose clusters are assessed for CD4/CD8 concordance
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1038/s41467-021-22008-3
    title: Model-based deep embedding for constrained clustering analysis of single
      cell RNA-seq data
  source_content_hash: sha256:0cc157392993d2407fc06219e922fec7ed6217a04acad24bc936c68605084882
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/ttgump/scDCC
  tools:
  - Nvidia Tesla P100
  - PyTorch
  - Python
  - Scanpy
  - scDCC
  edam_operation: http://edamontology.org/operation_3432
  edam_topics:
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 6
---

# t-cell-subtype-concordance-assessment

## Summary

Validates single-cell clustering of PBMC mRNA data by cross-tabulating cluster labels against CITE-seq protein-level CD4/CD8 phenotypes, checking that constrained clustering (e.g. scDCC with protein-based pairwise constraints) produces T-cell clusters consistent with surface-marker biology.

## When to use

Use when clustering scRNA-seq data from CITE-seq experiments where antibody-derived tags (ADTs) provide independent protein phenotypes such as CD4/CD8 that can be compared against mRNA-derived cluster labels.

## When NOT to use

- Data lack matched protein-level (CITE-seq ADT) measurements, so no independent CD4/CD8 phenotype exists to check clusters against — rely on NMI/ARI against known labels instead
- Cell types of interest are not T cells (e.g. the human liver dataset in the paper) — the CD4/CD8 rules do not apply
- Protein measurement normalization has not been performed; raw ADT values cannot yield reliable subtype calls

## Inputs

- Cluster label assignments (cluster ID per cell) from mRNA-based clustering (scDCC or baselines)
- Normalized CITE-seq protein/antibody-derived tag measurements (CD4, CD8 levels) for the same cells
- Comparison configuration: unconstrained vs constrained (n_pairwise=25000) scDCC runs

## Outputs

- Cross-tabulation of cluster labels vs CD4/CD8 protein-level phenotypes per method
- Concordance comparison between unconstrained and constrained scDCC (cd4_cd8_concordance record)
- Validation verdict: whether CD4+/CD8- cells are mislabeled

## How to apply

After clustering the mRNA matrix (e.g. with scDCC trained on 10X CITE-seq PBMC data using scDCC_pairwise_CITE_PBMC.py), normalize the CITE-seq protein measurements and derive per-cell CD4/CD8 subtype phenotypes from the ADT levels. Cross-tabulate the cluster assignments against these protein-level phenotypes for both the constraint-free scDCC run and the constrained run (up to n_pairwise=25000 must-link/cannot-link pairs, including CD4/CD8 subtype rules, with ml_weight and cl_weight loss terms). The key check is that constrained scDCC must not mislabel CD4+/CD8- cells, which the constraint-free variant does; unconstrained clustering can yield clusters whose labels conflict with protein-defined T-cell subtypes. Record the concordance comparison (e.g. in cd4_cd8_concordance output) and use it alongside NMI/CA/ARI metrics from cite_pbmc_metrics_table.csv to judge biological interpretability of clusters.

## Related tools

- **scDCC** (Source of the cluster labels being validated; constrained variant uses CD4/CD8 subtype rules as pairwise constraints whose effect is checked) — https://github.com/ttgump/scDCC
- **Scanpy** (Preprocessing of the mRNA counts matrix and handling of the 10X CITE-seq PBMC data before clustering and concordance checks)
- **PyTorch** (Backend for training scDCC with must-link/cannot-link loss terms (ml_weight, cl_weight) whose clusters are assessed for CD4/CD8 concordance)

## Examples

```
python scDCC_pairwise_CITE_PBMC.py
```

## Evaluation signals

- Constrained scDCC must not mislabel CD4+/CD8- cells, unlike constraint-free scDCC
- Cross-tabulation must show CD4/CD8 protein levels consistent with cluster labels for the constrained run
- Constrained scDCC achieves higher NMI, clustering accuracy (CA) and ARI than SC3 (mRNA), PhenoGraph and k-means (protein) in cite_pbmc_metrics_table.csv
- Concordance table must be computed for both unconstrained and constrained scDCC so the improvement is directly attributable to the pairwise constraints

## Limitations

- Purely unsupervised clustering may not produce biologically interpretable clusters, so protein-concordance failures indicate the constraint-free method (not the biology) is at fault
- Constraint generation relies on percentile cutoffs (0.5th and 95th) of protein distances, so mislabeling can persist if confident pairs are scarce
- Discussion section content is absent from the provided document

## Evidence

- [methods] constrained scDCC must not mislabel CD4+/CD8- cells: [evidence span withheld — anchor status: not_found]
- [abstract] purely unsupervised clustering may not produce biologically interpretable clusters: "purely unsupervised clustering methods may not produce biologically interpretable clusters, which complicates cell type assignment"
- [methods] constraint construction uses percentile cutoffs and CD4/CD8 rules: [evidence span withheld — anchor status: not_found]
