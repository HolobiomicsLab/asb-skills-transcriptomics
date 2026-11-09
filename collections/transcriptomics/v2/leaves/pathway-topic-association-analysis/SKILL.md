---
name: pathway-topic-association-analysis
description: Use when scRNA-seq AnnData matrices (HP, AD, MDD) with 'batch_indices'
  and 'cell_types' need direct, functionally meaningful pathway-topic interpretations
  without sacrificing clustering performance relative to unregularized scETM.
when_to_use_negative:
- When the goal is free-form topic discovery or transfer learning rather than direct
  interpretation against known pathway gene sets (use standard scETM, keeping trainable_gene_emb_dim
  positive so all genes are retained)
- When pathways of interest contain 5 or fewer genes, since the pathDIP4 matrix is
  filtered to keep only pathways with more than 5 genes
edam_operation: http://edamontology.org/operation_3432
edam_topics:
- http://edamontology.org/topic_0091
- http://edamontology.org/topic_3170
tools:
- name: scETM
  role: Provides the p-scETM variant (trainable_gene_emb_dim=0) and UnsupervisedTrainer
    used to fix ρ to the pathway matrix and learn topic embeddings
  repo: github.com/hui2000ji/scETM
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1038/s41467-021-25534-2
    title: Learning interpretable cellular and gene signature embeddings from single-cell
      transcriptomic data
  source_content_hash: sha256:0315090f8e918589a444673c0e10b1640ed6bc47d6e83d17039d98e6858e5362
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - scETM
  edam_operation: http://edamontology.org/operation_3432
  edam_topics:
  - http://edamontology.org/topic_0091
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.875
n_claims: 8
---

# pathway-topic-association-analysis

## Summary

Fit the pathway-informed scETM variant (p-scETM), fixing the gene embedding matrix ρ to a pathways-by-genes matrix from pathDIP4 and learning only the topic embeddings, so that pathway-topic associations are read directly from the model rather than inferred post hoc via gene set enrichment.

## When to use

Use when scRNA-seq AnnData matrices (HP, AD, MDD) with 'batch_indices' and 'cell_types' need direct, functionally meaningful pathway-topic interpretations without sacrificing clustering performance relative to unregularized scETM.

## When NOT to use

- When the goal is free-form topic discovery or transfer learning rather than direct interpretation against known pathway gene sets (use standard scETM, keeping trainable_gene_emb_dim positive so all genes are retained)
- When pathways of interest contain 5 or fewer genes, since the pathDIP4 matrix is filtered to keep only pathways with more than 5 genes

## Inputs

- pathDIP4 pathways-by-genes matrix (filtered to pathways with >5 genes)
- Cells-by-genes AnnData matrices (HP, AD, MDD) with adata.obs['batch_indices'] and adata.obs['cell_types']

## Outputs

- pathway_topic_associations.csv (ranked pathway-topic table per dataset)
- Learned topic embeddings (mp.uns['alpha']) and fixed pathway gene embeddings (mp.varm['rho'])
- Clustering evaluation metrics comparable to Supplementary Table 15

## How to apply

Download the pathways-by-genes matrix from the pathDIP4 pathway database and filter to pathways containing more than 5 genes. Load the cells-by-genes AnnData (batch in 'batch_indices', cell types in 'cell_types') and initialize p-scETM with trainable_gene_emb_dim set to zero so ρ is entirely fixed to the pathway matrix; the trained gene set becomes the intersection of genes in the scRNA-seq data and the pathway matrix. Train via UnsupervisedTrainer with batched gradient descent sized for at least 6k gradient updates (e.g. n_epochs=12000, eval_every=1000). Extract topic embeddings (mp.uns['alpha']) and the fixed pathway gene embeddings (mp.varm['rho']), score pathway-topic associations per dataset, verify expected hits, and write the ranked table to pathway_topic_associations.csv. Rationale: fixing ρ converts scETM's gene embedding space into known pathway space, so topics directly associate with pathways instead of requiring downstream GSEA.

## Related tools

- **scETM** (Provides the p-scETM variant (trainable_gene_emb_dim=0) and UnsupervisedTrainer used to fix ρ to the pathway matrix and learn topic embeddings) — github.com/hui2000ji/scETM

## Examples

```
from scETM import scETM, UnsupervisedTrainer; model = scETM(adata.n_vars, adata.obs.batch_indices.nunique(), trainable_gene_emb_dim=0); UnsupervisedTrainer(model, adata).train(n_epochs=12000, eval_every=1000)
```

## Evaluation signals

- pathway_topic_associations.csv contains the expected named hits: Insulin Signaling among top HP pathways and Topic 9 associated with Alzheimer Disease-Amyloid Secretase
- Clustering metrics on the HP, AD and MDD datasets do not degrade relative to the unregularized scETM baseline, matching the Supplementary Table 15 criterion
- The trained gene set equals the intersection of genes in the scRNA-seq data and the gene-by-pathway matrix (trainable_gene_emb_dim=0 was used)
- Input pathway matrix was filtered so only pathways with more than 5 genes remain
- Training performed at least 6k gradient updates (e.g. n_epochs=12000 with eval_every=1000) indicating convergence

## Limitations

- With trainable_gene_emb_dim=0, the gene set is restricted to the intersection of scRNA-seq genes and pathway-matrix genes, dropping scRNA-seq-unique genes
- For small datasets with large mini-batch sizes, fewer epochs may yield under 6k gradient updates and an unconverged model
- scETM's model handles only a single categorical batch variable, and p-scETM does not add multi-omic or knowledge-graph-pretrained gene embedding support

## Evidence

- [readme] p-scETM is a variant of scETM where part or all of the the gene embedding matrix ρ is fixed to a pathways-by-genes matrix, which can be downloaded from the: "p-scETM is a variant of scETM where part or all of the the gene embedding matrix ρ is fixed to a pathways-by-genes matrix, which can be downloaded from the"
- [readme] If it is desired to fix the gene embedding matrix ρ during training, let trainable_gene_emb_dim be zero.: "If it is desired to fix the gene embedding matrix ρ during training, let trainable_gene_emb_dim be zero."
- [methods] We only keep pathways that contain more than 5 genes.: "We only keep pathways that contain more than 5 genes."
- [abstract] scETM enables the incorporation of known gene sets into the gene embeddings, thereby directly learning the associations between pathways and topics via the topic embeddings.: "scETM enables the incorporation of known gene sets into the gene embeddings, thereby directly learning the associations between pathways and topics via the topic embeddings."
- [readme] it is recommended to set the number of training epochs such that there is at least 6k gradient updates to the model.: "it is recommended to set the number of training epochs such that there is at least 6k gradient updates to the model."
