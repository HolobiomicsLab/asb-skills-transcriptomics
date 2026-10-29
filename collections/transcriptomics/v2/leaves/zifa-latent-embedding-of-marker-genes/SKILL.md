---
name: zifa-latent-embedding-of-marker-genes
description: Use when you have an scRNA-seq dataset plus a marker gene set defining
  expected cell types, and you need a low-dimensional marker-gene embedding to seed
  pseudo-labels and pairwise constraints for constrained clustering.
when_to_use_negative:
- The analysis goal is purely unsupervised clustering with no marker gene set or domain
  knowledge available to define constraints (the article notes existing algorithms
  'only support clustering in an unsupervised fashion' and scDCC's value comes from
  integrating prior information).
- The input is not scRNA-seq count data subject to dropout — ZIFA's zero-inflated
  model is specific to scRNA-seq sparsity, and the article's motivating gap is dropout
  in scRNA-Seq data.
- You already have expert-verified cluster labels and need no pseudo-label-based constraints
  from the ZIFA+k-means reference.
edam_operation: http://edamontology.org/operation_3432
edam_topics:
- http://edamontology.org/topic_0203
tools:
- name: scDCC
  role: Constrained clustering method trained on the ZIFA-space pseudo-labels and
    pairwise constraints; run via the dataset-specific wrapper scDCC_pairwise_Human_liver.py
  repo: https://github.com/ttgump/scDCC
- name: Scanpy
  role: Single-cell processing library used in the scDCC pipeline environment
- name: PyTorch
  role: Deep learning backend for training scDCC
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1038/s41467-021-22008-3
    title: Model-based deep embedding for constrained clustering analysis of single
      cell RNA-seq data
  source_content_hash: sha256:065d703ed38f59a19ed0b5e02ec6730c2c36aeb5e1dd92bebdc2f6410cdcfc72
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/ttgump/scDCC
  tools:
  - PyTorch
  - Python
  - Scanpy
  - scDCC
  edam_operation: http://edamontology.org/operation_3432
  edam_topics:
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7142857142857143
n_claims: 7
---

# ZIFA latent embedding of marker genes

## Summary

Computes a ZIFA latent representation from a scRNA-seq expression matrix subsetted to a curated marker gene set (e.g., the 63 human liver marker genes in the scDCC paper), then uses k-means in that latent space to generate pseudo-labels and pairwise must-link/cannot-link constraints for downstream constrained clustering with scDCC.

## When to use

Use when you have an scRNA-seq dataset plus a marker gene set defining expected cell types, and you need a low-dimensional marker-gene embedding to seed pseudo-labels and pairwise constraints for constrained clustering.

## When NOT to use

- The analysis goal is purely unsupervised clustering with no marker gene set or domain knowledge available to define constraints (the article notes existing algorithms 'only support clustering in an unsupervised fashion' and scDCC's value comes from integrating prior information).
- The input is not scRNA-seq count data subject to dropout — ZIFA's zero-inflated model is specific to scRNA-seq sparsity, and the article's motivating gap is dropout in scRNA-Seq data.
- You already have expert-verified cluster labels and need no pseudo-label-based constraints from the ZIFA+k-means reference.

## Inputs

- scRNA-seq expression/count matrix for the human liver dataset (from the figshare scDCC_data deposit, https://figshare.com/articles/dataset/scDCC_data/21563517)
- Curated marker gene list (63 liver marker genes) used to subset the expression matrix
- Number of pairwise constraints (--n_pairwise) and number of clusters (--n_clusters)

## Outputs

- ZIFA latent representation of cells restricted to marker genes
- k-means pseudo-labels in ZIFA space
- Pairwise must-link/cannot-link constraint set
- Comparison table of NMI, CA, and ARI for scDCC vs. PhenoGraph vs. k-means (scdcc_human_liver_metrics.csv)

## How to apply

Download the dataset (the scDCC study's data are on figshare at https://figshare.com/articles/dataset/scDCC_data/21563517) and subset the expression matrix to the marker genes of interest — for the human liver case, the 63 liver marker genes. Compute a ZIFA latent representation of the cells from this subset, exploiting ZIFA's zero-inflated modeling of scRNA-seq dropout. Run k-means in the ZIFA space to obtain pseudo-labels, and derive pairwise must-link/cannot-link constraints from cells sharing or differing in pseudo-label, controlling the constraint count with the --n_pairwise flag. These constraints, together with the pseudo-label cluster number (--n_clusters), feed scDCC training via the dataset-specific wrapper (python scDCC_pairwise_Human_liver.py), with --gamma weighting the clustering loss and --ml_weight/--cl_weight weighting the must-link and cannot-link losses. Baselines (k-means and PhenoGraph on the same ZIFA representation) are computed in the same space so that NMI, CA, and ARI comparisons isolate the effect of the constraints.

## Related tools

- **scDCC** (Constrained clustering method trained on the ZIFA-space pseudo-labels and pairwise constraints; run via the dataset-specific wrapper scDCC_pairwise_Human_liver.py) — https://github.com/ttgump/scDCC
- **Scanpy** (Single-cell processing library used in the scDCC pipeline environment)
- **PyTorch** (Deep learning backend for training scDCC)

## Examples

```
python scDCC_pairwise_Human_liver.py --n_clusters <k> --gamma <g> --ml_weight <mlw> --cl_weight <clw> --n_pairwise <n>
```

## Evaluation signals

- scDCC with marker gene-based constraints reproduces the reported human liver metrics: NMI 0.905, CA 0.928, ARI 0.952.
- scDCC metrics exceed the k-means-on-ZIFA baseline (NMI 0.833, CA 0.928, ARI 0.924) and the PhenoGraph baseline computed on the same ZIFA representation.
- The marker-gene subset is exactly the expected size (63 liver marker genes) before computing the ZIFA embedding.
- The number of generated pairwise constraints matches the value set via --n_pairwise.
- Metrics are assembled into scdcc_human_liver_metrics.csv comparing scDCC, PhenoGraph, and k-means on the identical ZIFA representation.

## Limitations

- Constraint quality depends on k-means pseudo-labels in the ZIFA space — the reference is itself an unsupervised clustering, so metrics measure agreement with that reference rather than external ground truth.
- The embedding is marker-gene-specific (63 liver genes in the reported case); a different tissue or marker panel requires re-deriving the subset and re-tuning --n_clusters, --gamma, --ml_weight, and --cl_weight.
- Purely unsupervised clustering 'may not produce biologically interpretable clusters' on dropout-laden scRNA-seq data, which motivates but also constrains the approach: without usable marker genes, the constraint-generation path fails.

## Evidence

- [methods] human liver dataset wrapper: "the wrapper to run scDCC on the human liver dataset (Figure 6)"
- [methods] wrapper invocation: "python scDCC_pairwise_Human_liver.py"
- [methods] dataset availability: "Datasets used in the study is available in: https://figshare.com/articles/dataset/scDCC_data/21563517"
- [methods] constraint generation parameter: "--n_pairwise: number of pairwise constraints want to generate"
- [readme] scDCC README motivation: "purely unsupervised clustering methods may not produce biologically interpretable clusters, which complicates cell type assignment"
