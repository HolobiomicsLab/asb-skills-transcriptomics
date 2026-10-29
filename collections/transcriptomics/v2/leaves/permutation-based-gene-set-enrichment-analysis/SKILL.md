---
name: permutation-based-gene-set-enrichment-analysis
description: Use when a trained topic model (e.g. 100-topic scETM) needs biological
  interpretation of its topics via GSEA of per-topic ranked gene lists against pathway
  databases such as MSigDB Canonical Pathways or PsyGeNET.
when_to_use_negative:
- Input is raw single-cell data with no trained topic model — train the scETM model
  first to obtain gene and topic embeddings before running GSEA
- Goal is direct pathway assignment of topics rather than interpretation — use p-scETM,
  which fixes the gene embedding matrix rho to a pathways-by-genes matrix to directly
  learn pathway-topic associations
- Pathway gene sets contain 5 or fewer genes — the article explicitly only keeps pathways
  that contain more than 5 genes
edam_operation: http://edamontology.org/operation_2436
edam_topics:
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_0602
tools:
- name: scETM
  role: Trains the 100-topic generative topic model whose learned gene (rho) and topic
    (alpha) embeddings provide the per-topic ranked gene lists input to GSEA
  repo: https://github.com/hui2000ji/scETM
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1038/s41467-021-25534-2
    title: Learning interpretable cellular and gene signature embeddings from single-cell
      transcriptomic data
  source_content_hash: sha256:4b3ec89ae7523d6a4dee216c5bf06baf7b59983e69ab00154c0455272019875c
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/hui2000ji/scETM
  tools:
  - scETM
  edam_operation: http://edamontology.org/operation_2436
  edam_topics:
  - http://edamontology.org/topic_0203
  - http://edamontology.org/topic_0602
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 7
---

# permutation-based-gene-set-enrichment-analysis

## Summary

Interpret scETM-inferred topics by computing per-topic gene scores from the learned gene embedding matrix (rho) and topic embedding matrix (alpha), then running permutation-based gene set enrichment analysis against MSigDB Canonical Pathways or PsyGeNET gene sets to show topics are enriched in biologically meaningful and disease-related pathways.

## When to use

Use when a trained topic model (e.g. 100-topic scETM) needs biological interpretation of its topics via GSEA of per-topic ranked gene lists against pathway databases such as MSigDB Canonical Pathways or PsyGeNET.

## When NOT to use

- Input is raw single-cell data with no trained topic model — train the scETM model first to obtain gene and topic embeddings before running GSEA
- Goal is direct pathway assignment of topics rather than interpretation — use p-scETM, which fixes the gene embedding matrix rho to a pathways-by-genes matrix to directly learn pathway-topic associations
- Pathway gene sets contain 5 or fewer genes — the article explicitly only keeps pathways that contain more than 5 genes

## Inputs

- Trained scETM model with learned gene embedding matrix rho (mp.varm['rho']) and topic embedding matrix alpha (mp.uns['alpha'])
- AnnData cells-by-genes matrices for HP, AD and MDD datasets with batch indices in adata.obs
- Pathway gene sets: MSigDB Canonical Pathways (HP, AD) and PsyGeNET gene sets (MDD), filtered to pathways with more than 5 genes

## Outputs

- Per-topic ranked gene lists derived from rho x alpha topic scores
- GSEA results with permutation p-values and Benjamini-Hochberg adjusted q-values per topic-pathway pair
- topic_pathway_enrichment.tsv — a single table of significant topic-pathway associations across all three datasets

## How to apply

Train a 100-topic scETM model on the cells-by-genes AnnData (batch indices in adata.obs, enable_batch_bias=True) with UnsupervisedTrainer for 12000 epochs, evaluating every 1000 epochs with 4 minibatch samplers. Extract gene embeddings (mp.varm['rho']) and topic embeddings (mp.uns['alpha']) via model.get_all_embeddings_and_nll(mp), and compute per-topic gene scores by combining the gene embedding matrix with the topic embedding matrix (rho x alpha). Rank genes by their topic score and run permutation-based GSEA with 1000 cell-label shuffles and Benjamini-Hochberg correction, using MSigDB Canonical Pathways for HP/AD datasets and PsyGeNET gene sets for MDD. Keep only pathways that contain more than 5 genes, then compile significant topic-pathway associations into a single enrichment table (topic_pathway_enrichment.tsv).

## Related tools

- **scETM** (Trains the 100-topic generative topic model whose learned gene (rho) and topic (alpha) embeddings provide the per-topic ranked gene lists input to GSEA) — https://github.com/hui2000ji/scETM

## Examples

```
model.get_all_embeddings_and_nll(mp); topic_scores = mp.varm['rho'] @ mp.uns['alpha'].T
```

## Evaluation signals

- HP Topic 54 is significantly enriched in the Insulin Processing pathway with a BH-adjusted q-value of 0
- AD topics are significantly enriched in Reactome Amyloid Fiber Formation and KEGG Alzheimer's disease pathways after BH correction
- topic_pathway_enrichment.tsv contains all reported disease-relevant topic-pathway associations at BH-corrected significance
- Each pathway gene set in the analysis contains more than 5 genes
- GSEA used 1000 permutation (cell-label) shuffles and Benjamini-Hochberg correction for multiple testing

## Limitations

- Permutation-based GSEA results depend on the trained 100-topic model; convergence issues can arise if the number of training epochs yields fewer than ~6k gradient updates on small datasets
- Only topic-pathway associations from MSigDB Canonical Pathways (HP, AD) and PsyGeNET (MDD) were validated; other gene set databases were not assessed in this workflow
- scETM currently handles only a single categorical batch variable, which constrains how multi-study batch structure is modeled before enrichment testing

## Evidence

- [abstract] GSEA finds biologically meaningful pathway enrichment of scETM topics: "Using gene set enrichment analysis, we find that scETM-learned topics are enriched in biologically meaningful and disease-related pathways."
- [methods] Pathway filter threshold: "We only keep pathways that contain more than 5 genes."
- [methods] p-scETM fixes gene embeddings to pathway matrix: "p-scETM is a variant of scETM where part or all of the the gene embedding matrix ρ is fixed to a pathways-by-genes matrix"
- [readme] Embedding extraction locations: "Obtain scETM cell, gene and topic embeddings. Unnormalized cell embeddings will be stored at mp.obsm['delta'], normalized cell embeddings at mp.obsm['theta'], gene embeddings at mp.varm['rho'], topic"
- [readme] Training parameters for scETM model: "Train the model on adata for 12000 epochs, and evaluate every 1000 epochs. Use 4 threads to sample minibatches."
