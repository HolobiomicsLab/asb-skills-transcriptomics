---
name: pathway-disease-association-interpretation
description: Use when you have trained a topic model (e.g. 100-topic scETM) on scRNA-seq
  data and need to show the learned topics correspond to known human gene pathways
  or disease gene sets via gene set enrichment analysis.
when_to_use_negative:
- When you need de novo pathway discovery rather than testing topics against curated
  gene sets such as MSigDB or PsyGeNET
- When the model output is a cell embedding (e.g. delta/theta) and no gene-by-topic
  embedding matrices (rho, alpha) are available, since per-topic gene scores cannot
  be computed
- When gene sets of interest contain 5 or fewer genes, which are filtered out by the
  pathway-size cutoff
edam_operation: http://edamontology.org/operation_2436
edam_topics:
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_3407
tools:
- name: scETM
  role: Trains the 100-topic model whose gene (rho) and topic (alpha) embeddings are
    combined to produce per-topic gene scores for GSEA
  repo: github.com/hui2000ji/scETM
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1038/s41467-021-25534-2
    title: Learning interpretable cellular and gene signature embeddings from single-cell
      transcriptomic data
  source_content_hash: sha256:e25c457dd5986942335f4a7b35ab74441fab8aa0c94fb219fe2cffcecbfb5528
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - scETM
  edam_operation: http://edamontology.org/operation_2436
  edam_topics:
  - http://edamontology.org/topic_0203
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_3407
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 9
---

# pathway-disease-association-interpretation

## Summary

Interprets scETM-learned topics by ranking genes per topic (gene embedding rho combined with topic embedding alpha) and running permutation-based GSEA against MSigDB Canonical Pathways, PsyGeNET, or similar gene sets to test whether topics map to biologically meaningful and disease-related pathways.

## When to use

Use when you have trained a topic model (e.g. 100-topic scETM) on scRNA-seq data and need to show the learned topics correspond to known human gene pathways or disease gene sets via gene set enrichment analysis.

## When NOT to use

- When you need de novo pathway discovery rather than testing topics against curated gene sets such as MSigDB or PsyGeNET
- When the model output is a cell embedding (e.g. delta/theta) and no gene-by-topic embedding matrices (rho, alpha) are available, since per-topic gene scores cannot be computed
- When gene sets of interest contain 5 or fewer genes, which are filtered out by the pathway-size cutoff

## Inputs

- Trained 100-topic scETM model with gene embedding matrix rho (mp.varm['rho']) and topic embedding matrix alpha (mp.uns['alpha'])
- AnnData cells-by-genes matrices for HP, AD and MDD datasets with batch indices in adata.obs
- Gene set collections: MSigDB Canonical Pathways (HP, AD) and PsyGeNET gene sets (MDD)

## Outputs

- Per-topic ranked gene lists derived from rho x alpha gene scores
- GSEA results with permutation p-values and BH-adjusted q-values per topic-pathway pair
- Compiled enrichment table topic_pathway_enrichment.tsv

## How to apply

Train the 100-topic scETM model with enable_batch_bias=True using UnsupervisedTrainer for 12000 epochs (eval_every=1000, n_samplers=4) on AnnData cells-by-genes matrices with batch indices in adata.obs. Extract gene embeddings from mp.varm['rho'] and topic embeddings from mp.uns['alpha'], then compute a per-topic gene score by combining the gene embedding matrix with the topic embedding matrix. Rank genes by their topic score and run permutation-based GSEA with 1000 cell-label shuffles and Benjamini-Hochberg correction, using MSigDB Canonical Pathways for HP and AD data and PsyGeNET gene sets for MDD data. Keep only pathways that contain more than 5 genes, and compile significant topic-pathway associations into a single enrichment table (e.g. topic_pathway_enrichment.tsv).

## Related tools

- **scETM** (Trains the 100-topic model whose gene (rho) and topic (alpha) embeddings are combined to produce per-topic gene scores for GSEA) — github.com/hui2000ji/scETM

## Examples

```
from scETM import scETM, UnsupervisedTrainer; model = scETM(hp.n_vars, hp.obs.batch_indices.nunique(), enable_batch_bias=True); UnsupervisedTrainer(model, hp).train(n_epochs=12000, eval_every=1000, n_samplers=4); model.get_all_embeddings_and_nll(hp)
```

## Evaluation signals

- HP Topic 54 is significantly enriched in the Insulin Processing pathway with a BH-adjusted q-value of 0
- AD topics show significant BH-corrected enrichment in Reactome Amyloid Fiber Formation and KEGG Alzheimer's disease pathways
- topic_pathway_enrichment.tsv contains all reported disease-relevant topic-pathway associations at BH-corrected significance
- All retained gene sets contain more than 5 genes
- GSEA used 1000 cell-label shuffles as the permutation scheme

## Limitations

- Only pathways containing more than 5 genes are kept, so small pathways cannot be detected
- Interpretation depends on the choice of gene set database (MSigDB Canonical Pathways for HP/AD, PsyGeNET for MDD); associations are limited to what those collections cover
- Topic-pathway associations are statistical enrichments from permutation GSEA and require the topics themselves to be well-trained (e.g. at least ~6k gradient updates) to be meaningful

## Evidence

- [results] GSEA of Topic 54 / Insulin Processing finding: [evidence span withheld — anchor status: not_found]
- [abstract] Gene set enrichment of topics claim: "Using gene set enrichment analysis, we find that scETM-learned topics are enriched in biologically meaningful and disease-related pathways."
- [methods] Pathway size filter: "We only keep pathways that contain more than 5 genes."
- [readme] Embedding extraction locations: "gene embeddings at mp.varm['rho'], topic embeddings at mp.uns['alpha']."
- [readme] Training configuration: "trainer.train(n_epochs=12000, eval_every=1000, n_samplers=4)"
