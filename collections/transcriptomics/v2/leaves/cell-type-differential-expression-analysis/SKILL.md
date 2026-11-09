---
name: cell-type-differential-expression-analysis
description: Use when a single-cell dataset contains annotated cell types and two
  conditions, and you need per-cell-type DE gene counts and magnitudes to rank the
  biological response across populations.
when_to_use_negative:
- The data has not been clustered/annotated into cell types — per-cell-type DE requires
  a prior clustering step (e.g., DESC soft assignments with max.prob)
- 'Batch effect has not been assessed: if technical batch dominates true biological
  variation, DESC''s premise that ''technical differences across batches are smaller
  than true biological variations'' fails and DE results may reflect batch, not biology'
- The dataset lacks multiple conditions per cell type (e.g., only one condition present),
  making within-cell-type control-vs-stimulus DE impossible
edam_operation: http://edamontology.org/operation_3223
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_0203
tools:
- name: DESC
  role: Produces the soft cluster assignments (probability matrix, max.prob) used
    to define high-confidence cell-type groupings within which DE is run
  repo: https://github.com/eleozzr/desc
- name: Scanpy
  role: Preprocesses the Kang et al. PBMC count matrix (sc.pp.log1p, sc.pp.filter_genes_dispersion
    with n_top_genes=1000, sc.pp.scale max_value=6) and supports differential expression
    analysis between conditions within each cell type
  repo: https://github.com/scverse/scanpy
- name: Keras
  role: Backend used to train the DESC deep-embedding model whose clusters feed the
    per-cell-type DE analysis
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1038/s41467-020-15851-3
    title: Deep learning enables accurate clustering with batch effect removal in
      single-cell RNA-seq analysis
  source_content_hash: sha256:25f0efbfad4fa5210d30a1645d9be7a7cd5105a435ce0236bb0843c843a01945
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/eleozzr/desc
  tools:
  - DESC
  - Keras
  - Scanpy
  - desc
  edam_operation: http://edamontology.org/operation_3223
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.4
n_claims: 5
---

# cell-type-differential-expression-analysis

## Summary

Performs differential expression between conditions (e.g., control vs interferon-beta stimulated) within each annotated cell type of a PBMC scRNA-seq dataset, then ranks cell types by IFN-beta response magnitude to verify that the strongest biological signal lies in CD14+ monocytes.

## When to use

Use when a single-cell dataset contains annotated cell types and two conditions, and you need per-cell-type DE gene counts and magnitudes to rank the biological response across populations.

## When NOT to use

- The data has not been clustered/annotated into cell types — per-cell-type DE requires a prior clustering step (e.g., DESC soft assignments with max.prob)
- Batch effect has not been assessed: if technical batch dominates true biological variation, DESC's premise that 'technical differences across batches are smaller than true biological variations' fails and DE results may reflect batch, not biology
- The dataset lacks multiple conditions per cell type (e.g., only one condition present), making within-cell-type control-vs-stimulus DE impossible

## Inputs

- Cluster-annotated AnnData object (.h5ad) of PBMC cells with control vs interferon-beta-stimulated condition labels (e.g., Kang et al. GSE96583, 24,679 cells)
- Cluster-to-cell-type mapping based on canonical PBMC marker genes
- Condition annotation per cell (control vs stimulus)

## Outputs

- Per-cell-type DE summary table (e.g., kang_desc_de_results.csv) with number of DE genes and DE magnitude per cell type
- Ranked list of cell types by IFN-beta response magnitude
- Per-cell-type DE gene sets between control and stimulated cells

## How to apply

Start from a preprocessed, clustered AnnData object (e.g., adata_desc.h5ad produced by DESC training on the Kang et al. lupus PBMC dataset of 24,679 cells, preprocessed in Scanpy with sc.pp.log1p, sc.pp.filter_genes_dispersion with n_top_genes=1000, and sc.pp.scale with max_value=6). Map clusters to canonical PBMC cell types using known marker genes, then within each annotated cell type run differential expression between control and interferon-beta-stimulated cells, recording both the number of DE genes and the DE magnitude of the IFN-beta response per cell type. Rank cell types by IFN-beta response magnitude; a correct application of batch-effect-removal-with-preserved-biology will place CD14+ monocytes first, with DE genes and magnitude several orders greater than in other cell types, while cells of other types mix across conditions. Write the per-cell-type DE summary table (e.g., kang_desc_de_results.csv) as the final output.

## Related tools

- **DESC** (Produces the soft cluster assignments (probability matrix, max.prob) used to define high-confidence cell-type groupings within which DE is run) — https://github.com/eleozzr/desc
- **Scanpy** (Preprocesses the Kang et al. PBMC count matrix (sc.pp.log1p, sc.pp.filter_genes_dispersion with n_top_genes=1000, sc.pp.scale max_value=6) and supports differential expression analysis between conditions within each cell type) — https://github.com/scverse/scanpy
- **Keras** (Backend used to train the DESC deep-embedding model whose clusters feed the per-cell-type DE analysis)

## Examples

```
sc.pp.log1p(adata); sc.pp.filter_genes_dispersion(adata, n_top_genes=1000); sc.pp.scale(adata, max_value=6); desc.train(adata, dims=[adata.n_vars, 64, 32], louvain_resolution=[0.8, 1.0], tol=0.005, do_tsne=True)
```

## Evaluation signals

- CD14+ monocytes rank first in IFN-beta response magnitude, with DE gene count and magnitude several orders larger than other cell types
- DESC clustering yields high-confidence assignments (high max.prob) for the major PBMC populations
- Cells of all cell types except CD14+ monocytes randomly mix between control and stimulus conditions on the DESC embedding (t-SNE colored by condition)
- The per-cell-type DE summary table is produced (kang_desc_de_results.csv) with all annotated cell types represented

## Limitations

- DESC removes batch effects only as long as technical differences across batches are smaller than true biological variations
- Differential expression between control and stimulated cells is only interpretable per cell type; mixing conditions across unannotated clusters invalidates the ranking
- DESC results depend on software environment: to reproduce the paper's results, tensorflow must be below 2.0 (desc 2.0.3); desc requires Python 3.5.x or 3.6.x
- No software version numbers stated for Keras or Scanpy, which may affect reproducibility of preprocessing and DE calls

## Evidence

- [abstract] Through iterative self-learning, DESC gradually removes batch effects, as long as technical differences across batches are smaller than true biological variations: "Through iterative self-learning, DESC gradually removes batch effects, as long as technical differences across batches are smaller than true biological variations"
- [discussion] Highly variable genes are selected using the filter_genes_dispersion function from the Scanpy package25: "Highly variable genes are selected using the filter_genes_dispersion function from the Scanpy package25"
- [abstract] As a soft clustering algorithm, cluster assignment probabilities from DESC are biologically interpretable: "As a soft clustering algorithm, cluster assignment probabilities from DESC are biologically interpretable and can reveal both discrete and pseudotemporal structure of cells"
- [readme] Because desc depend on tensorflow, you should make sure the version of tensorflow is lower than 2.0: "**Note:** Because desc depend on `tensorflow`, you should make sure the version of `tensorflow` is lower than `2.0` if you want to get the same results as the results in our paper."
- [methods] For tensorflow 1*, we released desc(2.0.3): "For  `tensorflow 1*`, we released `desc(2.0.3)`"
