---
name: gene-set-enrichment-interpretation
description: Use when scRNA-seq topics/embeddings must be interpreted against known
  biological pathways, or when gene set/pathway knowledge should be directly embedded
  into a topic model so pathway-topic associations are learned rather than inferred
  post hoc.
when_to_use_negative:
- The goal is de novo topic discovery with freely learned gene embeddings and post
  hoc gene set enrichment analysis rather than direct pathway-constrained interpretation
  — use unregularized scETM instead
- Pathway definitions smaller than or equal to 5 genes are the analysis target, since
  the pipeline explicitly keeps only pathways with more than 5 genes
- The scRNA-seq gene set has minimal overlap with the pathDIP4 pathway matrix genes,
  since the trainable gene set is restricted to their intersection
edam_operation: null
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_0203
tools:
- name: scETM
  role: Provides the p-scETM variant and UnsupervisedTrainer used to fix the gene
    embedding matrix ρ to the pathway matrix and learn topic embeddings via batched
    gradient descent
  repo: github.com/hui2000ji/scETM
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1038/s41467-021-25534-2
    title: Learning interpretable cellular and gene signature embeddings from single-cell
      transcriptomic data
  source_content_hash: sha256:0ec49e5a708f3d8e036c1e3ffe0bc9635a068833cb30a44923631a8360963e19
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - scETM
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 7
---

# gene-set-enrichment-interpretation

## Summary

Interpret scETM-learned topics by fixing the gene embedding matrix ρ to a pathways-by-genes matrix (p-scETM), enabling direct learning of pathway-topic associations. This yields functionally meaningful pathway-topic hits (e.g., Insulin Signaling in HP, Alzheimer Disease-Amyloid Secretase) without compromising clustering performance.

## When to use

Use when scRNA-seq topics/embeddings must be interpreted against known biological pathways, or when gene set/pathway knowledge should be directly embedded into a topic model so pathway-topic associations are learned rather than inferred post hoc.

## When NOT to use

- The goal is de novo topic discovery with freely learned gene embeddings and post hoc gene set enrichment analysis rather than direct pathway-constrained interpretation — use unregularized scETM instead
- Pathway definitions smaller than or equal to 5 genes are the analysis target, since the pipeline explicitly keeps only pathways with more than 5 genes
- The scRNA-seq gene set has minimal overlap with the pathDIP4 pathway matrix genes, since the trainable gene set is restricted to their intersection

## Inputs

- Cells-by-genes AnnData matrix (HP, AD, or MDD datasets) with 'batch_indices' and 'cell_types' columns in adata.obs
- Pathways-by-genes matrix from the pathDIP4 pathway database (http://ophid.utoronto.ca/pathDIP/Download.jsp), filtered to pathways with more than 5 genes

## Outputs

- Ranked pathway-topic association table (pathway_topic_associations.csv)
- Learned topic embeddings (mp.uns['alpha']) and fixed pathway gene embedding matrix ρ (mp.varm['rho'])
- Clustering metrics (ARI/NMI-style evaluation from scETM evaluate()) comparable to Supplementary Table 15

## How to apply

Download the pathways-by-genes matrix from the pathDIP4 pathway database and keep only pathways containing more than 5 genes. Load the cells-by-genes AnnData matrices (batch information in 'batch_indices', cell type identity in 'cell_types') and instantiate p-scETM with trainable_gene_emb_dim set to zero so ρ is entirely fixed to the pathway matrix; the model gene set becomes the intersection of scRNA-seq genes and pathway-matrix genes, and only topic embeddings α are learned. Train with UnsupervisedTrainer via batched gradient descent for enough epochs to give at least 6k gradient updates (e.g., n_epochs=12000, eval_every=1000, n_samplers=4). Extract the learned topic embeddings and the fixed pathway gene embeddings, then score pathway-topic associations per dataset. Verify expected hits are recovered (Insulin Signaling among top HP pathways; Topic 9 associated with Alzheimer Disease-Amyloid Secretase) and that clustering performance matches Supplementary Table 15's criterion that fixing ρ does not degrade clustering versus unregularized scETM, writing the ranked table to pathway_topic_associations.csv.

## Related tools

- **scETM** (Provides the p-scETM variant and UnsupervisedTrainer used to fix the gene embedding matrix ρ to the pathway matrix and learn topic embeddings via batched gradient descent) — github.com/hui2000ji/scETM

## Examples

```
model = scETM(adata.n_vars, adata.obs.batch_indices.nunique(), trainable_gene_emb_dim=0, enable_batch_bias=True); UnsupervisedTrainer(model, adata, train_instance_name='p_scETM', ckpt_dir='../results').train(n_epochs=12000, eval_every=1000, n_samplers=4); model.get_all_embeddings_and_nll(adata)
```

## Evaluation signals

- pathway_topic_associations.csv contains the expected named hits: Insulin Signaling among top HP pathways and Topic 9 associated with Alzheimer Disease-Amyloid Secretase
- Clustering metrics from the run match the Supplementary Table 15 criterion: fixing ρ to pathway annotations does not degrade clustering relative to baseline scETM
- The model gene set equals the intersection of scRNA-seq genes and gene-by-pathway matrix genes (trainable_gene_emb_dim = 0)
- Training achieved at least 6k gradient updates (e.g., 12000 epochs evaluated every 1000), ensuring convergence on small datasets
- The pathDIP4 pathway matrix retains only pathways with more than 5 genes before model initialization

## Limitations

- Fixing ρ restricts the trainable gene set to the intersection of scRNA-seq and pathway-matrix genes; scRNA-seq-unique genes are dropped when trainable_gene_emb_dim is zero
- Setting trainable_gene_emb_dim to a positive value keeps all scRNA-seq genes, but then ρ is only partly fixed to pathways and pathway-topic associations are no longer fully direct
- For small datasets with large mini-batch sizes (e.g., 1886-cell MousePancreas, batch size 2000), one epoch gives only one gradient update, so too few epochs risks unconverged, noisy pathway-topic associations

## Evidence

- [readme] p-scETM is a variant of scETM where part or all of the the gene embedding matrix ρ is fixed to a pathways-by-genes matrix, which can be downloaded from the pathDIP4 pathway database: "p-scETM is a variant of scETM where part or all of the the gene embedding matrix ρ is fixed to a pathways-by-genes matrix"
- [readme] Filter pathways to more than 5 genes: "We only keep pathways that contain more than 5 genes."
- [readme] Fix ρ by setting trainable_gene_emb_dim to zero: "If it is desired to fix the gene embedding matrix ρ during training, let trainable_gene_emb_dim be zero. In this case, the gene set used to train the model would be the intersection of the genes in"
- [readme] At least 6k gradient updates recommended: "it is recommended to set the number of training epochs such that there is at least 6k gradient updates to the model. Otherwise, the model may not have converged when training stops."
- [abstract] scETM enables incorporation of known gene sets into gene embeddings to learn pathway-topic associations: "scETM enables the incorporation of known gene sets into the gene embeddings, thereby directly learning the associations between pathways and topics via the topic embeddings."
