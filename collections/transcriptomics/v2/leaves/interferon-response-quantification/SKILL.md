---
name: interferon-response-quantification
description: Use when scRNA-seq count data contain control vs interferon-beta-stimulated
  cells and you need to rank cell types by IFN-beta response magnitude while verifying
  batch effects were removed without erasing true biological variation.
when_to_use_negative:
- When technical batch differences are larger than true biological variation — DESC
  removes batch effects only 'as long as technical differences across batches are
  smaller than true biological variations'
- When the design requires explicitly providing batch labels to the batch-correction
  method — DESC does not require batch information, and this workflow deliberately
  trains desc.train without batch labels
- When cells have already been cluster-annotated and only a bulk-style DE test is
  needed — the DESC clustering pipeline is unnecessary overhead
edam_operation: http://edamontology.org/operation_3501
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: DESC (desc)
  role: Unsupervised deep-embedding clustering that removes batch effects without
    explicit batch labels and provides soft cluster probabilities (max.prob) used
    to gate high-confidence cell assignments before DE
  repo: https://github.com/eleozzr/desc
- name: Scanpy
  role: 'Preprocessing of the GSE96583 count matrix: sc.pp.log1p, sc.pp.filter_genes_dispersion
    (n_top_genes=1000), sc.pp.scale(max_value=6)'
  repo: https://github.com/theislab/scanpy
- name: Keras
  role: Backend used by DESC to train the stacked-autoencoder neural network during
    pretraining and self-training
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1038/s41467-020-15851-3
    title: Deep learning enables accurate clustering with batch effect removal in
      single-cell RNA-seq analysis
  source_content_hash: sha256:e6b63b8ad36b6458bb606f11a15a684008603477a8633646d451412ab3d5bb0c
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/eleozzr/desc
  tools:
  - DESC (desc)
  - Keras
  - Scanpy
  - desc
  edam_operation: http://edamontology.org/operation_3501
  edam_topics:
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6153846153846154
n_claims: 13
---

# interferon-response-quantification

## Summary

Quantifies cell-type-specific interferon-beta response in single-cell RNA-seq by clustering cells (e.g., with DESC on the Kang et al. lupus PBMC data), annotating clusters to PBMC cell types, and running per-cell-type differential expression between control and stimulated conditions, ranking cell types by response magnitude.

## When to use

Use when scRNA-seq count data contain control vs interferon-beta-stimulated cells and you need to rank cell types by IFN-beta response magnitude while verifying batch effects were removed without erasing true biological variation.

## When NOT to use

- When technical batch differences are larger than true biological variation — DESC removes batch effects only 'as long as technical differences across batches are smaller than true biological variations'
- When the design requires explicitly providing batch labels to the batch-correction method — DESC does not require batch information, and this workflow deliberately trains desc.train without batch labels
- When cells have already been cluster-annotated and only a bulk-style DE test is needed — the DESC clustering pipeline is unnecessary overhead

## Inputs

- Raw UMI count matrix from GEO accession GSE96583 (Kang et al. lupus PBMCs, control vs interferon-beta stimulated, 24,679 cells)
- Known PBMC marker genes for cluster-to-cell-type annotation
- Per-cell condition labels (control vs INF-beta stimulated) for within-type differential expression

## Outputs

- adata_desc.h5ad: AnnData with DESC soft-cluster assignments and max.prob per cell
- t-SNE scatter plots colored by cluster assignment, max.prob, cell type, and condition
- kang_desc_de_results.csv: per-cell-type DE summary table ranking IFN-beta response magnitude

## How to apply

Preprocess the count matrix (e.g., GSE96583 Kang et al. lupus PBMCs, 24,679 cells) in Scanpy: divide UMI counts by total UMIs per cell, multiply by 10,000, transform with sc.pp.log1p, select 1000 highly variable genes via sc.pp.filter_genes_dispersion, and standardize with sc.pp.scale(max_value=6). Train DESC via desc.train WITHOUT batch labels, using a stacked autoencoder (e.g., dims=[n_genes, 64, 32]), Louvain-initialized cluster centers over candidate resolutions such as [0.8, 1.0], Student's t soft assignments, and KL-divergence self-training with tol=0.005; save the annotated AnnData as adata_desc.h5ad. Compute per-cell maximum cluster probability (max.prob) from the soft-assignment matrix, map DESC clusters to canonical PBMC types with known marker genes, then run differential expression between control and stimulated cells within each annotated cell type. Rank cell types by IFN-beta response magnitude and write the per-cell-type DE summary to kang_desc_de_results.csv. The rationale is that DESC iteratively removes batch effects as long as technical differences are smaller than true biological variation, so the interferon signal—strongest in CD14+ monocytes—should survive batch correction while cells of other types mix across conditions.

## Related tools

- **DESC (desc)** (Unsupervised deep-embedding clustering that removes batch effects without explicit batch labels and provides soft cluster probabilities (max.prob) used to gate high-confidence cell assignments before DE) — https://github.com/eleozzr/desc
- **Scanpy** (Preprocessing of the GSE96583 count matrix: sc.pp.log1p, sc.pp.filter_genes_dispersion (n_top_genes=1000), sc.pp.scale(max_value=6)) — https://github.com/theislab/scanpy
- **Keras** (Backend used by DESC to train the stacked-autoencoder neural network during pretraining and self-training)

## Examples

```
sc.pp.log1p(adata); sc.pp.filter_genes_dispersion(adata, n_top_genes=1000); sc.pp.scale(adata, max_value=6)
```

## Evaluation signals

- DESC clustering yields high max.prob (high-confidence soft-assignment probabilities) for the major PBMC populations
- Ranked IFN-beta response magnitude places CD14+ monocytes first, several orders larger than other cell types
- t-SNE plots colored by condition show cells of all types except CD14+ monocytes randomly mixed between control and stimulus
- Number of DE genes and DE magnitude within CD14+ monocytes exceed those of other cell types by orders of magnitude
- Workflow completes with a valid kang_desc_de_results.csv containing per-cell-type DE summary rows

## Limitations

- Batch-effect removal by DESC only works when technical differences across batches are smaller than true biological variations
- desc (2.0.3) requires Python 3.5.x or 3.6.x; to reproduce paper results TensorFlow must be below 2.0, while desc 2.1.1 targets TensorFlow 2.x
- Supplementary defaults (hidden layers/nodes and autoencoder hyperparameters, Supplementary Tables 2–3) were not available in the provided text, so dims and resolutions must be set explicitly
- T-distribution kernel was found more robust than a Gaussian kernel (Supplementary Note 3), so kernel choice affects robustness

## Evidence

- [abstract] Through iterative self-learning, DESC gradually removes batch effects, as long as technical differences across batches are smaller than true biological variations: "Through iterative self-learning, DESC gradually removes batch effects, as long as technical differences across batches are smaller than true biological variations"
- [discussion] Normalization workflow step: "the UMI count for each gene in each cell is divided by the total number of UMIs in the cell, multiplied by 10,000, and then transformed to a natural log scale."
- [discussion] HVG selection step: "Highly variable genes are selected using the filter_genes_dispersion function from the Scanpy package25"
- [discussion] Soft assignment step: "we compute a soft assignment of each cell between the embedded points and the cluster centroids. Following van der Maaten and Hinton24, we use the Student’s t distribution as a kernel"
- [readme] DESC soft clustering capability: "DESC also enables soft clustering by assigning cluster-specific probabilities to each cell, which facilitates the identification of cells clustered with high-confidence"
