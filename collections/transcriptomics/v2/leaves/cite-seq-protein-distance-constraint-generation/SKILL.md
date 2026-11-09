---
name: cite-seq-protein-distance-constraint-generation
description: Use when a CITE-seq dataset provides protein (ADT) measurements on the
  same cells as mRNA counts, and the goal is to improve mRNA clustering accuracy or
  enforce biologically consistent cluster labels (e.g., CD4/CD8 T cell subtypes) via
  pairwise constraints.
when_to_use_negative:
- No CITE-seq/ADT protein measurements are available — constraints must then come
  from other domain knowledge (e.g., marker genes), not protein distances
- The downstream method only supports hard-constrained clustering (e.g., COP K-means
  style); these soft percentile-based constraints are designed for scDCC's loss-augmentation
  approach
- Only a small fraction of cells has protein data, so pairwise protein distances are
  too sparse to yield confident 0.5th/95th percentile pairs
edam_operation: http://edamontology.org/operation_3432
edam_topics:
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_0121
tools:
- name: scDCC
  role: Consumes the generated must-link/cannot-link constraints as soft loss terms
    (ml_weight, cl_weight) in its model-based deep embedding clustering
  repo: https://github.com/ttgump/scDCC
- name: Scanpy
  role: Preprocessing/normalization of the CITE-seq mRNA and protein matrices before
    constraint computation
  repo: https://scanpy.readthedocs.io/en/stable
- name: PyTorch
  role: Backend on which the scDCC deep embedding with pairwise-constraint loss terms
    is trained
  repo: https://pytorch.org
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1038/s41467-021-22008-3
    title: Model-based deep embedding for constrained clustering analysis of single
      cell RNA-seq data
  source_content_hash: sha256:a13ea7cf98c3ce5f433e111fbca7a18a9bb7b1c3261fe90bdc59c45bc9cb823d
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
  - http://edamontology.org/topic_0121
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5714285714285714
n_claims: 7
---

# CITE-seq protein-distance pairwise constraint generation

## Summary

Converts CITE-seq antibody-derived tag (ADT) protein measurements into soft pairwise constraints (must-link and cannot-link pairs) for scDCC, so that protein-level domain knowledge guides clustering of the matched scRNA-seq data. Rationale: purely unsupervised clustering may yield biologically uninterpretable clusters, and pairwise protein distances at confident extremes encode reliable cell-cell similarity/dissimilarity.

## When to use

Use when a CITE-seq dataset provides protein (ADT) measurements on the same cells as mRNA counts, and the goal is to improve mRNA clustering accuracy or enforce biologically consistent cluster labels (e.g., CD4/CD8 T cell subtypes) via pairwise constraints.

## When NOT to use

- No CITE-seq/ADT protein measurements are available — constraints must then come from other domain knowledge (e.g., marker genes), not protein distances
- The downstream method only supports hard-constrained clustering (e.g., COP K-means style); these soft percentile-based constraints are designed for scDCC's loss-augmentation approach
- Only a small fraction of cells has protein data, so pairwise protein distances are too sparse to yield confident 0.5th/95th percentile pairs

## Inputs

- 10X CITE-seq PBMC dataset: mRNA count matrix plus antibody-derived tag (ADT) protein matrix for the same cells (from figshare scDCC_data deposit, https://figshare.com/articles/dataset/scDCC_data/21563517)
- Preprocessed/normalized mRNA matrix (Scanpy preprocessing)
- Normalized CITE-seq protein measurements
- Optional cell-type-specific rule set (e.g., CD4/CD8 T cell subtype labels)
- Constraint budget parameter (n_pairwise, e.g., 25000)

## Outputs

- Must-link pairwise constraint list (cell pairs with confident protein similarity)
- Cannot-link pairwise constraint list (cell pairs at the 95th-percentile protein distance cutoff)
- Constraint lists consumed by scDCC as ml_weight- and cl_weight-weighted loss terms

## How to apply

Load the 10X CITE-seq PBMC dataset (mRNA counts plus antibody-derived tags) from the scDCC_data figshare deposit and preprocess the mRNA matrix with Scanpy (v1.0.4 in the paper). Normalize the protein measurements, compute pairwise cell-cell protein distances, and pick cells at the extreme percentiles — 0.5th percentile cutoffs to form confident must-link pairs and 95th percentile cutoffs to form confident cannot-link pairs — optionally augmented with CD4/CD8 subtype rules, up to a fixed budget such as n_pairwise=25000. Rationale: unlike hard-constrained clustering (e.g., COP-K-means), scDCC converts this partial prior knowledge into soft pairwise constraints added as additional loss terms, so constraint quality matters more than completeness; extreme distance percentiles select only the most confident pairs. Pass the resulting must-link and cannot-link lists to the scDCC PyTorch trainer (scDCC_pairwise_CITE_PBMC.py), where they are weighted by ml_weight and cl_weight alongside the clustering loss weighted by gamma for the known number of clusters.

## Related tools

- **scDCC** (Consumes the generated must-link/cannot-link constraints as soft loss terms (ml_weight, cl_weight) in its model-based deep embedding clustering) — https://github.com/ttgump/scDCC
- **Scanpy** (Preprocessing/normalization of the CITE-seq mRNA and protein matrices before constraint computation) — https://scanpy.readthedocs.io/en/stable
- **PyTorch** (Backend on which the scDCC deep embedding with pairwise-constraint loss terms is trained) — https://pytorch.org

## Examples

```
python scDCC_pairwise_CITE_PBMC.py --n_pairwise 25000
```

## Evaluation signals

- Constrained scDCC must not mislabel CD4+/CD8- cells, unlike constraint-free scDCC (cluster-vs-CD4/CD8-protein concordance check)
- Constrained scDCC must achieve higher NMI, clustering accuracy (CA), and ARI than constraint-free scDCC, SC3 (on mRNA), PhenoGraph, and k-means (on protein)
- Generated pairs should fall at confident extremes: must-link pairs at/below the 0.5th percentile of protein distance and cannot-link pairs at/above the 95th percentile
- Total constraint count should honor the specified budget (e.g., n_pairwise=25000)

## Limitations

- Constraints are soft and partial — they guide the embedding but cannot guarantee every cell is labeled correctly (constraint-free scDCC still mislabeled some CD4+/CD8- cells, so constraint quality/quantity drives improvement)
- Requires the number of clusters to be known in advance for scDCC training
- Protein-derived constraints reflect surface-marker similarity, not transcriptomic ground truth; inappropriate percentile cutoffs can inject wrong pairs

## Evidence

- [intro] Soft-constraint rationale: "Unlike traditional hard-constrained clustering algorithms (e.g., COP K-means36), we convert (partial) prior knowledge into soft pairwise constraints and add them as additional terms into the loss"
- [methods] Dataset and runner script: "the wrapper to run scDCC on the 10X CITE PBMC dataset (Figure 5)"
- [methods] Runner invocation: "python scDCC_pairwise_CITE_PBMC.py"
- [methods] Constraint-count parameter: "--n_pairwise: number of pairwise constraints want to generate"
- [methods] Data availability: "Datasets used in the study is available in: https://figshare.com/articles/dataset/scDCC_data/21563517"
- [other] Task-card constraint recipe (percentile cutoffs, CD4/CD8 augmentation, n_pairwise=25000): [evidence span withheld — anchor status: not_found]
