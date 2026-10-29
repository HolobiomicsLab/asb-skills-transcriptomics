---
name: k-means-baseline-clustering
description: Use when benchmarking a constrained or deep clustering method on single-cell
  RNA-seq data and you need an unsupervised k-means baseline computed in a marker-gene
  ZIFA latent space for NMI/CA/ARI comparison.
when_to_use_negative:
- When the goal is to integrate prior/marker-gene knowledge directly into clustering
  — k-means is the purely unsupervised baseline; use scDCC with pairwise constraints
  instead
- When working in raw high-dimensional gene space — the baseline here is defined on
  the ZIFA latent representation of marker genes, not the raw count matrix
- When the method under comparison (e.g., CellAssign, SCINA) requires predefined marker
  gene sets and cell types rather than a purely unsupervised baseline
edam_operation: http://edamontology.org/operation_3432
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: scDCC
  role: Method under evaluation; k-means serves as its unsupervised baseline on the
    ZIFA marker-gene latent features
  repo: https://github.com/ttgump/scDCC
- name: ZIFA
  role: Dimensionality reduction producing the latent space in which k-means is run
- name: Scanpy
  role: scRNA-seq preprocessing used within the scDCC analysis pipeline
- name: PyTorch
  role: Deep learning framework for scDCC training (PyTorch 1.5.1+cu101)
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1038/s41467-021-22008-3
    title: Model-based deep embedding for constrained clustering analysis of single
      cell RNA-seq data
  source_content_hash: sha256:4f579819aeb2a68638b1ff49bb662bbc026bb9df9c25d409d5214481bc20d409
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/ttgump/scDCC
  tools:
  - PyTorch
  - Python
  - Scanpy
  - ZIFA
  - scDCC
  edam_operation: http://edamontology.org/operation_3432
  edam_topics:
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.625
n_claims: 8
---

# k-means-baseline-clustering

## Summary

Runs k-means clustering on the ZIFA latent representation of marker genes as the unsupervised baseline against which scDCC with pairwise constraints is evaluated, using NMI, CA and ARI on the human liver scRNA-seq dataset.

## When to use

Use when benchmarking a constrained or deep clustering method on single-cell RNA-seq data and you need an unsupervised k-means baseline computed in a marker-gene ZIFA latent space for NMI/CA/ARI comparison.

## When NOT to use

- When the goal is to integrate prior/marker-gene knowledge directly into clustering — k-means is the purely unsupervised baseline; use scDCC with pairwise constraints instead
- When working in raw high-dimensional gene space — the baseline here is defined on the ZIFA latent representation of marker genes, not the raw count matrix
- When the method under comparison (e.g., CellAssign, SCINA) requires predefined marker gene sets and cell types rather than a purely unsupervised baseline

## Inputs

- Human liver scRNA-seq expression matrix (downloaded from the figshare scDCC_data deposit, https://figshare.com/articles/dataset/scDCC_data/21563517)
- Subset of 63 liver marker genes
- ZIFA latent representation of the cells
- Number of clusters k (set via --n_clusters)

## Outputs

- k-means cluster labels in the ZIFA marker-gene latent space
- Baseline NMI, CA and ARI values for comparison with scDCC and PhenoGraph
- Comparison table scdcc_human_liver_metrics.csv

## How to apply

Subset the normalized expression matrix to the 63 liver marker genes and embed cells with ZIFA. Run k-means in this ZIFA latent space to obtain cluster labels (and pseudo-labels used to derive pairwise must-link/cannot-link constraints controlled via --n_pairwise). Compute NMI, CA (clustering accuracy) and ARI for the k-means labels and compare with scDCC labels and PhenoGraph on the same representation. On the human liver dataset the expected baseline is NMI 0.833, CA 0.928, ARI 0.924, versus scDCC's NMI 0.905, CA 0.928, ARI 0.952; collect results into scdcc_human_liver_metrics.csv.

## Related tools

- **scDCC** (Method under evaluation; k-means serves as its unsupervised baseline on the ZIFA marker-gene latent features) — https://github.com/ttgump/scDCC
- **ZIFA** (Dimensionality reduction producing the latent space in which k-means is run)
- **Scanpy** (scRNA-seq preprocessing used within the scDCC analysis pipeline)
- **PyTorch** (Deep learning framework for scDCC training (PyTorch 1.5.1+cu101))

## Examples

```
python scDCC_pairwise_Human_liver.py --n_clusters <K> --gamma <g> --ml_weight <w_ml> --cl_weight <w_cl>
```

## Evaluation signals

- k-means baseline on ZIFA marker-gene features should yield approximately NMI 0.833, CA 0.928, ARI 0.924 on the human liver dataset
- scDCC with marker gene-based constraints should exceed the k-means baseline: NMI 0.905 vs 0.833 and ARI 0.952 vs 0.924, with CA matching at 0.928
- scDCC should also outperform the PhenoGraph baseline computed on the same ZIFA representation
- All three metrics must be computed for scDCC, PhenoGraph and k-means on the same ZIFA latent representation before assembling scdcc_human_liver_metrics.csv
- Pairwise constraint count must be controlled via the --n_pairwise parameter when generating must-link/cannot-link constraints from k-means pseudo-labels

## Limitations

- Purely unsupervised clustering methods may not produce biologically interpretable clusters on scRNA-seq data due to high dimensionality and pervasive dropout events
- Most existing algorithms for scRNA-seq only support unsupervised clustering and cannot integrate prior information
- Discussion section content is absent from the provided document, so no additional caveats from the discussion could be traced

## Evidence

- [methods] k-means baseline reported metrics: [evidence span withheld — anchor status: not_found]
- [abstract] unconstrained clustering limitation: "purely unsupervised clustering methods may not produce biologically interpretable clusters, which complicates cell type assignment"
- [methods] wrapper invocation: "the wrapper to run scDCC on the human liver dataset (Figure 6)"
- [methods] constraint generation flag: "--n_pairwise: number of pairwise constraints want to generate"
- [methods] data availability: "Datasets used in the study is available in: https://figshare.com/articles/dataset/scDCC_data/21563517"
