---
name: pathway-gene-matrix-filtering
description: Use when initializing p-scETM with trainable_gene_emb_dim set to zero,
  or whenever a known gene-set matrix must be size-filtered before being used as a
  fixed gene embedding in topic models.
when_to_use_negative:
- When trainable_gene_emb_dim is set to a positive value, since all scRNA-seq genes
  are kept and no pathway-fixed gene embedding is required
- When the analysis is standard unregularized scETM clustering with freely learned
  gene embeddings ρ rather than pathway-informed topics
edam_operation: http://edamontology.org/operation_3680
edam_topics:
- http://edamontology.org/topic_0605
- http://edamontology.org/topic_3170
tools:
- name: scETM
  role: Consumes the filtered pathways-by-genes matrix as the fixed gene embedding
    ρ (trainable_gene_emb_dim=0) in the p-scETM variant; installed via pip install
    scETM
  repo: github.com/hui2000ji/scETM
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1038/s41467-021-25534-2
    title: Learning interpretable cellular and gene signature embeddings from single-cell
      transcriptomic data
  source_content_hash: sha256:996fea504c566f7cc99b9f6e6242dcdf9d09c64706aa99094b1a7c1db589cc60
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - scETM
  edam_operation: http://edamontology.org/operation_3680
  edam_topics:
  - http://edamontology.org/topic_0605
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5714285714285714
n_claims: 7
---

# pathway-gene-matrix-filtering

## Summary

Filters a pathways-by-genes binary matrix downloaded from the pathDIP4 pathway database so that only pathways containing more than 5 genes are retained, preparing the fixed gene embedding matrix ρ used by p-scETM to learn pathway-topic associations.

## When to use

Use when initializing p-scETM with trainable_gene_emb_dim set to zero, or whenever a known gene-set matrix must be size-filtered before being used as a fixed gene embedding in topic models.

## When NOT to use

- When trainable_gene_emb_dim is set to a positive value, since all scRNA-seq genes are kept and no pathway-fixed gene embedding is required
- When the analysis is standard unregularized scETM clustering with freely learned gene embeddings ρ rather than pathway-informed topics

## Inputs

- pathways-by-genes matrix downloaded from the pathDIP4 pathway database
- cells-by-genes AnnData (.h5ad) matrices (HP, AD, MDD) with 'batch_indices' and 'cell_types' in adata.obs

## Outputs

- Filtered pathways-by-genes matrix containing only pathways with more than 5 genes
- Gene-restricted scRNA-seq matrix (intersection of scRNA-seq genes and pathway-matrix genes) for p-scETM training
- Ranked pathway-topic association table (e.g., pathway_topic_associations.csv)

## How to apply

Download the pathways-by-genes matrix from the pathDIP4 pathway database (http://ophid.utoronto.ca/pathDIP/Download.jsp) and keep only pathways that contain more than 5 genes; this size cutoff removes overly small gene sets that would be uninformative as embedding rows. Intersect the retained pathway genes with the genes of the scRNA-seq AnnData matrix (e.g., HP, AD, or MDD cells-by-genes .h5ad files with 'batch_indices' and 'cell_types' obs columns) so the fixed ρ aligns with the data. Instantiate scETM with trainable_gene_emb_dim=0 so ρ is entirely fixed to the filtered pathway matrix and only topic embeddings α are learned; note the model gene set then equals the intersection of scRNA-seq genes and pathway-matrix genes. Train with UnsupervisedTrainer for enough epochs to give at least 6k gradient updates (e.g., n_epochs=12000, eval_every=1000), then extract α and the fixed ρ to score pathway-topic associations.

## Related tools

- **scETM** (Consumes the filtered pathways-by-genes matrix as the fixed gene embedding ρ (trainable_gene_emb_dim=0) in the p-scETM variant; installed via pip install scETM) — github.com/hui2000ji/scETM

## Examples

```
model = scETM(adata.n_vars, adata.obs.batch_indices.nunique(), trainable_gene_emb_dim=0, enable_batch_bias=True); trainer = UnsupervisedTrainer(model, adata); trainer.train(n_epochs=12000, eval_every=1000)
```

## Evaluation signals

- Every retained pathway row has more than 5 genes after filtering
- The model gene set equals the intersection of scRNA-seq genes and genes in the pathway-by-gene matrix
- Expected pathway-topic hits are recovered (Insulin Signaling among top HP pathways; Topic 9 associated with Alzheimer Disease-Amyloid Secretase)
- Clustering metrics on HP/AD/MDD are not degraded relative to the unregularized scETM baseline (Supplementary Table 15 criterion)
- Output CSV pathway_topic_associations.csv contains the named pathway-topic hits

## Limitations

- Small pathways (≤5 genes) are discarded, so associations involving small gene sets cannot be learned
- Fixing ρ restricts the model gene set to the intersection with the pathway matrix, so scRNA-seq genes absent from pathDIP4 are excluded from training
- Small datasets with large mini-batches produce few gradient updates per epoch; epochs must be raised (≥6k updates) or the model may not converge

## Evidence

- [readme] p-scETM is a variant of scETM where part or all of the the gene embedding matrix ρ is fixed to a pathways-by-genes matrix, which can be downloaded from the pathDIP4 pathway database.: "p-scETM is a variant of scETM where part or all of the the gene embedding matrix ρ is fixed to a pathways-by-genes matrix, which can be downloaded from the [pathDIP4 pathway database]"
- [methods] We only keep pathways that contain more than 5 genes.: "We only keep pathways that contain more than 5 genes."
- [readme] If it is desired to fix the gene embedding matrix ρ during training, let trainable_gene_emb_dim be zero.: "If it is desired to fix the gene embedding matrix ρ during training, let trainable_gene_emb_dim be zero. In this case, the gene set used to train the model would be the intersection of the genes in"
- [readme] it is recommended to set the number of training epochs such that there is at least 6k gradient updates to the model.: "it is recommended to set the number of training epochs such that there is at least 6k gradient updates to the model. Otherwise, the model may not have converged when training stops."
