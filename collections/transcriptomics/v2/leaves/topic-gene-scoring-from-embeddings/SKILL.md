---
name: topic-gene-scoring-from-embeddings
description: Use when you have trained an scETM model (e.g. 100-topic model on HP,
  AD, or MDD AnnData) and need interpretable per-topic gene rankings to test topics
  against known pathway gene sets via GSEA.
when_to_use_negative:
- Input is raw counts or cell embeddings only — gene scoring requires the learned
  rho and alpha matrices from a trained scETM model, not cell-level data
- The model variant is p-scETM with the gene embedding matrix rho fixed to a pathways-by-genes
  matrix, since pathway structure is already imposed and scoring against the same
  gene sets is circular
- Genes in the scRNA-seq data do not sufficiently overlap the pathway gene sets after
  filtering (pathways must contain more than 5 genes to be kept)
edam_operation: http://edamontology.org/operation_2436
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_0203
tools:
- name: scETM
  role: Trains the embedded topic model whose rho (gene embedding) and alpha (topic
    embedding) matrices are combined to score genes per topic
  repo: https://github.com/hui2000ji/scETM
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1038/s41467-021-25534-2
    title: Learning interpretable cellular and gene signature embeddings from single-cell
      transcriptomic data
  source_content_hash: sha256:0bbd31923d384aa79e50e96d26303952d8d5dad66dbd3b72cd42e4ce6c36ad38
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/hui2000ji/scETM
  tools:
  - scETM
  edam_operation: http://edamontology.org/operation_2436
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 10
---

# topic-gene-scoring-from-embeddings

## Summary

Score genes for each topic learned by the scETM embedded topic model by combining the learned gene embedding matrix (rho) with the topic embedding matrix (alpha), producing per-topic gene rankings suitable for downstream gene set enrichment analysis of pathway associations.

## When to use

Use when you have trained an scETM model (e.g. 100-topic model on HP, AD, or MDD AnnData) and need interpretable per-topic gene rankings to test topics against known pathway gene sets via GSEA.

## When NOT to use

- Input is raw counts or cell embeddings only — gene scoring requires the learned rho and alpha matrices from a trained scETM model, not cell-level data
- The model variant is p-scETM with the gene embedding matrix rho fixed to a pathways-by-genes matrix, since pathway structure is already imposed and scoring against the same gene sets is circular
- Genes in the scRNA-seq data do not sufficiently overlap the pathway gene sets after filtering (pathways must contain more than 5 genes to be kept)

## Inputs

- Trained scETM model (e.g. 100-topic model with enable_batch_bias=True)
- AnnData cells-by-genes matrix (.h5ad) with batch indices in adata.obs (e.g. HumanPancreas_aligned.h5ad)
- Gene embedding matrix rho (adata.varm['rho'])
- Topic embedding matrix alpha (adata.uns['alpha'])
- Pathway gene set collections: MSigDB Canonical Pathways, PsyGeNET gene sets, or pathDIP4 pathways-by-genes matrix

## Outputs

- Per-topic ranked gene lists (genes sorted by topic score)
- Topic-pathway enrichment table with GSEA statistics and BH-adjusted q-values (e.g. topic_pathway_enrichment.tsv)

## How to apply

After training the scETM model (e.g. 100 topics, enable_batch_bias=True, UnsupervisedTrainer for 12000 epochs with eval_every=1000 and n_samplers=4), extract the learned gene embedding matrix rho (stored at adata.varm['rho']) and topic embedding matrix alpha (stored at adata.uns['alpha']) via model.get_all_embeddings_and_nll(adata). Combine these two matrices — the topic-gene score is the inner product of the topic embedding with the gene embedding over the latent embedding dimensions — to obtain one score per (topic, gene) pair. Rank genes in descending order of their topic score, then feed the ranked gene list into permutation-based GSEA (e.g. 1000 cell-label shuffles with Benjamini-Hochberg correction) using MSigDB Canonical Pathways (for HP and AD) or PsyGeNET gene sets (for MDD); keep only pathways containing more than 5 genes when preparing pathway matrices. The rationale is that the low-rank factorization view of scETM means a high-scoring gene for a topic has a gene embedding aligned with that topic's embedding, so the score directly reflects topic-gene association strength.

## Related tools

- **scETM** (Trains the embedded topic model whose rho (gene embedding) and alpha (topic embedding) matrices are combined to score genes per topic) — https://github.com/hui2000ji/scETM

## Examples

```
from scETM import scETM, UnsupervisedTrainer; model = scETM(mp.n_vars, mp.obs.batch_indices.nunique(), enable_batch_bias=True); trainer = UnsupervisedTrainer(model, mp); trainer.train(n_epochs=12000, eval_every=1000, n_samplers=4); model.get_all_embeddings_and_nll(mp)
```

## Evaluation signals

- HP Topic 54 must be significantly enriched in the Insulin Processing pathway at BH-adjusted significance (reported q-value of 0)
- AD topics must show significant enrichment in Reactome Amyloid Fiber Formation and KEGG Alzheimer's disease gene sets
- Enrichment table must be generated from GSEA with 1000 permutation shuffles and Benjamini-Hochberg correction
- Every retained pathway must contain more than 5 genes
- Output topic_pathway_enrichment.tsv must contain the reported disease-relevant topic-pathway associations across HP, AD and MDD

## Limitations

- Pathway enrichment results depend on the gene set collection choice (MSigDB Canonical Pathways for HP/AD vs PsyGeNET for MDD), so topic-pathway hits are not comparable across different gene set databases
- scETM currently handles only a single categorical batch variable, so batch confounding in the input AnnData can distort learned gene/topic embeddings
- scETM is trained via batched gradient descent; for small datasets with large mini-batches, fewer than ~6k gradient updates can leave the model unconverged and the embeddings unreliable

## Evidence

- [methods] Model factorization view of scETM as inner product of topic and gene embeddings: "model = scETM(mp.n_vars, mp.obs.batch_indices.nunique(), enable_batch_bias=True)"
- [readme] Embedding extraction locations documented in the README: "gene embeddings at mp.varm['rho'], topic embeddings at mp.uns['alpha']."
- [abstract] scETM-learned topics are biologically meaningful via GSEA: "Using gene set enrichment analysis, we find that scETM-learned topics are enriched in biologically meaningful and disease-related pathways."
- [methods] Pathway size filter for gene set preparation: "We only keep pathways that contain more than 5 genes."
- [readme] Training recipe for the models whose topics are scored: "trainer.train(n_epochs=12000, eval_every=1000, n_samplers=4)"
