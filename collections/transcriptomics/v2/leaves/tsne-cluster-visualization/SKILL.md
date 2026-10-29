---
name: tsne-cluster-visualization
description: Use when you have DESC soft-assignment probabilities for scRNA-seq cells
  (e.g. Kang et al. PBMC data) and need t-SNE plots to inspect cluster coherence,
  assignment confidence, and control-vs-stimulus mixing.
when_to_use_negative:
- Technical batch differences are comparable to or larger than true biological variation
  — DESC's iterative self-learning removes batch effects only when technical differences
  across batches are smaller than true biological variations
- You need hard, pre-defined cluster labels only — DESC is a soft clustering method
  whose value lies in interpretable per-cell cluster probabilities
edam_operation: http://edamontology.org/operation_0337
edam_topics:
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_0092
tools:
- name: DESC
  role: Produces the soft-clustering assignments and cluster probability matrix visualized
    on t-SNE; provides visualization of clustering results and gene expression patterns
  repo: https://github.com/eleozzr/desc
- name: Scanpy
  role: Preprocesses the count matrix (log1p, filter_genes_dispersion, scale) into
    the input representation used for DESC clustering before t-SNE plotting
  repo: https://github.com/theislab/scanpy
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1038/s41467-020-15851-3
    title: Deep learning enables accurate clustering with batch effect removal in
      single-cell RNA-seq analysis
  source_content_hash: sha256:7c734c0f145f456d3e544d93e01fcecf0471d9c190077a7ab92761dd59f45132
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/eleozzr/desc
  tools:
  - DESC
  - Keras
  - Scanpy
  - desc
  edam_operation: http://edamontology.org/operation_0337
  edam_topics:
  - http://edamontology.org/topic_3308
  - http://edamontology.org/topic_0092
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5384615384615384
n_claims: 13
---

# tsne-cluster-visualization

## Summary

Visualize DESC soft-clustering results of single-cell RNA-seq data as t-SNE scatter plots colored by cluster assignment, per-cell maximum cluster probability (max.prob), cell type, and condition, enabling assessment of high-confidence cluster assignments and batch-effect removal.

## When to use

Use when you have DESC soft-assignment probabilities for scRNA-seq cells (e.g. Kang et al. PBMC data) and need t-SNE plots to inspect cluster coherence, assignment confidence, and control-vs-stimulus mixing.

## When NOT to use

- Technical batch differences are comparable to or larger than true biological variation — DESC's iterative self-learning removes batch effects only when technical differences across batches are smaller than true biological variations
- You need hard, pre-defined cluster labels only — DESC is a soft clustering method whose value lies in interpretable per-cell cluster probabilities

## Inputs

- DESC-annotated AnnData object (adata_desc.h5ad) with soft-assignment probability matrix q_i
- Scanpy-preprocessed normalized gene expression matrix (UMI-normalized, log1p, 1000 HVGs, scaled max_value=6)
- Cell-type annotations from marker-gene mapping and condition labels (control vs IFN-beta)

## Outputs

- t-SNE scatter plots colored by cluster assignment, max.prob, cell type, and condition
- Per-cell maximum cluster probability (max.prob) values
- High-confidence cluster assignment annotations supporting downstream DE analysis

## How to apply

After training DESC on Scanpy-preprocessed data (sc.pp.log1p, sc.pp.filter_genes_dispersion with n_top_genes=1000, sc.pp.scale(max_value=6)) without batch labels, retrieve the soft-assignment probability matrix q_i from the Student's t-distribution kernel used for soft assignments. Compute per-cell max.prob (the maximum cluster probability) and generate t-SNE scatter plots colored by cluster assignment, max.prob, cell type, and condition to identify cells clustered with high confidence. Interpret the soft probabilities biologically: DESC cluster assignment probabilities can reveal discrete and pseudotemporal structure. Verify batch-effect removal visually by checking that cells from control and interferon-beta-stimulated conditions mix within clusters, except where true biological variation (CD14+ monocyte IFN response) separates them.

## Related tools

- **DESC** (Produces the soft-clustering assignments and cluster probability matrix visualized on t-SNE; provides visualization of clustering results and gene expression patterns) — https://github.com/eleozzr/desc
- **Scanpy** (Preprocesses the count matrix (log1p, filter_genes_dispersion, scale) into the input representation used for DESC clustering before t-SNE plotting) — https://github.com/theislab/scanpy

## Examples

```
import desc; adata = desc.train(adata, dims=[adata.shape[1], 64, 32], louvain_resolution=[0.8, 1.0], tol=0.005)
```

## Evaluation signals

- Major PBMC populations receive high max.prob assignments visible as coherent clusters on the t-SNE plot
- t-SNE colored by condition shows control and IFN-beta-stimulated cells mixed within clusters for all cell types except CD14+ monocytes, matching the paper's finding
- Cluster identities on the t-SNE correspond to canonical PBMC types assigned via known marker genes
- DESC self-training converged with tol=0.005 (proportion of cells changing cluster assignment between consecutive steps below tolerance)
- The ranked IFN-beta response from downstream DE places CD14+ monocytes largest, consistent with the visualized condition separation

## Limitations

- The Student's t distribution kernel was more robust than the Gaussian distribution, but the Gaussian comparison was relegated to Supplementary Note 3 not provided in the text
- desc 2.0.3 requires Python 3.5.x or 3.6.x and TensorFlow < 2.0 to reproduce the paper's results; desc 2.1.1 targets TensorFlow 2.x
- Batch-effect removal via iterative self-learning works only while technical differences across batches are smaller than true biological variations

## Evidence

- [abstract] As a soft clustering algorithm, cluster assignment probabilities from DESC are biologically interpretable and can reveal both discrete and pseudotemporal structure of cells: "As a soft clustering algorithm, cluster assignment probabilities from DESC are biologically interpretable and can reveal both discrete and pseudotemporal structure of cells"
- [discussion] soft assignment via t distribution kernel: "we compute a soft assignment of each cell between the embedded points and the cluster centroids"
- [readme] DESC enables soft clustering for high-confidence identification: "DESC also enables soft clustering by assigning cluster-specific probabilities to each cell, which facilitates the identification of cells clustered with high-confidence"
- [other] t-SNE scatter plots colored by cluster assignment, max.prob, cell type, and condition: [evidence span withheld — anchor status: not_found]
- [readme] Visualize the cell clustering results and the gene expression patterns.: "Visualize the cell clustering results and  the  gene expression patterns."
