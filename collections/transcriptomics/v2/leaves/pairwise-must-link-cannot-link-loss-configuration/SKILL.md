---
name: pairwise-must-link-cannot-link-loss-configuration
description: Use when clustering scRNA-seq data where purely unsupervised clustering
  yields biologically uninterpretable clusters and partial prior knowledge (e.g.,
  protein-level ADT distances or known marker subtypes like CD4/CD8) is available
  for a subset of cell pairs.
when_to_use_negative:
- Prior knowledge is limited to marker genes and predefined cell types with no pairwise
  cell-pair information — marker-based methods such as CellAssign or SCINA are the
  article's cited alternative
- No prior information at all is available and the user only wants unsupervised clustering
  — use constraint-free scDCC or SC3 instead
- The user wants hard-constrained clustering that forbids violations of constraints
  (e.g., COP-K-means style) rather than soft loss-term penalties
edam_operation: http://edamontology.org/operation_3432
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_2269
tools:
- name: scDCC
  role: Implements the deep-embedding constrained clustering; must-link and cannot-link
    losses are added as weighted terms via scDCC_pairwise_CITE_PBMC.py
  repo: https://github.com/ttgump/scDCC
- name: PyTorch
  role: Deep learning framework used to train the scDCC embedding with the constraint
    loss terms
  repo: https://github.com/ttgump/scDCC
- name: Scanpy
  role: Preprocessing of the mRNA count matrix before embedding
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1038/s41467-021-22008-3
    title: Model-based deep embedding for constrained clustering analysis of single
      cell RNA-seq data
  source_content_hash: sha256:846e7e2803fa949fb47a7ee48f385da24ae9a6a1d66759c8cdfbbd35b7659e93
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
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_2269
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8125
n_claims: 16
---

# pairwise-must-link-cannot-link-loss-configuration

## Summary

Configure scDCC's soft pairwise constraints by encoding partial prior knowledge (e.g., CITE-seq protein similarity, subtype marker rules) as must-link and cannot-link pairs added as weighted auxiliary loss terms on top of the deep-embedding clustering loss, so the latent representation and clusters are shaped by domain knowledge.

## When to use

Use when clustering scRNA-seq data where purely unsupervised clustering yields biologically uninterpretable clusters and partial prior knowledge (e.g., protein-level ADT distances or known marker subtypes like CD4/CD8) is available for a subset of cell pairs.

## When NOT to use

- Prior knowledge is limited to marker genes and predefined cell types with no pairwise cell-pair information — marker-based methods such as CellAssign or SCINA are the article's cited alternative
- No prior information at all is available and the user only wants unsupervised clustering — use constraint-free scDCC or SC3 instead
- The user wants hard-constrained clustering that forbids violations of constraints (e.g., COP-K-means style) rather than soft loss-term penalties

## Inputs

- Scanpy-preprocessed mRNA count matrix from the 10X CITE-seq PBMC dataset (from figshare scDCC_data deposit)
- Normalized CITE-seq protein (antibody-derived tag) measurements for the same cells
- Pairwise must-link / cannot-link constraints (0.5th and 95th percentile protein-distance cutoffs, plus CD4/CD8 subtype rules, n_pairwise=25000)
- Known number of clusters

## Outputs

- Constrained scDCC cluster labels for all cells
- Trained deep-embedding model with constraint-weighted loss (ml_weight, cl_weight, gamma)
- Cluster-by-CD4/CD8-phenotype cross-tabulation (cd4_cd8_concordance)

## How to apply

Convert prior knowledge into soft pairwise constraints rather than hard-constrained clustering: generate confident must-link pairs from cells below the 0.5th percentile and cannot-link pairs from cells above the 95th percentile of cell-cell protein distances computed from normalized CITE-seq ADT measurements, augmented with marker-based subtype rules, capped at n_pairwise=25000. Train the PyTorch deep embedding on the Scanpy-preprocessed mRNA matrix, adding must-link and cannot-link loss terms weighted by ml_weight and cl_weight on top of the clustering loss weighted by gamma, for the known number of clusters. Because constraints are soft, violating pairs incur loss instead of forbidding an assignment; run the constrained variant via scDCC_pairwise_CITE_PBMC.py on the 10X CITE PBMC dataset.

## Related tools

- **scDCC** (Implements the deep-embedding constrained clustering; must-link and cannot-link losses are added as weighted terms via scDCC_pairwise_CITE_PBMC.py) — https://github.com/ttgump/scDCC
- **PyTorch** (Deep learning framework used to train the scDCC embedding with the constraint loss terms) — https://github.com/ttgump/scDCC
- **Scanpy** (Preprocessing of the mRNA count matrix before embedding)

## Examples

```
python scDCC_pairwise_CITE_PBMC.py --n_pairwise 25000
```

## Evaluation signals

- Constrained scDCC must achieve higher NMI, clustering accuracy (CA) and ARI than SC3 (on mRNAs), PhenoGraph and k-means (on protein levels), and than constraint-free scDCC
- CD4+/CD8- cells must not be mislabeled by constrained scDCC, unlike constraint-free scDCC (recorded in cd4_cd8_concordance comparison)
- Constraint pair counts should match the requested n_pairwise (25000) with pairs drawn from the 0.5th/95th percentile distance cutoffs
- Cross-tabulated cluster labels against CD4/CD8 protein-level phenotypes should show concordance consistent with the constraints

## Limitations

- Constraints are soft, so violating pairs are penalized but not forbidden — final labels may still violate some constraints
- Constraint quality depends on the percentile cutoffs (0.5th/95th); mis-specified cutoffs or incorrect subtype rules inject misleading pairwise signal
- Prior-knowledge approaches limited to marker genes (CellAssign, SCINA) are noted in the article as inflexible and ignoring dropout events; the same caveat applies to rule-based constraints such as CD4/CD8

## Evidence

- [intro] Soft pairwise constraints encoded as loss terms: "Unlike traditional hard-constrained clustering algorithms (e.g., COP K-means36), we convert (partial) prior knowledge into soft pairwise constraints and add them as additional terms into the loss"
- [methods] Percentile cutoffs for confident pair selection: [evidence span withheld — anchor status: not_found]
- [methods] Constraint count parameter: "--n_pairwise: number of pairwise constraints want to generate"
- [methods] Running the CITE PBMC constrained pipeline: "python scDCC_pairwise_CITE_PBMC.py"
- [readme] CD4/CD8 mislabeling failure mode of constraint-free clustering: "purely unsupervised clustering methods may not produce biologically interpretable clusters, which complicates cell type assignment"
